# QAT Operations Dashboard — Netlify serverless starter

This package converts the supplied QAT server's **saved-data dashboard and planning CRUD** into a Netlify static site + Netlify Function. No always-on Node HTTP server or background polling loop is used.

## Included
- Responsive dashboard with dataset tabs for TL Hourly, Staff, Project/Task, Outflow, Queue Monitor and Cancellation.
- Read saved JSON from the corresponding Firebase Realtime Database URLs.
- Search/filter, pagination and CSV export.
- Planning list, create target and delete target, persisted in Firebase.
- Optional Basic Auth at the function boundary.
- Netlify config and environment variable template.

## Important integration boundary
The source you supplied uses a persistent Node server, Grafana session-cookie login, BigQuery jobs, and two automatic polling loops. Netlify Functions are short-lived/request-driven; those loops cannot be copied as-is. This package deliberately does **not** claim to perform live Grafana/BigQuery refresh: refresh endpoints return HTTP 501 with an explanatory message. Existing saved Firebase datasets can be viewed, and planning entries can be managed. To enable live refresh, port the BigQuery/Grafana functions into request-driven Netlify Functions (or schedule a separate scheduled function/workflow) and configure secrets below. Queue monitoring's 3-second forever loop must be replaced with a scheduled workflow; Netlify Scheduled Functions are not intended for 3-second polling.

## Deploy to Netlify
1. Unzip this folder and push its contents to a private Git repository.
2. In Netlify, **Add new site → Import an existing project** and select the repo. Build command can be blank; publish directory is `public`. `netlify.toml` already configures the function directory.
3. In **Site configuration → Environment variables**, copy the values you need from `.env.example`. Set `AUTH_USER` and a strong unique `AUTH_PASS`. Never use the old hard-coded password from the supplied source.
4. Redeploy after setting environment variables.
5. Open the deployed site and press **API login**. Enter the `AUTH_USER` and `AUTH_PASS` values. Login is kept in this browser tab's session storage only.
6. Check `https://YOUR-SITE.netlify.app/api/health` (when auth is enabled, provide Basic Auth; otherwise visit the dashboard).

## Firebase security
The supplied database URLs may allow public reads/writes depending on Firebase rules. Review and tighten Firebase Realtime Database Rules before production. If the database requires authentication, configure a supported server-side credential/token using `FIREBASE_DATABASE_SECRET` only if your project still supports that auth method. Do not put service-account JSON or secrets in `public/` or frontend JavaScript. Prefer a server-side Firebase Admin integration with a properly scoped service account if rules require it.

## Environment variables
See `.env.example`. URL variables should be full Firebase REST URLs ending in `.json`. `PROJECT_TASK_SHEET_URL` is optional; set it to the published CSV URL for your sheet. Keep secrets in Netlify environment variables, not in source control.

## Local development
Requires Node.js 20+:
```bash
npm install
cp .env.example .env
npx netlify dev
```
Netlify CLI loads `.env` locally. Do not commit `.env`.

## API endpoints
- `GET /api/health`
- `GET /api/tl-hourly`
- `GET /api/staff`
- `GET /api/filtered`
- `GET /api/outflow`
- `GET /api/queue`
- `GET /api/cancellation`
- `GET /api/planning`
- `POST /api/planning` JSON `{ "project_name": "Project", "task_name": "Task", "target": 100 }`
- `DELETE /api/planning?id=FIREBASE_KEY`
- `GET /api/staff-lookup`
- `GET /api/project-task-lookup` (requires `PROJECT_TASK_SHEET_URL`)
- Live source refresh endpoints currently return `501` until the Grafana/BigQuery integration is migrated.

## Security note
The uploaded source text contains hard-coded Grafana credentials and a default Basic Auth password. This package does not copy those passwords into the website. Rotate/revoke any credentials that were exposed in the uploaded file and put replacement values only in Netlify environment variables.
