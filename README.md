# ⚡ VoltShare India (P2P EV Charging Sharing Ecosystem)

VoltShare is a peer-to-peer (P2P) electric vehicle (EV) charging sharing mobile application tailored for the Indian market. It bridges the gap between apartment residents (**Hosts**) who have private chargers/sockets sitting idle while they are away at work, and EV owners (**Seekers**) traveling on highways and urban corridors who need reliable, nearby charging points.

---

## 🏗️ Architecture & Stack

- **Frontend:** Flutter (cross-platform iOS & Android) with **Clean Architecture** (`lib/features`, `lib/core`, `lib/services`).
- **State Management:** Riverpod (`flutter_riverpod: ^2.5.1`).
- **Backend & Database:** Firebase:
  - **Firebase Authentication:** Phone OTP login (+91 Indian mobile number support with demo fallback).
  - **Cloud Firestore:** Real-time sync for charger listings, bookings, and user profiles.
  - **Cloud Functions:** Automated booking expiry cron (15-min timeout), push notifications, and lifecycle triggers.
- **Geo-Discovery:** Firestore Geoqueries using Geohash bounding boxes and Haversine distance calculations (in kilometers) from highway corridors and live GPS positions.
- **Payments:** Indian UPI settlement flow (Google Pay, PhonePe, Paytm, BHIM UPI).

---

## 📁 Project Structure

```
VoltShare/
├── android/                        # Android project configuration & permissions
│   ├── app/
│   │   ├── build.gradle            # Firebase BoM & compileSdk 34
│   │   ├── google-services.json    # Android Firebase configuration template
│   │   └── src/main/AndroidManifest.xml # Location & UPI intents
│   ├── build.gradle
│   └── settings.gradle
├── ios/                            # iOS project configuration & permissions
│   └── Runner/
│       ├── GoogleService-Info.plist # iOS Firebase configuration template
│       └── Info.plist              # Location permissions & URL schemes
├── functions/                      # Firebase Cloud Functions (Node.js 18)
│   ├── index.js                    # Triggers: onBookingCreated, onBookingStatusChanged, expireStalePendingBookings
│   └── package.json
├── firestore.rules                 # Cloud Firestore security rules
├── firestore.indexes.json          # Composite indexes for Geo & Bookings
├── firebase.json                   # Firebase deployment configuration
├── pubspec.yaml                    # Flutter dependencies & assets
└── lib/
    ├── core/
    │   ├── constants/
    │   │   ├── app_constants.dart  # Indian highways, connectors, radii
    │   │   ├── colors.dart         # High-voltage green, slate, amber
    │   │   └── theme.dart          # Light Theme Material 3
    │   ├── errors/
    │   │   └── failures.dart
    │   ├── utils/
    │   │   ├── currency_formatter.dart # INR ₹ formatting
    │   │   ├── date_time_utils.dart
    │   │   └── geohash_helper.dart # Self-contained Geohash encoder/bounds
    │   └── widgets/
    │       ├── custom_button.dart
    │       ├── custom_text_field.dart
    │       ├── empty_state.dart
    │       ├── loading_indicator.dart
    │       └── status_badge.dart
    ├── services/
    │   ├── firebase_service.dart   # Firebase initialization
    │   ├── location_service.dart   # Geolocator permission & position stream
    │   └── notification_service.dart
    ├── features/
    │   ├── auth/
    │   │   ├── data/auth_repository.dart
    │   │   ├── domain/user_model.dart
    │   │   └── presentation/
    │   │       ├── controllers/auth_controller.dart
    │   │       └── screens/
    │   │           ├── phone_login_screen.dart
    │   │           ├── otp_verification_screen.dart
    │   │           └── profile_screen.dart
    │   ├── chargers/
    │   │   ├── data/charger_repository.dart
    │   │   ├── domain/
    │   │   │   ├── charger_model.dart
    │   │   │   └── connector_type.dart
    │   │   └── presentation/
    │   │       ├── controllers/charger_controller.dart
    │   │       └── screens/
    │   │           ├── charger_detail_screen.dart
    │   │           └── add_charger_screen.dart
    │   ├── bookings/
    │   │   ├── data/booking_repository.dart
    │   │   ├── domain/booking_model.dart
    │   │   └── presentation/
    │   │       ├── controllers/booking_controller.dart
    │   │       └── screens/
    │   │           ├── my_bookings_screen.dart
    │   │           ├── booking_detail_screen.dart
    │   │           └── mock_payment_sheet.dart
    │   ├── explore/
    │   │   └── presentation/
    │   │       ├── controllers/explore_controller.dart
    │   │       └── screens/
    │   │           ├── explore_map_screen.dart
    │   │           └── widgets/
    │   │               ├── charger_card.dart
    │   │               └── filter_bottom_sheet.dart
    │   ├── host/
    │   │   └── presentation/
    │   │       ├── controllers/host_controller.dart
    │   │       └── screens/
    │   │           └── host_dashboard_screen.dart
    │   └── navigation/
    │       └── main_nav_scaffold.dart
    ├── firebase_options.dart
    └── main.dart
```

---

## 🗄️ Database Schema (Firestore Collections)

### 1. `users` Collection
- `id` (String): Firebase Auth UID.
- `phoneNumber` (String): User's mobile number (`+91...`).
- `name` (String): Full name.
- `role` (String): `host`, `seeker`, or `both`.
- `upiId` (String): UPI VPA for Host payouts (e.g., `user@okhdfcbank`).
- `evModel` (String): EV brand & model (e.g., `Tata Nexon EV Max`).
- `apartmentName` (String): Community name.
- `rating` (Number): User rating (out of 5.0).
- `createdAt` (Timestamp).

### 2. `chargers` Collection
- `id` (String): Unique charger identifier.
- `hostId` (String): UID of host.
- `hostName` (String): Name of host.
- `hostPhone` (String): Contact phone for gate coordination.
- `title` (String): Listing display title.
- `apartmentName` (String): Society / Apartment name.
- `parkingSlotNumber` (String): e.g., `Basement 1, Slot B-104`.
- `societyGateInstructions` (String): Gate security guidelines (e.g. `Visiting Flat 402 for EV charging`).
- `address` (String): Full address.
- `latitude` (Number) & `longitude` (Number): Coordinates.
- `geohash` (String): Geohash for fast proximity range queries.
- `distanceFromHighwayKm` (Number): Distance in km from main highway corridor.
- `nearbyHighway` (String): e.g. `NH 44 (Hosur Road / Silk Board)`.
- `connectorType` (String): `ccs2`, `type2`, `socket15A`, `bharatAC001`.
- `powerOutputKw` (Number): Output capacity (e.g. 7.4 kW, 11 kW, 3.3 kW).
- `pricingType` (String): `per_kwh` or `per_hour`.
- `priceRateInr` (Number): Rate in INR (₹).
- `availableFrom` & `availableTo` (String): Time window (e.g., `09:00` - `18:30`).
- `availableDays` (Array): e.g. `['Mon', 'Tue', 'Wed', 'Thu', 'Fri']`.
- `isAutoApprove` (Boolean): Instant confirmation toggle.
- `isAvailable` (Boolean): Live Host status toggle (Online / Off-duty).
- `amenities` (Array): e.g. `['Covered Parking', '24/7 Guard', 'CCTV', 'Restroom Access']`.
- `rating` (Number): Average session rating.
- `totalSessions` (Number): Total completed charging sessions.

### 3. `bookings` Collection
- `id` (String): Unique booking ID.
- `chargerId` (String): Reference to charger.
- `chargerTitle` (String): Charger name.
- `hostId` & `seekerId` (String): Host and Seeker UIDs.
- `startTime` & `endTime` (Timestamp): Reserved charging window.
- `durationMinutes` (Number): Duration (e.g. 120 mins).
- `estimatedUnitsKwh` (Number): Anticipated energy consumption.
- `estimatedCostInr` (Number): Estimated total.
- `finalCostInr` (Number): Final billed total.
- `status` (String): `pending`, `confirmed`, `charging`, `completed`, `cancelled`, `rejected`.
- `paymentStatus` (String): `pending`, `paid`, `refunded`.
- `paymentMethod` (String): `UPI`.
- `paymentTransactionId` (String): UPI Reference Number.
- `otpStartCode` (String): 4-digit code given to host at plug-in.
- `otpEndCode` (String): 4-digit code given to host at unplugging.
- `vehicleNumber` (String): e.g. `KA 03 MM 8921`.
- `createdAt` (Timestamp).

---

## 🚀 Key User Workflows

### 1. Seeker: Discovery & Instant Booking
1. Seeker opens the **Explore** tab to view nearby chargers near their GPS coordinates or highway corridors (e.g. NH 44 Hosur Road).
2. Toggle between **Card List** and **Interactive Map/Radar** view.
3. Filter by radius (2km to 50km) and connector type (CCS2 DC Fast, Type 2 AC, 15A Socket).
4. Tap any charger to view society gate pass instructions, host rating, and power specs.
5. Select duration (1 to 4 hours) and reserve. If instant auto-approval is enabled, the booking is confirmed immediately!
6. Launch turn-by-turn navigation directly into Google Maps / Apple Maps.

### 2. Safe Host-Seeker Handshake via Two-Factor OTP
- When the traveler reaches the apartment parking bay, the Seeker provides the **Start OTP** to verify the vehicle plug-in.
- When charging completes, the Host inputs the **End OTP** to end the session and compute the final bill.

### 3. UPI Settlement
- Once the session is marked completed, the Seeker settles directly with the Host using digital UPI apps (Google Pay, PhonePe, Paytm, BHIM).

### 4. Apartment Host: Monetization & Management
- Apartment residents can list their private home charger or 15A socket in under 2 minutes.
- Toggle between **Online & Bookable** (while at office) and **Off-Duty** (for personal charging).
- View total earnings (₹), completed sessions, and incoming booking requests with one-tap Accept/Decline.

---

## ⚡ Cloud Functions & Backend Automation

VoltShare India uses serverless event-driven and scheduled Firebase Cloud Functions (`functions/index.js`):

1. **`onBookingCreated` (Notification Hook & Auto-Lock):**
   - Triggers when a new booking is written to Firestore.
   - Logs structured notification event formatted for Firebase Cloud Messaging (FCM).
   - Writes an in-app notification document to `users/{hostId}/notifications`.
   - Sends FCM push notification alerting the host with vehicle details and time slot.
   - Automatically locks the charger (`isAvailable: false`) if confirmed on creation.

2. **`onBookingStatusChanged` (Status Lifecycle & Charger Revert):**
   - Triggers when a booking status transitions.
   - When a booking concludes (`completed`, `cancelled`, or `rejected`), automatically reverts the charger's `isAvailable` property in Firestore back to `true`.
   - Dispatches status notifications to the Seeker (Plug-in start OTP on confirmation, session completion summary).

3. **`autoCompleteExpiredBookings` (Scheduled Expiry Cron):**
   - Scheduled Pub/Sub cron running every 5 minutes.
   - Automatically queries active bookings whose `endTime <= now`.
   - Marks booking status as `completed` and reverts `isAvailable: true` on associated chargers.

4. **`triggerBookingExpiryNow` (Manual Webhook Endpoint):**
   - HTTP endpoint allowing developers/testers to trigger the expiry engine on-demand.

