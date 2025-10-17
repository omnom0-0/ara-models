# ARA Radar v2 - Deployment Guide

## Quick Deploy (Recommended)

### Option 1: Deploy to Fly.io (Easiest)

**Backend:**
```bash
cd backend

# Install Fly CLI if needed
curl -L https://fly.io/install.sh | sh

# Login
fly auth login

# Create app (first time only)
fly apps create ara-radar-backend --region sin

# Set secrets
fly secrets set \
  SUPABASE_URL=https://wxddgrcnjesgumfztcdi.supabase.co \
  SUPABASE_ANON_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6Ind4ZGRncmNuamVzZ3VtZnp0Y2RpIiwicm9sZSI6ImFub24iLCJpYXQiOjE3NjA1OTA1NTQsImV4cCI6MjA3NjE2NjU1NH0.VhrJ5h7RNZNyWQS3E1fa4Rn0rvs17XAlNGAugWly2o0

# Deploy
fly deploy

# Get URL
fly status
```

**Frontend (Netlify):**
```bash
cd frontend

# Install Netlify CLI if needed
npm install -g netlify-cli

# Login
netlify login

# Initialize (first time only)
netlify init

# Set environment variables in Netlify dashboard:
NEXT_PUBLIC_BACKEND_URL=https://ara-radar-backend.fly.dev
NEXT_PUBLIC_SUPABASE_URL=https://wxddgrcnjesgumfztcdi.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6Ind4ZGRncmNuamVzZ3VtZnp0Y2RpIiwicm9sZSI6ImFub24iLCJpYXQiOjE3NjA1OTA1NTQsImV4cCI6MjA3NjE2NjU1NH0.VhrJ5h7RNZNyWQS3E1fa4Rn0rvs17XAlNGAugWly2o0

# Deploy (prints JSON with the public URL)
netlify deploy --prod --json

# Or run the helper to build + deploy + print the link
../deploy.sh  # choose option 4
```

### Option 2: Deploy to Render

**Backend:**
1. Go to https://render.com
2. Connect your GitHub repository
3. Create new Web Service
4. Select `backend` directory
5. Build command: `pip install -r requirements_prod.txt`
6. Start command: `uvicorn app:app --host 0.0.0.0 --port $PORT`
7. Add environment variables:
   ```
   SUPABASE_URL=https://wxddgrcnjesgumfztcdi.supabase.co
   SUPABASE_ANON_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6Ind4ZGRncmNuamVzZ3VtZnp0Y2RpIiwicm9sZSI6ImFub24iLCJpYXQiOjE3NjA1OTA1NTQsImV4cCI6MjA3NjE2NjU1NH0.VhrJ5h7RNZNyWQS3E1fa4Rn0rvs17XAlNGAugWly2o0
   GITHUB_REPO=allamrf865/ara-models
   ALERT_THRESHOLD=0.75
   ```
8. Deploy

**Frontend (Netlify):**
1. Go to https://netlify.com
2. Connect your GitHub repository
3. Build settings:
   - Base directory: `frontend`
   - Build command: `npm install && npm run build`
   - Publish directory: `frontend/.next`
4. Environment variables:
   ```
   NEXT_PUBLIC_BACKEND_URL=https://your-backend.onrender.com
   NEXT_PUBLIC_SUPABASE_URL=https://wxddgrcnjesgumfztcdi.supabase.co
   NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6Ind4ZGRncmNuamVzZ3VtZnp0Y2RpIiwicm9sZSI6ImFub24iLCJpYXQiOjE3NjA1OTA1NTQsImV4cCI6MjA3NjE2NjU1NH0.VhrJ5h7RNZNyWQS3E1fa4Rn0rvs17XAlNGAugWly2o0
   ```
5. Deploy

## Local Testing

### Backend
```bash
cd backend
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate
pip install -r requirements_prod.txt
uvicorn app:app --reload --port 8000
```

Test: http://localhost:8000/health

### Frontend
```bash
cd frontend
npm install
npm run build
npm start
```

Test: http://localhost:8888

## Troubleshooting

### Backend Issues

**"No model bundle available"**
- Model bundle is included in `/incoming` folder
- Will auto-load on startup
- If GitHub fetch fails, uses local bundle

**"Supabase connection error"**
- Verify SUPABASE_URL and SUPABASE_ANON_KEY
- Check network connectivity
- Verify RLS policies in Supabase dashboard

**"Module not found"**
- Ensure all dependencies in requirements_prod.txt are installed
- For OCR: Install tesseract-ocr system package

### Frontend Issues

**Build fails**
- Delete `node_modules` and `.next`
- Run `npm install` again
- Check all path aliases in tsconfig.json

**API calls fail**
- Verify NEXT_PUBLIC_BACKEND_URL is set correctly
- Check CORS is enabled on backend
- Test backend `/health` endpoint directly

## Post-Deployment Checklist

- [ ] Backend /health returns OK
- [ ] Frontend loads landing page
- [ ] Dashboard displays without errors
- [ ] Can navigate to /ingest
- [ ] Supabase tables are accessible
- [ ] SSE alerts stream works
- [ ] Can upload CSV file
- [ ] Auto-scrape from Yahoo works

## Monitoring

### Backend Logs
- Fly.io: `fly logs`
- Render: Check dashboard logs

### Frontend Logs
- Netlify: Check deploy logs and function logs

### Database
- Supabase dashboard: Check table contents and logs

## Scaling

### Backend
- Fly.io: `fly scale count 2`
- Render: Upgrade plan for auto-scaling

### Frontend
- Netlify auto-scales with CDN

### Database
- Supabase free tier: 500MB
- Upgrade if needed

## Support

If deployment fails:
1. Check logs for specific errors
2. Verify all environment variables
3. Test locally first
4. Check GitHub Issues: https://github.com/allamrf865/ara-models/issues
