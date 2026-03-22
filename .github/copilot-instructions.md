# Telimani Motorcycle Ride-Sharing App - AI Assistant Guidelines

## 🎯 Project Overview
Telimani is a production-grade motorcycle ride-sharing platform (like Uber). **Status: Phase 4 Complete** with real-time Socket.io, 11+ screens, full auth + payments.

- **Frontend**: React Native + Expo, Redux state management, Socket.io real-time
- **Backend**: Node.js/Express/TypeScript, 3-layer architecture (Routes → Controllers → Services)
- **Databases**: PostgreSQL (persistent) + Redis (real-time caching)
- **Authentication**: JWT tokens + Phone OTP, login lockout protection
- **Payment**: PayPal sandbox integration
- **Real-time**: Socket.io with `/rides` and `/notifications` namespaces

---

## ⚡ Quick Start Commands

### Root Level
```bash
npm run backend:dev       # Start backend on http://localhost:5000
npm run frontend:dev      # Start Expo dev (scan QR code with app)
npm run backend:build     # TypeScript compilation
npm run lint              # Run linter on both projects
```

### Backend Commands (`cd backend`)
```bash
npm run dev               # Auto-reload with ts-node-dev
npm run build             # Compile TypeScript to dist/
npm run start             # Run compiled output
npm run test              # Jest unit tests
npm run db:migrate        # Apply Knex migrations
npm run db:rollback       # Revert last migration
npm run db:seed           # Populate test data (optional)
```

### Frontend Commands (`cd frontend`)
```bash
npm start                 # Expo dev (web default, press 'a' for Android, 'i' for iOS)
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
screens/rider/BookRideScreen.tsx
  ↓ useAppDispatch(), dispatch thunks
redux/slices/rideSlice.ts
  ↓ state management (availableRides[], currentRide)
services/ride.ts
  ↓ API calls via api.ts
services/socket-ride.ts
  ↓ Real-time events via Socket.io
```
**Convention**: Redux for global state, Services for API/Socket calls, never direct API calls in components.

### Real-time Architecture (Socket.io)
- **Server**: Namespaces at `src/sockets/{rideNamespace.ts, notificationNamespace.ts}`
- **Client**: Services at `services/socket-ride.ts`, `services/socket-notifications.ts`
- **Pattern**: Emit event → Server broadcasts to room → Redux thunk receives update
- **Rooms**: `ride:${rideId}` for active rides, `user:${userId}` for notifications

---

## 📁 Key File Locations

| Purpose | Backend | Frontend |
|---------|---------|----------|
| Routes | `src/routes/*.ts` | `src/navigation/RootNavigator.tsx` |
| State | — | `src/redux/slices/*.ts` |
| Models | `src/models/*.ts` | — |
| Controllers | `src/controllers/*.ts` | — |
| Business Logic | `src/services/*.ts` | `src/services/*.ts` |
| Real-time | `src/sockets/*.ts` | `src/services/socket-*.ts` |
| Components | — | `src/components/` + `src/screens/` |
| Config/Env | `src/config/*.ts` | `services/api.ts` |

---

## 🧠 Redux Store Structure (6 Slices)

Each slice follows the **Thunk + Reducers** pattern:

```typescript
// ✅ authSlice (complete)
auth: { user, token, refreshToken, isAuthenticated, isLoading, error, loginAttempts, lockoutUntil }

// ✅ rideSlice (complete)  
ride: { availableRides, currentRide, rideHistory, isLoading, filter, searchResults }

// ✅ locationSlice (complete)
location: { userLocation, driverLocations, searchedLocation, routeCoordinates }

// ✅ paymentSlice (complete)
payment: { paymentMethods, paymentHistory, currentPayment, isProcessing }

// ✅ notificationSlice (complete)
notification: { notifications, unreadCount, filters, isLoading }

// ✅ realtimeRideSlice (complete)
realtimeRide: { liveRideId, driverInfo, rideStatus, driverLocationStream }
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
POST   /api/auth/signup              → Send OTP to phone
POST   /api/auth/verify-otp          → Verify OTP, return JWT
GET    /api/auth/profile             → Get current user (requires JWT)
PUT    /api/auth/profile             → Update user profile
POST   /api/auth/upgrade-to-driver   → Convert rider to driver
```

### Rides
```
POST   /api/rides                    → Create ride request (rider)
GET    /api/rides/available          → List pending rides (driver)
POST   /api/rides/:id/accept         → Accept ride (driver)
POST   /api/rides/:id/start          → Start ride (driver)
POST   /api/rides/:id/complete       → End ride, calculate final fare (driver)
POST   /api/rides/estimate-fare      → Get fare estimate (public)
GET    /api/rides/driver/earnings    → Driver earnings dashboard
POST   /api/rides/:id/rate           → Rate ride & driver (rider)
```

### Payments
```
POST   /api/payments/add-method      → Add PayPal/card (rider)
POST   /api/payments/process         → Process payment for ride
GET    /api/payments/history         → Payment history
```

### Notifications
```
GET    /api/notifications            → Fetch notifications
PUT    /api/notifications/:id/read   → Mark as read
DELETE /api/notifications/:id        → Delete notification
```

---

## 🗄️ Database Schema (PostgreSQL)

**Migrations**: `backend/migrations/001_initial_schema.js`

Key tables:
- **users**: id, phone (unique), email, name, role (rider|driver), avatar_url, rating, created_at
- **drivers**: id (FK users), license_number, vehicle_type, is_verified, is_online, monthly_earnings
- **rides**: id, rider_id (FK users), driver_id (FK drivers), pickup/dropoff coords, fare, status (requested|accepted|in_progress|completed|cancelled), created_at
- **payments**: id, ride_id (FK rides), amount, method (paypal|wallet), status, created_at
- **ride_locations**: ride_id, lat, lng, speed, recorded_at (for playback on map)
- **ratings**: id, ride_id, rater_id, ratee_id, score (1-5), comment, created_at
- **notifications**: id, user_id, type, message, data (JSON), is_read, created_at

---

## 💡 Development Conventions

### When Adding Features

1. **Auth-Protected Routes**: Use `auth` middleware in routes. Example: `router.post('/profile', auth, updateProfile)`
2. **Redux Actions**: Always use thunks for async operations; never setState in reducers
3. **Error Handling**: Return `{error: "message"}` from services; Controllers catch and respond with HTTP status
4. **Real-time Events**: Emit from client → broadcast from server to rooms → Redux thunk receives update
5. **Component Props**: Use TypeScript interfaces; avoid prop drilling (use Redux for global state)
6. **Testing**: Backend uses Jest; add tests in `__tests__/` folders

### Phone OTP Flow
1. User enters phone → `POST /auth/signup` → Backend generates 6-digit OTP, stores in Redis (5 min expiry)
2. SMS sent (mock in dev, real via Twilio in prod)
3. User enters OTP → `POST /auth/verify-otp` → Backend validates, returns JWT token
4. Frontend stores token in Redux `auth.token` + AsyncStorage (for persistence)

### Ride Lifecycle (Driver Perspective)
1. Driver comes online → `POST /rides/driver/go-online`
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
- [ ] **Frontend**: `npm start` launches Expo, shows login screen
- [ ] **Database**: PostgreSQL running, migrations applied (`npm run db:migrate`)
- [ ] **Redis**: Running on port 6379 (OTP storage, driver locations)
- [ ] **Auth Flow**: Phone signup → OTP → JWT token works end-to-end
- [ ] **Ride Flow**: Rider requests ride → Driver sees it → Driver accepts → Live location updates on map
- [ ] **Payment**: PayPal sandbox payment processes without error
- [ ] **Socket.io**: Real-time notifications appear without page refresh
- [ ] **Offline Resilience**: App doesn't crash if backend is down (graceful error handling)

---

## 📚 Further Reading

- Backend setup: See `backend/README.md`
- Frontend setup: See `frontend/README.md`
- Phase-specific details: Check `PHASE*_*.md` files for testing procedures
- Architecture decisions: See `backend/tsconfig.json` (TypeScript target), `frontend/app.json` (Expo config)
