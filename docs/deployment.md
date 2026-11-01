# Vercel frontend and Render backend

## Vercel

Set the project's **Root Directory** to `frontend`, **Framework Preset** to Vite,
**Build Command** to `npm run build`, and **Output Directory** to `dist`.
Set `VITE_DEMO_PURCHASES_ENABLED=true` for each deployment environment where the
purchase demo should be visible, then redeploy after changing that variable.
Visitors open the dashboard directly by default. Parent onboarding is retained
but hidden; set `VITE_ONBOARDING_ENABLED=true` and redeploy to restore it.

`frontend/vercel.json` forwards `/api/:path*` to
`https://monty-ipdk.onrender.com/api/:path*`. The browser continues requesting
`/api` on the frontend's own domain; no frontend API URL variable is required.
Vercel forwards the API path, query parameters, methods, and request bodies.
The API rewrite is first so API responses are not replaced by the SPA fallback.
Other application routes, including direct visits to `/dashboard`, serve
`index.html`. Existing static files are served by Vercel normally.

API responses opt out of rewrite caching and browser caching so assessment
polling and parent Updates fetch current data. Development still uses the local
Vite proxy to `http://localhost:8000`.

## Render

Set **Root Directory** to `backend`, **Build Command** to
`pip install -r requirements.txt`, **Start Command** to
`uvicorn app.main:app --host 0.0.0.0 --port $PORT`, and **Health Check Path** to
`/health`.

Configure `DATABASE_URL`, `JWT_SECRET`, and `GEMINI_API_KEY` as Render secrets.
Set `ENVIRONMENT=production`, `DEMO_PURCHASES_ENABLED=true`, and
`PURCHASE_ASSESSMENT_WORKER_ENABLED=true` for the public demo. Apply the latest
Alembic migrations against the intended database before enabling purchases.
Additional model and budget settings are documented in `backend/.env.example`.

The browser's API requests are same-origin through Vercel, so this proxy path
does not require adding each Vercel deployment to backend CORS origins. Existing
CORS settings remain relevant for browsers that call Render directly.

## Verify after deployment

1. Open `https://monty-ipdk.onrender.com/health` and confirm a successful response.
2. Open `/api/families` on the Vercel domain and confirm JSON, not an HTML page.
3. Reload `/dashboard` directly and confirm the app loads.
4. Create a demo purchase. Browser network requests should stay on the Vercel
   domain under `/api`, and assessment polling should return current statuses.

Changing the Render service URL requires updating the API rewrite destination
and redeploying the frontend. This configuration does not publish either service.
