```mermaid
graph TD
    subgraph "START: Monitoring SQS Queue"
        A(Start Monitoring) --> B{Is Age of Oldest Message > 5 mins?};
    end

    subgraph "SCALE-UP LOGIC (Failover)"
        B -- "Yes (Primary Worker is down)" --> C[TRIGGER ALARM: 'Primary_Worker_Failure'];
        C --> D{Is ASG Desired Capacity = 0?};
        D -- "Yes" --> E[ACTION: Set Desired Capacity = 2];
        E --> F[RESULT: AWS GPU Workers are now active];
        D -- "No (Already scaled up)" --> G[RESULT: No action needed];
    end

    subgraph "SCALE-DOWN LOGIC (Failback)"
        B -- "No (Primary Worker is OK)" --> H{Is Number of Visible Messages <= 10 for 15 mins?};
        H -- "Yes (Queue is clear)" --> I[TRIGGER ALARM: 'Queue_Is_Clear'];
        I --> J{Is ASG Desired Capacity > 0?};
        J -- "Yes" --> K[ACTION: Set Desired Capacity = 0];
        K --> L[RESULT: AWS GPU Workers are now terminated];
        J -- "No (Already scaled down)" --> M[RESULT: No action needed];
        H -- "No (Still processing)" --> N[RESULT: Continue monitoring];
    end

    %% Styling
    classDef decision fill:#FF9900,stroke:#333,stroke-width:2px,color:#000
    classDef action fill:#232F3E,stroke:#fff,stroke-width:1px,color:#fff
    classDef state fill:#2E73B8,stroke:#fff,stroke-width:1px,color:#fff
    classDef terminal fill:#5A6370,stroke:#fff,stroke-width:1px,color:#fff

    class B,D,H,J decision
    class C,I state
    class E,K action
    class F,G,L,M,N terminal
```
