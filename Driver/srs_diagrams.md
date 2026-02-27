ww# Mela Driver Application - SRS Diagrams

This document contains Mermaid diagrams that visually represent various aspects of the Software Requirements Specification (SRS) for the Mela Driver Application. Each diagram specifies the section of the SRS document where it should be referenced or inserted to provide visual context to the technical written requirements.

---

## 1. High-Level System Architecture
**Target SRS Section:** `6. System Architecture and Data Flow` -> `6.1 Technology Stack` & `6.2 Firebase Integration Architecture`

```mermaid
graph TD
    %% Node Definitions
    Client[Driver Mobile App<br/>Flutter]
    REST[Core Mela REST Backend<br/>Node.js/MySQL]
    FRT[Firebase Realtime Database<br/>Live Location Pool]
    CF[Cloud Firestore<br/>Trip State Machine]
    GMaps[Google Maps APIs<br/>Routing & Display]
    OS[OneSignal<br/>Push Notifications]

    %% Connections
    Client -- HTTPS / JWT --> REST
    Client -- WebSocket / WSS --> FRT
    Client -- WebSocket / WSS --> CF
    Client -- Native SDK --> GMaps
    
    REST -- Broadcast Updates --> FRT
    REST -- Push Trigger --> OS
    OS -- APNS / FCM Payload --> Client
    REST -- State Mutations --> CF
```

---

## 2. Driver Authentication & Onboarding Sequence
**Target SRS Section:** `3. System Features` -> `3.1 Driver Onboarding & Authentication`

```mermaid
sequenceDiagram
    actor Driver
    participant App as Mela Driver App
    participant REST as Mela Auth API
    participant SMS as SMS Gateway
    participant Hive as Local Storage (Hive)

    Driver->>App: Enters Phone Number (+251...)
    App->>REST: POST /send-mobile-otp (Phone)
    REST->>SMS: Request SMS dispatch
    SMS-->>Driver: SMS Delivery (OTP Code)
    REST-->>App: 200 OK (OTP Sent)
    
    Driver->>App: Inputs OTP
    alt is QA Test Number (+251940226005)
        App->>App: Auto-fill predefined OTP
    end
    App->>REST: POST /verify-mobile-otp (Phone, OTP)
    
    alt Verification Success
        REST-->>App: 200 OK + JWT Token + User Data
        App->>Hive: Securely serialize JWT & Configs
        App->>Driver: Navigate to Map Dashboard (or KYC Setup)
    else Verification Failed
        REST-->>App: 401 Unauthorized / 400 Bad Request
        App->>Driver: Display Form Validation / "Invalid OTP" Toast
    end
```

---

## 3. Trip Lifecycle State Machine
**Target SRS Section:** `3. System Features` -> `3.6 Trip State Management`

```mermaid
stateDiagram-v2
    [*] --> OFFLINE
    
    OFFLINE --> ONLINE : Driver toggles toggle-switch
    ONLINE --> OFFLINE : Driver toggles offline
    
    ONLINE --> BIDDING : Incoming Ride Payload Received
    
    state BIDDING {
        [*] --> Modal_Active
        Modal_Active --> ACCEPT : Driver Swipes Accept
        Modal_Active --> DECLINE : Driver Swipes Decline
        Modal_Active --> TIMEOUT : Countdown Expires
    }
    
    BIDDING --> ONLINE : DECLINE / TIMEOUT / LOST_BID
    BIDDING --> EN_ROUTE_TO_PICKUP : ACCEPT (Trip securely claimed)
    
    EN_ROUTE_TO_PICKUP --> ARRIVED_AT_PICKUP : Driver taps "ARRIVED"
    
    ARRIVED_AT_PICKUP --> TRIP_STARTED : Passenger verified / Picked up
    ARRIVED_AT_PICKUP --> CANCELLED : Driver or Rider cancels trip
    
    CANCELLED --> ONLINE
    
    TRIP_STARTED --> EN_ROUTE_TO_DROPOFF : Routing to destination
    
    EN_ROUTE_TO_DROPOFF --> TRIP_COMPLETED : Arrived at Drop-off
    
    TRIP_COMPLETED --> FARE_COLLECTION : Cash/Wallet settled
    FARE_COLLECTION --> RATING_MODULE : Rate passenger experience
    
    RATING_MODULE --> ONLINE : Evaluation complete, return to map
```

---

## 4. Background Telemetry (Location Tracking) Flow
**Target SRS Section:** `3. System Features` -> `3.9 Background Location Tracking Services`

```mermaid
flowchart TD
    Start[Driver Toggles ONLINE] --> PermCheck{Location Permissions<br/>Always Allow?}
    
    PermCheck -- NO --> ReqPerm[Prompt System Permission Request]
    ReqPerm --> PermCheck
    
    PermCheck -- YES --> InitService[Spawn Background Isolate<br/>via flutter_background_service]
    InitService --> StickyNotif[Display Sticky OS Notification<br/>Bypass OEM Process Killers]
    
    StickyNotif --> GPSPoll(geolocator: Listen to Geospatial Stream)
    
    GPSPoll --> Filter{Filter Criteria:<br/>Moved > 15m OR<br/>Time elapsed > 10s?}
    
    Filter -- NO --> GPSPoll
    Filter -- YES --> PushToFRT[w/ geoflutterfire_plus<br/>Write Lat/Lng & Geohash<br/>to Firebase RTDB]
    
    PushToFRT --> GPSPoll
    
    Stop[Driver Toggles OFFLINE or Session Terminated] --> Terminate[Destroy Background Isolate cleanly]
    Terminate --> FinalWrite[Remove Driver Node from Active Pool in Firebase]
```

---

## 5. KYC Document Verification Flow
**Target SRS Section:** `3. System Features` -> `3.2 Document Verification System`

```mermaid
flowchart LR
    Start(Unverified Driver Account) --> Cam[Image Picker: Activate Camera/Gallery]
    Cam --> Compress[flutter_image_compress<br/>Reduce payload to <70% JPEG quality]
    Compress --> Upload[Multipart Form Upload over HTTPS to REST API]
    
    Upload --> BackendProcessing[Admin Review / Automated OCR Pipeline]
    
    BackendProcessing -- Status: Pending --> UI_Pending[Dashboard Locked:<br/>'Waiting for Approval Overlay']
    BackendProcessing -- Status: Rejected --> UI_Rejected[Highlight specific document<br/>Prompt explicit re-upload]
    UI_Rejected --> Cam
    
    BackendProcessing -- Status: Approved --> UI_Unlocked[Enable Core Map Dashboard<br/>Unlock ONLINE toggle]
```
## 6. Threat Model
**Target SRS Section:** 
```mermaid
flowchart TD
    App((Mela Driver App))

    subgraph S ["Spoofing (Identity)"]
        S_Threat["Threat: Forged Driver Identity"]
        S_Mitigate["Mitigation: SMS OTP & Secure JWT"]
    end

    subgraph T ["Tampering (Data)"]
        T_Threat["Threat: Altered Location/Fare Data"]
        T_Mitigate["Mitigation: WSS & TLS 1.2+ Encryption"]
    end

    subgraph R ["Repudiation"]
        R_Threat["Threat: Denying Trip Acceptance"]
        R_Mitigate["Mitigation: Immutable Firebase Audit Logs"]
    end

    subgraph I ["Information Disclosure"]
        I_Threat["Threat: Leaked Rider/Driver PII"]
        I_Mitigate["Mitigation: Number Masking & Hive AES-256"]
    end

    subgraph D ["Denial of Service"]
        D_Threat["Threat: API/Dispatch Overload"]
        D_Mitigate["Mitigation: Client Timeouts & API Rate Limits"]
    end

    subgraph E ["Elevation of Privilege"]
        E_Threat["Threat: Driver Mutating Other Driver Data"]
        E_Mitigate["Mitigation: Strict Firebase Auth Rules"]
    end

    %% Relationships
    App -->|Defends Against| S
    App -->|Defends Against| T
    App -->|Defends Against| R
    App -->|Defends Against| I
    App -->|Defends Against| D
    App -->|Defends Against| E

    S_Threat --> S_Mitigate
    T_Threat --> T_Mitigate
    R_Threat --> R_Mitigate
    I_Threat --> I_Mitigate
    D_Threat --> D_Mitigate
    E_Threat --> E_Mitigate

```

---
