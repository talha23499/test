sequenceDiagram
    participant User
    participant FrontendApp as Frontend App
    participant BackendService as Backend API/Service
    participant Database as RPSP

    User->>FrontendApp: Attempts to Open Plan X for Editing
    FrontendApp->>BackendService: GET /api/plans/X?intent=edit
    BackendService->>Database: Fetch Plan X Data (includes its schemaVersionId)
    Database-->>BackendService: Plan X Data (e.g., version 'v1.2.0')
    BackendService->>BackendService: Compare Plan's version 'v1.2.0' with Latest Schema Version (e.g., 'v2.0.0')

    alt Plan Version is Outdated
        BackendService-->>FrontendApp: Response: { status: "outdated", currentVersion: "v1.2.0", latestVersion: "v2.0.0" }
        FrontendApp->>User: Prompt: "Plan is outdated. Update to version v2.0.0 to edit?"
        User->>FrontendApp: Confirms Update (Acknowledges)
        %% Frontend might need to fetch the plan data again if not sent with the 'outdated' response,
        %% or the backend could have sent it. Assuming frontend sends the original data back as per your Java code.
        FrontendApp->>BackendService: POST /migrate-plan (originalPlanData, fromVersion='v1.2.0', toVersion='v2.0.0')
        BackendService->>BackendService: Load 'fromSchema' (v1.2.0) and 'toSchema' (v2.0.0) (internally)
        BackendService->>BackendService: Perform recursive data migration
        BackendService->>BackendService: Update schemaVersionId in migrated data to 'v2.0.0'
        BackendService-->>FrontendApp: Migrated Plan Data (conforms to v2.0.0)
        FrontendApp->>FrontendApp: Load migrated data into editor UI (now based on v2.0.0 schema)
        User->>FrontendApp: Edits Plan
        FrontendApp->>BackendService: PUT /api/plans/X (with migrated & edited data, schemaVersionId='v2.0.0')
        BackendService->>Database: Save Updated Plan X (now v2.0.0)
        Database-->>BackendService: Save Confirmation
        BackendService-->>FrontendApp: Save Confirmation
        FrontendApp-->>User: Plan Saved Successfully
    else Plan Version is Current
        BackendService-->>FrontendApp: Plan X Data (version 'v2.0.0')
        FrontendApp->>FrontendApp: Load plan data into editor UI (based on v2.0.0 schema)
        User->>FrontendApp: Edits Plan
        FrontendApp->>BackendService: PUT /api/plans/X (edited data, schemaVersionId='v2.0.0')
        BackendService->>Database: Save Updated Plan X
        Database-->>BackendService: Save Confirmation
        BackendService-->>FrontendApp: Save Confirmation
        FrontendApp-->>User: Plan Saved Successfully
    end
