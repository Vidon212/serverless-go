## Serverless Go (Cloud Run sample)

Minimal serverless HTTP service written in Go. Suitable for Google Cloud Run.

### Endpoints
- `GET /` — Welcome JSON
- `GET /healthz` — Health check
- `POST /echo` — Echoes `{ "message": "..." }`

### Requirements
- Go 1.22+ (optional for local `go run`)
- Docker
- gcloud SDK (for Cloud Run deploy)

### Run locally (without Docker)
```bash
go run ./main.go
# in another terminal
curl -s localhost:8080/healthz
curl -s -X POST -H "Content-Type: application/json" localhost:8080/echo -d '{"message":"hello"}'
```

### Build and run with Docker
```bash
docker build -t serverlss-go:dev .
docker run --rm -p 8080:8080 serverlss-go:dev
```

### Deploy to Cloud Run
```bash
export PROJECT_ID=YOUR_PROJECT_ID
export REGION=us-central1
export REPO=three-tier-repo         # or any Artifact Registry (Docker) repo you have
export SERVICE_NAME=go-serverless

gcloud config set project "${PROJECT_ID}"
gcloud services enable run.googleapis.com artifactregistry.googleapis.com
gcloud auth configure-docker "${REGION}-docker.pkg.dev" --quiet

# Build and push
docker build -t "${REGION}-docker.pkg.dev/${PROJECT_ID}/${REPO}/${SERVICE_NAME}:v1" .
docker push "${REGION}-docker.pkg.dev/${PROJECT_ID}/${REPO}/${SERVICE_NAME}:v1"

# Deploy (fully managed)
gcloud run deploy "${SERVICE_NAME}" \
  --image "${REGION}-docker.pkg.dev/${PROJECT_ID}/${REPO}/${SERVICE_NAME}:v1" \
  --region "${REGION}" \
  --platform managed \
  --allow-unauthenticated

# Get URL
gcloud run services describe "${SERVICE_NAME}" --region "${REGION}" --format='value(status.url)'
```

### Test deployed service
```bash
URL=$(gcloud run services describe "${SERVICE_NAME}" --region "${REGION}" --format='value(status.url)')
curl -s "${URL}/healthz"
curl -s -X POST -H "Content-Type: application/json" "${URL}/echo" -d '{"message":"hello cloud run"}'
```

### Notes
- The service listens on `$PORT` (default `8080`), which is required by Cloud Run.
- The Docker image uses a multi-stage build and a distroless runtime.


