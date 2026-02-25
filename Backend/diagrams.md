# Mela Guzo Web Application Diagrams

These comprehensive reference diagrams are constructed explicitly to satisfy the **5.1 Business Architecture and Design** requirements prescribed by the INSA Web Application Security Testing Assessment.

---

### 1. Data Flow Diagrams (DFD) 
*(Referenced within Section 5.1.1)*

#### Level 0: Context Data Flow Diagram
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

#### Level 1: Detailed Data Flow Diagram
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

---

### 2. System Architecture Diagram
*(Referenced within Section 5.1.2)*

This blueprint maps the network topology, emphasizing the security perimeter, application tiers, and third-party interactions.

```mermaid
graph TD
    subgraph Public Internet Zone
        RiderApp[Mela Rider Mobile App]
        DriverApp[Mela Driver Mobile App]
        AdminPortal[Administrator Web Portal]
    end

    subgraph Perimeter Security Network & DMZ
        CDN[Cloudflare CDN & Edge Routing]
        WAF[Cloudflare Web App Firewall]
        Nginx[Nginx SSL Terminator & Load Balancer]
        UFW[Ubuntu UFW Port Restrictions]
    end

    subgraph Private Application Tier
        LaravelEngine[Laravel PHP 8.1 REST API]
        NodeSockets[Node.js Action Streamer / Websockets]
        SpatieAuth[RBAC Validation Middleware]
        Sanctum[Sanctum Token Gatekeeper]
    end

    subgraph Internal Isolated Data Tier
        DB_Master[(MySQL 8.x Master Database)]
        RedisInstance[(Redis In-Memory Cache/Queue)]
    end

    subgraph External Financial & Telemetry Partners
        FCMApi[Firebase Messaging]
        ChapaSDK[Chapa Financial Processors]
        GoogleMaps[Distance Matrix & Places API]
    end

    %% Network Connections
    RiderApp -->|TLS 1.2+ Encrypted JSON| CDN
    DriverApp -->|TLS 1.2+ Encrypted JSON| CDN
    AdminPortal -->|TLS 1.2+ Web Session| CDN

    CDN --> WAF
    WAF -->|DDoS Cleansed Traffic| UFW
    UFW --> Nginx

    Nginx -->|Proxy Pass 80/443| LaravelEngine
    Nginx -->|WSS Tunnel| NodeSockets

    LaravelEngine --> SpatieAuth
    LaravelEngine --> Sanctum
    SpatieAuth --> DB_Master
    Sanctum --> DB_Master

    LaravelEngine -->|Eloquent Queries| DB_Master
    LaravelEngine -->|Publish Events| RedisInstance
    NodeSockets <-->|Subscribe To Stream| RedisInstance

    LaravelEngine -->|cURL Payload| FCMApi
    LaravelEngine -->|Signed JWT/Webhook Auth| ChapaSDK
    LaravelEngine -->|Geospatial Queries| GoogleMaps
```

---

### 3. Entity Relationship Diagram (ERD) - Core Security Models
*(Referenced within Section 5.1.3)*

This ERD specifically highlights fields scrutinized for security, PII isolation, and financial integrity. 

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
