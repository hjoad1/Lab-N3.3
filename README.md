# Smart Speech Coach

Smart Speech Coach is an AI-enabled full-stack web application. The React/Vite
frontend sends questions to a Node.js/Express backend using `fetch`. The
backend calls Gemini and returns the answer to the frontend.

## Deployed Application

- Frontend: <https://gen-lang-client-0800605818.web.app>
- Backend: <https://smart-speech-coach-backend-389363739519.us-central1.run.app>
- Health check: <https://smart-speech-coach-backend-389363739519.us-central1.run.app/api/health>
- Google Cloud and Firebase project ID: `gen-lang-client-0800605818`

These URLs were generated at the time of deployment and are included as
deployment evidence. They may no longer be active or reachable. When
reproducing the deployment, use the URLs printed by the current Cloud Run and
Firebase Hosting commands instead.

The frontend is hosted by Firebase Hosting. The backend runs on Cloud Run in
the `us-central1` region.


## Environment Setup

Create these local files from the example files. They are ignored by Git:

```text
frontend/.env
backend/.env
backend/.env.cloudrun
```

The local backend file uses dotenv format:

```env
GEMINI_API_KEY=YOUR_GEMINI_API_KEY
port=3001
allowedOrigins=http://localhost:5173,http://127.0.0.1:5173,https://gen-lang-client-0800605818.web.app
geminiAskAnythingModel=gemini-3.8-flash
askAnythingPrompt="You are going to answer questions about Smart Speech Coach. Keep responses concise."
```

The Cloud Run file uses YAML format and does not define `PORT`, because Cloud
Run provides that variable:

```yaml
GEMINI_API_KEY: "YOUR_GEMINI_API_KEY"
geminiAskAnythingModel: "gemini-3.8-flash"
askAnythingPrompt: "You are going to answer questions about Smart Speech Coach. Keep responses concise."
allowedOrigins: "http://localhost:5173,http://127.0.0.1:5173,https://gen-lang-client-0800605818.web.app"
```

The example file is currently named `backend/.env.coudrun.example` (with a
typo). Copy it to `backend/.env.cloudrun` and use the YAML format above.

Set `frontend/.env` to use the deployed backend:

```env
VITE_SERVER_REMOTE=Y
VITE_SERVER_URL_REMOTE=https://smart-speech-coach-backend-389363739519.us-central1.run.app
VITE_SERVER_URL_LOCAL=http://localhost:3001
```

Never commit environment files because they contain the Gemini API key.

## Local Development

```bash
cd backend && npm install
cd ../frontend && npm install
npm run lint
npm run build
```

Start the backend with `npm run dev` from `backend`, then start the frontend
with `npm run dev` from `frontend`. The local frontend runs at
`http://localhost:5173` and calls `http://localhost:3001` when
`VITE_SERVER_REMOTE=N`.

Test the local backend:

```bash
curl http://localhost:3001/api/health
```

![Frontend lint and build](images/1_frontend_lint_build.png)

![Local frontend showing a Gemini answer](images/2_frontend_showing_gemini_ans.png)

![Local backend health response](images/3_frontend_showing_health.png)

## Google Cloud Run Deployment

```bash
gcloud auth login
gcloud auth list
gcloud config set project gen-lang-client-0800605818
```

![Google Cloud authentication](images/4_gcloud_auth_list.png)

![Google Cloud project ID](images/5_gcloud_project_ID.png)

Deploy from `backend`:

```bash
cd backend
gcloud run deploy smart-speech-coach-backend --source . --project gen-lang-client-0800605818 --region us-central1 --allow-unauthenticated --env-vars-file .env.cloudrun
```

`--source .` builds the Node.js service with Google Cloud Buildpacks.
`--allow-unauthenticated` permits the public Firebase frontend to call it.

![Cloud Run backend deployment](images/6_gcloud_backend_deploy.png)

Test the deployed backend:

```bash
export CLOUD_RUN_URL="https://smart-speech-coach-backend-389363739519.us-central1.run.app"
curl "$CLOUD_RUN_URL/api/health"
curl -X POST "$CLOUD_RUN_URL/api/askanything" -H "Content-Type: application/json" -d '{"message":"Give me one public speaking tip."}'
```

![Cloud Run health response](images/7_gcloud_backend_health.png)

![Cloud Run Gemini response](images/8_gcloud_backend_response.png)

## Firebase Hosting Deployment

Firebase was enabled on the existing Gemini project. The project ID
`gen-lang-client-0800605818` is used by both Google Cloud and Firebase; it is a
project identifier, not a Firebase account.

```bash
npx -y firebase-tools@latest login
npx -y firebase-tools@latest use
npx -y firebase-tools@latest projects:list
```

![Firebase setup](images/9_firebase_setup.png)

![Firebase project list](images/10_firebase_project_list.png)

Build the frontend after setting `VITE_SERVER_URL_REMOTE`:

```bash
cd frontend
npm run build
```

![Frontend production rebuild](images/14_frontend_rebuild.png)

Deploy the `frontend/dist` directory configured in `firebase.json`:

```bash
cd ..
npx -y firebase-tools@latest deploy --only hosting
```

![Firebase backend deployment](images/11_firebase_backend_deploy.png)

![Firebase Hosting deployment](images/15_firebase_frontend_deploy.png)

## Production Verification

```bash
curl https://smart-speech-coach-backend-389363739519.us-central1.run.app/api/health
```

![Deployed backend health check](images/12_firebase_backend_health.png)

![Deployed backend Gemini response](images/13_firebase_backend_response.png)

Open <https://gen-lang-client-0800605818.web.app>, submit a question, and
confirm that the answer is displayed. The browser sends `POST /api/askanything`
to Cloud Run and renders the returned `answer`.

![Live Firebase frontend](images/16_firebase_frontend_live_web.png)