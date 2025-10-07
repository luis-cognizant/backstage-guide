# Workshop Checklist: Backstage Integration Session

This checklist guides you through the complete workshop session, ensuring all key steps are completed and verified.

## Pre-Workshop Preparation

### Environment Setup
- [ ] Backstage instance running and accessible
  - URL: _________________
  - Admin credentials ready: [ ]
- [ ] Cloud SQL database provisioned
  - Instance name: _________________
  - Connection string tested: [ ]
- [ ] GitHub organization access confirmed
  - Organization name: _________________
  - Admin permissions verified: [ ]

### Repository Preparation
- [ ] Identify target repositories for integration
  - Repository 1: _________________
  - Repository 2: _________________
  - Repository 3: _________________
- [ ] Verify repositories have:
  - [ ] Existing tests configured
  - [ ] GitHub Actions workflows running
  - [ ] Test coverage reporting enabled
  - [ ] CodeQL scans enabled (or can be enabled)

### Materials Ready
- [ ] Workshop walkthrough document accessible
- [ ] Architecture diagrams available
- [ ] Command cheatsheet printed or accessible
- [ ] Troubleshooting guide ready
- [ ] Screen sharing setup tested

---

## Phase 1: Introduction (15 minutes)

### Opening (5 minutes)
- [ ] Welcome participants
- [ ] Introduce facilitators and participants
- [ ] State workshop objective:
  > "Demonstrate Backstage as a unified platform for test and security visibility"
- [ ] Show success criteria:
  - [ ] View GitHub repositories in Backstage
  - [ ] See test results in one centralized view
  - [ ] View CodeQL security findings
  - [ ] Understand data storage for historical analysis

### Environment Validation (10 minutes)
- [ ] Verify Backstage accessibility
  - [ ] Share URL with all participants
  - [ ] Confirm everyone can access: [ ]
  - [ ] Note any access issues: _________________
- [ ] Verify GitHub access
  - [ ] Organization admin access confirmed: [ ]
  - [ ] Target repositories identified: [ ]
  - [ ] List repositories: _________________
- [ ] Verify GCP access
  - [ ] Cloud SQL accessible: [ ]
  - [ ] Required permissions confirmed: [ ]

**🚨 Blockers Identified:**
- Issue 1: _________________
- Issue 2: _________________
- Resolution plan: _________________

---

## Phase 2: GitHub Integration (45 minutes)

### Step 1: Create/Configure GitHub App (15 minutes)

#### GitHub App Creation
- [ ] Navigate to GitHub Settings → Developer Settings → GitHub Apps
- [ ] Click "New GitHub App"
- [ ] Configure basic information:
  - [ ] App name: _________________
  - [ ] Homepage URL: _________________
  - [ ] Webhook URL: _________________ (if applicable)
  - [ ] Webhook secret: _________________ (generated)

#### Configure Permissions
- [ ] Repository permissions:
  - [ ] Contents: `Read-only`
  - [ ] Metadata: `Read-only`
  - [ ] Workflows: `Read-only`
  - [ ] Security events: `Read-only`
  - [ ] Administration: `Read-only` (optional)
  - [ ] Issues: `Read-only` (optional)
  - [ ] Pull requests: `Read-only` (optional)

#### Generate Credentials
- [ ] Generate private key
  - [ ] Download private key file
  - [ ] Store securely: _________________
- [ ] Note App ID: _________________
- [ ] Note Client ID: _________________
- [ ] Generate Client Secret: _________________

#### Install App
- [ ] Click "Install App" in left sidebar
- [ ] Select target organization
- [ ] Choose repositories:
  - [ ] All repositories, or
  - [ ] Selected repositories (list): _________________
- [ ] Complete installation
- [ ] Note Installation ID: _________________

**✅ Success Check:** GitHub App appears in organization's installed apps

### Step 2: Configure Backstage Integration (15 minutes)

#### Update Configuration
- [ ] Access Backstage backend configuration
- [ ] Open `app-config.yaml`
- [ ] Add GitHub integration configuration:
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

#### Set Environment Variables
- [ ] Set `GITHUB_APP_ID`
- [ ] Set `GITHUB_PRIVATE_KEY` (base64 encoded)
- [ ] Set `GITHUB_WEBHOOK_SECRET`
- [ ] Set `GITHUB_CLIENT_ID`
- [ ] Set `GITHUB_CLIENT_SECRET`

#### Restart Backend
- [ ] Save configuration changes
- [ ] Restart Backstage backend
- [ ] Wait for startup to complete
- [ ] Check logs for errors

**✅ Success Check:** No GitHub connection errors in Backstage logs

**📝 Log Snippet:** _________________

### Step 3: Register First Component (15 minutes)

#### Create catalog-info.yaml
- [ ] Choose target repository: _________________
- [ ] Create `catalog-info.yaml` in repository root:
  ```yaml
  apiVersion: backstage.io/v1alpha1
  kind: Component
  metadata:
    name: [component-name]
    description: [component-description]
    annotations:
      github.com/project-slug: [org-name]/[repo-name]
      backstage.io/techdocs-ref: dir:.
  spec:
    type: service
    lifecycle: production
    owner: [team-name]
  ```
- [ ] Customize values:
  - Component name: _________________
  - Description: _________________
  - Project slug: _________________
  - Owner: _________________

#### Register in Backstage
- [ ] Navigate to Backstage UI
- [ ] Click "Create" or "Register Existing Component"
- [ ] Enter repository URL: _________________
- [ ] Click "Analyze"
- [ ] Review detected entities
- [ ] Click "Import"
- [ ] Wait for import to complete

**✅ Success Check:** Component appears in Backstage catalog

**📸 Screenshot:** Component listed in catalog

### Step 4: Verify GitHub Actions Visibility (10 minutes)

#### Navigate to Component
- [ ] Open Backstage catalog
- [ ] Search for registered component: _________________
- [ ] Click on component name

#### Verify Tabs Present
- [ ] Overview tab visible
- [ ] CI/CD tab visible (or GitHub Actions tab)
- [ ] Code tab visible
- [ ] Other tabs: _________________

#### Check GitHub Actions Data
- [ ] Click CI/CD tab
- [ ] Verify recent workflows visible
- [ ] Check workflow statuses shown
- [ ] Verify run durations displayed
- [ ] Confirm triggered by information present

**✅ Success Check:** Can see GitHub Actions run history with details

**🚨 Troubleshooting (if not working):**
- [ ] Verify GitHub App permissions include "Workflows: Read"
- [ ] Check `catalog-info.yaml` annotation format
- [ ] Verify annotation matches: `github.com/project-slug: org/repo`
- [ ] Check Backstage backend logs for API errors
- [ ] Restart backend if configuration was modified
- [ ] Re-import component if needed

**📝 Notes:** _________________

---

## ☕ Break (10 minutes)

Use this time to:
- [ ] Answer participant questions
- [ ] Help anyone who fell behind catch up
- [ ] Verify all setups are working before continuing
- [ ] Address any issues encountered

**Issues to Address:**
- [ ] Issue 1: _________________
- [ ] Issue 2: _________________

---

## Phase 3: Test Coverage & Security (60 minutes)

### Step 1: Install Test Coverage Plugin (25 minutes)

#### Verify Coverage Reporting
- [ ] Check GitHub Actions workflow for coverage generation
- [ ] Identify coverage format used:
  - [ ] lcov
  - [ ] cobertura
  - [ ] jacoco
  - [ ] Other: _________________
- [ ] Verify coverage reports uploaded to GitHub
- [ ] Confirm coverage available in GitHub UI

#### Install Plugin
- [ ] Navigate to Backstage installation directory
- [ ] Run plugin installation:
  ```bash
  yarn --cwd packages/app add @backstage/plugin-code-coverage
  ```
- [ ] Or use alternative coverage plugin: _________________
- [ ] Wait for installation to complete

#### Configure Plugin
- [ ] Open `app-config.yaml`
- [ ] Add coverage configuration:
  ```yaml
  codeCoverage:
    providers:
      - type: github
        target: ^https://github\.com/.*/.*
  ```
- [ ] Adjust configuration for your setup
- [ ] Save configuration

#### Add to UI
- [ ] Open `packages/app/src/components/catalog/EntityPage.tsx`
- [ ] Import coverage plugin component
- [ ] Add coverage card to component overview
- [ ] Save file

#### Rebuild and Deploy
- [ ] Build frontend: `yarn build`
- [ ] Restart Backstage
- [ ] Wait for startup
- [ ] Clear browser cache if needed

#### Verify Coverage Display
- [ ] Navigate to component in Backstage
- [ ] Check Overview tab
- [ ] Verify coverage widget appears
- [ ] Confirm coverage percentage displayed

**✅ Success Check:** Coverage percentage visible on component overview

**📸 Screenshot:** Coverage widget showing percentage

**🚨 Troubleshooting:**
- [ ] If widget not appearing, check browser console for errors
- [ ] Verify coverage data exists in GitHub
- [ ] Check plugin configuration in `app-config.yaml`
- [ ] Ensure correct format parser is configured

### Step 2: Configure CodeQL Integration (25 minutes)

#### Verify CodeQL Setup
- [ ] Navigate to target repository on GitHub
- [ ] Go to Security tab → Code scanning alerts
- [ ] Verify CodeQL workflow exists:
  - [ ] Check `.github/workflows/` for CodeQL workflow
  - [ ] Workflow name: _________________
- [ ] Confirm recent scans completed:
  - [ ] Last scan date: _________________
  - [ ] Scan status: _________________
- [ ] Review sample findings (if any)

#### Enable CodeQL (if not already enabled)
- [ ] Navigate to repository Settings → Security & analysis
- [ ] Enable "Code scanning"
- [ ] Choose "Set up" → "CodeQL Analysis"
- [ ] Configure languages: _________________
- [ ] Commit workflow file
- [ ] Wait for initial scan to complete

#### Install Security Plugin
- [ ] Check if plugin already installed
- [ ] If not, install:
  ```bash
  yarn --cwd packages/app add @backstage/plugin-security-insights
  ```
- [ ] Or use alternative security plugin: _________________

#### Configure Plugin
- [ ] Add security plugin to component page
- [ ] Configure in `EntityPage.tsx`
- [ ] Import security components
- [ ] Add security tab or card
- [ ] Save changes

#### Rebuild and Verify
- [ ] Build frontend
- [ ] Restart Backstage
- [ ] Navigate to component
- [ ] Verify Security tab appears
- [ ] Check security findings displayed

**✅ Success Check:** CodeQL findings visible in Backstage

**📸 Screenshot:** Security findings with severity levels

### Step 3: Complete View Verification (10 minutes)

#### Overview Tab Review
- [ ] Navigate to component in Backstage
- [ ] Open Overview tab
- [ ] Verify displayed information:
  - [ ] Repository name and description
  - [ ] Test coverage percentage
  - [ ] Recent commits
  - [ ] Owner and team information
  - [ ] Links to GitHub repository
  - [ ] Additional metadata

#### CI/CD Tab Review
- [ ] Open CI/CD tab
- [ ] Verify displayed information:
  - [ ] Recent workflow runs (at least 3)
  - [ ] Run statuses (success/failure/in-progress)
  - [ ] Run durations
  - [ ] Triggered by information
  - [ ] Branch information
  - [ ] Commit messages

#### Security Tab Review
- [ ] Open Security tab
- [ ] Verify displayed information:
  - [ ] CodeQL findings count
  - [ ] Severity levels (Critical/High/Medium/Low)
  - [ ] Finding descriptions
  - [ ] Affected file paths
  - [ ] Line numbers
  - [ ] Remediation guidance

**✅ Success Check:** All three tabs show complete data

**📸 Screenshots:**
- [ ] Overview tab - Full view
- [ ] CI/CD tab - Workflow history
- [ ] Security tab - CodeQL findings

**Participant Confirmation:**
- [ ] Team confirms this meets requirements
- [ ] Feedback received: _________________
- [ ] Additional requests: _________________

---

## Phase 4: Data Foundation (30 minutes)

### Step 1: Database Schema Review (10 minutes)

#### Connect to Database
- [ ] Connect to Cloud SQL instance
- [ ] Connection method used:
  - [ ] Cloud SQL Proxy
  - [ ] Direct connection
  - [ ] Other: _________________
- [ ] Connection string: _________________

#### Review/Create Schema
- [ ] Check if tables exist
- [ ] If not, create test results table:
  ```sql
  CREATE TABLE test_results (
    id SERIAL PRIMARY KEY,
    repo_name VARCHAR(255),
    commit_sha VARCHAR(40),
    test_date TIMESTAMP,
    total_tests INT,
    passed_tests INT,
    failed_tests INT,
    coverage_percentage DECIMAL(5,2),
    workflow_run_id BIGINT,
    duration_seconds INT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
  );
  ```
- [ ] Create security scans table:
  ```sql
  CREATE TABLE security_scans (
    id SERIAL PRIMARY KEY,
    repo_name VARCHAR(255),
    scan_date TIMESTAMP,
    severity VARCHAR(20),
    finding_type VARCHAR(100),
    description TEXT,
    file_path VARCHAR(500),
    line_number INT,
    rule_id VARCHAR(100),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
  );
  ```
- [ ] Create coverage history table:
  ```sql
  CREATE TABLE coverage_history (
    id SERIAL PRIMARY KEY,
    repo_name VARCHAR(255),
    measurement_date TIMESTAMP,
    line_coverage DECIMAL(5,2),
    branch_coverage DECIMAL(5,2),
    total_lines INT,
    covered_lines INT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
  );
  ```

#### Explain Schema
- [ ] Walk through table structure
- [ ] Explain field purposes
- [ ] Discuss data retention
- [ ] Show indexing strategy

**✅ Success Check:** Tables created and accessible

### Step 2: Configure Data Collection (15 minutes)

#### Backend Plugin Configuration
- [ ] Review or create custom backend plugin for data storage
- [ ] Configure database connection in backend
- [ ] Set up data collection endpoints
- [ ] Configure webhook handlers (if applicable)

#### Test Data Insertion
- [ ] Insert sample test result:
  ```sql
  INSERT INTO test_results 
    (repo_name, commit_sha, test_date, total_tests, passed_tests, failed_tests, coverage_percentage)
  VALUES 
    ('example-org/example-repo', 'abc123', NOW(), 150, 148, 2, 85.5);
  ```
- [ ] Verify insertion successful
- [ ] Query data back:
  ```sql
  SELECT * FROM test_results ORDER BY test_date DESC LIMIT 5;
  ```

#### Webhook Setup (Optional)
- [ ] Configure GitHub webhook for repository
- [ ] Set webhook URL: _________________
- [ ] Select events to monitor:
  - [ ] Workflow runs
  - [ ] Code scanning alerts
  - [ ] Push events
- [ ] Set webhook secret
- [ ] Test webhook delivery

**✅ Success Check:** Can store and retrieve test data from database

### Step 3: Historical Data Demonstration (5 minutes)

#### Query Historical Trends
- [ ] Show test results over time:
  ```sql
  SELECT 
    repo_name,
    DATE(test_date) as date,
    AVG(coverage_percentage) as avg_coverage,
    SUM(failed_tests) as total_failures
  FROM test_results
  WHERE test_date > NOW() - INTERVAL '30 days'
  GROUP BY repo_name, DATE(test_date)
  ORDER BY date DESC;
  ```

#### Explain Analytics Potential
- [ ] Discuss trend analysis capabilities
- [ ] Show coverage trajectory
- [ ] Identify patterns in failures
- [ ] Explain how this data enables:
  - Quality trend monitoring
  - Predictive insights
  - Automated recommendations
  - Custom reporting

**✅ Success Check:** Participants understand data foundation value

---

## Phase 5: Wrap-Up (45 minutes)

### Demonstration Review (20 minutes)

#### Showcase Complete Solution
- [ ] Open Backstage to catalog view
- [ ] Navigate through registered components
- [ ] Show each major feature:
  - [ ] Component catalog with metadata
  - [ ] Test coverage visualization
  - [ ] GitHub Actions integration
  - [ ] CodeQL security findings
  - [ ] Historical data in database

#### Highlight Key Achievements
- [ ] Single pane of glass for test visibility
- [ ] Unified security and quality view
- [ ] Historical data foundation
- [ ] Extensible platform for future enhancements

**📸 Final Screenshots:**
- [ ] Complete catalog view
- [ ] Component overview with all data
- [ ] Database query results

### Q&A Session (25 minutes)

#### Questions to Address
- [ ] Q: _________________
  - A: _________________
- [ ] Q: _________________
  - A: _________________
- [ ] Q: _________________
  - A: _________________

#### Common Topics
- [ ] Scaling to more repositories
- [ ] Performance considerations
- [ ] Security and access control
- [ ] Customization options
- [ ] Integration with other tools
- [ ] Cost considerations
- [ ] Maintenance requirements
- [ ] Team onboarding process

### Next Steps Discussion

#### Immediate Actions
- [ ] Identify additional repositories to onboard
- [ ] Assign ownership and responsibilities
- [ ] Schedule follow-up sessions if needed
- [ ] Document any custom requirements

#### Short-Term Goals (1-2 weeks)
- [ ] Roll out to priority repositories
- [ ] Configure remaining teams
- [ ] Set up additional plugins
- [ ] Create team documentation

#### Medium-Term Goals (1-3 months)
- [ ] Complete repository onboarding
- [ ] Establish monitoring and alerts
- [ ] Build custom dashboards
- [ ] Implement advanced analytics
- [ ] Integrate additional data sources

#### Long-Term Vision (3+ months)
- [ ] Advanced automation
- [ ] Custom plugin development
- [ ] Organization-wide adoption
- [ ] Continuous improvement process

---

## Post-Workshop Tasks

### Documentation
- [ ] Compile workshop notes
- [ ] Save all screenshots
- [ ] Document configurations made
- [ ] Record credentials (securely)
- [ ] Create runbook for future reference

### Deliverables
- [ ] Share workshop materials with participants
- [ ] Provide access to documentation
- [ ] Send follow-up email with:
  - [ ] Summary of what was accomplished
  - [ ] Links to Backstage instance
  - [ ] Documentation links
  - [ ] Contact information for support

### Follow-Up
- [ ] Schedule follow-up session (if needed)
- [ ] Set up regular check-ins
- [ ] Create support channel
- [ ] Plan for next phase

---

## Success Criteria Summary

### Technical Achievements
- [x] Backstage connected to GitHub
- [x] Components registered and visible
- [x] Test coverage displayed
- [x] CodeQL integration working
- [x] GitHub Actions history visible
- [x] Database schema created
- [x] Data collection configured

### Participant Outcomes
- [ ] Participants can navigate Backstage
- [ ] Participants understand integration architecture
- [ ] Participants can register new components
- [ ] Participants see value in unified view
- [ ] Participants understand data foundation

### Business Outcomes
- [ ] Demonstrated single pane of glass capability
- [ ] Showed test and security visibility
- [ ] Established foundation for analytics
- [ ] Identified next steps for full implementation
- [ ] Received positive feedback

---

## Lessons Learned

### What Went Well
- _________________
- _________________
- _________________

### Challenges Encountered
- _________________
- _________________
- _________________

### Improvements for Next Time
- _________________
- _________________
- _________________

### Participant Feedback
- _________________
- _________________
- _________________

---

## Workshop Metrics

- **Total Duration:** _________ hours
- **Participants:** _________ people
- **Repositories Integrated:** _________
- **Components Registered:** _________
- **Plugins Installed:** _________
- **Issues Encountered:** _________
- **Issues Resolved:** _________

**Overall Success Rating:** _____ / 10

---

## Additional Notes

_________________
_________________
_________________
_________________
