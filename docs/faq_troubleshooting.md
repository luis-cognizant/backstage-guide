## FAQ

**Q: What is the `PROJECT_ID`, and where do I find it?**  
A: The `PROJECT_ID` is the unique identifier for your Google Cloud Platform (GCP) project, used in commands like `gcloud` and for Docker images (e.g., `us-central1-docker.pkg.dev/[PROJECT_ID]/backstage-repo/backstage`). Find it in the GCP Console (top-left project dropdown) or run `gcloud projects list`. Example: `my-backstage-project-123`. Set it with `gcloud config set project [PROJECT_ID]`. See the [main workshop guide](../readme.md) for detailed setup instructions.

**Q: Why use a GitHub App instead of a Personal Access Token (PAT)?**  
A: A GitHub App is more secure because it offers fine-grained permissions, can be restricted to specific repositories, and supports webhook integration. PATs are user-specific, have broader access, and are less manageable for teams. For the workshop demo, a PAT is simpler, but GitHub Apps are recommended for production.

**Q: Can I run Backstage locally instead of on GCP?**  
A: Yes, for testing! Clone the Backstage repo, run `yarn install`, set up a local PostgreSQL database, configure `app-config.yaml` with the database details, and start with `yarn dev`. However, the workshop uses GCP for a production-like setup. Local runs are great for plugin development or quick tests.

**Q: Why don’t I see my GitHub Actions workflows in Backstage?**  
A: Ensure the GitHub integration is configured in `app-config.yaml` with a valid token (`repo` scope). Verify the repo has a `catalog-info.yaml` file and is registered in Backstage’s catalog. Check the CI/CD tab on the component’s page. If still missing, confirm the workflow runs exist in GitHub and redeploy Backstage.

**Q: Do I need GitHub Advanced Security for CodeQL?**  
A: Yes, for private repositories, CodeQL requires a GitHub Advanced Security license. For public repos, it’s free. The workshop assumes a public repo for simplicity. If using a private repo, ensure your organization has the license enabled.

**Q: How do I scale Backstage for production?**  
A: Increase GKE node count (`--num-nodes`), use a managed database with backups, enable autoscaling (`kubectl autoscale`), and secure with HTTPS and RBAC. Add monitoring (e.g., Google Cloud Monitoring) and use Kubernetes Secrets for sensitive data. This workshop focuses on a demo setup, but these are next steps.

**Q: Can I customize the Backstage UI further?**  
A: Yes! Modify `packages/app/src/components/catalog/EntityPage.tsx` for custom tabs/layouts or create new plugins with `yarn create-plugin`. Use Backstage’s theming (`themes.ts`) for branding. The workshop covers basic customization; explore the Backstage docs for advanced options.

**Facilitator Tip:** Display these FAQs on a slide or handout. Encourage participants to ask questions during Q&A, referencing these answers to save time.

## Troubleshooting Guide

**Issue: GKE cluster creation fails or takes too long.**  
- **Symptoms:** `gcloud container clusters create` errors or hangs.  
- **Fixes:**  
  - Check quotas: In GCP Console, go to `IAM & Admin > Quotas` and ensure enough CPUs/VMs are available in `us-central1-a`.  
  - Verify APIs: Run `gcloud services list` to confirm `container.googleapis.com` is enabled.  
  - Retry with a smaller cluster: `gcloud container clusters create backstage-cluster --zone=us-central1-a --num-nodes=1 --machine-type=e2-medium`.  
  - Check logs: `gcloud logging read "resource.type=gke_cluster"`.  
- **Facilitator Tip:** If someone’s cluster fails, suggest switching to Cloud Run for simplicity (Section 1.2, Alternative).

**Issue: Backstage UI is inaccessible (no external IP or 404).**  
- **Symptoms:** `kubectl get services backstage-service` shows no external IP, or the IP returns a 404.  
- **Fixes:**  
  - Wait 5-10 minutes for the LoadBalancer IP to provision.  
  - Check service status: `kubectl describe service backstage-service`. Ensure `type: LoadBalancer`.  
  - Verify deployment: `kubectl get pods`. If pods are crashing, check logs: `kubectl logs [POD_NAME]`.  
  - Ensure `backstage-deployment.yaml` uses the correct `us-central1-docker.pkg.dev/[PROJECT_ID]/backstage-repo/backstage` image.  
  - For Cloud Run, confirm `--allow-unauthenticated` and check the service URL.  
- **Facilitator Tip:** Show `kubectl get services` on your screen. Help participants debug by sharing pod logs if needed.

**Issue: Database connection fails.**  
- **Symptoms:** Backstage logs show `ECONNREFUSED` or database errors.  
- **Fixes:**  
  - Verify Cloud SQL connection details in `app-config.yaml` (host, port, user, password).  
  - If using Cloud SQL Proxy, ensure it’s running: `cloud_sql_proxy -instances=[PROJECT_ID]:us-central1:backstage-db=tcp:5432`.  
  - Check Cloud SQL instance status: `gcloud sql instances describe backstage-db`.  
  - Test connectivity: `psql -h 127.0.0.1 -U postgres -d backstage`.  
  - Ensure the database and user exist: `gcloud sql databases list --instance=backstage-db`.  
- **Facilitator Tip:** Demonstrate proxy setup if using GKE. Have participants note their Cloud SQL instance name.

**Issue: GitHub integration fails (no repos or workflows visible).**  
- **Symptoms:** CI/CD tab is empty, or catalog doesn’t show repos.  
- **Fixes:**  
  - Verify `app-config.yaml` has the correct GitHub token with `repo` scope.  
  - Check GitHub App/PAT permissions: Include `repo`, `workflow`, `security_events` for CodeQL.  
  - Ensure repos have `catalog-info.yaml` with correct annotations (Section 9).  
  - Test the integration: In Backstage UI, go to `Settings > Integrations > GitHub`.  
  - Redeploy Backstage after config changes: `kubectl rollout restart deployment backstage`.  
- **Facilitator Tip:** Share a test repo with a `catalog-info.yaml` for participants to fork and test. Show the Integrations page in the UI.

**Issue: Plugins don’t appear in the UI.**  
- **Symptoms:** CI/CD, Deployments, or Insights tabs are missing.  
- **Fixes:**  
  - Verify plugin installation: `yarn list | grep @backstage-community` in `packages/app`.  
  - Check `packages/app/src/plugins.ts` for correct imports (e.g., `GithubActionsPlugin`).  
  - Ensure `EntityPage.tsx` includes the plugin’s component (e.g., `<EntityGithubActionsContent />`).  
  - Run `yarn dev` locally to test changes before redeploying.  
  - Clear cache: `yarn clean && yarn install`.  
- **Facilitator Tip:** Walk through `EntityPage.tsx` edits live. If a participant’s plugin fails, check their file changes line-by-line.

**Issue: Webhooks don’t trigger updates.**  
- **Symptoms:** Catalog or widgets don’t update after GitHub events (e.g., commits).  
- **Fixes:**  
  - Verify webhook URL: `http://[BACKSTAGE_EXTERNAL_IP]/api/webhook`.  
  - Check webhook secret in `app-config.yaml` matches GitHub’s webhook settings.  
  - In GitHub, go to Repo > Settings > Webhooks and check “Recent Deliveries” for errors.  
  - Ensure Backstage’s backend is publicly accessible (use `kubectl get services` to confirm IP).  
  - Test with a manual push to a repo with `catalog-info.yaml`.  
- **Facilitator Tip:** Simulate a webhook by pushing a commit to a test repo. Show the webhook logs in GitHub.

**Issue: CodeQL scans don’t show up.**  
- **Symptoms:** No CodeQL results in CI/CD or Insights tab.  
- **Fixes:**  
  - Confirm GitHub Advanced Security is enabled (Repo > Settings > Code security).  
  - Verify `.github/workflows/codeql.yml` exists and runs successfully in GitHub.  
  - Ensure the GitHub token has `security_events` scope.  
  - Check the GitHub Insights plugin if used (`@roadiehq/backstage-plugin-github-insights`).  
- **Facilitator Tip:** Use a public repo for CodeQL to avoid licensing issues. Show the workflow run in GitHub’s Actions tab.

**General Debugging Tips:**  
- Check Backstage logs: `kubectl logs [POD_NAME]` or Cloud Run logs in GCP Console.  
- Use Backstage’s debug mode: Add `logLevel: debug` to `app-config.yaml`.  
- Consult Backstage docs (https://backstage.io/docs) or community Slack for help.  
- If stuck, simplify: Test locally with `yarn dev` or switch to Cloud Run.

**Facilitator Tip:** Create a shared doc or chat channel for participants to report issues during the workshop. Assign a co-facilitator to assist stragglers while you move forward with the group.