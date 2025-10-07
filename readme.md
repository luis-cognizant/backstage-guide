# Backstage Workshop: Installation on Google Cloud, GitHub Integration, and Advanced Features

## Introduction
Welcome to this hands-on workshop! This guide will walk you through installing Backstage (Spotify's open-source developer portal) on Google Cloud Platform (GCP) from scratch, connecting it to GitHub, and enabling features like viewing GitHub Actions test results (triggered by pull requests), historical run data, CodeQL scans, and deployments. We'll cover community plugins, creating custom views, the anatomy of a plugin, and how to register components using YAML files. Security configurations and GitHub webhooks for real-time notifications/widgets are included for a production-ready setup.

This guide is designed for a group workshop with clear, step-by-step instructions and checklists to ensure everyone can follow along. Assume participants have basic familiarity with Git, YAML, and command-line tools, but we'll keep steps explicit to accommodate beginners. The workshop will take 2-4 hours, so adjust pacing based on your group’s progress.

**Workshop Goals:**
- Deploy Backstage on GCP using Google Kubernetes Engine (GKE) and Cloud SQL.
- Integrate with GitHub to display CI/CD workflows, CodeQL scans, and deployments.
- Explore community plugins and customize views.
- Understand component registration and secure GitHub integration with webhooks.

## Prerequisites
Before starting, ensure all participants have:
- A GCP account with billing enabled (free tier may suffice for small setups).
- Access to Google Kubernetes Engine (GKE), Cloud Run, Cloud SQL, and Artifact Registry.
- A GitHub account with admin access to an organization or repository.
- Local tools installed: Node.js (v18+), Yarn, Docker, `gcloud` CLI, and `kubectl`.
- A code editor (e.g., VS Code) for editing files.
- Basic knowledge of Git, YAML, and command-line operations.

**Pre-Workshop Setup Checklist:**
- [ ] Create a GCP project via the GCP Console (`IAM & Admin > Create a Project`).
  - Note the `PROJECT_ID` (e.g., `my-backstage-project-123`) displayed in the Console or via `gcloud projects list`.
- [ ] Enable GCP APIs: Compute Engine, GKE, Cloud SQL, Artifact Registry.
  - Run: `gcloud services enable compute.googleapis.com container.googleapis.com sqladmin.googleapis.com artifactregistry.googleapis.com`
- [ ] Install `gcloud` CLI (https://cloud.google.com/sdk/docs/install).
- [ ] Install `kubectl`: `gcloud components install kubectl`.
- [ ] Install Node.js (v18+), Yarn (`npm install -g yarn`), and Docker.
- [ ] Set up a GitHub personal access token (PAT) with `repo` scope or a GitHub App (details in Security section).
- [ ] Ensure all participants have their `PROJECT_ID` and GitHub credentials ready.

**Facilitator Tip:** Before the workshop, have participants verify their `PROJECT_ID` and tool installations. Share a pre-workshop email with setup instructions and links to install tools. During the session, display your screen for commands and file edits to guide the group.

## Section 1: Installing Backstage on Google Cloud
We'll deploy Backstage on GKE with Cloud SQL for PostgreSQL storage. This setup is robust for a workshop demo, but we’ll also mention a simpler Cloud Run option for reference. Replace `[PROJECT_ID]` in all commands with your actual GCP `PROJECT_ID`.

### Step 1.1: Set Up the GCP Environment
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

**Facilitator Tip:** Pause after this step to ensure everyone’s cluster and database are running. Show how to check status in the GCP Console (GKE > Clusters, Cloud SQL > Instances).

### Step 1.2: Download and Configure Backstage
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
  Replace `[YOUR_SECURE_PASSWORD]` with the password set earlier. For GKE, you’ll update this later with Cloud SQL Proxy or public IP settings.
- [ ] Build a Docker image:
  ```bash
  docker build -t gcr.io/[PROJECT_ID]/backstage .
  ```
  Replace `[PROJECT_ID]` with your GCP project ID (e.g., `gcr.io/my-backstage-project-123/backstage`).
- [ ] Set up Artifact Registry:
  ```bash
  gcloud artifacts repositories create backstage-repo \
    --repository-format=docker \
    --location=us-central1
  ```
- [ ] Authenticate Docker: `gcloud auth configure-docker us-central1-docker.pkg.dev`.
- [ ] Push the image:
  ```bash
  docker push gcr.io/[PROJECT_ID]/backstage
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
          image: gcr.io/[PROJECT_ID]/backstage
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

**Alternative: Cloud Run Deployment (Optional for Simpler Setup):**
- [ ] Deploy the same Docker image to Cloud Run:
  ```bash
  gcloud run deploy backstage \
    --image=gcr.io/[PROJECT_ID]/backstage \
    --platform=managed \
    --region=us-central1 \
    --allow-unauthenticated
  ```
- [ ] For Cloud SQL, use the Cloud SQL Auth Proxy or public IP (update `app-config.yaml` accordingly).
- [ ] Get the Cloud Run URL from the command output or GCP Console.

**Facilitator Tip:** Demonstrate accessing the Backstage UI. If anyone’s deployment fails, check `kubectl logs` or Cloud Run logs in the GCP Console. Ensure all participants can access their instance before proceeding.

## Section 2: Connecting Backstage to GitHub
Integrate Backstage with GitHub to discover repositories and enable catalog functionality.

**Checklist:**
- [ ] Create a GitHub Personal Access Token (PAT):
  - Go to GitHub > Settings > Developer settings > Personal access tokens > Tokens (classic).
  - Generate a token with `repo` scope (for Actions, PRs, etc.).
  - Copy the token (e.g., `ghp_xxxxxxxxxxxxxxxx`).
- [ ] Add GitHub integration to `app-config.yaml`:
  ```yaml
  integrations:
    github:
      - host: github.com
        token: ${GITHUB_TOKEN}
  ```
  Replace `${GITHUB_TOKEN}` with the PAT (e.g., set as an environment variable or hardcode for demo, but use secrets in production).
- [ ] Restart the Backstage deployment:
  ```bash
  kubectl rollout restart deployment backstage
  ```
  Or, for Cloud Run: Redeploy with `gcloud run deploy`.
- [ ] Verify in Backstage UI: Go to `Settings > Integrations > GitHub` to confirm the connection.
- [ ] Add a GitHub discovery provider to auto-import repos:
  ```yaml
  catalog:
    providers:
      github:
        my-org:
          organization: '[YOUR_GITHUB_ORG]'
          catalogPath: catalog-info.yaml
  ```
  Replace `[YOUR_GITHUB_ORG]` with your GitHub organization name (e.g., `mycompany`).

**Facilitator Tip:** Have participants share their GitHub org name (or use a test org you control). Show how to check the integration in the UI. If errors occur, verify the PAT scope and network access.

## Section 3: Integrating GitHub Actions for Test Results and History
Use the `@backstage-community/plugin-github-actions` plugin to display GitHub Actions workflows, including PR-triggered tests and historical runs.

**Checklist for Plugin Installation:**
- [ ] Install the plugin in the `packages/app` directory:
  ```bash
  cd packages/app
  yarn add @backstage-community/plugin-github-actions
  ```
- [ ] Register the plugin in `packages/app/src/plugins.ts` (create if it doesn’t exist):
  ```typescript
  // packages/app/src/plugins.ts
  import { GithubActionsPlugin } from '@backstage-community/plugin-github-actions';

  export const githubActionsPlugin = GithubActionsPlugin;
  ```
- [ ] Add the plugin’s component to the entity page in `packages/app/src/components/catalog/EntityPage.tsx`. Find the `serviceEntityPage` section (or similar, depending on your setup) and add a CI/CD tab:
  ```typescript
  import { EntityGithubActionsContent } from '@backstage-community/plugin-github-actions';

  // In the serviceEntityPage definition
  const serviceEntityPage = (
    <EntityLayout>
      {/* Existing tabs, e.g., Overview */}
      <EntityLayout.Route path="/ci-cd" title="CI/CD">
        <EntityGithubActionsContent />
      </EntityLayout.Route>
    </EntityLayout>
  );
  ```
  This adds a “CI/CD” tab to component pages, showing Actions workflows.
- [ ] Ensure the GitHub integration in `app-config.yaml` has a token with `repo` scope (from Section 2).
- [ ] Test locally: Run `yarn dev` in the `backstage` root to preview changes.
- [ ] Redeploy to GKE:
  ```bash
  docker build -t gcr.io/[PROJECT_ID]/backstage .
  docker push gcr.io/[PROJECT_ID]/backstage
  kubectl rollout restart deployment backstage
  ```
  Or, for Cloud Run: `gcloud run deploy` as above.
- [ ] Verify: In Backstage UI, navigate to a component’s page (requires a `catalog-info.yaml` in the repo, see Section 9), click the “CI/CD” tab, and confirm workflows (including PR-triggered tests and history) are displayed.

**Example GitHub Actions Workflow (for PR Tests):**
Add to `.github/workflows/test.yml` in a repo:
```yaml
name: Test
on: [pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: echo "Running tests"
```
This triggers on PRs and will appear in Backstage’s CI/CD tab.

**Facilitator Tip:** Walk through editing `EntityPage.tsx` on your screen, as this is a common tripping point. Show the CI/CD tab in the UI and explain how it pulls live data from GitHub. Have a test repo ready with a sample workflow.

## Section 4: Integrating CodeQL Scans
CodeQL scans (part of GitHub Advanced Security) run as Actions workflows and can be viewed via the GitHub Actions plugin or a dedicated insights plugin.

**Checklist:**
- [ ] Enable GitHub Advanced Security in your repo: GitHub > Repo > Settings > Code security and analysis > Enable GitHub Advanced Security.
- [ ] Add a CodeQL workflow to `.github/workflows/codeql.yml`:
  ```yaml
  name: CodeQL
  on: [push, pull_request]
  jobs:
    analyze:
      runs-on: ubuntu-latest
      steps:
      - uses: actions/checkout@v4
      - uses: github/codeql-action/init@v3
        with:
          languages: javascript
      - uses: github/codeql-action/analyze@v3
  ```
- [ ] View CodeQL runs in Backstage’s CI/CD tab (via the GitHub Actions plugin).
- [ ] (Optional) For dedicated security insights, install `@roadiehq/backstage-plugin-github-insights`:
  ```bash
  cd packages/app
  yarn add @roadiehq/backstage-plugin-github-insights
  ```
- [ ] Register in `packages/app/src/plugins.ts`:
  ```typescript
  import { GithubInsightsPlugin } from '@roadiehq/backstage-plugin-github-insights';
  export const githubInsightsPlugin = GithubInsightsPlugin;
  ```
- [ ] Add to `EntityPage.tsx`:
  ```typescript
  import { EntityGithubInsightsContent } from '@roadiehq/backstage-plugin-github-insights';

  // In serviceEntityPage
  <EntityLayout.Route path="/insights" title="Insights">
    <EntityGithubInsightsContent />
  </EntityLayout.Route>
  ```
- [ ] Update GitHub token in `app-config.yaml` to include `security_events` scope.
- [ ] Redeploy and verify the “Insights” tab shows CodeQL results and history.

**Facilitator Tip:** Explain that CodeQL requires a GitHub Advanced Security license for private repos. Use a public repo for the demo if licensing is an issue. Show the workflow run in the CI/CD tab.

## Section 5: Integrating Deployments
Use the `@backstage-community/plugin-github-deployments` plugin for deployment status and history.

**Checklist:**
- [ ] Install: `yarn add @backstage-community/plugin-github-deployments` (in `packages/app`).
- [ ] Register in `packages/app/src/plugins.ts`:
  ```typescript
  import { GithubDeploymentsPlugin } from '@backstage-community/plugin-github-deployments';
  export const githubDeploymentsPlugin = GithubDeploymentsPlugin;
  ```
- [ ] Add to `EntityPage.tsx`:
  ```typescript
  import { EntityGithubDeploymentsCard } from '@backstage-community/plugin-github-deployments';

  // In serviceEntityPage
  <EntityLayout.Route path="/deployments" title="Deployments">
    <EntityGithubDeploymentsCard />
  </EntityLayout.Route>
  ```
- [ ] Annotate components in `catalog-info.yaml` (see Section 9): `github.com/project-slug: your-org/repo`.
- [ ] Redeploy and verify the “Deployments” tab shows status, environments, and history.

**Example Deployment Workflow:**
Add to `.github/workflows/deploy.yml`:
```yaml
name: Deploy
on:
  push:
    branches: [main]
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: echo "Deploying to production"
```
This appears in the Deployments tab.

**Facilitator Tip:** Show how deployments tie to Actions. Use a test repo with a deployment workflow to demonstrate.

## Section 6: Community Plugins for Functionality
Recommended plugins for this use case:
- `@backstage-community/plugin-github-actions`: For tests/history.
- `@backstage-community/plugin-github-deployments`: For deployments.
- `@roadiehq/backstage-plugin-github-pull-requests`: For PR views.
- `@roadiehq/backstage-plugin-github-insights`: For CodeQL/security.

**Checklist for Adding Plugins:**
- [ ] Search the Backstage plugin directory (https://backstage.io/plugins) for updates.
- [ ] Install via `yarn add [PLUGIN_NAME]` in `packages/app`.
- [ ] Import and export in `plugins.ts`.
- [ ] Add components to `EntityPage.tsx` or custom routes.
- [ ] Configure in `app-config.yaml` if needed (e.g., API tokens).
- [ ] Redeploy and test.

**Facilitator Tip:** Highlight the plugin directory as a resource. Install one additional plugin (e.g., PRs) if time allows, to show the process.

## Section 7: Creating Custom Views
Customize entity pages or create new tabs for aggregated views.

**Checklist:**
- [ ] In `packages/app/src/components/catalog/EntityPage.tsx`, add a custom tab:
  ```typescript
  import { EntityGithubActionsContent } from '@backstage-community/plugin-github-actions';

  const customTab = (
    <EntityLayout.Route path="/custom" title="Custom View">
      <Card>
        <Typography variant="h6">CI/CD Overview</Typography>
        <EntityGithubActionsContent />
      </Card>
    </EntityLayout.Route>
  );
  // Add to serviceEntityPage
  ```
- [ ] For filtering, use `EntityListProvider` in a custom component:
  ```typescript
  import { EntityListProvider } from '@backstage/plugin-catalog-react';

  const CustomComponent = () => (
    <EntityListProvider query={{ kind: 'Component' }}>
      {/* Custom rendering */}
    </EntityListProvider>
  );
  ```
- [ ] Test locally: `yarn dev`.
- [ ] Redeploy to GKE or Cloud Run.

**Facilitator Tip:** Show a simple custom tab addition. If time is short, skip advanced filtering but mention it as an option.

## Section 8: Anatomy of a Plugin with Configuration
A Backstage plugin has frontend and/or backend parts.

**Plugin Structure:**
- **Folder:** `plugins/my-plugin/` (with `src/`, `package.json`).
- **Files:** `plugin.ts` (exports plugin), `index.ts`, React components (e.g., `MyCard.tsx`).
- **Backend (Optional):** In `plugins/my-plugin-backend/` for API logic.
- **Registration:** Import in `packages/app/src/plugins.ts` and add to routes.

**Example Configuration in `app-config.yaml`:**
```yaml
myPlugin:
  baseUrl: https://api.example.com
  token: ${MY_TOKEN}
```

**Checklist for Building a Plugin:**
- [ ] Create: `yarn create-plugin --id my-plugin` (in `backstage` root).
- [ ] Add components in `plugins/my-plugin/src/components/`.
- [ ] Configure proxies/secrets in `app-config.yaml`.
- [ ] Register in `plugins.ts` and add to `EntityPage.tsx` or `App.tsx`.
- [ ] Test and redeploy.

**Facilitator Tip:** Explain the plugin structure briefly. If time allows, demo creating a simple plugin with `yarn create-plugin`.

## Section 9: Listing Components in Backstage
Register components using `catalog-info.yaml` in each repo.

**Example `catalog-info.yaml` (in repo root):**
```yaml
apiVersion: backstage.io/v1alpha1
kind: Component
metadata:
  name: my-service
  annotations:
    github.com/project-slug: [YOUR_ORG]/[REPO_NAME]
spec:
  type: service
  lifecycle: production
  owner: team-a
```
Replace `[YOUR_ORG]/[REPO_NAME]` with your GitHub org and repo (e.g., `mycompany/my-service`).

**Checklist:**
- [ ] Create `catalog-info.yaml` in each repo’s root.
- [ ] Register manually: In Backstage UI, go to `Create > Register Component` and enter the YAML file’s URL (e.g., `https://github.com/[YOUR_ORG]/[REPO_NAME]/blob/main/catalog-info.yaml`).
- [ ] Or use discovery (from Section 2) to auto-import all repos in the org.
- [ ] Verify: Check the Catalog in Backstage UI; search/filter by name or type.

**Facilitator Tip:** Have participants add `catalog-info.yaml` to a test repo. Show the Catalog UI and how components appear.

## Section 10: Security Configurations and Permissions
Use a GitHub App for secure integration (preferred over PATs).

**GitHub App Permissions (Read-Only Where Possible):**
- Repository: Contents (read), Issues (read), Metadata (read), Pull requests (read), Workflows (read), Deployments (read/write if triggering).
- Organization: Members (read) for discovery.
- Security events (read) for CodeQL.

**Checklist:**
- [ ] Create a GitHub App: GitHub > Org Settings > Developer settings > GitHub Apps > New App.
  - Set Homepage URL to your Backstage URL.
  - Enable required permissions (above).
  - Generate a client secret and note the App ID.
- [ ] Install the App on target repos or the org.
- [ ] Update `app-config.yaml`:
  ```yaml
  integrations:
    github:
      - host: github.com
        apps:
          - appId: [APP_ID]
            clientId: [CLIENT_ID]
            clientSecret: ${CLIENT_SECRET}
            webhookUrl: http://[BACKSTAGE_URL]/api/webhook
            webhookSecret: [WEBHOOK_SECRET]
  ```
  Replace placeholders with your App details and Backstage URL.
- [ ] Use Kubernetes Secrets or environment variables for sensitive data in production.

**Facilitator Tip:** Explain why GitHub Apps are more secure than PATs. For the demo, a PAT may be simpler, but show the App setup in GitHub for awareness.

## Section 11: Webhooks for Notifications and Real-Time Widgets
Set up GitHub webhooks for real-time updates (e.g., catalog refreshes or dashboard widgets).

**Checklist:**
- [ ] In GitHub: Repo/Org > Settings > Webhooks > Add webhook.
  - Payload URL: `http://[BACKSTAGE_EXTERNAL_IP]/api/webhook`.
  - Content type: `application/json`.
  - Secret: Generate a random string (e.g., `openssl rand -hex 16`) and note it.
  - Events: Select `Push`, `Repository`, `Pull requests`.
- [ ] In `app-config.yaml`, add:
  ```yaml
  catalog:
    processors:
      githubOrg:
        webhookSecret: [WEBHOOK_SECRET]
  ```
- [ ] Redeploy Backstage.
- [ ] Verify: Make a commit or PR in a repo; check if Backstage updates the catalog or shows live data in plugins (e.g., GitHub Actions widget).
- [ ] For widgets: The GitHub Actions plugin’s `<EntityGithubActionsContent />` shows live workflow status on entity pages.

**Facilitator Tip:** Demonstrate a webhook-triggered update (e.g., push to a repo and show the catalog refresh). Highlight how widgets enhance the dashboard.

## Wrap-Up and Q&A
**Key Takeaways:**
- Backstage is deployed on GCP with GKE or Cloud Run.
- GitHub integration enables CI/CD visibility (Actions, CodeQL, deployments).
- Components are registered via `catalog-info.yaml`.
- Plugins and custom views extend functionality.
- Secure with GitHub Apps and webhooks for real-time updates.

**Next Steps:**
- Explore more plugins in the Backstage directory.
- Set up monitoring, scaling, and backups on GCP for production.
- Check Backstage docs (https://backstage.io/docs) and community forums for support.

**Bonus:** Webhooks enable real-time widgets, making the Backstage dashboard dynamic with live GitHub data (e.g., test statuses, PR updates).

**Facilitator Tip:** Allocate 15-20 minutes for Q&A. Encourage participants to experiment with plugins or custom views post-workshop. If issues arise, use `kubectl logs`, Cloud Run logs, or Backstage’s error messages to debug. Share a link to this guide for reference.
