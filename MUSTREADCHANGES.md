# Repository Impact Analysis - Smart Requisition System

This document highlights where in the MerchP.A backend code repository we would implement changes to introduce the smart requisition system without complex AI implementation.

## 1. Directory Structure Impact

```mermaid
flowchart TB
    classDef new fill:#4CAF50,stroke:#388E3C,color:white,stroke-width:2px
    classDef modified fill:#FFC107,stroke:#FFA000,color:black,stroke-width:2px
    classDef existing fill:#2196F3,stroke:#0D47A1,color:white,stroke-width:1px
    classDef heading fill:#607D8B,stroke:#37474F,color:white,stroke-width:2px,font-weight:bold

    title["DIRECTORY STRUCTURE IMPACT"]:::heading
    
    existing["EXISTING FILES"]:::heading
    usersFile["models/users.py"]:::existing
    taskFile["models/task.py"]:::existing
    taskRoutesFile["apis/task_routes.py"]:::existing
    seedUsersFile["seed/seed_users.py"]:::existing
    
    mUsers["Add client & professional fields"]:::modified
    mTask["Enhance relationship with new models"]:::modified
    mTaskRoutes["Extend with quotation handling"]:::modified
    mSeedUsers["Add client seeding"]:::modified
    
    usersFile --> mUsers
    taskFile --> mTask
    taskRoutesFile --> mTaskRoutes
    seedUsersFile --> mSeedUsers
    
    new["NEW FILES"]:::heading
    
    newModels["MODELS"]:::heading
    reqModel["requisition.py"]:::new
    quotModel["quotation.py"]:::new
    progModel["progress.py"]:::new
    
    newAPIs["APIs"]:::heading
    reqRoutes["requisition_routes.py"]:::new
    quotRoutes["quotation_routes.py"]:::new
    clientRoutes["client_routes.py"]:::new
    
    newServices["SERVICES"]:::heading
    matchService["matching_service.py"]:::new
    scoreService["scoring_service.py"]:::new
    
    newSeed["SEED DATA"]:::heading
    seedReq["seed_requisitions.py"]:::new
    seedQuot["seed_quotations.py"]:::new
    
    title --> existing
    title --> new
    
    new --> newModels
    new --> newAPIs
    new --> newServices
    new --> newSeed
    
    newModels --> reqModel
    newModels --> quotModel
    newModels --> progModel
    
    newAPIs --> reqRoutes
    newAPIs --> quotRoutes
    newAPIs --> clientRoutes
    
    newServices --> matchService
    newServices --> scoreService
    
    newSeed --> seedReq
    newSeed --> seedQuot
```

## 2. Model Relationships Impact

```mermaid
flowchart TB
    classDef new fill:#4CAF50,stroke:#388E3C,color:white,stroke-width:2px
    classDef existing fill:#2196F3,stroke:#0D47A1,color:white,stroke-width:1px
    classDef rel color:#333,font-style:italic

    title["MODEL RELATIONSHIPS"]
    
    subgraph Existing["Existing Model Relationships"]
        direction TB
        Users["Users\n(with role='contractor')"]:::existing
        Task["Task"]:::existing
        
        Users -->|"manages"| Task
        Users -->|"assigned to"| Task
    end
    
    subgraph New["New Model Relationships"]
        direction TB
        Client["Client"]:::new
        Requisition["Requisition"]:::new
        Quotation["Quotation"]:::new
        Progress["Progress"]:::new
        
        Client -->|"creates"| Requisition
        Users2["Users\n(with role='professional')"]:::existing
        Users2 -->|"bids on"| Requisition
        Users2 -->|"submits"| Quotation
        Quotation -->|"generates"| Task2["Task"]:::existing
        Task2 -->|"tracks"| Progress
    end
    
    Existing -.- New
```

## 3. API Routes Impact

```mermaid
mindmap
    root((API Routes))
        Existing Routes
            ::icon(fa fa-bolt)
            task_routes.py
                :::modified Extend with quotation selection
            contractor_routes.py
                :::modified Add oversight endpoints
        New Routes
            ::icon(fa fa-plus)
            requisition_routes.py
                :::new Client requisition endpoints
                Create requisition
                List requisitions
                View quotations
                Select winning bid
            quotation_routes.py
                :::new Professional bidding endpoints
                Submit quotation
                View requisition details
                Update quotation
            client_routes.py
                :::new Client management endpoints
                Register client
                Manage profile
                View task progress

%%    linkStyle 0 stroke:#FFC107,stroke-width:2px
%%    linkStyle 2 stroke:#4CAF50,stroke-width:2px
%%    linkStyle 3 stroke:#4CAF50,stroke-width:2px
%%    linkStyle 4 stroke:#4CAF50,stroke-width:2px
```

## 4. Service Layer Implementation

```mermaid
graph TB
    classDef new fill:#4CAF50,stroke:#388E3C,color:white,stroke-width:2px
    
    A["NEW SERVICE LAYER"]
    
    A --> B["Matching Services"]
    A --> C["Evaluation Services"]
    A --> D["Workflow Services"] 
    A --> E["Notification Services"]
    
    subgraph matching ["Matching Services"]
        M1["/services/matching_service.py"]:::new
        M2["rule_based_matching()"]
        M3["tag_based_categorization()"]
        M4["location_based_matching()"]
        
        M1 --> M2
        M1 --> M3
        M1 --> M4
    end
    
    subgraph scoring ["Evaluation Services"]
        S1["/services/scoring_service.py"]:::new
        S2["evaluate_quotation()"]
        S3["smart_merge_requisition()"]
        S4["calculate_quality_score()"]
        
        S1 --> S2
        S1 --> S3
        S1 --> S4
    end
    
    subgraph workflow ["Workflow Services"]
        W1["/services/workflow_service.py"]:::new
        W2["process_requisition()"]
        W3["handle_quotation_selection()"]
        W4["track_progress()"]
        
        W1 --> W2
        W1 --> W3
        W1 --> W4
    end
    
    subgraph notification ["Notification Services"]
        N1["/services/notification_service.py"]:::new
        N2["notify_professional()"]
        N3["notify_client()"]
        N4["notify_contractor()"]
        
        N1 --> N2
        N1 --> N3
        N1 --> N4
    end
    
    B --- matching
    C --- scoring
    D --- workflow
    E --- notification
```

## 5. Core Implementation Logic

```mermaid
classDiagram
    class TagManager {
        +extract_tags_from_requisition(requisition)
        +get_professional_tags(pro_id)
        +match_professional_by_tags(tags, min_match_percent)
        +filter_relevant_tags(all_tags)
    }
    
    class RequisitionWorkflow {
        -db
        -notification_service
        +process_new_requisition(requisition)
        +select_winning_quotation(requisition_id, quotation_id)
        +create_task_from_quotation(quotation)
        +notify_task_creation(task_id, requisition_id, quotation_id)
    }
    
    class ScoreCalculator {
        +calculate_price_score(amount, budget)
        +calculate_time_score(days, expected_timeframe)
        +calculate_quality_score(rating, completion_rate)
        +evaluate_quotation(quotation, requisition, historical_data)
    }
    
    class LocationMatcher {
        +get_coordinates(location)
        +calculate_distance(loc1, loc2)
        +match_by_location(requisition, max_distance_km)
        +calculate_proximity_score(pro_location, req_location)
    }
    
    TagManager --> RequisitionWorkflow : provides matching
    ScoreCalculator --> RequisitionWorkflow : evaluates bids
    LocationMatcher --> RequisitionWorkflow : finds nearby pros
    RequisitionWorkflow --> Task : creates
```

## 6. Database Migration Impact

```mermaid
pie showData
    title Database Migration Complexity
    "New Tables" : 35
    "Foreign Key Relationships" : 25
    "Indexes" : 15
    "User Role Updates" : 15
    "Task Relationships" : 10
```

## 7. Implementation Timeline & Phases

```mermaid
gantt
    title Implementation Timeline
    dateFormat  YYYY-MM-DD
    section Database
    Create models           :a1, 2025-06-01, 14d
    Apply migrations        :a2, after a1, 7d
    Seed initial data       :a3, after a2, 7d
    
    section Backend Services
    Matching service        :b1, after a3, 21d
    Scoring service         :b2, after a3, 21d
    Workflow automation     :b3, after b1, 14d
    Notification system     :b4, after b1, 14d
    
    section API Development
    Client routes           :c1, after a3, 21d
    Professional routes     :c2, after a3, 21d
    Update task routes      :c3, after c1, 7d
    
    section Integration
    Backend integration     :d1, after b3, 14d
    UI development          :d2, after c2, 28d
    Testing                 :d3, after d1, 21d
    Deployment              :d4, after d3, 7d
```

## 8. Implementation Workflow

```mermaid
stateDiagram-v2
    [*] --> DatabaseMigrations
    
    state DatabaseMigrations {
        [*] --> CreateModels
        CreateModels --> ApplyMigrations
        ApplyMigrations --> SeedData
        SeedData --> [*]
    }
    
    DatabaseMigrations --> CoreServices
    
    state CoreServices {
        [*] --> MatchingServices
        [*] --> ScoringServices
        MatchingServices --> WorkflowServices
        ScoringServices --> WorkflowServices
        MatchingServices --> NotificationServices
        WorkflowServices --> [*]
        NotificationServices --> [*]
    }
    
    CoreServices --> APILayer
    
    state APILayer {
        [*] --> ClientRoutes
        [*] --> ProfessionalRoutes
        ClientRoutes --> TaskRoutesUpdate
        ProfessionalRoutes --> TaskRoutesUpdate
        TaskRoutesUpdate --> [*]
    }
    
    APILayer --> UIIntegration
    UIIntegration --> Testing
    Testing --> Deployment
    Deployment --> [*]
```

## Key Implementation Files and Changes

| Type | File Path | Change Type | Priority | Description |
|------|-----------|-------------|----------|-------------|
| Model | `/models/requisition.py` | New | High | Stores client service requests and metadata |
| Model | `/models/quotation.py` | New | High | Stores professional bids on requisitions |
| Model | `/models/progress.py` | New | Medium | Tracks task progress and milestone completion |
| Model | `/models/users.py` | Modify | High | Add client role and professional attributes |
| API | `/apis/requisition_routes.py` | New | High | Client-facing endpoints for requisition management |
| API | `/apis/quotation_routes.py` | New | High | Professional-facing endpoints for bidding |
| API | `/apis/task_routes.py` | Modify | Medium | Update to handle tasks created from quotations |
| Service | `/services/matching_service.py` | New | High | Rule-based matching algorithms |
| Service | `/services/scoring_service.py` | New | High | Bid evaluation and ranking logic |
| Service | `/services/workflow_service.py` | New | Medium | Process automation for requisition workflow |
| Migration | `migrations/versions/new_migration.py` | New | High | Database schema updates for new models |
| Seed | `/seed/seed_clients.py` | New | Medium | Sample data for client testing |
| Seed | `/seed/seed_requisitions.py` | New | Medium | Sample data for requisition testing |

This implementation strategy focuses on practical, rule-based approaches rather than complex AI, while still delivering a comprehensive requisition automation system.
