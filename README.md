# ARA Radar

Advanced Ranking & Alert system for stock market analysis using machine learning models.

## Project Structure

```
ara-models/
├── backend/          # FastAPI backend
│   ├── app.py       # Main API endpoints
│   ├── model_loader.py
│   ├── utils.py
│   ├── requirements.txt
│   ├── Dockerfile
│   └── render.yaml
├── frontend/         # Next.js 14 frontend
│   ├── app/         # App Router pages
│   ├── components/  # React components
│   ├── lib/         # Utilities and API
│   └── public/      # Static assets
├── incoming/         # Model bundles from GitHub Releases
└── .github/workflows/promote-if-better.yml
```

## Features

### Backend (FastAPI)

- Automatic model loading from GitHub Releases
- XGBoost ensemble with isotonic calibration
- RESTful API endpoints:
  - `GET /health` - Health check
  - `GET /metrics` - Model performance metrics
  - `GET /score_latest` - Get top K candidates
  - `GET /equity` - Equity curve data
  - `GET /alerts/stream` - SSE real-time alerts
  - `POST /score` - Score uploaded data
  - `GET /meta` - Model card information
  - `GET /bundle/info` - Bundle diagnostics

### Frontend (Next.js 14 + App Router)

- Landing page with animated aurora + starfield (Canvas API)
- 3D rotating logo (react-three-fiber)
- Preloader with progress bar
- Dashboard with:
  - Filter panel (K selector, liquidity slider, toggles)
  - Top candidates table with real-time highlights
  - Charts panel (equity curve, metrics)
  - Real-time SSE alerts bar
  - Web Push notifications support
- Settings page
- Model card page
- Error boundaries
- PWA support with service worker
- Playwright e2e tests

## Deployment

### Backend (Render)

1. Push code to GitHub
2. Connect repository to Render
3. Set environment variables:
   ```
   GITHUB_REPO=allamrf865/ara-models
   GITHUB_TOKEN=(optional, for private repos)
   ARTIFACT_TAG=(optional, specific tag)
   ARTIFACT_ZIP_URL=(optional, direct URL)
   ALERT_THRESHOLD=0.75
   ```
4. Deploy using `render.yaml`

Backend will auto-fetch the latest model bundle from GitHub Releases.

### Frontend (Netlify)

1. Push code to GitHub
2. Connect repository to Netlify
3. Configure build settings:
   - Build command: `cd frontend && npm ci && npm run build`
   - Publish directory: `frontend/.next`
4. Set environment variable:
   ```
   NEXT_PUBLIC_BACKEND_URL=https://your-backend-url.onrender.com
   ```
5. Deploy

### Auto-Promotion Workflow

The GitHub Action `.github/workflows/promote-if-better.yml` automatically:
- Checks `incoming/` for new model bundles
- Compares `ap_valid` with current release
- Creates new GitHub Release if improvement detected
- Backend auto-fetches the new release

## Local Development

### Backend

```bash
cd backend
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt
uvicorn app:app --reload --port 8000
```

### Frontend

```bash
cd frontend
npm ci
npm run dev
```

Visit http://localhost:8888

#### Preview the production bundle

To confirm the pinned React 18.2 runtime behaves correctly before deploying, run the bundled preview locally:

```bash
npm run preview
```

This command builds the production assets and boots the same server you will deploy, allowing you to verify there are no `ReactCurrentBatchConfig` runtime errors.

If a Netlify build fails with an `integrity checksum` error for `react-dom@18.2.0`, clear the cached artifacts before redeploying:

```bash
rm -rf ~/.npm
npm cache clean --force
```

Then retry the deployment (`netlify deploy --prod --json` or the hosted pipeline). The project is pinned to React 18.2.0, so a clean cache guarantees the lockfile checksum matches what Netlify installs.

### Environment Variables

Backend `.env`:
```
GITHUB_REPO=allamrf865/ara-models
GITHUB_TOKEN=
ARTIFACT_TAG=
ARTIFACT_ZIP_URL=
ALERT_THRESHOLD=0.75
```

Frontend `.env.local`:
```
NEXT_PUBLIC_BACKEND_URL=http://localhost:8000
```

## Testing

### Backend
```bash
cd backend
pytest
```

### Frontend E2E
```bash
cd frontend
npm run test:e2e
```

## Tech Stack

### Backend
- FastAPI
- Pandas, NumPy
- XGBoost, scikit-learn
- Uvicorn

### Frontend
- Next.js 14 (App Router)
- TypeScript
- Tailwind CSS
- Framer Motion
- React Three Fiber
- Zustand
- SWR
- Playwright

## API Examples

### Get Latest Scores
```bash
curl "http://localhost:8000/score_latest?k=50&liq=0.5&exclude_pemantauan=true"
```

### Subscribe to Alerts
```javascript
const eventSource = new EventSource('http://localhost:8000/alerts/stream');
eventSource.onmessage = (event) => {
  const alert = JSON.parse(event.data);
  console.log(alert);
};
```

### Upload New Data
```bash
curl -X POST "http://localhost:8000/score?k=50&liq=0.5&exclude_pemantauan=true" \
  -F "features_csv=@features.csv" \
  -F "raw_csv=@raw.csv" \
  -F "meta_xlsx=@metadata.xlsx"
```

## Model Bundle Format

Expected ZIP structure:
```
ara_model_bundle.zip
├── model_card.json           # Metadata and metrics
├── feature_cols_final.json   # Feature list
├── isotonic_calibrator.pkl   # Calibrator
└── xgb_cls_seed*.json        # XGBoost models
```

## License

MIT
