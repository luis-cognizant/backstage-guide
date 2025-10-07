# Workshop Walkthrough: Backstage Integration

## Workshop Objective

**Primary Goal:** Demonstrate that Backstage can serve as a "single pane of glass" for test visibility by integrating with GitHub repositories and CodeQL scans.

**Success Criteria:** By end of session, you'll see:
- ✅ Your GitHub repositories listed in Backstage
- ✅ Test results from GitHub Actions visible in one view
- ✅ CodeQL security scan results displayed alongside test data
- ✅ Historical data stored in database for analysis

## Architecture Overview

```mermaid
graph TB
    subgraph "Core Integration"
        A[GitHub Repositories] -->|GitHub App| B[Backstage Instance]
        B --> C[Test Coverage View]
        B --> D[CodeQL Security View]
        B --> E[GitHub Actions History]
        B -->|Store Data| F[Cloud SQL Database]
        
        style A fill:#e1f5ff
        style B fill:#fff4e1
        style C fill:#e8f5e9
        style D fill:#e8f5e9
        style E fill:#e8f5e9
        style F fill:#e8f5e9
    end
    
    subgraph "Future Capabilities"
        F -.->|Analyze Trends| G[Analytics Engine]
        G -.->|Insights| H[Automated Reports]
        G -.->|Recommendations| I[Quality Improvements]
        
        style G fill:#f3e5f5
        style H fill:#f3e5f5
        style I fill:#f3e5f5
    end
    
    classDef coreClass fill:#e8f5e9,stroke:#4caf50,stroke-width:3px
    classDef futureClass fill:#f3e5f5,stroke:#9c27b0,stroke-width:2px,stroke-dasharray: 5 5
    
    class C,D,E,F coreClass
    class G,H,I futureClass
```

## Workshop Timeline

**Total Duration:** 3 hours (approximately 2:00 PM - 5:00 PM)

```mermaid
gantt
    title Workshop Session Timeline
    dateFormat YYYY-MM-DD HH:mm
    axisFormat %H:%M
    
    section Introduction (15 min)
    Workshop Goals & Setup          :intro, 2024-01-01 00:00, 15m
    
    section Phase 1: Quick Wins (85 min)
    Verify Backstage Access         :access, 2024-01-01 00:15, 15m
    Configure GitHub Integration    :github, 2024-01-01 00:30, 30m
    Register First Component        :comp, 2024-01-01 01:00, 20m
    View Test Results               :tests, 2024-01-01 01:20, 20m
    
    section Break (10 min)
    Coffee Break                    :crit, break1, 2024-01-01 01:40, 10m
    
    section Phase 2: Test Visibility (70 min)
    Install Test Coverage Plugin    :plugin, 2024-01-01 01:50, 25m
    Configure CodeQL Integration    :codeql, 2024-01-01 02:15, 25m
    Verify Data Flow                :verify, 2024-01-01 02:40, 20m
    
    section Phase 3: Data Foundation (45 min)
    Database Setup Review           :db, 2024-01-01 03:00, 20m
    Configure Data Collection       :collect, 2024-01-01 03:20, 25m
    
    section Wrap-Up (45 min)
    Review & Demonstration          :demo, 2024-01-01 02:45, 20m
    Q&A and Next Steps              :qa, 2024-01-01 03:05, 25m
```

### Time Breakdown by Phase

| Phase | Start Time | Duration | Activities |
|-------|-----------|----------|------------|
| **Introduction** | Start | 15 min | Workshop goals, environment validation |
| **Phase 1: Quick Wins** | +15 min | 85 min | GitHub integration, component registration |
| **Break** | +1h 40min | 10 min | Coffee break, Q&A |
| **Phase 2: Test Visibility** | +1h 50min | 70 min | Coverage plugin, CodeQL integration |
| **Phase 3: Data Foundation** | +3h 00min | 45 min | Database setup, data collection |
| **Wrap-Up** | +2h 45min | 45 min | Review, demonstration, Q&A |

**Note:** Timeline is flexible. Adjust pacing based on group progress and questions.

## Phase 1: GitHub Integration Setup

### Integration Flow

```mermaid
flowchart LR
    A[Start: Existing Backstage] --> B{GitHub Connected?}
    B -->|No| C[Configure GitHub App]
    B -->|Yes| D[Verify Token Scopes]
    C --> D
    D --> E[Register Test Repo]
    E --> F[View in Catalog]
    F --> G[✅ See GitHub Actions Tab]
    
    style A fill:#e3f2fd
    style G fill:#c8e6c9
```

### Required GitHub Permissions

```mermaid
graph TD
    A[GitHub App Configuration] --> B[Repository Permissions]
    B --> C[Contents: Read-only]
    B --> D[Metadata: Read-only]
    B --> E[Workflows: Read-only]
    B --> F[Security Events: Read-only]
    
    style A fill:#fff4e1
    style C fill:#e8f5e9
    style D fill:#e8f5e9
    style E fill:#e8f5e9
    style F fill:#e8f5e9
```

### Steps

1. **Create GitHub App**
   - Navigate to GitHub Settings > Developer Settings > GitHub Apps
   - Configure required permissions (see diagram above)
   - Generate private key
   - Install on target organization

2. **Configure Backstage**
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

3. **Register Component**
   - Create `catalog-info.yaml` in repository
   - Import component in Backstage
   - Verify visibility

**Deliverables:**
- GitHub integration configured
- At least one repository visible in Backstage
- GitHub Actions history displayed

---

## Phase 2: Test Visibility Configuration

### Test Coverage Integration

```mermaid
flowchart TD
    A[GitHub Actions Workflow] --> B[Generate Coverage Report]
    B --> C[Upload to GitHub]
    C --> D[Backstage Coverage Plugin]
    D --> E[Display in Component View]
    
    style E fill:#c8e6c9
```

### CodeQL Security Integration

```mermaid
flowchart TD
    A[CodeQL Workflow] --> B[Security Scan]
    B --> C[Upload Results to GitHub]
    C --> D[Backstage Security Plugin]
    D --> E[Display Findings]
    
    F[Severity Levels] --> G[Critical]
    F --> H[High]
    F --> I[Medium]
    F --> J[Low]
    
    E --> F
    
    style E fill:#c8e6c9
```

### Complete View Architecture

```mermaid
flowchart LR
    A[Component Page] --> B[Overview Tab]
    A --> C[CI/CD Tab]
    A --> D[Security Tab]
    
    B --> E[Repository Info]
    B --> F[Test Coverage %]
    B --> G[Recent Activity]
    
    C --> H[Workflow Runs]
    C --> I[Run Status]
    C --> J[Run Duration]
    
    D --> K[CodeQL Findings]
    D --> L[Severity Breakdown]
    D --> M[Affected Files]
    
    style A fill:#fff4e1
    style B fill:#e8f5e9
    style C fill:#e8f5e9
    style D fill:#e8f5e9
```

**Deliverables:**
- Test coverage percentages visible per component
- CodeQL security findings displayed
- Historical test run data accessible
- Single unified view of all test and security information

---

## Phase 3: Data Foundation

### Data Storage Architecture

```mermaid
flowchart LR
    A[GitHub Actions] -->|Webhook| B[Backstage Backend]
    B -->|Store| C[Cloud SQL Database]
    C -->|Schema| D[Test Results Table]
    C -->|Schema| E[Coverage Table]
    C -->|Schema| F[Security Scans Table]
    
    D -->|Enable| G[Historical Analysis]
    E -->|Enable| G
    F -->|Enable| G
    
    style C fill:#fff9c4
    style G fill:#e8f5e9
```

### Database Schema

```mermaid
erDiagram
    TEST_RESULTS {
        int id PK
        string repo_name
        string commit_sha
        timestamp test_date
        int total_tests
        int passed_tests
        int failed_tests
        decimal coverage_percentage
        bigint workflow_run_id
    }
    
    SECURITY_SCANS {
        int id PK
        string repo_name
        timestamp scan_date
        string severity
        string finding_type
        text description
        string file_path
        int line_number
    }
    
    COVERAGE_HISTORY {
        int id PK
        string repo_name
        timestamp measurement_date
        decimal line_coverage
        decimal branch_coverage
        int total_lines
        int covered_lines
    }
```

### Data Collection Flow

```mermaid
sequenceDiagram
    participant GHA as GitHub Actions
    participant GH as GitHub
    participant BS as Backstage
    participant DB as Cloud SQL
    
    GHA->>GHA: Run Tests
    GHA->>GHA: Generate Coverage
    GHA->>GH: Upload Results
    GH->>BS: Webhook Notification
    BS->>GH: Fetch Results
    BS->>DB: Store Historical Data
    BS->>BS: Update UI
    
    Note over DB: Data available for<br/>trend analysis
```

**Deliverables:**
- Database schema configured
- Data collection pipeline established
- Historical data queryable
- Foundation for advanced analytics

---

## Success Metrics

### Verification Checklist

```mermaid
graph LR
    A[Open Backstage] --> B[Navigate to Catalog]
    B --> C[Click Your Repository]
    C --> D[See GitHub Actions Tab]
    D --> E[View Test Results]
    
    C --> F[See Overview Tab]
    F --> G[View Test Coverage %]
    
    C --> H[See Security Tab]
    H --> I[View CodeQL Findings]
    
    E --> J[✅ Success: Single Pane of Glass]
    G --> J
    I --> J
    
    style J fill:#4caf50,color:#fff
```

### What You Should See

1. **Backstage Catalog**
   - All registered repositories listed
   - Component metadata displayed
   - Owner and lifecycle information

2. **Component View - Overview Tab**
   - Repository description and links
   - Test coverage percentage
   - Recent commit history
   - Team ownership information

3. **Component View - CI/CD Tab**
   - GitHub Actions workflow runs
   - Run statuses (success/failure)
   - Run durations
   - Triggered by information

4. **Component View - Security Tab**
   - CodeQL security findings
   - Severity classifications
   - Affected file paths
   - Remediation guidance

5. **Database**
   - Historical test results stored
   - Coverage trends over time
   - Security scan history

---

## Troubleshooting Decision Tree

```mermaid
flowchart TD
    A[Issue Encountered] --> B{Where is the problem?}
    
    B -->|GitHub Connection| C[Check Token Scopes]
    C --> D[Regenerate Token with Correct Permissions]
    
    B -->|Plugin Not Showing| E[Check app-config.yaml]
    E --> F[Restart Backstage Backend]
    
    B -->|Database Connection| G[Verify Cloud SQL Proxy]
    G --> H[Check Connection String]
    
    B -->|No Test Data| I[Verify GitHub Actions Workflow]
    I --> J[Check Webhook Configuration]
    
    D --> K[Re-test Connection]
    F --> K
    H --> K
    J --> K
    
    style K fill:#c8e6c9
```

### Common Issues and Solutions

| Issue | Likely Cause | Solution |
|-------|-------------|----------|
| GitHub repos not showing | Incorrect GitHub App permissions | Verify app has Contents and Metadata read access |
| No test data visible | Coverage report not uploaded | Check GitHub Actions workflow uploads coverage artifacts |
| CodeQL findings missing | CodeQL not enabled on repo | Enable CodeQL in repository security settings |
| Database connection fails | Cloud SQL proxy not configured | Verify connection string and proxy setup |
| Plugin not appearing | Configuration not loaded | Restart Backstage backend after config changes |

---

## Permissions & Access Requirements

### GitHub Requirements

```mermaid
graph TB
    A[GitHub Access] --> B[Organization Admin]
    B --> C[Can Create GitHub Apps]
    B --> D[Can Install Apps on Org]
    
    A --> E[Repository Access]
    E --> F[Read Repository Contents]
    E --> G[Read Workflows]
    E --> H[Read Security Events]
    
    style A fill:#fff4e1
    style B fill:#e8f5e9
    style E fill:#e8f5e9
```

### Google Cloud Requirements

```mermaid
graph TB
    A[GCP Access] --> B[Project Permissions]
    B --> C[Cloud SQL Admin]
    B --> D[GKE Cluster Access]
    B --> E[Secret Manager Access]
    
    A --> F[Resource Access]
    F --> G[Database Read/Write]
    F --> H[Pod Logs Access]
    
    style A fill:#fff4e1
    style B fill:#e8f5e9
    style F fill:#e8f5e9
```

### Validation Checklist

- [ ] GitHub organization admin access
- [ ] At least one repository with existing tests
- [ ] GitHub Actions workflows configured
- [ ] CodeQL enabled on target repositories
- [ ] GCP project access
- [ ] Cloud SQL instance accessible
- [ ] Backstage instance running

---

## Implementation Timeline

```mermaid
timeline
    title Typical Implementation Phases
    
    section Phase 1: Foundation
        Week 1-2 : Configure GitHub integration
                 : Register initial repositories
                 : Install core plugins
    
    section Phase 2: Expansion
        Week 3-4 : Roll out to additional repositories
                 : Configure webhooks for real-time updates
                 : Set up monitoring and alerts
    
    section Phase 3: Data Collection
        Month 2 : Gather historical data
                : Validate data quality
                : Create baseline metrics
    
    section Phase 4: Advanced Analytics
        Month 3+ : Implement trend analysis
                 : Create custom dashboards
                 : Enable automated insights
```

---

## Key Capabilities Demonstrated

### 1. Unified Test Visibility
- View test results across all repositories
- Track test execution history
- Monitor test duration trends
- Identify flaky tests

### 2. Security Integration
- Display CodeQL security findings
- Track vulnerability remediation
- Monitor security posture over time
- Prioritize security work

### 3. Coverage Tracking
- Display test coverage percentages
- Track coverage trends
- Identify uncovered code paths
- Set coverage targets

### 4. Data Foundation
- Store historical test data
- Enable trend analysis
- Support data-driven decisions
- Foundation for advanced analytics

---

## Next Steps

After completing the workshop, consider:

1. **Expand Coverage**
   - Register additional repositories
   - Configure additional teams
   - Customize component views

2. **Enhance Integration**
   - Set up real-time webhooks
   - Configure custom notifications
   - Integrate with other tools

3. **Enable Analytics**
   - Build custom dashboards
   - Create trend reports
   - Implement automated insights

4. **Customize Experience**
   - Create custom plugins
   - Design team-specific views
   - Add organization branding

---

## Questions & Discussion Points

### Integration Questions
- How many repositories need to be integrated?
- Are all repositories in a single GitHub organization?
- Are there specific teams or projects to prioritize?

### Data Questions
- What historical data retention period is required?
- What specific metrics are most important?
- How should data be shared across teams?

### Workflow Questions
- What is the current test execution process?
- How are security findings currently tracked?
- What reporting is currently manual?

### Technical Questions
- What is the current Backstage version?
- Are there existing custom plugins?
- What is the preferred deployment method?

---

## Additional Resources

- [Installation Guide](../installation_guide.md) - Detailed Backstage setup on GCP
- [GitHub Integration Guide](../github_integration.md) - Complete GitHub configuration
- [Project Resilience Guide](../project_resilience_guide.md) - Test coverage configuration
- [Command Cheatsheet](../command_cheatsheet.md) - Quick reference commands
- [FAQ & Troubleshooting](../faq_troubleshooting.md) - Common issues and solutions

---

## Workshop Support

For questions during or after the workshop:
- Reference the [FAQ & Troubleshooting](../faq_troubleshooting.md) guide
- Check the [Workshop Facilitation Guide](../workshop_facilitation.md)
- Review the [Visual Diagrams](../diagrams.md) for architecture clarity
