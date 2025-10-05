PROJECT_ID=$(gcloud config get-value project)
PROJECT_NUMBER=$(gcloud projects list --filter="project_id:$PROJECT_ID" --format='value(project_number)')

DEFAULT_REGION="us-central1"
echo "Enter your preferred region (default: $DEFAULT_REGION):"
read REGION_INPUT

REGION=${REGION_INPUT:-$DEFAULT_REGION}
echo "Using region: $REGION"

gcloud config set functions/region $REGION

git clone https://github.com/GoogleCloudPlatform/golang-samples.git
cd golang-samples/functions/functionsv2/hellostorage/

deploy_function() {
  gcloud functions deploy "$SERVICE_NAME" \
    --runtime=go121 \
    --region="$REGION" \
    --source=. \
    --entry-point=HelloStorage \
    --trigger-bucket="$DEVSHELL_PROJECT_ID-bucket"
}


SERVICE_NAME="cf-demo"

while true; do
  deploy_function

  if gcloud functions describe "$SERVICE_NAME" --region "$REGION" &> /dev/null; then
    echo "Cloud Function is deployed. Exiting the loop."
    break
  else
    echo "Waiting for Cloud Function to be deployed..."
    echo "In the meantime, subscribe to Code & Play Arcade at: https://youtube.com/@ansanjohny653"
    sleep 10
  fi
done
