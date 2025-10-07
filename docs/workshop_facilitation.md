# Workshop Facilitation

This guide provides additional insights for facilitators running the [Backstage Workshop](../readme.md). It covers common issues, limitations, and practical tips for managing group sessions effectively.

## Additional Insights: Quirks, Gotchas, and Limitations of Backstage

### Quirks and Gotchas
These are common stumbling blocks or unintuitive behaviors in Backstage that participants might encounter during the workshop or ask about.

1. **Configuration Sensitivity in `app-config.yaml`**  
   - **Quirk:** Backstage is highly sensitive to YAML syntax errors in `app-config.yaml`. A missing colon or incorrect indentation can cause the app to fail silently or throw cryptic errors.  
   - **Gotcha:** Environment variables (e.g., `${GITHUB_TOKEN}`) must be set in the deployment environment (e.g., Kubernetes `env` or Cloud Run environment variables). If not, Backstage falls back to defaults or fails.  
   - **Tip:** Always validate `app-config.yaml` with a YAML linter (e.g., `yamllint`). Use `kubectl set env` or Cloud Run’s environment variable settings to inject secrets securely.  
   - **Workshop Prep:** Show participants how to check logs (`kubectl logs [POD_NAME]`) for config errors. Have a sample `app-config.yaml` ready to copy-paste.

2. **Catalog Discovery Can Be Slow or Incomplete**  
   - **Quirk:** The GitHub discovery provider (`catalog.providers.github`) may take time to scan an organization’s repositories, especially if there are many repos or complex `catalog-info.yaml` files.  
   - **Gotcha:** Repos without a valid `catalog-info.yaml` won’t appear in the catalog, and errors in the YAML (e.g., wrong `kind` or missing annotations) can silently skip repos.  
   - **Tip:** Use the Backstage UI’s “Register Component” feature to manually test a single repo’s YAML before enabling org-wide discovery. Monitor the catalog refresh process in logs (`logLevel: debug`).  
   - **Workshop Prep:** Prepare a test repo with a correct `catalog-info.yaml` and demonstrate manual registration. Explain that discovery is asynchronous and may require patience.

3. **Plugin Compatibility and Versions**  
   - **Quirk:** Community plugins (e.g., `@backstage-community/plugin-github-actions`) may not always be compatible with the latest Backstage version due to rapid development cycles.  
   - **Gotcha:** Installing a plugin with `yarn add` might pull an outdated or incompatible version if not specified.  
   - **Tip:** Check the plugin’s GitHub page or `package.json` for Backstage version compatibility. Pin versions (e.g., `yarn add @backstage-community/plugin-github-actions@0.6.5`). Run `yarn install --check-files` after adding plugins to ensure dependencies resolve correctly.  
   - **Workshop Prep:** Pre-test all plugins with your Backstage version (e.g., check `package.json` for `@backstage/core`). Share a list of exact plugin versions used in the workshop.

4. **UI Customization Requires Frontend Knowledge**  
   - **Quirk:** Modifying `EntityPage.tsx` or adding custom tabs requires React and TypeScript knowledge, which can be a barrier for non-frontend developers.  
   - **Gotcha:** Incorrect changes to `EntityPage.tsx` (e.g., wrong imports or syntax) can break the entire UI without clear error messages.  
   - **Tip:** Start with small changes (e.g., adding a single plugin component) and test locally with `yarn dev`. Use Backstage’s scaffolding tools (`yarn create-plugin`) for custom components to avoid boilerplate errors.  
   - **Workshop Prep:** Provide a pre-edited `EntityPage.tsx` snippet (as shown in the guide) and walk through it live. Emphasize testing changes locally before deploying.

5. **GitHub Integration Scope Creep**  
   - **Quirk:** GitHub tokens or Apps need specific scopes (e.g., `repo`, `workflow`, `security_events`), but requesting too many scopes can alarm security-conscious teams.  
   - **Gotcha:** Missing scopes (e.g., forgetting `security_events` for CodeQL) cause plugins to fail silently or show partial data.  
   - **Tip:** Use the least-privilege principle: Start with minimal scopes and add as needed. Test integration in Backstage’s `Settings > Integrations` page.  
   - **Workshop Prep:** Prepare a slide showing required GitHub App permissions. Demonstrate adding a scope to a PAT or App during the session.

6. **Performance on Large Organizations**  
   - **Quirk:** Backstage can slow down when cataloging hundreds of repos or displaying complex plugin data (e.g., thousands of Actions runs).  
   - **Gotcha:** Default GKE or Cloud Run setups may not handle high loads without scaling.  
   - **Tip:** For production, increase GKE nodes or Cloud Run instances, enable caching (`backend.cache` in `app-config.yaml`), and limit catalog discovery to specific repos with filters.  
   - **Workshop Prep:** Mention this as a production consideration, but note the demo setup (3-node GKE) is sufficient for small-scale testing.

### Limitations of Backstage
These are inherent constraints or areas where Backstage may not meet expectations, which participants might ask about.

1. **Limited Built-In Authentication**  
   - **Limitation:** Backstage’s default setup uses guest login, which is insecure for production. Built-in authentication providers (e.g., GitHub OAuth) require additional setup, and custom providers are complex to implement.  
   - **Workaround:** Enable GitHub OAuth (`auth.providers.github`) in `app-config.yaml` for secure logins. For enterprise needs, integrate with Okta or Auth0, but this requires backend plugin development.  
   - **Workshop Answer:** For the demo, guest mode is fine. For production, plan for OAuth setup (see Backstage docs: https://backstage.io/docs/auth).

2. **No Native Support for Non-GitHub CI/CD**  
   - **Limitation:** The GitHub Actions plugin is tailored to GitHub; other CI/CD systems (e.g., Jenkins, CircleCI) require separate plugins, which may be less mature.  
   - **Workaround:** Use community plugins like `@backstage-community/plugin-jenkins` or build custom integrations.  
   - **Workshop Answer:** Focus on GitHub Actions for the workshop. Mention other plugins exist but require similar setup steps.

3. **Heavy Resource Usage**  
   - **Limitation:** Backstage’s backend and frontend can be resource-intensive, especially with many plugins or large catalogs. Small GKE clusters or Cloud Run instances may struggle under load.  
   - **Workaround:** Scale GKE nodes (`kubectl scale`) or increase Cloud Run concurrency. Optimize by disabling unused plugins or limiting catalog scope.  
   - **Workshop Answer:** The workshop’s 3-node GKE setup is sufficient for demo purposes. Discuss scaling in the wrap-up.

4. **Learning Curve for Customization**  
   - **Limitation:** Advanced customization (e.g., custom plugins, themes) requires React, TypeScript, and Node.js expertise, which may intimidate non-developers.  
   - **Workaround:** Use existing community plugins and Backstage’s scaffolding tools to simplify development. Start with small UI tweaks (e.g., adding tabs).  
   - **Workshop Answer:** Show simple UI changes (Section 7) and reassure participants that community plugins cover most use cases.

5. **Webhook Reliability**  
   - **Limitation:** Webhooks depend on Backstage’s public accessibility and GitHub’s delivery reliability. Network issues or misconfigured secrets can cause missed events.  
   - **Workaround:** Use a stable public URL (e.g., via LoadBalancer or Cloud Run) and monitor webhook deliveries in GitHub. Implement retry logic in production.  
   - **Workshop Answer:** Test webhooks with a simple commit (Section 11). For production, consider a reverse proxy or ingress controller.

6. **Plugin Ecosystem Maturity**  
   - **Limitation:** Some community plugins (e.g., GitHub Insights) are less polished or lack features compared to core plugins. Documentation may be sparse.  
   - **Workaround:** Check plugin GitHub issues for known bugs. Contribute fixes or use alternative plugins if needed.  
   - **Workshop Answer:** Stick to well-tested plugins like GitHub Actions for the demo. Point to the plugin directory (https://backstage.io/plugins) for exploration.

### Practical Tips for the Workshop
These tips address potential challenges and help you manage group questions effectively.

1. **Prepare a Test Environment**  
   - Set up a demo GCP project, GitHub org, and Backstage instance in advance. Use a public repo with `catalog-info.yaml`, sample Actions workflows, and CodeQL enabled to demonstrate features without relying on participants’ setups.  
   - **Why?** This ensures you can show working examples even if someone’s setup fails.  
   - **Action:** Share the repo URL (e.g., `github.com/your-org/backstage-demo`) and pre-configured `app-config.yaml` snippets.

2. **Handle Varying Skill Levels**  
   - Some participants may struggle with command-line tools or React. Pair beginners with more experienced attendees for hands-on steps like editing `EntityPage.tsx`.  
   - **Why?** Keeps the group moving and prevents frustration.  
   - **Action:** Prepare a “cheat sheet” with key commands (e.g., `kubectl`, `gcloud`, `yarn`) and share it digitally or as a handout.

3. **Anticipate Common Questions**  
   - Expect queries like “Can Backstage integrate with [tool X]?”, “How do I secure it?”, or “Why is my UI blank?” Use the FAQ and Troubleshooting Guide to answer quickly.  
   - **Why?** Saves time and keeps the workshop on track.  
   - **Action:** Create a slide deck with the FAQ and key diagrams (e.g., Backstage architecture, plugin flow). Reference the Troubleshooting Guide during Q&A.

4. **Time Management**  
   - The workshop covers a lot (GCP setup, GitHub integration, plugins). If time runs short, prioritize Sections 1-3 (installation, GitHub integration, Actions plugin) and summarize Sections 4-11.  
   - **Why?** These cover the core functionality your client wants (test results, history).  
   - **Action:** Plan a 2.5-hour agenda: 45 min for setup, 45 min for integration/plugins, 30 min for demos/customization, 30 min for Q&A.

5. **Debugging Live**  
   - Be ready to debug live if a participant’s setup fails. Common issues include wrong `PROJECT_ID`, missing GitHub scopes, or syntax errors in `EntityPage.tsx`.  
   - **Why?** Builds confidence and shows real-world problem-solving.  
   - **Action:** Keep a terminal open with `kubectl logs` and GCP Console tabs ready. Use the Troubleshooting Guide to walk through fixes.

6. **Highlight Production Considerations**  
   - Participants may ask about scaling, security, or maintenance. Briefly mention RBAC, HTTPS, backups, and monitoring (e.g., Google Cloud Monitoring) without diving deep.  
   - **Why?** Shows Backstage’s enterprise readiness without overwhelming the demo.  
   - **Action:** Add a slide on “Next Steps for Production” (e.g., Secrets, scaling, OAuth).

### Workshop Facilitation Tips
- **Start with a Demo:** Begin with a 5-minute demo of your pre-configured Backstage instance showing the Catalog, CI/CD tab, and a webhook-triggered update. This sets expectations and motivates participants.  
- **Use Visuals:** Show diagrams of Backstage’s architecture (e.g., catalog, plugins, integrations) and the GCP setup (GKE, Cloud SQL). Use a whiteboard or slides for clarity.  
- **Breakout for Hands-On:** After each major section (e.g., GKE setup, plugin installation), give 10-15 minutes for participants to follow along, with you circulating to assist.  
- **Encourage Questions:** Allocate time after each section for quick clarifications, saving deeper questions for the final Q&A.  
- **Backup Plan:** If GCP setup is too slow, have participants run Backstage locally (`yarn dev`) with a pre-configured `app-config.yaml` and GitHub PAT. Provide a ZIP file with the setup ready to go.  
- **Post-Workshop Resources:** Share links to Backstage docs (https://backstage.io/docs), the plugin directory, and the community Slack. Provide your guide as a PDF or GitHub repo for reference.

### Potential Questions and Answers
- **Q: Can Backstage handle non-GitHub repos (e.g., GitLab)?**  
  A: Yes, with plugins like `@backstage-community/plugin-gitlab`. Setup is similar: configure the integration in `app-config.yaml` and add relevant components.  
- **Q: How do I back up the catalog?**  
  A: Export catalog data via the API or back up the PostgreSQL database (e.g., `gcloud sql export`). For production, schedule regular Cloud SQL backups.  
- **Q: What if my team doesn’t use Kubernetes?**  
  A: Use Cloud Run (Section 1.2, Alternative) or a VM with Docker. Cloud Run is simpler but less customizable than GKE.  
- **Q: Can Backstage show metrics or dashboards?**  
  A: Yes, with plugins like `@backstage-community/plugin-grafana` or custom components pulling from APIs. This requires additional setup beyond the workshop scope.