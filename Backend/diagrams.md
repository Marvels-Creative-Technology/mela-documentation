# Mela Guzo Web Application Diagrams

These comprehensive reference diagrams are constructed explicitly to satisfy the **5.1 Business Architecture and Design** requirements prescribed by the INSA Web Application Security Testing Assessment.

---

### 1. Data Flow Diagrams (DFD) 

#### 1.1 Level 0: Context Data Flow Diagram
This indicates the high-level boundary interaction of the Mela Guzo platform with external actants.

```mermaid
graph TD
    %% External Entities
    Rider([Rider Customer])
    Driver([Fleet Driver])
    Admin([Corporate Administrator])
    Chapa([Chapa Payment Gateway])
    Firebase([Firebase / FCM Endpoint])
    MapAPI([Google Maps / Routing])

    %% Main System Process
    System{{Mela Guzo Core Backend System}}

    %% Data Flows
    Rider <-->|Ride Request, Profile Updates, Payment Initiation| System
    Driver <-->|Live Telemetry, Status Toggles, Accept Rides| System
    Admin <-->|Configuration Changes, Audit Requests, Fleet Management| System
    System <-->|Financial Auth Tokens & Webhooks| Chapa
    System -->|Push Notifications Broadcast| Firebase
    System <-->|Distance Matrix Queries| MapAPI
```

#### 1.2 Level 1: Detailed Data Flow Diagram
This elaborates on the internal data transformations occurring within the `Mela Guzo Core Backend System`.

```mermaid
graph TD
    %% Components
    MobileApp(Mobile Client / Web Browser)
    WAF[Cloudflare WAF Perimeter]
    AuthLayer[Sanctum Authentication Logic]
    RoutingEngine[Ride Matching & Routing Engine]
    LedgerWorker[Wallet Transaction Event Bus]
    Database[(MySQL Master Database)]
    Queue[(Redis Action Queues)]

    %% Flows
    MobileApp -->|1. Transmit Encrypted Payload HTTPS| WAF
    WAF -->|2. Scrutinized API Request| AuthLayer
    AuthLayer -->|3. Validate Bearer Token & RBAC| Database
    Database -.->|4. Token Valid/Invalid Response| AuthLayer
    
    AuthLayer -->|5. Authorized Request| RoutingEngine
    RoutingEngine -->|6. Query Pricing & Geography| Database
    RoutingEngine -->|7. Push Background Calculation| Queue
    
    AuthLayer -->|5b. Payment Authorized| LedgerWorker
    LedgerWorker -->|8. Record Financial State| Database
    LedgerWorker -->|9. Dispatch Receipt Event| Queue
```

#### 1.3 Level 2: Authentication & Token Lifecycle Processing
A deep-dive sequence DFD illustrating how the system verifies user identity via OTP and generates the dual-layered authentication tokens (Sanctum + Session Keys).

```mermaid
sequenceDiagram
    autonumber
    actor Mobile Client
    participant API Gateway (WAF)
    participant Mela Auth Controller
    participant OTP Dispatch Provider
    participant MySQL Database

    Mobile Client->>API Gateway (WAF): POST /api/v1/sendMobileLoginOtp {phone}
    API Gateway (WAF)->>Mela Auth Controller: Proxied Request (TLS 1.3)
    Mela Auth Controller->>MySQL Database: Generate & Insert bcrypt encrypted OTP
    Mela Auth Controller->>OTP Dispatch Provider: Dispatch SMS String
    OTP Dispatch Provider-->>Mobile Client: Receive SMS
    
    Mobile Client->>API Gateway (WAF): POST /api/v1/userMobileLogin {phone, otp_value}
    API Gateway (WAF)->>Mela Auth Controller: Proxied Request
    Mela Auth Controller->>MySQL Database: Lookup & Evaluate bcrypt string
    MySQL Database-->>Mela Auth Controller: OTP Validated
    Mela Auth Controller->>MySQL Database: Flag OTP as consumed, Generate 120-char Session Token
    Mela Auth Controller->>Mela Auth Controller: Generate Sanctum Bearer Token
    Mela Auth Controller-->>Mobile Client: Return HTTP 200 {Bearer Token, Session Key}
```

#### 1.4 Level 2: Secure Ride Booking & Wallet Settlement Flow
A sequence flow mapping the complex interactions when dispatching a vehicle and locking financial states securely into the ledger.

```mermaid
sequenceDiagram
    autonumber
    actor Rider
    actor Driver
    participant Ride Aggregator Logic
    participant Distance Matrix Module
    participant Wallet Ledger Engine
    participant App Database

    Rider->>Ride Aggregator Logic: POST /api/v1/bookItem {coords, token}
    Ride Aggregator Logic->>Distance Matrix Module: Query exact road distance/duration
    Distance Matrix Module-->>Ride Aggregator Logic: Return {distance: 12km, time: 25m}
    Ride Aggregator Logic->>App Database: INSERT booking_requests {status: pending, fare: calculated}
    
    Ride Aggregator Logic->>Driver: Broadcast Push Notification via FCM
    Driver->>Ride Aggregator Logic: POST /api/v1/driverAcceptBooking
    Ride Aggregator Logic->>App Database: UPDATE booking {driver_id: X, status: active}
    
    Driver->>Ride Aggregator Logic: Ride Completed Event
    Ride Aggregator Logic->>Wallet Ledger Engine: Trigger Settlement Procedure
    Wallet Ledger Engine->>App Database: Verify Rider Wallet Balance
    Wallet Ledger Engine->>App Database: INSERT wallet_transactions {Debit Rider, Credit Driver}
    Wallet Ledger Engine-->>Rider: Return Digital Receipt
```

---

### 2. System Architecture Diagrams

#### 2.1 Ecosystem Architecture Diagram (Global View)
This blueprint maps the overall platform, emphasizing the security perimeter, application tiers, and third-party interactions.

```mermaid
graph TD
    subgraph Client Tier
        RiderApp[Mela Rider Mobile App]
        DriverApp[Mela Driver Mobile App]
        AdminPortal[Administrator Web Portal]
    end

    subgraph CDN Edge & DMZ
        CDN[Cloudflare CDN & Edge Routing]
        WAF[Cloudflare Web App Firewall]
        Nginx[Nginx SSL Terminator & Load Balancer]
    end

    subgraph Internal Application Tier
        LaravelEngine[Laravel PHP 8.1 API]
        NodeSockets[Node.js Action Streamer / WSS]
        Sanctum[Sanctum Token Gatekeeper]
    end

    subgraph Isolated Data Tier
        DB_Master[(MySQL Master Database)]
        RedisInstance[(Redis In-Memory Cache)]
    end

    %% Network Connections
    RiderApp -->|HTTPS| CDN
    DriverApp -->|HTTPS| CDN
    AdminPortal -->|HTTPS| CDN

    CDN --> WAF
    WAF --> Nginx
    Nginx --> LaravelEngine
    Nginx --> NodeSockets

    LaravelEngine --> Sanctum
    Sanctum --> DB_Master
    LaravelEngine --> DB_Master
    LaravelEngine --> RedisInstance
    NodeSockets <--> RedisInstance
```

#### 2.2 Deployment Architecture Topology (Network Zoning)
This visualizes the physical network hosting paradigm, showcasing how servers are isolated via Virtual Private Clouds (VPC) and Subnets.

```mermaid
graph TB
    subgraph "External Web / Internet"
        Internet((Public Traffic))
    end

    subgraph "Cloud Provider Environment (e.g. AWS/DigitalOcean)"
        subgraph "VPC (Virtual Private Cloud - 10.0.0.0/16)"
            subgraph "Public Subnet (DMZ - 10.0.1.0/24)"
                NLB[Network Load Balancer]
                WebNode1[Nginx Reverse Proxy 1]
                WebNode2[Nginx Reverse Proxy 2]
            end

            subgraph "Private App Subnet (10.0.2.0/24) - NO DIRECT INTERNET"
                AppNode1[PHP-FPM Worker 1]
                AppNode2[PHP-FPM Worker 2]
                SocketNode[NodeJS Socket Handler]
            end

            subgraph "Private DB Subnet (10.0.3.0/24) - RESTRICTED PORT ACCESS"
                MySQL_Primary[(MySQL 8 Primary)]
                Redis_Cluster[(Redis Cluster)]
            end
        end
    end

    Internet -->|HTTPS :443| NLB
    NLB -->|Proxy| WebNode1
    NLB -->|Proxy| WebNode2
    WebNode1 -->|Local :9000| AppNode1
    WebNode2 -->|Local :9000| AppNode2
    WebNode1 -->|Local WS| SocketNode
    
    AppNode1 -->|Port 3306| MySQL_Primary
    AppNode2 -->|Port 3306| MySQL_Primary
    AppNode1 -->|Port 6379| Redis_Cluster
    SocketNode -->|Port 6379| Redis_Cluster
```

#### 2.3 Application Component Architecture (Module Isolation)
Highlights the micro-modular approach inside the monolithic API structure, describing how distinct logic blocks interface internally.

```mermaid
graph LR
    subgraph "Mela Guzo API Application Kernel"
        API_Route[API Router & Sanitizer]
        AuthModule[[Authentication & Session Module]]
        FleetModule[[Fleet & Dispatch Module]]
        PricingModule[[Dynamic Pricing Module]]
        WalletModule[[Ledger & Wallet Module]]
        NotifyModule[[Push & Event Module]]

        API_Route --> AuthModule
        AuthModule --> FleetModule
        AuthModule --> WalletModule
        
        FleetModule --> PricingModule
        FleetModule --> NotifyModule
        WalletModule --> NotifyModule
    end

    subgraph "External Hooks"
        Chapa[Chapa SDK]
        FCM[Firebase FCM]
    end

    WalletModule -.->|Webhooks| Chapa
    NotifyModule -.->|Socket Push| FCM
```

#### 2.4 Security Layers & Threat Mitigation Zones
A structural representation mapping specific security standards and protocols applied across different system OSI layers.

```mermaid
graph TD
    subgraph "Layer 7: Edge Perimeter Defense"
        DDoS[Unmetered DDoS Mitigation]
        GeoBlock[Geo IP Rate Limiting]
        WAF_Rules[OWASP Top 10 Injection Blocking]
    end

    subgraph "Layer 4-6: Network & Transport Sec"
        TLS_Drop[Strict Transport Security HTTPS Only]
        UFW[Ubuntu Firewall - Custom Port Safelisting]
    end

    subgraph "Layer 7: Application Code Security"
        CSRF[CSRF Token Validation]
        Bcrypt[Bcrypt Password Hashing]
        ORM[Prepared Statement SQL Mitigations]
        XSS[Blade Entity Escaping]
    end

    DDoS --> GeoBlock
    GeoBlock --> WAF_Rules
    WAF_Rules --> TLS_Drop
    TLS_Drop --> UFW
    UFW --> CSRF
    CSRF --> Bcrypt
    Bcrypt --> ORM
    ORM --> XSS
```

---

### 3. Entity Relationship Diagram (ERD) - Core Security Models

#### 3.1 ERD: Identifying PII & Transaction Keys
This diagram specifically highlights fields scrutinized for security, PII isolation, and financial integrity. 

```mermaid
erDiagram
    app_users ||--o{ app_user_otps : validates
    app_users ||--o{ booking_requests : initiates
    app_users ||--o{ wallet_transactions : executes
    fleet_drivers ||--o{ booking_requests : fulfills
    app_users ||--|{ fleet_drivers : "1-to-1 Profile Mapping"
    
    app_users {
        bigint id PK
        string mobile_number "Sanitized Regex Validated"
        string email_address "Encrypted at Rest"
        string role "SuperAdmin, Dispatcher, Rider, Driver"
        string sanctum_session_hash "120-Char Validation Key"
        boolean is_banned 
        timestamp created_at
    }

    app_user_otps {
        bigint id PK
        bigint app_user_id FK
        string raw_otp_hash "Bcrypt Encrypted OTP"
        timestamp expires_at "Temporal Session Window"
        boolean is_consumed
    }

    fleet_drivers {
        bigint id PK
        bigint app_user_id FK
        string license_image_path "Secured AWS/Local Storage Link"
        float live_latitude "Real-time Update"
        float live_longitude "Real-time Update"
        boolean is_approved "Requires RBAC Admin Signature"
    }

    booking_requests {
        bigint id PK
        bigint rider_id FK
        bigint driver_id FK "Nullable until dispatch resolution"
        string route_polyline "Encoded Geopoints"
        decimal calculated_fare "Locked via Server Distance Matrix"
        string security_sos_status "Active/None"
        string journey_status "Pending, Active, Completed, Cancelled"
    }

    wallet_transactions {
        bigint id PK
        bigint booking_id FK "Nullable for Top-ups"
        bigint user_id FK
        decimal immutable_amount
        string gateway_reference "Chapa verification signature"
        string transaction_type "Credit, Debit"
        timestamp executed_at
    }
```
