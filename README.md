# Integration

Below is a **complete, production‑grade `README.md`** for your project.  
It is structured, professional, and ready to drop into GitHub.

You can paste it directly into your repository.

---

# Graph‑Based Email Ingestion & Phishing Detection Service  
### *FastAPI • Microsoft Graph • GCP Cloud Run • Docker • OAuth2 • SIEM Integration*

---

## Overview

This service ingests emails from **Exchange Online** and **Hybrid Exchange Server** using **Microsoft Graph API**, analyzes them for phishing using an AI/ML model, and sends alerts to SIEM or other security repositories. It supports both **scheduled batch scanning** and **real‑time webhook ingestion**, and is deployed as a **containerized FastAPI application on Google Cloud Run**.

This repository includes:

- Email ingestion pipeline (cron + webhook)
- Microsoft Graph OAuth2 integration
- Delta query support for incremental scanning
- AI‑based phishing detection model (starter implementation)
- Alerting to SIEM endpoints
- Optional remediation (quarantine/move message)
- Subscription management for Graph webhooks
- GitHub Actions CI/CD pipeline for Cloud Run deployment

---

## Features

### Email Ingestion
- Pulls emails from Exchange Online / Hybrid Exchange via Microsoft Graph.
- Supports:
  - **All mailboxes**
  - **Range‑based mailboxes**
  - **Group‑based mailboxes**
- Incremental ingestion using **delta queries**.
- Real‑time ingestion using **Graph subscriptions**.

### Phishing Detection
- AI/ML model analyzes:
  - Headers
  - Body content
  - URLs
  - Attachments (metadata)
- Produces:
  - `phishingScore`
  - `label` (phishing / suspicious / benign)
  - `reasons`

### Remediation
- Optional quarantine via Graph (`Mail.ReadWrite`).
- Alerting to SIEM or HTTP endpoints.
- Reporting via `/scan` results.

### Deployment
- Dockerized FastAPI app.
- Runs on **Google Cloud Run**.
- CI/CD via **GitHub Actions**.
- Secrets stored in **GCP Secret Manager**.

### Observability
- Structured JSON logging.
- Error handling with retries/backoff.
- Health endpoint for uptime monitoring.

---

## Architecture

```
Exchange Online / Hybrid Exchange
            │
            ▼
   Microsoft Graph API
            │
 ┌──────────┴──────────┐
 │  Cloud Run (FastAPI) │
 └──────────┬──────────┘
            │
   ┌────────┼─────────┐
   ▼        ▼          ▼
Cron     Webhook     Manual
/scan    /webhook    /subscribe
            │
            ▼
   Phishing ML Model
            │
            ▼
     Alerts / SIEM
```

---

## Project Structure

```
graph-phishing-service/
  app/
    __init__.py
    main.py
    config.py
    auth.py
    graph_client.py
    mailbox.py
    subscriptions.py
    model.py
    alerting.py
    utils.py
  tests/
    test_basic.py
  Dockerfile
  requirements.txt
  README.md
  .github/
    workflows/
      deploy.yaml
```

---

## Authentication (Microsoft Graph)

This service uses **OAuth2 client credentials** with Microsoft Entra ID.

Required Graph permissions:

- `Mail.Read` (minimum)
- `Mail.ReadWrite` (optional, for quarantine)

Token endpoint:

```
POST https://login.microsoftonline.com/{tenant}/oauth2/v2.0/token
```

---

## Configuration

All configuration is handled via environment variables (preferably from GCP Secret Manager).

| Variable | Description |
|---------|-------------|
| `MS_TENANT_ID` | Entra ID tenant |
| `MS_CLIENT_ID` | App registration client ID |
| `MS_CLIENT_SECRET` | App secret |
| `WEBHOOK_CLIENT_STATE` | Secret for validating Graph notifications |
| `WEBHOOK_BASE_URL` | Cloud Run webhook URL |
| `SIEM_ENDPOINT` | Optional SIEM ingestion URL |
| `SIEM_API_KEY` | Optional SIEM auth token |

---

## Running Locally

### 1. Install dependencies

```
pip install -r requirements.txt
```

### 2. Run FastAPI

```
uvicorn app.main:app --reload
```

### 3. Test health endpoint

```
curl http://localhost:8000/health
```

---

## Docker

### Build

```
docker build -t graph-phishing-service .
```

### Run

```
docker run -p 8080:8080 graph-phishing-service
```

---

## Deploying to Cloud Run

### Build & push image

```
gcloud builds submit --tag gcr.io/PROJECT_ID/graph-phishing-service
```

### Deploy

```
gcloud run deploy graph-phishing-service \
  --image gcr.io/PROJECT_ID/graph-phishing-service \
  --platform managed \
  --region REGION \
  --allow-unauthenticated
```

---

## Scheduled Scanning (Cron)

Use **Cloud Scheduler** to POST to:

```
POST https://<cloud-run-url>/scan
```

Example schedule:

- Every 10 minutes for high‑risk groups
- Every 30 minutes for full tenant

---

## Real-Time Webhooks

Create subscriptions via:

```
POST /subscribe
```

Graph will send notifications to:

```
POST /webhook
```

Webhook flow:

1. Validate `clientState`
2. Fetch full message
3. Analyze with ML model
4. Send alert

---

## Phishing Detection Model

Starter model uses:

- Keyword heuristics
- Subject/body analysis
- Sender mismatch detection

Output example:

```json
{
  "messageId": "abc123",
  "phishingScore": 0.92,
  "label": "phishing",
  "reasons": ["Detected suspicious keywords: verify, urgent"]
}
```

You can replace this with:

- scikit‑learn model
- ONNX model
- Transformer/NLP model
- External ML endpoint

---

## Remediation

If `Mail.ReadWrite` is enabled, the service can:

- Move messages to a **Quarantine** folder
- Add categories/flags
- Tag messages for SOC review

Example:

```python
await move_to_quarantine(user_id, message_id, quarantine_folder_id)
```

---

## Observability

### Logging
- Structured JSON logs
- Includes:
  - Graph errors
  - Message processing stats
  - Alert counts

### Monitoring
- Cloud Monitoring dashboards
- Logging-based metrics:
  - `emails_processed_total`
  - `alerts_raised_total`
  - `graph_errors_total`

### Alerting
- Trigger alerts when:
  - No scans succeed within X minutes
  - Webhook failures spike
  - Graph API returns repeated 429/5xx

---

## Testing

Run tests:

```
pytest
```

---

## CI/CD (GitHub Actions)

Included workflow:

- Runs tests
- Builds Docker image
- Pushes to GCP registry
- Deploys to Cloud Run

File: `.github/workflows/deploy.yaml`

---
