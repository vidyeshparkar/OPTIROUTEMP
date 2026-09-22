# OptiRoute

Multi-objective route optimization. Instead of one "best" route, OptiRoute computes a
Pareto-optimal set of routes trading off **Time**, **Distance**, **Fuel Cost**, and
**Accident Risk**, using real OpenStreetMap road data and **NSGA-III** (via `pymoo`).
Pick a priority (fastest / shortest / eco / safest / balanced, or custom weights) and get
the best-fit route from the Pareto set instantly — no re-optimization needed.

## How it works

1. Geocodes your **From** and **To** locations (OSMnx / Nominatim).
2. Downloads the drivable road network around the midpoint, out to the radius you set.
3. Computes 4 objective costs per road segment (time, distance, fuel, accident risk).
4. Finds a baseline route with A*.
5. Runs NSGA-III to search for a diverse, genuinely different set of Pareto-optimal routes.
6. Validates every candidate route against the real graph.
7. Lets you switch priorities instantly (cheap re-scoring, no recompute) and shows the
   recommended route on an interactive map plus a Pareto-front chart.

## Run locally

```bash
pip install -r requirements.txt
streamlit run app.py
```

Then open the local URL Streamlit prints (usually `http://localhost:8501`).

## Optional: live traffic (TomTom)

The app works fully without this. To enable it:

1. Get a free API key at https://developer.tomtom.com/ (Traffic API, free tier).
2. Locally, create `.streamlit/secrets.toml`:
   ```toml
   TOMTOM_API_KEY = "your-key-here"
   ```
3. On Streamlit Community Cloud, add the same key under **App settings → Secrets**.

Never commit your API key to the repo. If no key is configured, the "Use live traffic
data" checkbox is simply disabled and the app uses speed-limit-based times.

## Deploy to Streamlit Community Cloud

1. Push this folder (`app.py`, `requirements.txt`, `README.md`) to a GitHub repo.
2. Go to https://share.streamlit.io/, sign in, and click **New app**.
3. Select your repo/branch and set the main file path to `app.py`.
4. (Optional) Under **Advanced settings → Secrets**, paste your `TOMTOM_API_KEY` as shown
   above.
5. Click **Deploy**. First load will take a minute or two (dependency install); after that,
   graph downloads are cached per area for the lifetime of the app instance.

## Practical limits

- Area radius is capped at 15 km. Larger areas mean much bigger road graphs, which slows
  down both the OSM download and the NSGA-III search — this cap keeps each request
  practical for a live web request (roughly 20–60s per search).
- Sparsely-mapped regions may lack `maxspeed` or `lit` tags in OpenStreetMap; the weighting
  function falls back to reasonable defaults in that case.
- If your two locations are too far apart for the selected radius, or geocoding fails,
  the app shows a clear error instead of crashing — try a larger radius or a more specific
  address.

## Files

- `app.py` — the full Streamlit application
- `requirements.txt` — Python dependencies
- `README.md` — this file
