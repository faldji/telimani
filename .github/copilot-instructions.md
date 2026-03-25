# Telimani Motorcycle Ride-Sharing App - AI Assistant Guidelines

## 🎯 Project Overview
Telimani is a production-grade motorcycle ride-sharing platform (Uber-style). **Current status: full feature set implemented — multi-role auth (OTP + email/password), live ride tracking, admin panel, PayPal payments, ratings, support cases, and payment disputes.**

- **Frontend**: React Native + Expo + expo-router, Redux Toolkit, Socket.io client
- **Backend**: Node.js/Express/TypeScript, 3-layer architecture (Routes → Controllers → Services)
- **Databases**: PostgreSQL + Redis
- **Maps/Routes**: MapLibre + OpenStreetMap tiles, OSRM route coordinate fetching
- **Realtime**: Socket.io `/rides` + `/notifications`, ride room `ride:${rideId}` updates

---

## ⚡ Quick Start Commands

### Root Level
```bash
npm run backend:dev       # Start backend on http://localhost:5000
npm run frontend:dev      # Start Expo dev (scan QR code with app)
npm run backend:build     # TypeScript compilation
npm run lint              # Run linter on both projects

# Docker-based backend operations (recommended for CI/CD)
docker compose up --build # Start backend stack (API + PostgreSQL + Redis)
docker compose --profile test run --rm backend-test  # Run tests in isolated container
```

### Docker Policy (IMPORTANT)
- Docker Compose is for **backend stack only** (backend API + PostgreSQL + Redis).
- Frontend must run locally with Expo (`cd frontend && npx expo start`).
- Do not introduce or restore frontend Docker services unless explicitly requested.

### Docker Development Workflow
- **Development**: `docker compose up --build` (hot-reload enabled)
- **Testing**: `docker compose --profile test run --rm backend-test` (isolated test environment)
- **Database**: `docker compose exec backend npm run db:migrate` (migrations in container)
- **CI/CD**: All backend operations containerized for consistent environments

### Backend Commands inside docker container
```bash
npm run dev               # Auto-reload with ts-node-dev
npm run build             # Compile TypeScript to dist/
npm start                 # Run compiled output
npm test                  # Jest unit tests (local)
npm run test:docker       # Jest tests in Docker container (recommended)
npm run build:docker      # Build TypeScript in Docker
npm run lint:docker       # Run ESLint in Docker
npm run db:migrate        # Apply Knex migrations
npm run db:rollback       # Revert last migration batch
npm run db:seed           # Populate test data (optional)
npm run db:migrate:docker # Run migrations in Docker container
npm run db:rollback:docker # Rollback migrations in Docker container
```

### Testing Status
- **✅ Working**: AuthService, RideService, AdminService, AdminController, AuthMiddleware — all pass with Docker containerized testing
- **❌ Known Issue**: RatingModel tests fail due to top-level database import (architecture limitation)
- **🔧 Workaround**: Focus testing on services with proper dependency injection patterns

### Frontend Commands (`cd frontend`)
```bash
npx expo start            # Expo dev (web default, press 'a' for Android, 'i' for iOS)
npx expo start -c         # Clear Metro cache
npm run android           # Android emulator
npm run ios               # iOS simulator
npm run build:android     # EAS build Android APK
```

---

## 🏗️ Architecture Patterns

### Backend: 3-Layer Pattern
```
routes/auth.ts
  ↓ HTTP POST /api/auth/signup
controllers/AuthController.ts
  ↓ validateInput(), check auth
services/AuthService.ts
  ↓ business logic: generateOTP(), verifyOTP()
models/User.ts + Config/database.ts
  ↓ DB queries via Knex
```
**Convention**: Routes → Controllers → Services → Models. Never skip layers for direct DB access.

### Frontend: React Component + Redux + Services
```
app/map.tsx                      ← expo-router map screen
  ↓ selectors from locationSlice
src/components/map/*             ← presentational map primitives
  ↓ no business logic in components
app/ride-tracking.tsx            ← active ride tracking screen
  ↓ useAppDispatch(), dispatch thunks
redux/slices/rideSlice.ts
  ↓ state management (availableRides[], currentRide)
redux/slices/locationSlice.ts
  ↓ userLocation, driverLocations, routeCoordinates
services/ride.ts
  ↓ API calls + OSRM route coordinates
services/socket-ride.ts
  ↓ socket events + room join/leave (no direct component socket usage)
```
**Convention**: Redux for global state, Services for API/Socket calls, never direct API calls in components.
**Navigation**: expo-router file-based routing (`app/` directory) — no React Navigation.

### Real-time Architecture (Socket.io)
- **Server**: Namespaces at `src/sockets/{rideNamespace.ts, notificationNamespace.ts}`
- **Client**: Services at `services/socket-ride.ts`, `services/socket-notifications.ts`
- **Pattern**: Emit event → Server broadcasts to room → Redux thunk receives update
- **Rooms**: `ride:${rideId}` for active rides, `user:${userId}` for notifications
- **Driver location updates**: `driver:location:update` emitted every ~5s to ride room during active rides

---

## 📁 Key File Locations

| Purpose | Backend | Frontend |
|---------|---------|----------|
| Routes | `src/routes/*.ts` | `app/` (expo-router file-based) |
| State | — | `src/redux/slices/*.ts` |
| Models | `src/models/*.ts` | — |
| Controllers | `src/controllers/*.ts` | — |
| Business Logic | `src/services/*.ts` | `src/services/*.ts` |
| Real-time | `src/sockets/*.ts` | `src/services/socket-*.ts` |
| Components | — | `src/components/` |
| Config/Env | `src/config/*.ts` | `src/services/api.ts` |

---

## 🧠 Redux Store Structure (6 Slices)

Each slice follows the **Thunk + Reducers** pattern. The store has **7 slices** total:

```typescript
// ✅ authSlice (complete)
// NOTE: no async thunks — auth calls happen in the service layer and are manually dispatched
auth: { user, token, isAuthenticated, isLoading, error }
// user shape: { id, phone, email, firstName, lastName, role: 'rider'|'driver'|'admin', rating }

// ✅ rideSlice (complete)
ride: { currentRide, rideHistory, availableRides, fareEstimate, isLoading, error }
// Thunks: requestRide, getAvailableRides, acceptRide, startRide, completeRide,
//         cancelRide, getRideHistory, estimateFare, getRideDetails

// ✅ locationSlice (complete)
location: {
  userLocation, driverLocation, driverLocations,
  routeCoordinates, isTracking, isLoading, permissionGranted, error
}
// Thunks: fetchUserLocation, watchUserLocation, stopWatchingUserLocation

// ✅ paymentSlice (complete)
payment: { currentPayment, paymentHistory, isLoading, error }
// Thunks: initializePayPal, confirmPayment, getPaymentHistory, requestRefund

// ✅ notificationSlice (complete)
notification: { notifications, unreadCount, isLoading, error, isConnected, filter }
// filter: 'all' | 'unread' | 'read'
// Thunks: fetchNotifications, getUnreadCount, markNotificationAsRead, markAllAsRead, deleteNotification

// ✅ realtimeRideSlice (complete)
realtimeRide: {
  liveRideId, rideStatus, driverLocation, driverInfo, locationHistory,
  distanceRemaining, etaMinutes, totalDistance, totalDuration,
  fare, finalFare, isTracking, lastUpdateTime, error
}
// rideStatus: 'idle' | 'requested' | 'accepted' | 'started' | 'completed' | 'cancelled'
// locationHistory: last 100 GPS points (for route playback)

// ✅ adminSlice (complete)
admin: { users, rides, payments, supportCases, disputes, isLoading, error }
// Thunks: fetchAdminUsers, updateAdminUser, fetchAdminRides, fetchAdminPayments,
//         resolveAdminPaymentDispute
```

**When adding features**:
1. Add state + initial state in `slices/*.ts`
2. Create async thunk for API calls
3. Add reducers for state updates
4. Export thunk and selectors
5. Use in components via `useAppDispatch()` and `useAppSelector()`

---

## 🎨 Design System

### Colors
```typescript
PRIMARY: "#2E86AB"      // Call-to-action buttons
SECONDARY: "#A23B72"    // Accent elements
SUCCESS: "#27AE60"      // Confirmations (ride accepted, payment success)
WARNING: "#F39C12"      // Cautions (low balance, high demand)
ERROR: "#E74C3C"        // Errors (invalid OTP, failed payment)
LIGHT_BG: "#F5F5F5"     // Card backgrounds
DARK_TEXT: "#1A1A1A"    // Primary text
```

### Typography
- **Headers**: 700 weight, 28px size
- **Subheaders**: 700 weight, 20px size
- **Labels**: 600 weight, 16px size
- **Body**: 400 weight, 14px size
- **Small**: 400 weight, 12px size

### Components
- Uses React Native built-ins (View, TouchableOpacity, FlatList, ScrollView)
- Custom: `Input` (TextInput wrapper), `Button` (gradient + loading state), `Card` (shadow + radius), `Badge`
- Responsive: Use Dimensions for screen size, no hardcoded widths/heights

---

## 📡 API Endpoints Reference

### Authentication
```
POST   /api/auth/signup                  → Send OTP to phone
POST   /api/auth/verify-otp             → Verify OTP, return JWT
POST   /api/auth/login                  → Phone + OTP login (alias)
POST   /api/auth/login-email            → Email + password login
POST   /api/auth/forgot-password        → Send password reset email
POST   /api/auth/reset-password         → Reset password via token
GET    /api/auth/profile                → Get current user (requires JWT)
PUT    /api/auth/profile                → Update user profile
POST   /api/auth/set-password           → Set/update password for email login (requires JWT)
POST   /api/auth/upgrade-to-driver      → Convert rider to driver (requires JWT)
GET    /api/auth/users                  → List users — paginated, role filter (admin only)
POST   /api/auth/drivers/go-online      → Mark driver available (driver only)
POST   /api/auth/drivers/go-offline     → Mark driver unavailable (driver only)
```

### Rides
```
POST   /api/rides                        → Create ride request (rider)
GET    /api/rides/available              → List pending rides (driver)
POST   /api/rides/estimate-fare          → Get fare estimate (public, no auth)
GET    /api/rides/history                → Rider or driver ride history (requires JWT)
GET    /api/rides/:id                    → Get ride details (requires JWT)
POST   /api/rides/:id/accept             → Accept ride (driver only)
POST   /api/rides/:id/start             → Start ride (driver only)
POST   /api/rides/:id/complete           → End ride, calculate actual fare (driver only)
POST   /api/rides/:id/cancel             → Cancel ride — rider or driver (requires JWT)
POST   /api/rides/:id/rate               → Rate driver (rider only)
GET    /api/rides/driver/earnings        → Driver earnings dashboard (driver only)
```

### Payments
```
POST   /api/payments/initialize-paypal  → Create PayPal order for a completed ride (rider)
POST   /api/payments/confirm            → Confirm payment with transaction_id (requires JWT)
GET    /api/payments/history            → Payment history (requires JWT)
POST   /api/payments/refund             → Request payment refund (rider)
GET    /api/payments/paypal/return      → PayPal redirect callback (public)
GET    /api/payments/paypal/cancel      → PayPal cancel callback (public)
```

### Notifications
```
GET    /api/notifications                → Fetch notifications (requires JWT)
GET    /api/notifications/unread         → Unread notification count (requires JWT)
POST   /api/notifications/:id/read      → Mark one notification as read (requires JWT)
POST   /api/notifications/read-all      → Mark all notifications as read (requires JWT)
DELETE /api/notifications/:id           → Delete one notification (requires JWT)
DELETE /api/notifications               → Delete all notifications (requires JWT)
```

### Admin (admin role required for all)
```
GET    /api/admin/users                  → List all users (paginated, role filter)
PATCH  /api/admin/users/:id             → Update user role / is_active / profile fields
GET    /api/admin/rides                  → List all rides (status, date range filter)
GET    /api/admin/rides/:id             → Get ride details
POST   /api/admin/rides/:id/support     → Create support case for a ride
GET    /api/admin/support                → List support cases (status, ride_id filter)
GET    /api/admin/support/:id           → Get support case details
PATCH  /api/admin/support/:id           → Update support case status / resolution notes
GET    /api/admin/payments               → List all payments (paginated)
GET    /api/admin/payments/disputes      → List payment disputes (status, ride_id filter)
GET    /api/admin/payments/disputes/:id → Get dispute details
POST   /api/admin/payments/:id/disputes/resolve → Resolve dispute (approve or reject)
```

---

## 🗄️ Database Schema (PostgreSQL)

**Migrations** (applied in order):
- `backend/migrations/001_initial_schema.js` — all core tables
- `backend/migrations/002_add_admin_role_to_users_enum.js` — adds `admin` to the `role` enum
- `backend/migrations/003_admin_support_and_disputes.js` — adds support cases + payment disputes

Key tables:
- **users**: id, phone (unique), email (unique), password_hash, role (`rider`|`driver`|`admin`), first_name, last_name, profile_image_url, rating, total_rides, is_active, timestamps
- **drivers**: id, user_id (FK users, unique), license_number, vehicle_plate (unique), vehicle_type (motorcycle), is_verified, is_online, rating, total_completed_rides, total_earnings, today_earnings, timestamps
- **rides**: id, rider_id, driver_id, pickup/dropoff coords + addresses, status (`pending`|`accepted`|`in_progress`|`completed`|`cancelled`), estimated_fare, actual_fare, distance_km, duration_minutes, payment_status, cancellation_reason, accepted_at, started_at, completed_at, cancelled_at, timestamps
- **payments**: id, ride_id (unique), payer_id, amount, currency, payment_method (`paypal`|`wallet`|`card`), status (`pending`|`completed`|`failed`|`refunded`), transaction_id (unique), paypal_payment_id, paid_at, timestamps
- **ride_locations**: ride_id, lat, lng, speed, recorded_at (for playback on map)
- **ratings**: id, ride_id (unique), rated_by_id, rated_user_id, rating (1–5), comment, timestamps
- **ride_support_cases**: id, ride_id, admin_id, case_type, status (`open`|`investigating`|`resolved`|`escalated`), description, resolution_notes, resolved_at, timestamps
- **payment_disputes**: id, payment_id (unique), ride_id, payer_id, support_case_id, reason, status (`open`|`approved`|`rejected`), resolved_at, resolution_notes, timestamps

> **Note:** Notifications are stored in **Redis** (30-day TTL), not PostgreSQL.

---

## 💡 Development Conventions

### When Adding Features

1. **Auth-Protected Routes**: Use `auth` middleware in routes. Example: `router.post('/profile', auth, updateProfile)`
2. **Redux Actions**: Always use thunks for async operations; never setState in reducers
3. **Error Handling**: Return `{error: "message"}` from services; Controllers catch and respond with HTTP status
4. **Real-time Events**: Emit from client → broadcast from server to rooms → Redux thunk receives update
5. **Component Props**: Use TypeScript interfaces; avoid prop drilling (use Redux for global state)
6. **Testing**: Backend uses Jest; add tests in `__tests__/` folders. Use `npm run test:docker` for CI/CD testing in isolated containers

### Unified Authentication Flow
**Goal**: Single login experience that automatically routes users to their correct flow based on backend role response.

**Entry Point**: `/(auth)` landing screen → unified "Sign In or Create Account" button → `/(auth)/login`

**Login Screen** (unified for all roles):
- Supports both **Phone OTP** and **Email/Password** modes
- No role selection in UI
- User identity determined by backend, not frontend selection

**Post-Login Auto-Routing**:
After successful login, the app automatically redirects based on `authSlice.user.role`:
- `role: 'rider'` → `/(tabs)` (rider map/home flow)
- `role: 'driver'` → `/driver-online` (driver dashboard)
- `role: 'admin'` → `/(admin)` (admin panel)

Implemented in `app/(auth)/_layout.tsx` using `getRouteFromRole()` helper from `src/utils/roleRouting.ts`.

**Phone OTP Flow** (default for new signups):
1. User enters phone → `POST /api/auth/signup` → Backend generates 6-digit OTP (Redis, 10-min expiry)
2. SMS sent (mock in dev, Twilio in prod)
3. User enters OTP → `POST /api/auth/verify-otp` → Backend validates, returns user + JWT
4. Backend defaults new signups to `role: 'rider'`; users can upgrade to driver from profile
5. Frontend dispatches `setUser()` + `setToken()`, auth layout redirects based on role

**Email/Password Flow** (existing accounts):
1. User enters email + password → `POST /api/auth/login-email`
2. Backend returns user object with their current role
3. Frontend handles redirect via auth layout (same as OTP flow)

**Role Upgrade**:
- Riders can upgrade to drivers via `PUT /api/auth/upgrade-to-driver`
- Called from profile screen; redirects to driver flow after success

### Ride Lifecycle (Driver Perspective)
1. Driver comes online → `POST /api/auth/drivers/go-online`
2. Driver sees available rides → `GET /rides/available` (Socket.io update every 10 sec)
3. Driver accepts ride → `POST /rides/:id/accept` → Socket.io notifies rider (emit to room `ride:${rideId}`)
4. Driver travels to pickup → Location streamed via Socket.io every 5 sec
5. Driver starts ride → `POST /rides/:id/start` → Fare meter begins
6. Driver reaches destination → `POST /rides/:id/complete` → Final fare calculated, payment processed
7. Rider rates driver → `POST /rides/:id/rate`

### PayPal Payment (Sandbox)
- Uses PayPal Sandbox API keys (configured in `.env`)
- Payment processed in `services/PaymentService.ts` → `services/payment.ts` (frontend)
- After successful payment: mark ride as `completed`, update driver earnings, send notification

---

## 🧪 Testing Checklist (Before Deploy)

- [ ] **Backend**: `npm run dev` starts, server listens on port 5000
- [ ] **Backend Tests**: `npm run test:docker` passes (AuthService, RideService, AdminService, AdminController, AuthMiddleware)
- [ ] **Frontend**: `npx expo start` launches Expo, shows login screen
- [ ] **Database**: PostgreSQL running, migrations applied (`npm run db:migrate`)
- [ ] **Redis**: Running on port 6379 (OTP storage, driver locations)
- [ ] **Auth Flow**: Phone signup → OTP → JWT token works end-to-end
- [ ] **Email Auth Flow**: Email + password login → JWT token works end-to-end
- [ ] **Ride Flow**: Rider requests ride → Driver sees it → Driver accepts → Live location updates on map
- [ ] **Admin Panel**: Admin user can access `/admin` routes and manage users/rides/payments
- [ ] **Payment**: PayPal sandbox payment processes without error
- [ ] **Socket.io**: Real-time notifications appear without page refresh
- [ ] **Offline Resilience**: App doesn't crash if backend is down (graceful error handling)

---

## 📚 Further Reading

- Backend setup: See `backend/README.md`

---

## 🤖 Copilot Agent Usage Examples

Use these prompts to interact with the AI assistant in this codebase:

1. "Add surge pricing logic to `RideService.estimateFare` when demand is high and write Jest tests for it."
2. "Wire up the admin `disputes` and `supportCases` Redux thunks that are currently missing reducers in `adminSlice.ts`."
3. "Fix `DriverOnlineScreen.tsx` to use the authenticated user ID from `authSlice` instead of the hardcoded `driver_001`."
4. "Add a `ride:driver_arrived` Socket.io event: server side in `rideNamespace.ts`, frontend service in `socket-ride.ts`, and Redux reducer in `realtimeRideSlice.ts`."
5. "Write Jest tests for `AdminService.resolvePaymentDispute` covering approve and reject paths with mocked DB calls."
6. "Add Docker-based testing for `NotificationService` and update `backend/TESTING.md` with examples."

> Tip: Keep prompts concise and refer to file paths (e.g., `routes`, `services`, `redux/slices`) to get precise, actionable results.

## 📚 Current References

- Project setup: See `SETUP_INSTRUCTIONS.md`
- Backend guide: See `backend/README.md`
- Frontend guide: See `frontend/README.md`
- Testing guidance: See `backend/TESTING.md` and `frontend/TESTING.md`
- Architecture decisions: See `backend/tsconfig.json` (TypeScript target), `frontend/app.json` (Expo config)

## ✅ Current Map/Tracking Source of Truth
- Screen: `frontend/app/map.tsx`
- Active tracking: `frontend/app/ride-tracking.tsx`, `frontend/src/screens/RideTrackingScreen.tsx`
- Reusable map components: `frontend/src/components/map/*` (markers, polyline), `frontend/src/components/Map/*` (RideMap native + web platform split), `frontend/src/components/Ride/*` (DriverInfo, ETADisplay, RideActionButtons)
- Location state + thunks: `frontend/src/redux/slices/locationSlice.ts`
- Ride socket service: `frontend/src/services/socket-ride.ts`
- Ride room backend emit loop: `backend/src/sockets/rideNamespace.ts`

## ⚠️ Known Issues / TODOs

1. **Type strictness debt** — several frontend modules still use broad `any` types; lint now permits current patterns, but deeper typing cleanup is still recommended.
