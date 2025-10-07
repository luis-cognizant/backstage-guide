# Backstage Workshop Command Cheatsheet

Quick reference for all commands used in the Backstage workshop. All commands are ready to copy and paste!

## Table of Contents
- [Prerequisites Setup](#prerequisites-setup)
- [Google Cloud Setup](#google-cloud-setup)
- [Docker Commands](#docker-commands)
- [Kubernetes Commands](#kubernetes-commands)
- [Backstage Development](#backstage-development)
- [GitHub Integration](#github-integration)
- [Troubleshooting Commands](#troubleshooting-commands)
- [Useful One-liners](#useful-one-liners)

## Prerequisites Setup

| Purpose | Command | Notes |
|---------|---------|-------|
| Check Node.js version | `node --version` | Should be v18+ |
| Install Yarn globally | `npm install -g yarn` | |
| Check Yarn version | `yarn --version` | |
| Check Docker version | `docker --version` | |
| Install gcloud CLI | `curl https://sdk.cloud.google.com \| bash` | macOS/Linux |
| Check gcloud version | `gcloud --version` | |
| Install kubectl | `gcloud components install kubectl` | |
| Check kubectl version | `kubectl version --client` | |

## Google Cloud Setup

| Purpose | Command | Notes |
|---------|---------|-------|
| Login to gcloud | `gcloud auth login` | Opens browser |
| Set project | `gcloud config set project PROJECT_ID` | Replace PROJECT_ID |
| List projects | `gcloud projects list` | Find your PROJECT_ID |
| Create new project | `gcloud projects create PROJECT_ID --name="Backstage Workshop"` | |
| Enable required APIs | `gcloud services enable compute.googleapis.com container.googleapis.com sqladmin.googleapis.com artifactregistry.googleapis.com` | |
| Set default region | `gcloud config set compute/region us-central1` | |
| Set default zone | `gcloud config set compute/zone us-central1-a` | |
| Get current config | `gcloud config list` | |

## Docker Commands

| Purpose | Command | Notes |
|---------|---------|-------|
| Build Backstage image | `docker build -t backstage-app .` | Run from app root |
| Build with tag | `docker build -t gcr.io/PROJECT_ID/backstage:latest .` | |
| Tag for Artifact Registry | `docker tag backstage-app us-central1-docker.pkg.dev/PROJECT_ID/backstage/backstage:latest` | |
| Push to Artifact Registry | `docker push us-central1-docker.pkg.dev/PROJECT_ID/backstage/backstage:latest` | |
| Run locally | `docker run -p 3000:3000 -p 7007:7007 backstage-app` | |
| List images | `docker images` | |
| Remove image | `docker rmi IMAGE_ID` | |
| Configure Docker auth | `gcloud auth configure-docker us-central1-docker.pkg.dev` | |

## Kubernetes Commands

| Purpose | Command | Notes |
|---------|---------|-------|
| Create GKE cluster | `gcloud container clusters create backstage-cluster --zone=us-central1-a --num-nodes=3` | |
| Get cluster credentials | `gcloud container clusters get-credentials backstage-cluster --zone=us-central1-a` | |
| List clusters | `gcloud container clusters list` | |
| Apply Kubernetes manifest | `kubectl apply -f kubernetes-manifest.yaml` | |
| Get pods | `kubectl get pods` | |
| Get services | `kubectl get services` | |
| Get deployments | `kubectl get deployments` | |
| Get pod logs | `kubectl logs POD_NAME` | |
| Follow pod logs | `kubectl logs -f POD_NAME` | |
| Describe pod | `kubectl describe pod POD_NAME` | |
| Delete deployment | `kubectl delete deployment backstage` | |
| Port forward | `kubectl port-forward service/backstage 3000:3000` | |
| Get external IP | `kubectl get service backstage-service` | |

## Backstage Development

| Purpose | Command | Notes |
|---------|---------|-------|
| Create Backstage app | `npx @backstage/create-app@latest` | Interactive setup |
| Install dependencies | `yarn install` | Run from app root |
| Start development server | `yarn dev` | Frontend + backend |
| Start only backend | `yarn start-backend` | Port 7007 |
| Start only frontend | `yarn start` | Port 3000 |
| Build for production | `yarn build` | Creates dist/ |
| Run tests | `yarn test` | |
| Type checking | `yarn tsc` | |
| Lint code | `yarn lint` | |
| Install new plugin | `yarn workspace backend add @backstage/plugin-NAME` | |

## GitHub Integration

| Purpose | Command | Notes |
|---------|---------|-------|
| Create GitHub PAT | Visit: `https://github.com/settings/tokens/new` | Select 'repo' scope |
| Test GitHub connection | `curl -H "Authorization: token YOUR_PAT" https://api.github.com/user` | |
| Add GitHub integration | `yarn workspace backend add @backstage/plugin-github-actions` | |
| Add catalog integration | `yarn workspace backend add @backstage/plugin-catalog-backend-module-github` | |

## Cloud SQL Commands

| Purpose | Command | Notes |
|---------|---------|-------|
| Create Cloud SQL instance | `gcloud sql instances create backstage-db --database-version=POSTGRES_13 --tier=db-f1-micro --region=us-central1` | |
| Create database | `gcloud sql databases create backstage --instance=backstage-db` | |
| Create user | `gcloud sql users create backstage-user --instance=backstage-db --password=YOUR_PASSWORD` | |
| Get connection name | `gcloud sql instances describe backstage-db --format="value(connectionName)"` | |
| Connect to database | `gcloud sql connect backstage-db --user=backstage-user --database=backstage` | |

## Artifact Registry Commands

| Purpose | Command | Notes |
|---------|---------|-------|
| Create repository | `gcloud artifacts repositories create backstage --repository-format=docker --location=us-central1` | |
| List repositories | `gcloud artifacts repositories list` | |
| Configure Docker auth | `gcloud auth configure-docker us-central1-docker.pkg.dev` | |

## Troubleshooting Commands

| Purpose | Command | Notes |
|---------|---------|-------|
| Check pod status | `kubectl get pods -o wide` | Shows node assignment |
| Get pod events | `kubectl get events --sort-by=.metadata.creationTimestamp` | |
| Debug pod | `kubectl exec -it POD_NAME -- /bin/bash` | Interactive shell |
| Check service endpoints | `kubectl get endpoints` | |
| View cluster info | `kubectl cluster-info` | |
| Check node status | `kubectl get nodes` | |
| Restart deployment | `kubectl rollout restart deployment/backstage` | |
| Scale deployment | `kubectl scale deployment backstage --replicas=3` | |
| Check resource usage | `kubectl top pods` | Requires metrics-server |
| Port forward debug | `kubectl port-forward pod/POD_NAME 8080:3000` | |

## Useful One-liners

| Purpose | Command | Notes |
|---------|---------|-------|
| Get external IP quickly | `kubectl get svc backstage-service -o jsonpath='{.status.loadBalancer.ingress[0].ip}'` | |
| Watch pod status | `kubectl get pods -w` | Live updates |
| Get all resources | `kubectl get all` | |
| Delete all pods | `kubectl delete pods --all` | ⚠️ Use carefully |
| Tail all pod logs | `kubectl logs -f deployment/backstage` | |
| Check if port is open | `nc -zv EXTERNAL_IP 3000` | |
| Get current context | `kubectl config current-context` | |
| Switch context | `kubectl config use-context CONTEXT_NAME` | |
| Generate secret | `kubectl create secret generic backstage-secrets --from-literal=GITHUB_TOKEN=your_token --dry-run=client -o yaml` | |
| Apply with watch | `kubectl apply -f manifest.yaml && kubectl get pods -w` | |

## Environment Variables Template

Copy and customize these environment variables:

```bash
# Project Configuration
export PROJECT_ID="your-project-id"
export REGION="us-central1"
export ZONE="us-central1-a"
export CLUSTER_NAME="backstage-cluster"

# GitHub Configuration
export GITHUB_TOKEN="your_github_token"
export GITHUB_CLIENT_ID="your_github_client_id"
export GITHUB_CLIENT_SECRET="your_github_client_secret"

# Database Configuration
export POSTGRES_HOST="localhost"
export POSTGRES_PORT="5432"
export POSTGRES_USER="backstage-user"
export POSTGRES_PASSWORD="your_password"
export POSTGRES_DB="backstage"

# Backstage Configuration
export NODE_ENV="production"
export APP_CONFIG_backend_baseUrl="http://localhost:7007"
export APP_CONFIG_app_baseUrl="http://localhost:3000"
```

## Quick Setup Script

Save this as `quick-setup.sh` for rapid environment setup:

```bash
#!/bin/bash
set -e

echo "🚀 Setting up Backstage workshop environment..."

# Set your project ID here
PROJECT_ID="your-project-id-here"

# Configure gcloud
gcloud config set project $PROJECT_ID
gcloud config set compute/region us-central1
gcloud config set compute/zone us-central1-a

# Enable APIs
echo "📡 Enabling required APIs..."
gcloud services enable compute.googleapis.com container.googleapis.com sqladmin.googleapis.com artifactregistry.googleapis.com

# Create Artifact Registry repository
echo "📦 Creating Artifact Registry repository..."
gcloud artifacts repositories create backstage --repository-format=docker --location=us-central1

# Configure Docker authentication
echo "🔐 Configuring Docker authentication..."
gcloud auth configure-docker us-central1-docker.pkg.dev

# Create GKE cluster
echo "☸️ Creating GKE cluster..."
gcloud container clusters create backstage-cluster --zone=us-central1-a --num-nodes=3

# Get cluster credentials
echo "🔑 Getting cluster credentials..."
gcloud container clusters get-credentials backstage-cluster --zone=us-central1-a

echo "✅ Setup complete! You can now proceed with the workshop."
```

## Tips for Copy-Pasting

1. **Replace placeholders**: Always replace `PROJECT_ID`, `YOUR_TOKEN`, etc. with your actual values
2. **Check context**: Make sure you're in the right directory before running commands
3. **Verify first**: Use `--dry-run` flag when available to test commands
4. **One at a time**: Copy and run one command at a time for easier debugging
5. **Save frequently**: Save your progress with `kubectl apply` after making changes

Remember to check the [FAQ & Troubleshooting](./faq_troubleshooting.md) if any commands fail!