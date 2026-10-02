# Running Rateverse on Replit

## Frontend

The React development server runs on port 5000:

```sh
cd frontend && HOST=0.0.0.0 PORT=5000 BROWSER=none yarn start
```

In development, requests use `/api` and the frontend dev server proxies them to
the FastAPI service on port 8000. In production, the frontend calls
`https://rate.indetail.in/api`.

## Backend

The backend uses Replit-managed PostgreSQL. Its development tables are defined
in `backend/schema.sql` and applied to the development database by
`scripts/post-merge.sh`. The app does not create or alter tables during startup.
Install the dependencies listed in `backend/requirements.txt`, then start the
API from the `backend` directory:

```sh
cd backend && python -m uvicorn server:app --host 0.0.0.0 --port 8000
```

The backend uses Replit's injected `DATABASE_URL`. Authentication uses
`JWT_SECRET` when present, otherwise the existing `SESSION_SECRET`. Set the
optional `ADMIN_PASSWORD` secret to create the seeded admin account; without it,
the server skips admin account creation rather than using an insecure default.


Records are stored in per-resource PostgreSQL tables with JSONB payloads. This
preserves the flexible, category-specific listing metadata and existing API
response shapes. The backend seeds demo data, creates an admin account, and
runs a user-data backfill when it starts. The PostgreSQL development database
was selected as a fresh database; no MongoDB data is imported.

## Domain routing

The frontend production API base is `https://rate.indetail.in`; the backend API
routes are under `/api`. The domain or its reverse proxy must route `/api/*` to
the FastAPI service on port 8000. `FRONTEND_URL` is set to the same domain for
backend CORS.

## Optional features

The imported AI summary and object-storage code expects the `emergentintegrations`
package and `EMERGENT_LLM_KEY`. That package is not available from the package
registry in this environment, so those integrations are not enabled by this
setup.