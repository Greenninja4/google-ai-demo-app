# Vertex AI Workshop Project

The following steps are explcitly written to help figure out how to run this docker image in the Google Cloud.

## Task 1. Configure your environment and project

To set your project ID and region environment variables, in Cloud Shell, run the following commands:
```bash
PROJECT_ID=$(gcloud config get-value project)
REGION=set at lab start
echo "PROJECT_ID=${PROJECT_ID}"
echo "REGION=${REGION}"
```

Enable APIs:
```bash
gcloud services enable cloudbuild.googleapis.com cloudfunctions.googleapis.com run.googleapis.com logging.googleapis.com storage-component.googleapis.com aiplatform.googleapis.com
```

## Task 2. Set up the application environment

Set up a Python virtual environment:
```bash
python3 -m venv gemini-streamlit
```

Activate Python virtual environment:
```bash
source gemini-streamlit/bin/activate
```

Install application dependencies
```bash
pip install -r requirements.txt
```

## Task 3. Develop the app

<Developed and code written>

## Task 4. Run and test the app locally

Run the app lcoally:
```bash
streamlit run app.py \
--browser.serverAddress=localhost \
--server.enableCORS=false \
--server.enableXsrfProtection=false \
--server.port 8080
```

## <Tasks 5-13 to set up app>

## Task 14. Deploy the app to Cloud Run

Set env vars:
```bash
PROJECT_ID=$(gcloud config get-value project)
REGION=set at lab start
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
