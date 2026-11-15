# Flask CI/CD to Render

A small Flask API with a GitHub Actions workflow that **tests**, **builds** a Docker image, **deploys** it to Render, and **verifies** the live endpoint.

## Project structure

```
.
├── .github/workflows/ci-cd.yml   # CI/CD pipeline
├── app.py                        # Flask app (/ and /health)
├── test_app.py                   # pytest test suite
├── requirements.txt
├── Dockerfile
└── README.md
```

## Endpoints

| Method | Path | Response |
|---|---|---|
| GET | `/` | `{"message": "Hello from Flask on Render"}` |
| GET | `/health` | `{"status": "ok"}` |

## Run locally

```bash
pip install -r requirements.txt
pytest
python app.py                 # http://localhost:5000
```

With Docker:

```bash
docker build -t flask-render-demo .
docker run -p 5000:5000 flask-render-demo
curl localhost:5000/health
```

## CI/CD pipeline

Defined in `.github/workflows/ci-cd.yml`.

| Job | Runs on | What it does |
|---|---|---|
| `test` | every push and pull request | installs dependencies and runs `pytest` |
| `build-and-push` | push to `main`, after `test` passes | builds the Docker image and pushes it to Docker Hub (`latest` and commit SHA tags) |
| `deploy` | after `build-and-push` | calls the Render deploy hook, then polls `/health` until it returns HTTP 200 |

## Setup

### 1. GitHub repository
Create a repo and push all files, keeping the workflow at `.github/workflows/ci-cd.yml`.

### 2. Docker Hub
Create an account and an access token (Account Settings → Security → New Access Token).

### 3. Render
1. New → **Web Service** → **Existing image**.
2. Image URL: `docker.io/<your-dockerhub-username>/flask-render-demo:latest`
3. Create the service, then copy its **Deploy Hook** URL from Settings.

### 4. GitHub secrets
In the repo: Settings → Secrets and variables → Actions → New repository secret.

| Secret | Value |
|---|---|
| `DOCKERHUB_USERNAME` | your Docker Hub username |
| `DOCKERHUB_TOKEN` | the Docker Hub access token |
| `RENDER_DEPLOY_HOOK_URL` | the Render deploy hook URL |
| `RENDER_APP_URL` | e.g. `https://your-app.onrender.com` (no trailing slash) |

### 5. Deploy
Push to `main`. Watch the run under the repo's **Actions** tab. When it is green, open `https://<your-app>.onrender.com/health`.

## Troubleshooting

- **No workflow run appears:** check the file path is exactly `.github/workflows/ci-cd.yml` and you pushed to `main`.
- **`build-and-push` fails:** the Docker Hub secrets are missing or the token is wrong.
- **`deploy` fails at the hook:** check `RENDER_DEPLOY_HOOK_URL`.
- **Health check times out:** Render's free tier can take a few minutes to start; increase the retry count in the workflow.
- **Render serves an old image:** append `&imgURL=docker.io/<user>/flask-render-demo:<commit-sha>` to the deploy hook URL to deploy a specific tag.
