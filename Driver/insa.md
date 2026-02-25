
### 4.3 System Architecture Diagram
The architecture relies on high-speed event streams for dispatch operations.

```mermaid
graph TD
    Client[Driver Mobile App - Flutter]
    REST[Core Mela REST Backend - MySQL]
    FRT[Firebase Realtime Database - Live Location Pool]
    CF[Cloud Firestore - Trip State Machine]
    GMaps[Google Maps APIs - Routing & Display]
    OS[OneSignal - Push Notifications]

    Client -- HTTPS / JWT --> REST
    Client -- WebSocket / WSS --> FRT
    Client -- WebSocket / WSS --> CF
    Client -- Native SDK --> GMaps
    
    REST -- Broadcast Updates --> FRT
    REST -- Push Trigger --> OS
    OS -- APNS / FCM Payload --> Client
    REST -- State Mutations --> CF
```

### 4.4 Data Flow Diagram (DFD)
The core location tracking and backend synchronization flow is highlighted below:

```mermaid
flowchart TD
    Start[Driver Toggles ONLINE] --> PermCheck{Location Permissions?}
    PermCheck -- YES --> InitService[Spawn Background Isolate]
    InitService --> StickyNotif[Display Sticky OS Notification]
    StickyNotif --> GPSPoll(geolocator: Listen to Geospatial Stream)
    GPSPoll --> Filter{Filter Criteria:<br/>Moved > 15m OR Time > 10s}
    Filter -- YES --> PushToFRT[Write Lat/Lng & Geohash to Firebase RTDB]
    PushToFRT --> GPSPoll
    Stop[Driver Toggles OFFLINE] --> Terminate[Destroy Background Isolate]
    Terminate --> FinalWrite[Remove Driver Node in Firebase]
```

### 4.5 Driver Authentication & Onboarding Sequence
To ensure accountability and security, the application mandates a stringent OTP-based login flow.

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

### 4.6 Trip Lifecycle State Machine
Trips are strictly managed through a state machine, guaranteeing synchronization between the driver, rider, and backend.

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

### 4.7 KYC Document Verification Flow
Drivers must be verified before going online. The verification handles sensitive images and compresses them before transmission.

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
rstanding of the Mela Driver Application for the purposes of security validation. We welcome INSA's thorough analysis of our infrastructure, data pathways, and cryptographic implementations. We are highly committed to providing whatever further logs, access, or binaries are required to successfully certify the safety and privacy standards of our platform.
