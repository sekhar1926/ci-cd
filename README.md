graph TD
    subgraph "Phase 1: Development & CI (Dev Environment)"
        A[Developer merges 'feature' branch to 'dev'] --> B["'dev' branch push triggers Application Pipeline"];
        B --> C["Build Stage: <br>Builds image, tags as 'dev-commit', pushes to ACR"];
        C --> D["Deploy Dev Stage: <br>Deploys 'dev-commit' image to DEV Cluster"];
    end

    %% Dotted line indicates the start of the formal release process
    D-.->|"When ready for release"|E

    subgraph "Phase 2: Build Release Candidate (The 'One Build')"
        E["Release Manager runs 'Create Release' pipeline"] --> F["Pipeline creates 'release/1.1.0' branch"];
        F --> G["'release/1.1.0' push triggers Application Pipeline"];
        G --> H["<b>BUILD STAGE (THE ONLY BUILD)</b><br>Builds ONE image, tags as '1.1.0-rc', pushes to ACR"];
    end

    subgraph "Phase 3: QA Validation"
        H --> I["Deploy QA Stage: <br>Deploys '1.1.0-rc' image to QA Clusters"];
        I --> J{"Wait for Manual Approval <br>on QA Environments"};
        J -- Approved --> K["QA Team tests the exact '1.1.0-rc' image artifact"];
        K -- Bug Found --> L["Bug fixed in 'release/1.1.0' branch, triggering a new build (e.g., '1.1.0-rc2')"];
        L --> H;
        K -- All Good --> M{QA Sign-off on '1.1.0-rc'};
    end

    subgraph "Phase 4: Promotion & Production Release"
        M --> N["Release Manager runs 'Finalize Release' pipeline"];
        N --> O["Pipeline merges code & creates 'v1.1.0' tag on 'main'"];
        O --> P["'v1.1.0' tag triggers Application Pipeline"];
        P --> Q["<b>PROMOTE STAGE (NO BUILD)</b><br>Finds '1.1.0-rc' image in ACR and adds the final '1.1.0' tag to it"];
        Q --> R["Deploy Prod Stage: <br>Deploys the image using the '1.1.0' tag"];
        R --> S{"Wait for Manual Approval <br>on PROD Environments"};
        S -- Approved --> T["The QA-approved artifact is deployed to PROD Clusters"];
        T --> U[Application is LIVE in Production!];
    end

    style H fill:#d4edda,stroke:#155724,stroke-width:4px
    style Q fill:#fff3cd,stroke:#856404,stroke-width:4px
    style J fill:#fff2cc,stroke:#333,stroke-width:2px
    style S fill:#fff2cc,stroke:#333,stroke-width:2px
    style U fill:#ccffcc,stroke:#333,stroke-width:4px