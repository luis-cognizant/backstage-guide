# GitHub Integration for Backstage

This guide covers connecting Backstage to GitHub to discover repositories and enable catalog functionality.

## Integration Flow

For a visual representation of the GitHub integration flow, see [diagrams.md](./diagrams.md#github-integration-flow).

## Setting Up GitHub Integration

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

**Facilitator Tip:** Have participants share their GitHub org name (or use a test org you control). Show how to check the integration in the UI. If errors occur, verify the PAT scope and network access. For common GitHub integration issues, see the [Troubleshooting Guide](./faq_troubleshooting.md#issue-github-integration-fails-no-repos-or-workflows-visible).

## Installing GitHub Actions Plugin

Use the `@backstage-community/plugin-github-actions` plugin to display GitHub Actions workflows, including PR-triggered tests and historical runs.

**Checklist for Plugin Installation:**
- [ ] Install the plugin in the `packages/app` directory:
  ```bash
  cd packages/app
  yarn add @backstage-community/plugin-github-actions
  ```
- [ ] Register the plugin in `packages/app/src/plugins.ts` (create if it doesn't exist):
  ```typescript
  // packages/app/src/plugins.ts
  import { GithubActionsPlugin } from '@backstage-community/plugin-github-actions';

  export const githubActionsPlugin = GithubActionsPlugin;
  ```
- [ ] Add the plugin's component to the entity page in `packages/app/src/components/catalog/EntityPage.tsx`. Find the `serviceEntityPage` section and add a CI/CD tab:
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
  This adds a "CI/CD" tab to component pages, showing Actions workflows.
- [ ] Ensure the GitHub integration in `app-config.yaml` has a token with `repo` scope (from previous section).
- [ ] Test locally: Run `yarn dev` in the `backstage` root to preview changes.
- [ ] Redeploy to GKE:
  ```bash
  docker build -t us-central1-docker.pkg.dev/[PROJECT_ID]/backstage-repo/backstage .
  docker push us-central1-docker.pkg.dev/[PROJECT_ID]/backstage-repo/backstage
  kubectl rollout restart deployment backstage
  ```
  Or, for Cloud Run: `gcloud run deploy` as above.
- [ ] Verify: In Backstage UI, navigate to a component's page (requires a `catalog-info.yaml` in the repo), click the "CI/CD" tab, and confirm workflows (including PR-triggered tests and history) are displayed.

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
This triggers on PRs and will appear in Backstage's CI/CD tab.

**Facilitator Tip:** Walk through editing `EntityPage.tsx` on your screen, as this is a common tripping point. Show the CI/CD tab in the UI and explain how it pulls live data from GitHub. Have a test repo ready with a sample workflow. For plugin installation issues, see the [Troubleshooting Guide](./faq_troubleshooting.md#issue-plugins-dont-appear-in-the-ui).

## CodeQL Integration

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
- [ ] View CodeQL runs in Backstage's CI/CD tab (via the GitHub Actions plugin).
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
- [ ] Redeploy and verify the "Insights" tab shows CodeQL results and history.

**Facilitator Tip:** Explain that CodeQL requires a GitHub Advanced Security license for private repos. Use a public repo for the demo if licensing is an issue. Show the workflow run in the CI/CD tab. For more details on security configurations, see the [Project Resilience Guide](./project_resilience_guide.md).

## Deployments Integration

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
- [ ] Annotate components in `catalog-info.yaml`: `github.com/project-slug: your-org/repo`.
- [ ] Redeploy and verify the "Deployments" tab shows status, environments, and history.

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