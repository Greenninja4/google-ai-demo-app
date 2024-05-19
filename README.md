# Vertex AI Workshop Project

The following steps are explcitly written to help figure out how to run this docker image in the Google Cloud.

## Prerequisites for local development

- [Docker](https://docs.docker.com/desktop/)

## Run the app via docker

Create `.env` file with the following variables:
```bash
PROJECT_ID=
REGION=
```

Build docker image:
```bash
docker build -t my-python-app .
```

Run image via docker:
```bash
docker run -p 8080:8080 my-python-app
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
