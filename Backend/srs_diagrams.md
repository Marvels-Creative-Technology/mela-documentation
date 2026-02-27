# Software Requirements Specification (SRS) Diagrams for Mela Guzo
**Prepared By:** Marvels Creative Technology  
**Version:** 1.0  

This document contains 16 comprehensive system diagrams constructed to support the Software Requirements Specification (SRS) for the Mela Guzo platform.

---

### 1. Overall System Use Case Diagram
This visualizes the high-level boundary and the primary actors interacting with the entire Mela Guzo ecosystem.

```mermaid
graph TD
    classDef actor fill:#f9f,stroke:#333,stroke-width:2px;
    Rider((Rider)):::actor
    Driver((Driver)):::actor
    Dispatcher((Dispatcher)):::actor
    SuperAdmin((SuperAdmin)):::actor
    
    subgraph "Mela Guzo System"
        UC1(Manage Ride Bookings)
        UC2(Manage Digital Wallet)
        UC3(Track Real-time Locations)
        UC4(Manage Fleet Operations)
        UC5(Provide Support/SOS)
        UC6(Configure System Architecture)
    end
    
    Rider --> UC1
    Rider --> UC2
    Rider --> UC3
    Driver --> UC1
    Driver --> UC2
    Driver --> UC3
    Driver --> UC5
    Dispatcher --> UC4
    Dispatcher --> UC3
    SuperAdmin --> UC6
    SuperAdmin --> UC4
```

---

### 2. Rider Specific Use Case Diagram
Details the exact systemic permissions and triggers available specifically to the passenger client app.

```mermaid
graph TD
    classDef actor fill:#f9f,stroke:#333,stroke-width:2px;
    Rider((Rider)):::actor
    PaymentGateway((PaymentGateway)):::actor
    
    subgraph "Rider Domain"
        R1(Register & Verify OTP)
        R2(Request Ride Estimate)
        R3(Book Ride)
        R4(Cancel Ride Request)
        R5(Top-up Wallet)
        R6(Pay via Wallet)
        R7(Submit Driver Review)
    end
    
    Rider --> R1
    Rider --> R2
    Rider --> R3
    Rider --> R4
    Rider --> R5
    Rider --> R6
    Rider --> R7
    R5 --> PaymentGateway
```

---

### 3. Driver Specific Use Case Diagram
Focuses on the mobile application capabilities exclusively tailored for fleet drivers.

```mermaid
graph TD
    classDef actor fill:#f9f,stroke:#333,stroke-width:2px;
    Driver((Driver)):::actor
    TelemetryNode((TelemetryNode)):::actor
    
    subgraph "Driver Domain"
        D1(Submit KYC Docs)
        D2(Toggle Online Status)
        D3(Accept/Reject Ride)
        D4(Broadcast Location)
        D5(Complete Trip)
        D6(Withdraw Earnings)
    end
    
    Driver --> D1
    Driver --> D2
    Driver --> D3
    Driver --> D4
    Driver --> D5
    Driver --> D6
    D4 --> TelemetryNode
```

---

### 4. Admin & Dispatcher Use Case Diagram
Visualizes the backend operations portal mapping for corporate staff.

```mermaid
graph TD
    classDef actor fill:#f9f,stroke:#333,stroke-width:2px;
    ZonalDispatcher((ZonalDispatcher)):::actor
    SuperAdmin((SuperAdmin)):::actor
    
    subgraph "Admin Portal"
        A1(Monitor Regional Fleet)
        A2(Assign Manual Ride)
        A3(Approve/Ban Driver)
        A4(Manage Promo Codes)
        A5(Configure Dynamic Pricing)
        A6(Generate Financial Reports)
        A7(Manage User Roles)
    end
    
    ZonalDispatcher --> A1
    ZonalDispatcher --> A2
    ZonalDispatcher --> A3
    
    SuperAdmin --> A1
    SuperAdmin --> A3
    SuperAdmin --> A4
    SuperAdmin --> A5
    SuperAdmin --> A6
    SuperAdmin --> A7
```

---

### 5. Sequence Diagram: Authentication & OTP Verification
Illustrates the chronological API calls required to securely onboard a new rider using SMS OTP.

```mermaid
sequenceDiagram
    autonumber
    actor User
    participant App
    participant AuthEngine
    participant SMSGateway
    participant DB
    
    User->>App: Submits Phone Number
    App->>AuthEngine: POST /sendMobileLoginOtp
    AuthEngine->>DB: Store hashed OTP & timestamp
    AuthEngine->>SMSGateway: Trigger SMS Delivery
    SMSGateway-->>User: Receives Code
    
    User->>App: Inputs Code
    App->>AuthEngine: POST /userMobileLogin
    AuthEngine->>DB: Validate OTP hash & expiry
    DB-->>AuthEngine: Valid
    AuthEngine->>DB: Revoke OTP, Create User Session
    AuthEngine-->>App: Return 120-Char Session Token + Sanctum Bearer
    App-->>User: Dashboard Unlocked
```

---

### 6. Sequence Diagram: Ride Booking & Matching Algorithm
Steps executed when matching a rider to the nearest compatible driver.

```mermaid
sequenceDiagram
    autonumber
    actor Rider
    participant RideEngine
    participant MapsAPI
    participant DB
    participant Driver
    
    Rider->>RideEngine: Request Ride (Pickup, Dropoff, Class)
    RideEngine->>MapsAPI: Calculate Distance Matrix
    MapsAPI-->>RideEngine: Distance, Polyline, Est. Time
    RideEngine->>DB: Generate Pending Booking Record
    RideEngine->>DB: Execute Nearest-Neighbor Query (Drivers < 5km)
    DB-->>RideEngine: List of Candidates
    
    RideEngine-->>Driver: FCM Push Notification (Ping)
    Driver->>RideEngine: Accepts Ping
    RideEngine->>DB: Update Booking (driver_id matched)
    RideEngine-->>Rider: Ride Confirmed (Driver Details)
```

---

### 7. Sequence Diagram: Wallet Top-Up (Chapa Gateway)
Maps the secure asynchronous webhook processing for topping up a rider wallet.

```mermaid
sequenceDiagram
    autonumber
    actor Rider
    participant WalletModule
    participant ChapaAPI
    participant WebhookHandler
    participant DB
    
    Rider->>WalletModule: Request 500 ETB Top-up
    WalletModule->>ChapaAPI: Initialize Transaction Checkout
    ChapaAPI-->>WalletModule: Secure Checkout URL
    WalletModule-->>Rider: Redirect to Chapa Payment Page
    
    Rider->>ChapaAPI: Submits Banking Details
    ChapaAPI->>WebhookHandler: Asynchronous POST Webhook (Success)
    WebhookHandler->>WebhookHandler: Verify Cryptographic Hash
    WebhookHandler->>DB: INSERT Transaction (Credit 500 ETB)
    WebhookHandler->>DB: UPDATE Wallet Balance (+500 ETB)
    WebhookHandler-->>ChapaAPI: HTTP 200 OK
    WalletModule-->>Rider: Balance Updated UI Refresh
```

---

### 8. Sequence Diagram: Ride Completion & automated Payment Settlement
Shows how the system executes a ride-end trigger and balances the internal ledgers.

```mermaid
sequenceDiagram
    autonumber
    actor Driver
    participant RideEngine
    participant WalletLedger
    participant DB
    participant Rider
    
    Driver->>RideEngine: Trigger "End Ride" Request
    RideEngine->>DB: Update Booking Status = "Completed"
    RideEngine->>WalletLedger: Settle Fare (e.g. 200 ETB, Driver Cut 80%)
    
    WalletLedger->>DB: Deduct 200 ETB from Rider Wallet
    WalletLedger->>DB: Credit 160 ETB to Driver Wallet
    WalletLedger->>DB: Credit 40 ETB to Corporate Commission Account
    
    RideEngine-->>Driver: Settlement Complete
    RideEngine-->>Rider: Emit Receipt Notification
```

---

### 9. Activity Diagram: Driver Onboarding Workflow
Details the conditional branching involved in verifying a driver's legal application to join the fleet.

```mermaid
stateDiagram-v2
    [*] --> Start
    Start --> DriverRegisters : Basic PII
    DriverRegisters --> UploadsDocs : License & Car Details
    UploadsDocs --> SubmitApplication
    
    SubmitApplication --> FormatValidation_Check
    
    state FormatValidation_Check <<choice>>
    FormatValidation_Check --> PENDING_REVIEW : Valid
    FormatValidation_Check --> RejectedDocs : Invalid
    
    RejectedDocs --> [*] : Requires Re-upload
    
    PENDING_REVIEW --> DispatcherDownloads
    DispatcherDownloads --> DocsLegallyValid_Check
    
    state DocsLegallyValid_Check <<choice>>
    DocsLegallyValid_Check --> ApproveApplication : Valid
    DocsLegallyValid_Check --> RejectWithReason : Invalid
    
    RejectWithReason --> [*] : Notify Email/SMS
    
    ApproveApplication --> GenerateAuth
    GenerateAuth --> ACTIVE
    ACTIVE --> [*]
```

---

### 10. Activity Diagram: The Complete Ride Lifecycle
Outlines the logic from booking creation to completion or cancellation.

```mermaid
stateDiagram-v2
    [*] --> RiderRequestsEstimate
    RiderRequestsEstimate --> ConfirmBooking
    ConfirmBooking --> BroadcastToDrivers
    
    BroadcastToDrivers --> DriverAccepts_Check
    
    state DriverAccepts_Check <<choice>>
    DriverAccepts_Check --> ACCEPTED : Yes (Within 30s)
    DriverAccepts_Check --> TIMEOUT : No / Timeout
    
    TIMEOUT --> [*] : Notify "No Drivers Found"
    
    ACCEPTED --> DriverTravels
    DriverTravels --> DriverArrived
    DriverArrived --> RiderBoards
    RiderBoards --> ACTIVE_RIDE
    ACTIVE_RIDE --> ReachesDestination
    ReachesDestination --> EndRideTrigger
    EndRideTrigger --> SettleWallets
    SettleWallets --> [*] : Generate Receipt
```

---

### 11. State Machine Diagram: Ride Status
Tracks the exact lifecycle of the `booking_requests` database entity.

```mermaid
stateDiagram-v2
    [*] --> PENDING_ESTIMATE: Rider opens app
    PENDING_ESTIMATE --> SEARCHING: Request confirmed
    SEARCHING --> ACCEPTED: Driver accepts
    SEARCHING --> CANCELLED_TIMEOUT: No driver found
    ACCEPTED --> DRIVER_ARRIVED: Driver geofenced
    DRIVER_ARRIVED --> EN_ROUTE: Start trip
    EN_ROUTE --> COMPLETED: Destination reached
    
    ACCEPTED --> CANCELLED_BY_RIDER
    ACCEPTED --> CANCELLED_BY_DRIVER
    
    COMPLETED --> [*]
    CANCELLED_TIMEOUT --> [*]
    CANCELLED_BY_RIDER --> [*]
    CANCELLED_BY_DRIVER --> [*]
```

---

### 12. State Machine Diagram: Driver Operational Status
Shows how a driver toggles availability to the algorithmic dispatcher.

```mermaid
stateDiagram-v2
    [*] --> OFFLINE: Initial Login
    OFFLINE --> ONLINE: Toggle Dashboard
    ONLINE --> OFFLINE: Toggle Offline
    
    ONLINE --> ON_TRIP: Accepts a Ride
    ON_TRIP --> ONLINE: Completes Trip
    ON_TRIP --> OFFLINE: Force Disconnect
```

---

### 13. System Entity Relationship Diagram (Class Level)
Displays how relational tables map in the MySQL Database structure underlying the software requirements.

```mermaid
erDiagram
    USER ||--o{ WALLET : possesses
    USER ||--o{ BOOKING : requests
    DRIVER ||--o{ BOOKING : accepts
    USER ||--|{ DRIVER : "Role Morph"
    BOOKING ||--|| TRANSACTION : creates
    
    USER {
        bigint id
        string mobile
        string role "Rider/Driver/Admin"
    }
    
    DRIVER {
        bigint id
        int user_id
        string current_lat
        string current_lng
        bool is_online
    }
    
    BOOKING {
        bigint id
        int user_id
        int driver_id
        string status
        decimal final_fare
    }
    
    WALLET {
        bigint id
        int user_id
        decimal balance_etb
    }
    
    TRANSACTION {
        bigint id
        int wallet_id
        decimal amount
        string type "credit/debit"
    }
```

---

### 14. Component Diagram: System Modules (Micro-Level)
Illustrates the internal software component relationships within the monolithic core.

```mermaid
graph TD
    subgraph Mobile_Clients ["Mobile Clients"]
        RiderApp[Rider Vue/Flutter App]
        DriverApp[Driver Vue/Flutter App]
    end
    
    subgraph Laravel_Backend_Engine ["Laravel Backend Engine"]
        SanctumAPI[Sanctum API Router]
        RideEngine[Ride Matching Engine]
        LedgerLogic[Digital Ledger Logic]
        FleetControl[Fleet Config Control]
    end
    
    subgraph Infrastructure_Components ["Infrastructure Components"]
        RedisNode[(Redis Cluster)]
        MySQLNode[(Cloud MySQL Node)]
    end
    
    RiderApp --> SanctumAPI
    DriverApp --> SanctumAPI
    
    SanctumAPI --> RideEngine
    SanctumAPI --> LedgerLogic
    
    RideEngine --> RedisNode
    LedgerLogic --> MySQLNode
    FleetControl --> MySQLNode
```

---

### 15. Deployment Architecture Diagram
Demonstrates how the Mela Guzo platform fulfills the performance requirement of high availability through cloud architecture.

```mermaid
graph TD
    Internet((Public Internet))
    
    subgraph "VPC Zone (AWS/CloudProvider)"
        WAF[Cloudflare WAF Proxy]
        LB[Network Load Balancer]
        
        subgraph "Auto-Scaling App Layer"
            Node1[Nginx + PHP-FPM Node 1]
            Node2[Nginx + PHP-FPM Node 2]
            WSServer[Node.js WSS Socket Instance]
        end
        
        subgraph "Stateful Data Layer (Isolated)"
            DBPrimary[(MySQL Writer Node)]
            DBReplica[(MySQL Read-Replica)]
            Redis[(Redis Cache Endpoint)]
        end
    end
    
    Internet --> WAF
    WAF --> LB
    LB --> Node1
    LB --> Node2
    LB --> WSServer
    
    Node1 --> DBPrimary
    Node1 --> Redis
    Node2 --> DBPrimary
    Node2 --> Redis
    
    DBPrimary -.->|Asynchronous Sync| DBReplica
    WSServer <--> Redis
```

---

### 16. Communication Diagram (WebSocket Real-Time Tracking)
Maps the event-driven broadcast system responsible for updating driver cars moving across the map UI.

```mermaid
flowchart LR
    DriverApp([Driver GPS Chip]) -->|WSS Event Emit| SocketEngine[Node.js / Laravel WebSockets]
    SocketEngine -->|Publish to Channel 'Region-X'| Redis[Redis Pub/Sub]
    Redis -->|Notify Subscribers| EngineWorker[Broadcast Worker]
    EngineWorker -->|Stream Payload| RiderApp1([Nearby Rider 1])
    EngineWorker -->|Stream Payload| RiderApp2([Nearby Rider 2])
    
    subgraph "High Frequency Cycle (Every 5s)"
    SocketEngine
    Redis
    EngineWorker
    end
```

## 7. Threat Modeling (STRIDE)

### 7.1 STRIDE Threat Assessment Model
To ensure rigorous security compliance, the platform's core architecture has been analyzed against the STRIDE threat classification model (Spoofing, Tampering, Repudiation, Information Disclosure, Denial of Service, Elevation of Privilege).

The following diagram maps these theoretical vulnerability vectors against the primary system components and illustrates the enforced systemic mitigations.

```mermaid
graph TD
    classDef spoofing fill:#ff9999,stroke:#333,stroke-width:2px;
    classDef tampering fill:#ffcc99,stroke:#333,stroke-width:2px;
    classDef repudiation fill:#ffff99,stroke:#333,stroke-width:2px;
    classDef infodisc fill:#ccff99,stroke:#333,stroke-width:2px;
    classDef dos fill:#99ccff,stroke:#333,stroke-width:2px;
    classDef eop fill:#cc99ff,stroke:#333,stroke-width:2px;

    %% Components
    MobileApp(Mobile Client)
    API[API Gateway / WAF]
    AuthLayer[Auth Layer Sanctum]
    DB[(Core MySQL DB)]
    Ledger[Wallet Ledger]

    %% Threat Classifications Mapping
    Spoofing(Spoofing Identity):::spoofing
    Tampering(Tampering with Data):::tampering
    Repudiation(Repudiation):::repudiation
    InfoDisc(Information Disclosure):::infodisc
    DoS(Denial of Service):::dos
    EoP(Elevation of Privilege):::eop

    %% Link threats to vulnerable modules
    Spoofing -.->|Mitigated by OTP & MFA| MobileApp
    Spoofing -.->|Mitigated by Bearer Tokens| AuthLayer
    
    Tampering -.->|Mitigated by TLS 1.3 & WAF| API
    Tampering -.->|Mitigated by Prepared Statements| DB
    
    Repudiation -.->|Mitigated by Audit Logging| Ledger
    Repudiation -.->|Mitigated by App Logs| API
    
    InfoDisc -.->|Mitigated by Data Encryption| DB
    InfoDisc -.->|Mitigated by App Obfuscation| MobileApp
    
    DoS -.->|Mitigated by Cloudflare Shield| API
    DoS -.->|Mitigated by Rate Limiting| AuthLayer
    
    EoP -.->|Mitigated by Role Based Access| AuthLayer
    EoP -.->|Mitigated by Least Privilege Accounts| DB
```
