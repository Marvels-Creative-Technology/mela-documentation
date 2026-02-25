# Mermaid Diagrams for Mela Rider SRS

> These diagrams are formatted in Mermaid syntax. They correspond to the Software Requirements Specification (SRS) document (`Mela_Rider_SRS.md`). Each diagram includes a reference to its intended placement within the SRS document headers.

---

## 1. High-Level Data Flow Diagram
**(Belongs in SRS Section 1.4: Product Scope)**

```mermaid
flowchart TB
    subgraph Client["📱 Rider App (Flutter — Android / iOS)"]
        UI["Presentation Layer\n(Screens & Cubits)"]
        Domain["Domain Layer\n(Entities / Models)"]
        Data["Data Layer\n(Repositories & HTTP Client)"]
        LocalStore["Local Storage\n(Hive: appBox, lanBox)"]
        UI --> Domain --> Data
        Data --> LocalStore
    end

    subgraph Backend["🖥️ Backend Server"]
        API["REST API\n/api/v1/*"]
        DB["Server Database\n(Relational DB)"]
        API --> DB
    end

    subgraph Firebase["🔥 Firebase Platform"]
        FAuth["Firebase Auth\n(OTP Verification)"]
        FRTDB["Realtime Database\n(Ride Requests,\nDriver Locations)"]
        FFirestore["Cloud Firestore\n(Supplementary Data)"]
        FFCM["Firebase Cloud\nMessaging (FCM)"]
    end

    subgraph GoogleSvc["🌐 Google Services"]
        GMaps["Google Maps SDK\n(Map Rendering)"]
        GPlaces["Google Places API\n(Address Search)"]
        GGeo["Geocoding API\n(Reverse Geocoding)"]
        GDir["Directions API\n(Routes & Polylines)"]
        GSignIn["Google Sign-In\n(OAuth 2.0)"]
    end

    subgraph External["🔗 External Services"]
        OneSignal["OneSignal\n(Push Notifications)"]
        AppleSignIn["Apple Sign-In\n(OAuth 2.0)"]
        PayGW["Payment Gateway\n(WebView)"]
    end

    Data -- "HTTPS\n(JSON + Bearer Token)" --> API
    Data -- "WebSocket\n(Real-time)" --> FRTDB
    Data -- "HTTPS" --> FFirestore
    Data -- "SDK" --> FAuth
    UI -- "SDK" --> GMaps
    UI -- "SDK" --> GPlaces
    Data -- "SDK" --> GGeo
    Data -- "SDK" --> GDir
    Data -- "OAuth 2.0" --> GSignIn
    Data -- "OAuth 2.0" --> AppleSignIn
    Data -- "SDK" --> OneSignal
    FFCM -- "Push Notifications" --> Client
    OneSignal -- "Push Notifications" --> Client
    UI -- "WebView\n(HTTPS)" --> PayGW
```

---

## 2. Business Flow Diagram
**(Belongs in SRS Section 2.1: Product Perspective)**

```mermaid
flowchart LR
    A["🚀 App Launch"] --> B{"Logged In?"}
    B -- "Yes" --> D["🏠 Home Screen"]
    B -- "No" --> C["🔐 Login / Register"]
    C -->|"Phone OTP\nor Google\nor Apple"| D

    D --> E["🔍 Search\nDestination"]
    E --> F["📍 Select\nPickup & Dropoff"]
    F --> G["🚗 Select\nVehicle Category"]
    G --> H["💰 View\nFare Estimate"]
    H --> I["📤 Send\nRide Request"]
    I --> J{"Driver\nAccepts?"}
    J -- "Yes" --> K["📡 Live\nTracking"]
    J -- "No / Timeout" --> I
    K --> L["✅ Ride\nCompleted"]
    L --> M["💳 Payment"]
    M --> N["⭐ Rating\n& Review"]
    N --> D

    D --> O["📜 Ride History"]
    D --> P["👤 Profile\nManagement"]
    D --> Q["🎫 Referral\nSystem"]
    D --> R["🆘 SOS /\nEmergency"]
    D --> S["⚙️ Settings &\nLanguage"]
```

---

## 3. System Architecture Diagram
**(Belongs in SRS Section 3.3: Software Interfaces)**

```mermaid
flowchart TB
    subgraph PresentationLayer["🎨 PRESENTATION LAYER"]
        direction TB
        subgraph Screens["Screens"]
            S1["Splash / Initial"]
            S2["Onboarding"]
            S3["Login"]
            S4["Signup"]
            S5["OTP Verification"]
            S6["Home"]
            S7["Search Map"]
            S8["Route Location"]
            S9["Vehicle Selection"]
            S10["Send Ride Request"]
            S11["Nearby Drivers Loading"]
            S12["Payment"]
            S13["Payment Gateway\n(WebView)"]
            S14["Payment Success"]
            S15["Ride History"]
            S16["Account / Profile"]
            S17["Email / Phone\nUpdate"]
        end
        subgraph Cubits["Cubits (BLoC State Management)"]
            C1["AuthLoginCubit"]
            C2["AuthSignUpCubit"]
            C3["AuthOtpVerifyCubit"]
            C4["GoogleLoginCubit"]
            C5["AppleLoginCubit"]
            C6["BookRideRealTimeDBCubit"]
            C7["RideRequestCubit"]
            C8["GetRideRequestStatusCubit"]
            C9["GetVehicleDataCubit"]
            C10["GetItemPriceCubit"]
            C11["LocationUserCubit"]
            C12["PaymentCubit"]
            C13["HistoryCubit"]
            C14["ReviewCubit"]
            C15["ReferralCubit"]
            C16["GeneralCubit"]
            C17["LanguageCubit"]
            C18["BannerCubit"]
            C19["LogoutCubit"]
            C20["SosCubit"]
        end
        subgraph Widgets["Reusable Widgets"]
            W1["Common Widgets\n(common_widget.dart)"]
        end
    end

    subgraph DomainLayer["📦 DOMAIN LAYER"]
        subgraph Entities["Entities (Data Models)"]
            E1["LoginModel"]
            E2["CategoryModel"]
            E3["RideRequestModel"]
            E4["BookingSuccess"]
            E5["DriverData"]
            E6["HistoryData"]
            E7["GeneralData"]
            E8["GetItemPrice"]
            E9["BannerModel"]
            E10["ReferralModel"]
            E11["ReferralDiscount"]
            E12["SOSData"]
            E13["CheckEmail"]
            E14["CheckMobile"]
            E15["StaticData"]
        end
    end

    subgraph DataLayer["💾 DATA LAYER"]
        subgraph Repos["Repositories"]
            R1["AuthRepository"]
            R2["VehicleRepository"]
            R3["PaymentRepository"]
            R4["ProfileRepository"]
            R5["HistoryRepository"]
            R6["ReviewRepository"]
            R7["ReferralRepository"]
            R8["BannerRepository"]
        end
        subgraph Core["Core Services"]
            CS1["HTTP Client\n(http.dart)"]
            CS2["Config\n(config.dart)"]
            CS3["DataStore\n(Hive)"]
            CS4["Push Notifications\n(OneSignal + FCM)"]
        end
    end

    Screens --> Cubits
    Cubits --> Entities
    Cubits --> Repos
    Repos --> Core
```

---

## 4. Database Entity Relationship Diagram
**(Belongs in SRS Section 5.2: Server Database & API Mapping)**

```mermaid
erDiagram
    USERS {
        int id PK
        string first_name
        string last_name
        string email
        string phone
        string phone_country
        string default_country
        string gender
        string birthdate
        json profile_image
        json identity_image
        string token
        string otp_value
        string reset_token
        float wallet
        string login_type
        string social_id
        string fcm
        string device_id
        string verified
        string status
        string avr_guest_rate
        int remaining_items
        datetime created_at
        datetime updated_at
    }

    BOOKINGS {
        int id PK
        string ride_id
        int user_id FK
        int Driver_id FK
        int item_type_id FK
        string item_id
        string pickup_latitude
        string pickup_longitude
        string dropoff_latitude
        string dropoff_longitude
        string pickup_address
        string dropoff_address
        string estimated_distance_km
        string estimated_duration_min
        string amount_to_pay
        string payment_method
        string currency_code
        string coupon_code
        string coupon_discount
        string discount_price
        string service_charge
        string wallet_amount
        string status
        string ride_date
        datetime created_at
        datetime updated_at
    }

    DriverS {
        int id PK
        string name
        string phone
        string email
        json vehicle_info
        float latitude
        float longitude
        float heading
        string status
        float rating
        datetime created_at
        datetime updated_at
    }

    VEHICLE_TYPES {
        int id PK
        string name
        string description
        float base_fare
        float per_km_rate
        string image
        string status
        datetime created_at
    }

    REVIEWS {
        int id PK
        int booking_id FK
        int user_id FK
        int Driver_id FK
        string rating
        string message
        datetime created_at
    }

    PAYMENTS {
        int id PK
        int booking_id FK
        string payment_method
        float amount
        string status
        string transaction_ref
        datetime created_at
    }

    REFERRALS {
        int id PK
        int user_id FK
        string referral_code
        int referred_count
        float discount_amount
        string status
        datetime created_at
    }

    SOS_CONTACTS {
        int id PK
        string name
        string phone
        string relationship
        string status
        datetime created_at
    }

    STATIC_PAGES {
        int id PK
        string title
        text content
        string type
        string status
        datetime created_at
        datetime updated_at
    }

    BANNERS {
        int id PK
        string title
        string image
        string link
        string status
        int sort_order
        datetime created_at
    }

    USERS ||--o{ BOOKINGS : "places"
    DriverS ||--o{ BOOKINGS : "fulfills"
    VEHICLE_TYPES ||--o{ BOOKINGS : "categorizes"
    BOOKINGS ||--o| REVIEWS : "receives"
    BOOKINGS ||--o| PAYMENTS : "has"
    USERS ||--o{ REFERRALS : "creates"
    USERS ||--o{ REVIEWS : "submits"
```

---

## 5. Authentication Flow Diagram
**(Belongs in SRS Section 4.1: Authentication & Authorization)**

```mermaid
sequenceDiagram
    actor User as 👤 Rider
    participant App as 📱 Rider App
    participant Backend as 🖥️ Backend API
    participant Firebase as 🔥 Firebase
    participant Google as 🌐 Google
    participant Apple as 🍎 Apple

    Note over User,Apple: === Phone OTP Login ===
    User->>App: Enter phone number (+251)
    App->>Backend: POST /sendMobileLoginOtp<br/>{phone, phone_country}
    Backend-->>App: OTP sent confirmation
    User->>App: Enter OTP code
    App->>Backend: POST /userMobileLogin<br/>{phone, phone_country, otp_value}
    Backend-->>App: {user_data, token}
    App->>App: Store token in Hive (appBox)
    App->>Backend: POST /api/generateToken<br/>{secret, user_token}
    Backend-->>App: {bearer_token}
    App->>App: Store bearer token in Hive

    Note over User,Apple: === Google Sign-In ===
    User->>App: Tap "Sign in with Google"
    App->>Google: OAuth 2.0 Request
    Google-->>App: {displayName, email, id, profileImage}
    App->>Backend: POST /socialLogin<br/>{displayName, email, id,<br/>profile_image, login_type: "google"}
    Backend-->>App: {user_data, token}

    Note over User,Apple: === Apple Sign-In (iOS) ===
    User->>App: Tap "Sign in with Apple"
    App->>Apple: Authorization Request
    Apple-->>App: {identityToken, authorizationCode,<br/>email, displayName}
    App->>Backend: POST /socialLogin<br/>{identityToken, authorizationCode,<br/>login_type: "apple"}
    Backend-->>App: {user_data, token}

    Note over User,Apple: === Token Refresh ===
    App->>Backend: Any API call (Bearer token expired)
    Backend-->>App: HTTP 498 (Token Expired)
    App->>Backend: POST /api/generateToken<br/>{secret, user_token}
    Backend-->>App: {new_bearer_token}
    App->>App: Update bearer token in Hive
    App->>Backend: Retry original API call

    Note over User,Apple: === Session Expiration ===
    App->>Backend: Any API call
    Backend-->>App: HTTP 401 / 419 (Unauthorized)
    App->>App: Clear Hive data
    App->>User: Redirect to Login Screen
```

---

## 6. Ride Booking Data Flow (Detailed)
**(Belongs in SRS Section 4.2: Ride Booking & Dispatch)**

```mermaid
sequenceDiagram
    actor Rider as 👤 Rider
    participant App as 📱 Rider App
    participant Maps as 🗺️ Google Maps
    participant Backend as 🖥️ Backend API
    participant FRTDB as 🔥 Firebase RTDB
    participant PayGW as 💳 Payment Gateway

    Rider->>App: Open app
    App->>App: getUserLocation() — GPS
    App->>Maps: Render map at current location

    Rider->>App: Search destination
    App->>Maps: Places API autocomplete
    Maps-->>App: Place suggestions
    Rider->>App: Select destination

    App->>Maps: Directions API<br/>(pickup → dropoff)
    Maps-->>App: Route polyline + distance

    App->>Backend: GET /getAllCategories
    Backend-->>App: Vehicle categories list
    Rider->>App: Select vehicle type

    App->>Backend: POST /getItemPrices<br/>{item_type_id, distance}
    Backend-->>App: Fare estimate

    Rider->>App: Confirm ride request
    App->>FRTDB: Find nearby Drivers<br/>(GeoFlutterFire query)
    FRTDB-->>App: Available Drivers list

    App->>FRTDB: Write ride request<br/>{user_id, pickup, dropoff, fare}
    
    loop Waiting for Driver
        App->>FRTDB: Listen for acceptance
        FRTDB-->>App: Status update
    end

    Note over App,FRTDB: Driver accepts ride
    FRTDB-->>App: Driver accepted<br/>{Driver_id, Driver_location}

    App->>Backend: POST /bookItem<br/>{ride details, Driver_id, fare}
    Backend-->>App: Booking confirmation

    loop Live Tracking
        FRTDB-->>App: Driver location updates
        App->>Maps: Update Driver marker
    end

    Note over App,FRTDB: Ride completed
    FRTDB-->>App: Ride status: completed

    Rider->>App: Select payment method
    App->>PayGW: Open payment WebView
    PayGW-->>App: Payment success callback

    App->>Backend: POST /updatePaymentStatusByUser<br/>{booking_id, payment_method}
    Backend-->>App: Payment confirmed

    Rider->>App: Rate Driver (1-5 stars)
    App->>Backend: POST /giveReviewByUser<br/>{booking_id, rating, message}
    Backend-->>App: Review submitted
```

---

## 7. Threat Model Mapping (STRIDE)
**(Belongs in SRS Section 6.3: Security Requirements)**

```mermaid
mindmap
    root((STRIDE<br/>Threat Model))
        🎭 Spoofing
            User impersonation
                Mitigation: OTP verification
                Mitigation: OAuth 2.0 (Google/Apple)
            Session hijacking
                Mitigation: Bearer token + auto-refresh
                Mitigation: Session invalidation (401/419)
            Token theft
                Mitigation: Hive local storage
                Mitigation: OS app sandboxing
        🔧 Tampering
            API request modificatin
                Mitigation: HTTPS/TLS encryption
                Mitigation: Server-side validation
            Local data tampering
                Mitigation: Hive file-based storage
                Mitigation: OS sandboxing
        🚫 Repudiation
            Denial of booking/payment
                Mitigation: Server-side transaction logging
                Mitigation: Payment gateway receipts
                Mitigation: Timestamped booking records
        🔓 Information Disclosure
            PII exposure
                Mitigation: HTTPS for all data in transit
                Mitigation: Authenticated API access only
            API key exposure
                Mitigation: Platform-restricted API keys
                Mitigation: dart-define env variables
        ⛔ Denial of Service
            API flooding
                Mitigation: 15s request timeouts
                Mitigation: Server-side rate limiting
                Mitigation: Client error handling
        🔑 Elevation of Privilege
            Unauthorized role access
                Mitigation: module_id + user_type per request
                Mitigation: Separate Driver/admin apps
                Mitigation: Server-side role enforcement
```

---

## 8. Push Notification Flow
**(Belongs in SRS Section 4.10: Push Notifications)**

```mermaid
sequenceDiagram
    participant App as 📱 Rider App
    participant FCM as 🔔 Firebase FCM
    participant OS as 📲 OneSignal
    participant Backend as 🖥️ Backend

    Note over App,Backend: === Initialization ===
    App->>FCM: Get FCM Token
    FCM-->>App: fcmToken
    App->>OS: OneSignal.initialize(appId)
    App->>OS: OneSignal.login(fcmToken)
    App->>OS: addTag("FCMToken", fcmToken)
    App->>OS: Get Player ID
    OS-->>App: playerId, pushToken

    App->>Backend: POST /fcmUpdate<br/>{fcm: fcmToken, player_id: playerId}
    Backend-->>App: OK

    Note over App,Backend: === Foreground Notification ===
    FCM->>App: onMessage (RemoteMessage)
    App->>App: showFlutterNotification()

    OS->>App: ForegroundWillDisplay event
    App->>App: Deduplicate (processedNotificationIds)
    App->>App: showOneSignalNotification()

    Note over App,Backend: === Background / Click ===
    OS->>App: Notification Click event
    App->>App: handleNotificationClick(route, data)
    App->>App: Navigate to relevant screen
```

---

## 9. Local Data Storage Diagram
**(Belongs in SRS Section 5.1: Local Storage Strategy)**

```mermaid
flowchart LR
    subgraph Device["📱 Device (App Sandbox)"]
        subgraph HiveDB["Hive Database"]
            AppBox["appBox"]
            LanBox["lanBox"]
        end

        AppBox -->|"UserData (JSON)"| UD["User Profile\n• id, name, email\n• phone, country\n• token, profile_image\n• login_type, status"]
        AppBox -->|"bearerToken"| BT["Bearer Token\n(API Authentication)"]
        AppBox -->|"isGuest"| GF["Guest Flag\n(Boolean)"]

        LanBox -->|"language"| LP["Language Code\n(en, am, ar, or, ti, so)"]
    end

    subgraph InMemory["🧠 In-Memory (Runtime)"]
        Token["token\n(User Session Token)"]
        BearerMem["bearerToken\n(Cached Bearer)"]
        LoginMdl["loginModel\n(LoginModel Instance)"]
        LocData["latitudeGlobal\nlongitudeGlobal"]
        OSData["oneSignalPlayerId\noneSignalToken"]
        Flags["isGuest\nisManuallyCancelled\nactiveRideRequestId"]
    end

    AppBox -.->|"Loaded on\napp start"| InMemory
```

---

## 10. Android & iOS Permission Model
**(Belongs in SRS Section 6.3: Security Requirements)**

```mermaid
flowchart TB
    subgraph Android["🤖 Android Permissions"]
        A1["INTERNET"]
        A2["ACCESS_FINE_LOCATION"]
        A3["ACCESS_COARSE_LOCATION"]
        A4["READ_EXTERNAL_STORAGE"]
        A5["WRITE_EXTERNAL_STORAGE"]
        A6["ACCESS_NETWORK_STATE"]
        A7["WAKE_LOCK"]
        A8["VIBRATE"]
        A9["C2DM RECEIVE\n(Push Notifications)"]
    end

    subgraph iOS["🍎 iOS Permissions (Info.plist)"]
        I1["NSCameraUsageDescription\n→ Profile photo & ID capture"]
        I2["NSLocationWhenInUseUsageDescription\n→ Nearby rides & navigation"]
        I3["NSLocationAlwaysAndWhenInUseUsageDescription\n→ Background location updates"]
        I4["NSPhotoLibraryUsageDescription\n→ Upload ID & profile images"]
    end

    subgraph BGModes["iOS Background Modes"]
        B1["fetch"]
        B2["location"]
        B3["remote-notification"]
    end

    subgraph Usage["Feature → Permission Mapping"]
        F1["🗺️ Map & Ride Tracking"] --> A2
        F1 --> A3
        F1 --> I2
        F1 --> I3
        F2["📷 Profile Photo / ID Upload"] --> A4
        F2 --> A5
        F2 --> I1
        F2 --> I4
        F3["🔔 Push Notifications"] --> A7
        F3 --> A8
        F3 --> A9
        F3 --> B3
        F4["📍 Background Location"] --> I3
        F4 --> B2
    end
```
