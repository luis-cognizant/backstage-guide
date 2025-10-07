# GitHub Permissions Guide for Backstage Integration

This guide provides a comprehensive reference for all GitHub permissions required to integrate Backstage with your GitHub repositories, organizations, and workflows.

## Overview

Backstage can connect to GitHub using two methods:
1. **GitHub App** (Recommended for organizations)
2. **Personal Access Token** (PAT) (Suitable for individual use or testing)

Each method requires specific permissions to access different GitHub features.

---

## GitHub App Permissions (Recommended)

GitHub Apps provide fine-grained, scoped access to organization resources with better security and audit trails.

### Repository Permissions

These permissions apply to repositories where the app is installed:

| Permission | Access Level | Required For | Description |
|------------|-------------|--------------|-------------|
| **Contents** | `Read-only` | ✅ Required | Read repository files, commits, branches, and tags |
| **Metadata** | `Read-only` | ✅ Required | Read basic repository information (always granted) |
| **Workflows** | `Read-only` | ✅ Required | View GitHub Actions workflows and run history |
| **Security events** | `Read-only` | ✅ Required | Access CodeQL scans, Dependabot alerts, secret scanning |
| **Administration** | `Read-only` | Optional | View repository settings and team access |
| **Issues** | `Read-only` | Optional | View issues for documentation or tracking |
| **Pull requests** | `Read-only` | Optional | View pull requests and reviews |
| **Commit statuses** | `Read-only` | Optional | View commit status checks |
| **Deployments** | `Read-only` | Optional | View deployment history and environments |
| **Checks** | `Read-only` | Optional | View check runs and check suites |

### Organization Permissions

These permissions apply at the organization level:

| Permission | Access Level | Required For | Description |
|------------|-------------|--------------|-------------|
| **Members** | `Read-only` | Optional | Read organization membership to display team info |
| **Administration** | `Read-only` | Optional | View organization settings |

### Account Permissions

| Permission | Access Level | Required For | Description |
|------------|-------------|--------------|-------------|
| **Email addresses** | `Read-only` | Optional | Access user email addresses for contact info |

### Webhook Events (Subscribe to events)

Select these events if you want real-time updates:

- [x] **Workflow run** - Receive notifications when GitHub Actions complete
- [x] **Code scanning alert** - Get notified of new CodeQL findings
- [x] **Push** - Receive push events for catalog updates
- [x] **Release** - Track new releases
- [x] **Repository** - Monitor repository changes

---

## Personal Access Token (PAT) Permissions

If using a Personal Access Token instead of a GitHub App, select these scopes:

### Classic Token Scopes

| Scope | Required For | Description |
|-------|--------------|-------------|
| **repo** | ✅ Required | Full control of private repositories (includes all sub-scopes below) |
| ↳ `repo:status` | Required | Access commit status |
| ↳ `repo_deployment` | Required | Access deployment status |
| ↳ `public_repo` | Required | Access public repositories |
| ↳ `security_events` | Required | Read security events (CodeQL, Dependabot) |
| **workflow** | ✅ Required | Update GitHub Actions workflows |
| **read:org** | Optional | Read organization membership and teams |
| **read:user** | Optional | Read user profile information |
| **user:email** | Optional | Access user email addresses |

### Fine-Grained Token Permissions

For fine-grained personal access tokens (beta), configure:

**Repository access:**
- Select: "All repositories" or "Only select repositories"

**Repository permissions:**
- Actions: `Read-only`
- Contents: `Read-only`
- Metadata: `Read-only` (automatically included)
- Workflows: `Read-only`
- Security events: `Read-only`
- Issues: `Read-only` (optional)
- Pull requests: `Read-only` (optional)

**Organization permissions:**
- Members: `Read-only` (optional)

---

## Permission Requirements by Feature

### Core Backstage Features

| Feature | Required Permissions | Notes |
|---------|---------------------|-------|
| **View Repositories** | Contents, Metadata | Basic catalog functionality |
| **View Commits** | Contents | Show commit history |
| **View Branches** | Contents | Display branch information |
| **View Tags** | Contents | Show release tags |

### CI/CD Integration

| Feature | Required Permissions | Notes |
|---------|---------------------|-------|
| **View GitHub Actions** | Workflows | Display workflow runs |
| **View Run History** | Workflows | Show past executions |
| **View Run Logs** | Workflows | Access execution logs |
| **View Job Status** | Workflows, Checks | Show individual job results |
| **View Artifacts** | Workflows | Access build artifacts |

### Security & Quality

| Feature | Required Permissions | Notes |
|---------|---------------------|-------|
| **View CodeQL Scans** | Security events | Display code scanning alerts |
| **View Dependabot Alerts** | Security events | Show dependency vulnerabilities |
| **View Secret Scanning** | Security events | Display leaked secrets |
| **View Test Coverage** | Workflows, Contents | Requires coverage reports in Actions |

### Advanced Features

| Feature | Required Permissions | Notes |
|---------|---------------------|-------|
| **View Deployments** | Deployments | Show deployment history |
| **View Environments** | Deployments | Display environment status |
| **View Issues** | Issues | Show issue tracking |
| **View Pull Requests** | Pull requests | Display PR information |
| **View Team Info** | Members (org) | Show repository owners |

---

## Setting Up GitHub App (Step-by-Step)

### 1. Create GitHub App

```bash
# Navigate to GitHub Settings
https://github.com/settings/apps
# Or for organizations:
https://github.com/organizations/[YOUR-ORG]/settings/apps
```

### 2. Configure Basic Information

- **GitHub App name:** `backstage-[your-org]` (must be unique across GitHub)
- **Homepage URL:** `https://your-backstage-instance.com`
- **Webhook URL:** `https://your-backstage-instance.com/api/github/webhook` (optional)
- **Webhook secret:** Generate a secure random string

### 3. Set Repository Permissions

Apply the permissions from the table above:

```yaml
Repository Permissions:
  Contents: Read-only
  Metadata: Read-only (automatic)
  Workflows: Read-only
  Security events: Read-only
  
# Optional but recommended:
  Administration: Read-only
  Issues: Read-only
  Pull requests: Read-only
  Deployments: Read-only
  Checks: Read-only
```

### 4. Set Organization Permissions (Optional)

```yaml
Organization Permissions:
  Members: Read-only
  Administration: Read-only
```

### 5. Subscribe to Events (Optional)

If using webhooks for real-time updates:

- [x] Workflow run
- [x] Code scanning alert
- [x] Push
- [x] Release

### 6. Generate Credentials

1. **Generate Private Key**
   - Scroll to "Private keys" section
   - Click "Generate a private key"
   - Download the `.pem` file
   - Store securely (you'll need this for Backstage config)

2. **Note App Credentials**
   - **App ID:** Found at top of settings page
   - **Client ID:** Found in "About" section
   - **Client Secret:** Generate in "Client secrets" section

### 7. Install App

1. Click "Install App" in left sidebar
2. Select your organization
3. Choose repositories:
   - **All repositories:** Grant access to all current and future repos
   - **Only select repositories:** Choose specific repos
4. Click "Install"
5. **Note Installation ID:** Found in the URL after installation
   - URL format: `https://github.com/settings/installations/[INSTALLATION_ID]`

---

## Setting Up Personal Access Token (Step-by-Step)

### Classic Token

1. Navigate to: `https://github.com/settings/tokens`
2. Click "Generate new token" → "Generate new token (classic)"
3. Give your token a descriptive name: `backstage-integration`
4. Set expiration: Choose based on your security policy
5. Select scopes:
   - [x] **repo** (all sub-scopes)
   - [x] **workflow**
   - [x] **read:org** (optional)
   - [x] **read:user** (optional)
6. Click "Generate token"
7. **Copy token immediately** (you won't see it again!)

### Fine-Grained Token

1. Navigate to: `https://github.com/settings/tokens?type=beta`
2. Click "Generate new token"
3. Configure token:
   - **Name:** `backstage-integration`
   - **Expiration:** Choose based on policy
   - **Repository access:** Select "All repositories" or specific repos
4. Set permissions (see table above)
5. Click "Generate token"
6. **Copy token immediately**

---

## Configuring Backstage

### For GitHub App

Add to `app-config.yaml`:

```yaml
integrations:
  github:
    - host: github.com
      apps:
        - appId: ${GITHUB_APP_ID}
          privateKey: ${GITHUB_PRIVATE_KEY}
          webhookSecret: ${GITHUB_WEBHOOK_SECRET}
          clientId: ${GITHUB_CLIENT_ID}
          clientSecret: ${GITHUB_CLIENT_SECRET}
```

Set environment variables:

```bash
export GITHUB_APP_ID="123456"
export GITHUB_CLIENT_ID="Iv1.a1b2c3d4e5f6g7h8"
export GITHUB_CLIENT_SECRET="abc123def456..."
export GITHUB_WEBHOOK_SECRET="your-webhook-secret"
export GITHUB_PRIVATE_KEY="$(cat path/to/your-app.private-key.pem)"
# Or base64 encode for some environments:
export GITHUB_PRIVATE_KEY="$(cat path/to/your-app.private-key.pem | base64)"
```

### For Personal Access Token

Add to `app-config.yaml`:

```yaml
integrations:
  github:
    - host: github.com
      token: ${GITHUB_TOKEN}
```

Set environment variable:

```bash
export GITHUB_TOKEN="ghp_your_personal_access_token_here"
```

---

## Verifying Permissions

### Check GitHub App Permissions

1. Navigate to: `https://github.com/settings/installations`
2. Click on your Backstage app installation
3. Review "Repository access" and "Permissions"
4. Verify all required permissions are granted

### Check Token Scopes

Using GitHub API:

```bash
# Check token scopes
curl -H "Authorization: token YOUR_TOKEN" https://api.github.com/user

# Verify workflow access
curl -H "Authorization: token YOUR_TOKEN" \
  https://api.github.com/repos/OWNER/REPO/actions/runs

# Verify security events access
curl -H "Authorization: token YOUR_TOKEN" \
  https://api.github.com/repos/OWNER/REPO/code-scanning/alerts
```

### Test in Backstage

Check Backstage backend logs:

```bash
# Look for successful GitHub API calls
kubectl logs -f deployment/backstage-backend | grep github

# Or for local development:
yarn start-backend | grep github
```

---

## Permission Troubleshooting

### Common Issues

#### "Resource not accessible by integration"

**Cause:** Missing required permission
**Solution:** Add the missing permission to your GitHub App or token

```bash
# Check which permission is needed from the API response
# Example: "Resource not accessible by integration" when accessing workflows
# → Add "Workflows: Read-only" permission
```

#### "Not Found" (404) Error

**Cause:** 
- App not installed on repository
- Token doesn't have access to repository
- Repository doesn't exist

**Solution:**
1. Verify repository exists and name is correct
2. For GitHub Apps: Install app on the repository
3. For tokens: Ensure token has access to the repository

#### "Bad credentials" (401) Error

**Cause:**
- Invalid or expired token
- Incorrect app credentials
- Private key mismatch

**Solution:**
1. Verify token hasn't expired
2. Check environment variables are set correctly
3. For apps: Ensure private key matches the app

#### CodeQL Scans Not Visible

**Cause:** Missing "Security events" permission

**Solution:**
1. Add "Security events: Read-only" to GitHub App
2. Or add `security_events` scope to token
3. Ensure CodeQL is actually running on the repository

#### Workflow Runs Not Showing

**Cause:** Missing "Workflows" permission

**Solution:**
1. Add "Workflows: Read-only" to GitHub App
2. Or ensure `workflow` scope is included in token
3. Verify GitHub Actions are enabled on repository

---

## Security Best Practices

### For GitHub Apps

✅ **Do:**
- Use GitHub Apps instead of PATs for organization use
- Limit app installation to required repositories only
- Regularly review app permissions and access
- Rotate webhook secrets periodically
- Store private keys securely (use secret managers)
- Monitor app usage through GitHub's audit log

❌ **Don't:**
- Share private keys in version control
- Grant more permissions than needed
- Install apps on personal repositories for testing production apps
- Use the same app across multiple Backstage instances

### For Personal Access Tokens

✅ **Do:**
- Set reasonable expiration dates (30-90 days)
- Use fine-grained tokens when possible
- Limit token scope to minimum required
- Store tokens in secret managers
- Rotate tokens regularly
- Use different tokens for different purposes

❌ **Don't:**
- Commit tokens to version control
- Share tokens between team members
- Use tokens with no expiration
- Grant full `repo` scope if you only need read access

### Token Storage

**Recommended storage methods:**

1. **Kubernetes Secrets:**
   ```yaml
   apiVersion: v1
   kind: Secret
   metadata:
     name: backstage-github-credentials
   type: Opaque
   stringData:
     GITHUB_TOKEN: "ghp_your_token"
     GITHUB_APP_ID: "123456"
     GITHUB_PRIVATE_KEY: |
       -----BEGIN RSA PRIVATE KEY-----
       ...
       -----END RSA PRIVATE KEY-----
   ```

2. **Google Secret Manager:**
   ```yaml
   integrations:
     github:
       - host: github.com
         token: ${GITHUB_TOKEN_FROM_SECRET_MANAGER}
   ```

3. **Environment Variables (Development Only):**
   ```bash
   # .env file (add to .gitignore!)
   GITHUB_TOKEN=ghp_your_token
   ```

---

## Permission Audit Checklist

Use this checklist to verify your GitHub integration has proper permissions:

### Basic Integration
- [ ] Can view public repositories
- [ ] Can view private repositories (if applicable)
- [ ] Can view repository metadata (description, topics, etc.)
- [ ] Can view commit history
- [ ] Can view branches and tags

### CI/CD Features
- [ ] Can view GitHub Actions workflows
- [ ] Can view workflow run history
- [ ] Can view workflow run logs
- [ ] Can view job statuses
- [ ] Can view artifacts (if needed)

### Security Features
- [ ] Can view CodeQL scanning results
- [ ] Can view Dependabot alerts
- [ ] Can view secret scanning alerts
- [ ] Can view security advisories

### Optional Features
- [ ] Can view deployment history
- [ ] Can view issues
- [ ] Can view pull requests
- [ ] Can view organization members
- [ ] Can view team assignments

### Webhooks (If Configured)
- [ ] Webhook endpoint is accessible
- [ ] Webhook secret is configured
- [ ] Subscribed to relevant events
- [ ] Webhooks are delivering successfully

---

## Quick Reference

### Minimum Required Permissions for Basic Backstage

**GitHub App:**
- Contents: Read-only
- Metadata: Read-only
- Workflows: Read-only
- Security events: Read-only

**Personal Access Token:**
- `repo`
- `workflow`

### Recommended Permissions for Full Features

**GitHub App:**
- All minimum permissions, plus:
- Administration: Read-only
- Issues: Read-only
- Pull requests: Read-only
- Deployments: Read-only
- Checks: Read-only
- Members (org): Read-only

**Personal Access Token:**
- `repo`
- `workflow`
- `read:org`

---

## Additional Resources

- [GitHub Apps Documentation](https://docs.github.com/en/developers/apps)
- [Personal Access Tokens](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token)
- [GitHub API Scopes](https://docs.github.com/en/developers/apps/building-oauth-apps/scopes-for-oauth-apps)
- [Backstage GitHub Integration](https://backstage.io/docs/integrations/github/locations)
- [GitHub API Documentation](https://docs.github.com/en/rest)

---

## Related Workshop Documents

- [Workshop Walkthrough](../workshop/workshop_walkthrough.md) - Complete workshop guide with diagrams
- [Workshop Checklist](../workshop/workshop_checklist.md) - Detailed step-by-step checklist
- [GitHub Integration Guide](./github_integration.md) - Full GitHub setup instructions
- [Command Cheatsheet](./command_cheatsheet.md) - Quick command reference
- [FAQ & Troubleshooting](./faq_troubleshooting.md) - Common issues and solutions
