# Component Registration in Backstage

This guide covers how to register components in Backstage using `catalog-info.yaml` files.

## What is Component Registration?

Component registration is the process of making your services, libraries, and resources discoverable in the Backstage catalog. This enables features like:
- Centralized service discovery
- Documentation and metadata display
- Integration with GitHub Actions, deployments, and other plugins
- Ownership and team assignment

## Creating catalog-info.yaml

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

## Registration Methods

### Manual Registration

**Checklist:**
- [ ] Create `catalog-info.yaml` in each repo's root
- [ ] Register manually: In Backstage UI, go to `Create > Register Component` and enter the YAML file's URL (e.g., `https://github.com/[YOUR_ORG]/[REPO_NAME]/blob/main/catalog-info.yaml`)
- [ ] Verify: Check the Catalog in Backstage UI; search/filter by name or type

### Automatic Discovery

**Checklist:**
- [ ] Configure GitHub discovery provider in `app-config.yaml`:
  ```yaml
  catalog:
    providers:
      github:
        my-org:
          organization: '[YOUR_GITHUB_ORG]'
          catalogPath: catalog-info.yaml
  ```
  Replace `[YOUR_GITHUB_ORG]` with your GitHub organization name
- [ ] Restart Backstage deployment to apply changes
- [ ] Wait for discovery to scan all repositories (may take several minutes)

## Component Types and Examples

### Service Component
```yaml
apiVersion: backstage.io/v1alpha1
kind: Component
metadata:
  name: user-service
  description: User management service
  annotations:
    github.com/project-slug: myorg/user-service
    backstage.io/code-coverage: "true"
spec:
  type: service
  lifecycle: production
  owner: backend-team
  system: user-management
```

### Library Component
```yaml
apiVersion: backstage.io/v1alpha1
kind: Component
metadata:
  name: shared-utils
  description: Shared utility library
  annotations:
    github.com/project-slug: myorg/shared-utils
spec:
  type: library
  lifecycle: production
  owner: platform-team
```

### Website Component
```yaml
apiVersion: backstage.io/v1alpha1
kind: Component
metadata:
  name: company-website
  description: Main company website
  annotations:
    github.com/project-slug: myorg/website
spec:
  type: website
  lifecycle: production
  owner: frontend-team
```

## Important Annotations

### GitHub Integration
```yaml
annotations:
  github.com/project-slug: [ORG]/[REPO]  # Links to GitHub repository
```

### Test Coverage
```yaml
annotations:
  backstage.io/code-coverage: "true"  # Enables coverage plugin
```

### Documentation
```yaml
annotations:
  backstage.io/techdocs-ref: dir:.  # Enables TechDocs
```

### Monitoring
```yaml
annotations:
  backstage.io/kubernetes-label-selector: 'app=user-service'  # Kubernetes integration
```

## Systems and Resources

### Defining a System
```yaml
apiVersion: backstage.io/v1alpha1
kind: System
metadata:
  name: user-management
  description: User management system
spec:
  owner: backend-team
```

### Defining an API
```yaml
apiVersion: backstage.io/v1alpha1
kind: API
metadata:
  name: user-api
  description: User management API
spec:
  type: openapi
  lifecycle: production
  owner: backend-team
  system: user-management
  definition: |
    openapi: 3.0.0
    info:
      title: User API
      version: 1.0.0
    # ... rest of OpenAPI spec
```

## Best Practices

### Naming Conventions
- Use lowercase, kebab-case for component names
- Include team or domain prefix for clarity (e.g., `frontend-dashboard`, `backend-user-service`)
- Keep names concise but descriptive

### Ownership
- Always specify an owner (team or individual)
- Use consistent team names across all components
- Consider creating Team entities for better organization

### Lifecycle Management
- Use appropriate lifecycle values: `experimental`, `production`, `deprecated`
- Update lifecycle as components evolve
- Remove deprecated components from catalog when no longer needed

### Documentation
- Include meaningful descriptions for all components
- Use annotations to link to external documentation
- Consider using TechDocs for comprehensive documentation

## Troubleshooting Component Registration

### Component Not Appearing in Catalog
- Verify `catalog-info.yaml` syntax with a YAML validator
- Check Backstage logs for parsing errors: `kubectl logs [POD_NAME]`
- Ensure the GitHub integration has proper access to the repository
- Wait for discovery provider to refresh (can take 10-15 minutes)

### Missing GitHub Data
- Verify `github.com/project-slug` annotation is correct
- Ensure GitHub token has `repo` scope
- Check that the repository exists and is accessible

### Plugin Features Not Working
- Verify required annotations are present (e.g., `backstage.io/code-coverage`)
- Ensure corresponding plugins are installed and configured
- Check plugin-specific requirements in their documentation

**Facilitator Tip:** Have participants add `catalog-info.yaml` to a test repo. Show the Catalog UI and how components appear. For automated component creation, consider exploring [Scaffolder Templates](./backstage_scaffolder_templates.md) which can generate projects with `catalog-info.yaml` included. For troubleshooting registration issues, see the [FAQ & Troubleshooting Guide](./faq_troubleshooting.md).

## Integration with Other Features

Once components are registered, they can take advantage of:
- **CI/CD Visibility:** View GitHub Actions workflows in the CI/CD tab
- **Security Insights:** Display CodeQL scans and vulnerability reports
- **Test Coverage:** Show coverage metrics and trends
- **Deployments:** Track deployment status and history
- **Custom Views:** Create dashboard views filtered by teams or systems

For more advanced integration options, see:
- [GitHub Integration Guide](./github_integration.md) for CI/CD and security features
- [Project Resilience Guide](./project_resilience_guide.md) for metrics and monitoring
- [Scaffolder Templates](./backstage_scaffolder_templates.md) for automated component creation