# Backstage Workshop Diagrams

This document contains visual diagrams to help understand the Backstage workshop flow and architecture.

## Workshop Overview Flow

```mermaid
flowchart TD
    A[Prerequisites Check] --> B[Deploy Backstage on GCP]
    B --> C[GitHub Integration]
    C --> D[Install Plugins]
    D --> E[Configure Security and Webhooks]
    E --> F[Test and Validate]
    F --> G[Advanced Features]
    
    B1[GKE Cluster Setup] --> B2[Cloud SQL Database] --> B3[Docker Build and Deploy]
    C1[GitHub PAT or App] --> C2[Configure Integration] --> C3[Test Connection]
    D1[GitHub Actions Plugin] --> D2[CodeQL Integration] --> D3[Custom Views]
    E1[Security Config] --> E2[Webhook Setup] --> E3[Real-time Updates]
    G1[Resilience Metrics] --> G2[Data Storage] --> G3[LLM Analysis] --> G4[Scaffolder Templates]
    
    B --> B1
    C --> C1
    D --> D1
    E --> E1
    G --> G1
    
    style A fill:#e1f5fe
    style B fill:#f3e5f5
    style C fill:#e8f5e8
    style D fill:#fff3e0
    style E fill:#fce4ec
    style F fill:#f1f8e9
    style G fill:#e0f2f1
```

## Backstage Architecture on GCP

```mermaid
graph TB
    subgraph "Google Cloud Platform"
        subgraph "GKE Cluster"
            BS[Backstage Frontend]
            API[Backstage Backend API]
            PL[Plugins]
        end
        
        subgraph "Cloud SQL"
            DB[(PostgreSQL Database)]
        end
        
        subgraph "Artifact Registry"
            CR[Container Images]
        end
        
        LB[Load Balancer] --> BS
        BS --> API
        API --> PL
        API --> DB
        CR --> BS
    end
    
    subgraph "External Services"
        GH[GitHub]
        INT[Other Integrations]
    end
    
    API --> GH
    PL --> INT
    
    style BS fill:#4285f4
    style API fill:#34a853
    style DB fill:#ea4335
    style GH fill:#333
```

## GitHub Integration Flow

```mermaid
sequenceDiagram
    participant B as Backstage
    participant G as GitHub API
    participant R as Repository
    participant W as Webhooks
    
    Note over B,R: Initial Setup
    B->>G: Authenticate with PAT/App
    B->>G: Discover repositories
    G->>B: Repository metadata
    
    Note over B,R: Catalog Registration
    B->>R: Read catalog-info.yaml
    R->>B: Component definition
    B->>B: Register in catalog
    
    Note over B,R: Real-time Updates
    R->>W: Code push/PR event
    W->>B: Webhook notification
    B->>G: Fetch latest data
    G->>B: Updated metrics/status
    B->>B: Update UI components
```

## Project Resilience Metrics

```mermaid
mindmap
  root((Project Resilience))
    Test Coverage
      Unit Tests
      Integration Tests
      Coverage Percentage
      Historical Trends
    Security
      CodeQL Scans
      Vulnerability Count
      Dependency Health
      Security Alerts
    Code Quality
      Code Smells
      Technical Debt
      Documentation Coverage
      Complexity Metrics
    Build Health
      Success Rate
      Build Duration
      Deployment Frequency
      MTTR
```

## Test Coverage Integration Flow

```mermaid
sequenceDiagram
    participant Dev as Developer
    participant GH as GitHub Actions
    participant B as Backstage
    participant DB as Database
    
    Dev->>GH: Push code with tests
    GH->>GH: Run test suite
    GH->>GH: Generate coverage report
    GH->>B: POST coverage data
    B->>DB: Store coverage metrics
    B->>B: Update UI components
    
    Note over B: Coverage visualization
    B->>Dev: Display coverage percentage
    B->>Dev: Show trends and flags
    
    alt Coverage < 80%
        B->>Dev: Flag as low resilience
    end
```

## Scaffolder Template Workflow

```mermaid
flowchart LR
    A[Template Definition] --> B[Template Repository]
    B --> C[Backstage Catalog]
    C --> D[Developer Portal]
    
    D --> E[Create Component]
    E --> F[Fill Parameters]
    F --> G[Execute Actions]
    
    subgraph "Template Actions"
        G1[Create GitHub Repo]
        G2[Fetch Skeleton Files]
        G3[Apply Parameters]
        G4[Publish to GitHub]
        G5[Register in Catalog]
    end
    
    G --> G1
    G1 --> G2
    G2 --> G3
    G3 --> G4
    G4 --> G5
    
    G5 --> H[New Project with:]
    H --> I[Test Framework]
    H --> J[CodeQL Setup]
    H --> K[Catalog Integration]
    H --> L[CI/CD Pipeline]
    
    style A fill:#e3f2fd
    style H fill:#e8f5e8
```

## Data Storage and LLM Analysis Architecture

```mermaid
graph TD
    A[GitHub Repos] -->|Webhooks/Actions| B[Backstage Plugins e.g., Code Coverage, Security Insights]
    B -->|API Calls/Data Processing| C[Custom Backend Plugin]
    C -->|Insert Queries| D[Cloud SQL PostgreSQL Database]
    E[Backstage UI] -->|Visualization| B
    style A fill:#0000FF,stroke:#333,stroke-width:2px
    style D fill:#0000FF,stroke:#000,stroke-width:2px
```

```mermaid
flowchart LR
    D[Cloud SQL Database] -->|SQL Queries| F[LLM API e.g., Grok]
    F -->|Analysis/Trends| G[Insights Dashboard/Reports]
    F -->|Decision Triggers| H[AI Agent e.g., LangChain]
    H -->|Proactive Actions| I[GitHub: Generate PR with Unit Tests]
    H -->|Notifications| J[Backstage UI/Alerts]
    style D fill:#0000FF,stroke:#333,stroke-width:2px
    style F fill:#0000FF,stroke:#333,stroke-width:2px
    style H fill:#0000FF,stroke:#333,stroke-width:2px
    style I fill:#0000FF,stroke:#333,stroke-width:2px
```