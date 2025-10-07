# Backstage Installation on Google Cloud

This guide covers installing Backstage on Google Cloud Platform using Google Kubernetes Engine (GKE) and Cloud SQL.

## Architecture Overview

For visual architecture diagrams, see [diagrams.md](./diagrams.md#backstage-architecture-on-gcp).

We'll deploy Backstage on GKE with Cloud SQL for PostgreSQL storage. This setup is robust for a workshop demo, but we'll also mention a simpler Cloud Run option for reference. Replace `[PROJECT_ID]` in all commands with your actual GCP `PROJECT_ID`.

## Step 1.1: Set Up the GCP Environment

Create a GKE cluster for Backstage and a Cloud SQL instance for the database.

**Checklist for GKE and Cloud SQL Setup:**
- [ ] Set your `PROJECT_ID` in `gcloud`: `gcloud config set project [PROJECT_ID]`.
- [ ] Create a GKE cluster:
  ```bash
  gcloud container clusters create backstage-cluster \
    --zone=us-central1-a \
    --num-nodes=3 \
    --machine-type=e2-standard-4
  ```
  This creates a cluster with 3 nodes for reliability. Wait ~5-10 minutes for completion.
- [ ] Get GKE credentials: `gcloud container clusters get-credentials backstage-cluster --zone=us-central1-a`.
- [ ] Create a Cloud SQL PostgreSQL instance:
  ```bash
  gcloud sql instances create backstage-db \
    --database-version=POSTGRES_13 \
    --cpu=2 \
    --memory=4GB \
    --region=us-central1
  ```
- [ ] Create a database: `gcloud sql databases create backstage --instance=backstage-db`.
- [ ] Set a root password:
  ```bash
  gcloud sql users set-password postgres \
    --instance=backstage-db \
    --password=[YOUR_SECURE_PASSWORD]
  ```
  Replace `[YOUR_SECURE_PASSWORD]` with a strong password (e.g., `MySecurePass123!`).
- [ ] Note the Cloud SQL connection details (instance connection name, e.g., `[PROJECT_ID]:us-central1:backstage-db`) from the GCP Console or `gcloud sql instances describe backstage-db`.

**Facilitator Tip:** Pause after this step to ensure everyone's cluster and database are running. Show how to check status in the GCP Console (GKE > Clusters, Cloud SQL > Instances). If you encounter issues, refer to the [Troubleshooting Guide](./faq_troubleshooting.md#troubleshooting-guide).

## Step 1.2: Download and Configure Backstage

Clone the Backstage repository, configure it, and build a Docker image.

**Checklist for Backstage Installation:**
- [ ] Clone the Backstage repo:
  ```bash
  git clone https://github.com/backstage/backstage.git
  cd backstage
  ```
- [ ] Install dependencies: `yarn install` (run in the `backstage` directory).
- [ ] Create `app-config.yaml` in the `backstage` root directory with database settings:
  ```yaml
  backend:
    database:
      client: pg
      connection:
        host: 127.0.0.1
        port: 5432
        user: postgres
        password: [YOUR_SECURE_PASSWORD]
  ```
  Replace `[YOUR_SECURE_PASSWORD]` with the password set earlier. For GKE, you'll update this later with Cloud SQL Proxy or public IP settings.
- [ ] Build a Docker image:
  ```bash
  docker build -t us-central1-docker.pkg.dev/[PROJECT_ID]/backstage-repo/backstage .
  ```
  Replace `[PROJECT_ID]` with your GCP project ID (e.g., `us-central1-docker.pkg.dev/my-backstage-project-123/backstage-repo/backstage`).
- [ ] Set up Artifact Registry:
  ```bash
  gcloud artifacts repositories create backstage-repo \
    --repository-format=docker \
    --location=us-central1
  ```
- [ ] Authenticate Docker: `gcloud auth configure-docker us-central1-docker.pkg.dev`.
- [ ] Push the image:
  ```bash
  docker push us-central1-docker.pkg.dev/[PROJECT_ID]/backstage-repo/backstage
  ```
- [ ] Create a Kubernetes deployment YAML (`backstage-deployment.yaml`):
  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: backstage
  spec:
    replicas: 1
    selector:
      matchLabels:
        app: backstage
    template:
      metadata:
        labels:
          app: backstage
      spec:
        containers:
        - name: backstage
          image: us-central1-docker.pkg.dev/[PROJECT_ID]/backstage-repo/backstage
          ports:
          - containerPort: 7007
          env:
          - name: POSTGRES_HOST
            value: "127.0.0.1" # Update with Cloud SQL Proxy or public IP
          - name: POSTGRES_USER
            value: "postgres"
          - name: POSTGRES_PASSWORD
            value: "[YOUR_SECURE_PASSWORD]"
  ---
  apiVersion: v1
  kind: Service
  metadata:
    name: backstage-service
  spec:
    selector:
      app: backstage
    ports:
    - port: 80
      targetPort: 7007
    type: LoadBalancer
  ```
  Replace `[PROJECT_ID]` and `[YOUR_SECURE_PASSWORD]`. For production, use Kubernetes Secrets for the password.
- [ ] Deploy to GKE:
  ```bash
  kubectl apply -f backstage-deployment.yaml
  ```
- [ ] Get the external IP: `kubectl get services backstage-service`. Wait ~2-5 minutes for the IP to appear.
- [ ] Access Backstage at `http://[EXTERNAL_IP]`. Log in as a guest (default mode).

## Alternative: Cloud Run Deployment (Optional for Simpler Setup)

- [ ] Deploy the same Docker image to Cloud Run:
  ```bash
  gcloud run deploy backstage \
    --image=us-central1-docker.pkg.dev/[PROJECT_ID]/backstage-repo/backstage \
    --platform=managed \
    --region=us-central1 \
    --allow-unauthenticated
  ```
- [ ] For Cloud SQL, use the Cloud SQL Auth Proxy or public IP (update `app-config.yaml` accordingly).
- [ ] Get the Cloud Run URL from the command output or GCP Console.

**Facilitator Tip:** Demonstrate accessing the Backstage UI. If anyone's deployment fails, check `kubectl logs` or Cloud Run logs in the GCP Console. Ensure all participants can access their instance before proceeding. For troubleshooting deployment issues, see the [Troubleshooting Guide](./faq_troubleshooting.md#issue-backstage-ui-is-inaccessible-no-external-ip-or-404).