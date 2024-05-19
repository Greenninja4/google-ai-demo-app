# Vertex AI Workshop Project

The following steps are written to run a Docker image locally and in the Google cloud with Vertex AI.

<img width="450" alt="Screenshot 2024-05-19 at 11 01 26 AM" src="https://github.com/Greenninja4/google-ai-demo-app/assets/15525028/ef81ec52-560b-4fa1-85a7-f1e3302a0513">
<img width="450" alt="Screenshot 2024-05-19 at 11 02 59 AM" src="https://github.com/Greenninja4/google-ai-demo-app/assets/15525028/6aa91c4d-1c79-420e-94e8-f23d1c16b43d">

## Prerequisites for local development

- [Docker](https://docs.docker.com/desktop/)

## Run the app via docker

Create project credentials by following the steps [here](https://developers.google.com/workspace/guides/create-credentials#create_credentials_for_a_service_account). Save the credentials JSON to `./streamlit-app/google-creds.json`.

Create `.env` file with the following variables:
```bash
PROJECT_ID=<Your Project ID>
REGION=<Your Google Cloud Region>
GOOGLE_APPLICATION_CREDENTIALS=google-creds.json
```

Deploy service:
```bash
docker-compose up --build -d
```

Bring down service:
```bash
docker-compose down
```

## Deploy the app to Cloud Run

Enable APIs:
```bash
gcloud services enable cloudbuild.googleapis.com cloudfunctions.googleapis.com run.googleapis.com logging.googleapis.com storage-component.googleapis.com aiplatform.googleapis.com
```

Add the following to your `.env` file:
```bash
SERVICE_NAME='gemini-app-playground' # Name of your Cloud Run service.
AR_REPO='gemini-app-repo'            # Name of your repository in Artifact Registry that stores your application container image.
```

Create repo in Artifact Registry:
```bash
gcloud artifacts repositories create "$AR_REPO" --location="$REGION" --repository-format=Docker
```

Set up authentication to repo:
```bash
gcloud auth configure-docker "$REGION-docker.pkg.dev"
```

Build container image for app:
```bash
gcloud builds submit --tag "$REGION-docker.pkg.dev/$PROJECT_ID/$AR_REPO/$SERVICE_NAME"
```

Deploy app on Cloud Run:
```bash
gcloud run deploy "$SERVICE_NAME" \
  --port=8080 \
  --image="$REGION-docker.pkg.dev/$PROJECT_ID/$AR_REPO/$SERVICE_NAME" \
  --allow-unauthenticated \
  --region=$REGION \
  --platform=managed  \
  --project=$PROJECT_ID \
  --set-env-vars=PROJECT_ID=$PROJECT_ID,REGION=$REGION
```
