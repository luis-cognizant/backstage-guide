# Backstage Workshop: Installation on Google Cloud, GitHub Integration, and Advanced Features

## Introduction

Welcome to this hands-on workshop! This guide will walk you through installing Backstage (Spotify's open-source developer portal) on Google Cloud Platform (GCP) from scratch, connecting it to GitHub, and enabling features like viewing GitHub Actions test results, historical run data, CodeQL scans, and deployments.

This workshop is designed for groups with clear, step-by-step instructions and checklists to ensure everyone can follow along. The workshop will take 2-4 hours, so adjust pacing based on your group's progress.

**Workshop Goals:**
- Deploy Backstage on GCP using Google Kubernetes Engine (GKE) and Cloud SQL
- Integrate with GitHub to display CI/CD workflows, CodeQL scans, and deployments
- Explore community plugins and customize views
- Understand component registration and secure GitHub integration with webhooks

## Workshop Structure

This workshop is organized into focused modules for easier navigation and maintenance:

### Core Setup
1. **[Installation Guide](./docs/installation_guide.md)** - Deploy Backstage on GCP with GKE and Cloud SQL
2. **[GitHub Integration](./docs/github_integration.md)** - Connect to GitHub and install plugins
3. **[Component Registration](./docs/component_registration.md)** - Register components using catalog-info.yaml

### Advanced Features
4. **[Project Resilience Guide](./docs/project_resilience_guide.md)** - Test coverage and security metrics
5. **[Scaffolder Templates](./docs/backstage_scaffolder_templates.md)** - Create standardized project templates
6. **[Storing GitHub Data for LLM Insights](./docs/storing_github_data_for_llm_insights.md)** - Advanced data analysis

### Optional: AI/LLM Integration
7. **[LLM Integrations with LangChain](./docs/llm_integrations.md)** - AI-powered analysis and automated actions (Optional advanced topic)

### Workshop Support
- **[Visual Diagrams](./docs/diagrams.md)** - Architecture and workflow diagrams
- **[Workshop Facilitation Guide](./docs/workshop_facilitation.md)** - Tips for workshop leaders
- **[FAQ & Troubleshooting](./docs/faq_troubleshooting.md)** - Common issues and solutions

## Prerequisites

Before starting, ensure all participants have:
- A GCP account with billing enabled (free tier may suffice for small setups)
- Access to Google Kubernetes Engine (GKE), Cloud Run, Cloud SQL, and Artifact Registry
- A GitHub account with admin access to an organization or repository
- Local tools installed: Node.js (v18+), Yarn, Docker, `gcloud` CLI, and `kubectl`
- A code editor (e.g., VS Code) for editing files
- Basic knowledge of Git, YAML, and command-line operations

**Pre-Workshop Setup Checklist:**
- [ ] Create a GCP project via the GCP Console (`IAM & Admin > Create a Project`)
  - Note the `PROJECT_ID` (e.g., `my-backstage-project-123`) displayed in the Console or via `gcloud projects list`
- [ ] Enable GCP APIs: Compute Engine, GKE, Cloud SQL, Artifact Registry
  - Run: `gcloud services enable compute.googleapis.com container.googleapis.com sqladmin.googleapis.com artifactregistry.googleapis.com`
- [ ] Install `gcloud` CLI (https://cloud.google.com/sdk/docs/install)
- [ ] Install `kubectl`: `gcloud components install kubectl`
- [ ] Install Node.js (v18+), Yarn (`npm install -g yarn`), and Docker
- [ ] Set up a GitHub personal access token (PAT) with `repo` scope or a GitHub App
- [ ] Ensure all participants have their `PROJECT_ID` and GitHub credentials ready

**Facilitator Tip:** Before the workshop, have participants verify their `PROJECT_ID` and tool installations. Share a pre-workshop email with setup instructions and links to install tools. During the session, display your screen for commands and file edits to guide the group. For additional facilitation guidance, see the [Workshop Facilitation Guide](./docs/workshop_facilitation.md).

## Quick Start

If you want to jump straight into the hands-on work:

1. **Start Here:** [Installation Guide](./docs/installation_guide.md) - Set up Backstage on GCP
2. **Then:** [GitHub Integration](./docs/github_integration.md) - Connect to GitHub and enable plugins
3. **Next:** [Component Registration](./docs/component_registration.md) - Add your first component

For visual learners, check out the [diagrams](./docs/diagrams.md) to understand the architecture and workflow.

## Getting Help

If you encounter issues during the workshop:
- Check the [FAQ & Troubleshooting](./docs/faq_troubleshooting.md) for common solutions
- Refer to the [Workshop Facilitation Guide](./docs/workshop_facilitation.md) for tips on debugging
- Use `kubectl logs` or Cloud Run logs to diagnose deployment issues
- Consult Backstage docs (https://backstage.io/docs) for detailed documentation

## Next Steps After the Workshop

Once you complete the core workshop:
- Explore the [Project Resilience Guide](./docs/project_resilience_guide.md) to add test coverage and security metrics
- Try creating templates with the [Scaffolder Templates](./docs/backstage_scaffolder_templates.md) guide
- Set up advanced analytics with [Storing GitHub Data for LLM Insights](./docs/storing_github_data_for_llm_insights.md)
- **AI Integation:** Explore [LLM Integrations with LangChain](./docs/llm_integrations.md) for AI-powered analysis and automated actions
- Scale your deployment for production use
- Explore more plugins in the Backstage community directory

**Workshop Time Estimates:**
- Core Setup (Sections 1-3): 90-120 minutes
- Advanced Features (Sections 4-6): 60-90 minutes
- Q&A and Troubleshooting: 30 minutes
- **Total:** 2.5-4 hours depending on group size and experience level

## Wrap-Up and Key Takeaways

**Key Takeaways:**
- Backstage is deployed on GCP with GKE or Cloud Run
- GitHub integration enables CI/CD visibility (Actions, CodeQL, deployments)
- Components are registered via `catalog-info.yaml`
- Plugins and custom views extend functionality
- Secure with GitHub Apps and webhooks for real-time updates

**Next Steps:**
- Explore more plugins in the Backstage directory
- Set up monitoring, scaling, and backups on GCP for production
- Check Backstage docs (https://backstage.io/docs) and community forums for support

**Facilitator Tip:** Allocate 15-20 minutes for Q&A. Encourage participants to experiment with plugins or custom views post-workshop. If issues arise, use `kubectl logs`, Cloud Run logs, or Backstage's error messages to debug. Share a link to this guide for reference.