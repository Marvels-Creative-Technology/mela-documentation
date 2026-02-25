# Mela Rider Application - Cybersecurity Audit Diagrams

As requested for the INSA Mobile Application Security Testing Requirements Document (Document No: OF/AEAD/001), the following diagrams map out the core architecture, data flow, structure, and threat models of the "Mela for Mekedonia" Rider Application.

---

## 1. Business Architecture and Design Diagram (Ecosystem)
This diagram illustrates the high-level business ecosystem, showing how the Rider App interacts with other components of the platform like the Driver App, Backend Server, Firebase, and third-party APIs.

```mermaid
graph TD
    A[Mela Rider App] <-->|Secure HTTPS / WSS| B(Firebase Services)
    C[Mela Driver App] <-->|Secure HTTPS / WSS| B
    A <-->|REST API| D{Mela Backend System}
    C <-->|REST API| D
    E[Admin Dashboard] <-->|REST API| D
    
    D <-->|CRUD Operations| F[(Relational Database)]
    D <-->|Transactions| G[Payment Gateways]
    D <-->|Routing & Maps| H[Google Maps APIs]
    
    B -->|Authentication| I(Firebase Auth)
    B -->|Live Sync| J(Firebase Realtime DB)
    B -->|Notifications| K(Firebase Cloud Messaging)
```

---

## 2. Data Flow Diagram
This shows the logical flow of data from the initial user request to backend processing, matchmaking via real-time databases, and payment confirmation.

```mermaid
flowchart TD
    subgraph Rider Interface
        R[Rider] -->|1. Request Ride| RA(Rider App)
        RA -->|2. Get Location & ETA| MAP[Google Maps API]
    end
    
    subgraph Backend Infrastructure
        RA -->|3. Send Ride Data| API[REST API / Backend Server]
        API -->|4. Save Ride Intent| DB[(SQL Database)]
    end
    
    subgraph Real-Time Dispatch Engine
        API -->|5. Match & Dispatch Request| FB[Firebase RTDB]
        FB <-->|6. Broadcast Driver Location| DA(Driver App)
        DA -->|7. Accept Ride| FB
        FB -->|8. Notify Match Success| RA
    end

    subgraph Payment Processing
        RA -->|9. Finalize Payment| PG[Payment Gateway WebView]
        PG -->|10. Callback Verification| API
        API -->|11. Update Status| DB
    end
```

---

## 3. System Architecture Diagram with Database Relation
This section covers both the topological system architecture layout and the ERD (Entity Relationship Diagram) of the core underlying database models.

### System Infrastructure
```mermaid
graph TB
    subgraph Mobile Clients [Frontend]
        R[Flutter Rider App]
        D[Flutter Driver App]
    end
    
    subgraph Third-Party Services
        F[Firebase Auth / FCM / RTDB]
        M[Google Maps API]
        P[Payment Gatways]
    end
    
    subgraph Server Infrastructure [Backend]
        LB[Load Balancer / Proxy]
        API[Core REST API Server]
        
        subgraph Database Layer
            SQL[(Primary SQL Database)]
            Redis[(Redis Cache)]
        end
    end
    
    R -- TLS 1.2+ --> LB
    D -- TLS 1.2+ --> LB
    R -- SDK/WSS --> F
    D -- SDK/WSS --> F
    R -- HTTPS --> M
    
    LB --> API
    API --> SQL
    API --> Redis
    API --> P
    API --> F
```

### Database Entity Relationship Diagram
```mermaid
erDiagram
    Rider ||--o{ Booking : creates
    Driver ||--o{ Booking : accepts
    Booking ||--|| Payment : triggers
    Driver }|--|| Vehicle : drives
    
    Rider {
        uuid id PK
        string phone_number
        string name
        string email
        float wallet_balance
    }
    Driver {
        uuid id PK
        string phone_number
        string name
        boolean is_active
        float rating
    }
    Vehicle {
        uuid id PK
        string license_plate
        string category
    }
    Booking {
        uuid id PK
        uuid rider_id FK
        uuid driver_id FK
        string pickup_location
        string dropoff_location
        float fare_estimated
        string status
        timestamp created_at
    }
    Payment {
        uuid id PK
        uuid booking_id FK
        float amount
        string method
        string status
    }
```

---

## 4. Threat Model Mapping
Using the STRIDE framework, this highlights primary attack vectors against the mobile application endpoints and their corresponding server-side & client-side mitigations.

```mermaid
flowchart TD
    A[Malicious Actor] -->|Spoofing Auth Tokens| Auth(Firebase Auth / Custom API)
    A -->|Tampering with Payload| Network(HTTP Transit / WebSockets)
    A -->|Repudiation of Transactions| Logs(Booking / Transactions DB)
    A -->|Information Disclosure| LocalStorage(Hive DB / Cached Data)
    A -->|Denial of Service/Spam| API(REST API Framework)
    A -->|Elevation of Privilege| AppRoute(Parameter Manipulation)
    
    Auth -->|MITIGATION: Require OTP & MFA | Mit1[Secure Identity Verification]
    Network -->|MITIGATION: TLS 1.2+ & Pinning | Mit2[Encrypted Transit Pipes]
    Logs -->|MITIGATION: Immutable Ledgers | Mit3[Server-side Audit Logging]
    LocalStorage -->|MITIGATION: Keystore / keychain encryption | Mit4[Encrypted Hive Boxes & Obfuscated Code]
    API -->|MITIGATION: Rate Limiting & WAF | Mit5[Cloudflare & Request Throttling]
    AppRoute -->|MITIGATION: Strict Server-Side Validation | Mit6[No Client-Side Authorization Trust]
```

---

## 5. Role / System Actor Relationship
Defines the boundary relationships and use cases accessible to the differing classes of authenticated entities within the system.

```mermaid
graph LR
    subgraph Users & Entities
        R((Rider App User))
        D((Driver App User))
        SA((System Administrator))
    end
    
    subgraph Platform Actions
        R_Req[Request Ride & Live Track]
        R_Pay[Initiate Gateway Payment]
        R_SOS[Trigger Emergency SOS]
        
        D_Acc[Accept Pending Rides]
        D_Nav[GPS Navigation Integration]
        D_Earn[Manage Wallet & Earnings]
        
        SA_User[Verify Driver KYC Documents]
        SA_Fare[Manipulate Active Fare Logic]
        SA_Audit[Review Trip & System Audit Logs]
    end
    
    R --> R_Req
    R --> R_Pay
    R --> R_SOS
    
    D --> D_Acc
    D --> D_Nav
    D --> D_Earn
    
    SA --> SA_User
    SA --> SA_Fare
    SA --> SA_Audit
    
    subgraph Mela API Backend
        API[API Orchestrator / Enforcer]
    end
    
    R_Req --> API
    R_Pay --> API
    R_SOS --> API
    D_Acc --> API
    D_Nav --> API
    D_Earn --> API
    SA_User --> API
    SA_Fare --> API
    SA_Audit --> API
```
