# Phase 3 Files Reference Guide

Complete navigation map for Phase 3 implementation with file locations, purposes, and quick links.

---

## Backend Files

### Models Layer (`src/models/`)

#### `Ride.ts` (330 lines)
**Purpose:** Ride data model with all CRUD operations

**Key Classes/Interfaces:**
- `Ride` interface - Complete ride data structure
- `RideModel` - Static methods for database operations

**Key Methods:**
- `create()` - Create new ride request
- `findById()` - Get ride by ID
- `findAvailableRidesNearby()` - Find rides within radius (driver view)
- `acceptRide()` - Assign driver to ride
- `startRide()` - Begin journey
- `completeRide()` - Finish ride & record metrics
- `cancelRide()` - Cancel with reason
- `getRiderRideHistory()` - Get rider's past rides
- `getDriverRideHistory()` - Get driver's past rides
- `getCompletedRidesToday()` - Driver's today rides
- `updatePaymentStatus()` - Update payment state

**Database Tables Accessed:**
- `rides` (INSERT, SELECT, UPDATE)
- `drivers` (JOIN for driver info)
- `users` (JOIN for user info)

---

#### `Payment.ts` (130 lines)
**Purpose:** Payment tracking and history management

**Key Methods:**
- `create()` - Initialize payment record
- `findById()` - Get payment by ID
- `findByRideId()` - Get payment for specific ride
- `findByTransactionId()` - Look up by PayPal transaction
- `updateStatus()` - Update payment state
- `markAsCompleted()` - Record successful payment
- `getPaymentHistory()` - User's payment history
- `getTotalEarningsByDriver()` - Aggregate driver earnings

**Database Tables Accessed:**
- `payments` (INSERT, SELECT, UPDATE)

---

### Services Layer (`src/services/`)

#### `RideService.ts` (110 lines)
**Purpose:** Business logic for rides - fare calculation, distance computation, duration estimation

**Key Constants:**
```typescript
BASE_FARE = $2.00
PER_KM_RATE = $1.50
PER_MINUTE_RATE = $0.25
MIN_FARE = $3.00
```

**Key Methods:**
- `calculateDistance(lat1, lng1, lat2, lng2)` - Haversine formula
- `toRad(degrees)` - Convert degrees to radians
- `estimateDuration(distanceKm)` - Calculate ETA (15 km/h avg)
- `estimateFare(distanceKm, estimatedMinutes?)` - Get complete fare breakdown
- `calculateActualFare(distanceKm, durationMinutes)` - Final charge after ride
- `findNearbyDrivers()` - Placeholder for Phase 4 real-time lookup

**Used By:** RideController

---

#### `PaymentService.ts` (85 lines)
**Purpose:** Payment processor interface (mock PayPal integration)

**Key Methods:**
- `getConfig()` - Get PayPal mode (sandbox/production)
- `initializePayment()` - Create payment session (mock ID: PAY-{timestamp})
- `executePayment()` - Simulate payment execution (90% success)
- `refundPayment()` - Process refund request
- `getPaymentStatus()` - Check payment state

**Mock Implementation Details:**
- Returns payment IDs in format: `PAY-{timestamp}`
- Simulates 90% success rate for testing
- In production, would use PayPal REST SDK

**Used By:** PaymentController

---

### Controllers Layer (`src/controllers/`)

#### `RideController.ts` (380 lines)
**Purpose:** HTTP request handlers for all ride endpoints

**Endpoint Handlers:**
1. `requestRide()` - POST /api/rides → Create new ride request
2. `getAvailableRides()` - GET /api/rides/available → List rides near driver
3. `acceptRide()` - POST /api/rides/:id/accept → Driver accepts ride
4. `startRide()` - POST /api/rides/:id/start → Begin journey
5. `completeRide()` - POST /api/rides/:id/complete → Finish & calculate fare
6. `cancelRide()` - POST /api/rides/:id/cancel → Cancel ride
7. `getRideDetails()` - GET /api/rides/:id → Get full ride info
8. `getRideHistory()` - GET /api/rides/history → Get past rides
9. `estimateFare()` - POST /api/rides/estimate-fare → Public fare quote
10. `getDriverEarnings()` - GET /api/drivers/earnings → Driver stats

**Key Features:**
- Authorization checks (token verification)
- Role validation (driver-only, rider-only endpoints)
- Input validation (coordinates, IDs)
- Error responses (400, 403, 404, 500)

**Used By:** Mounted to Express app via `routes/rides.ts`

---

#### `PaymentController.ts` (220 lines)
**Purpose:** HTTP handlers for payment workflow

**Endpoint Handlers:**
1. `initializePayPal()` - POST /api/payments/initialize-paypal → Start payment
2. `handlePayPalReturn()` - GET /api/payments/paypal/return → Webhook after approval
3. `handlePayPalCancel()` - GET /api/payments/paypal/cancel → User cancels payment
4. `confirmPayment()` - POST /api/payments/confirm → Verify & record payment
5. `getPaymentHistory()` - GET /api/payments/history → User's payments
6. `requestRefund()` - POST /api/payments/refund → Request money back

---

### Routes Layer (`src/routes/`)

#### `rides.ts` (25 lines)
**Purpose:** Route definitions for all ride endpoints

**Route Definitions:**
```
POST   /api/rides                    (auth, everyone)
GET    /api/rides/history            (auth, everyone)
POST   /api/rides/estimate-fare      (public)
GET    /api/rides/:id                (auth, everyone)
POST   /api/rides/:id/cancel         (auth, everyone)
GET    /api/rides/available          (auth, driver-only)
POST   /api/rides/:id/accept         (auth, driver-only)
POST   /api/rides/:id/start          (auth, driver-only)
POST   /api/rides/:id/complete       (auth, driver-only)
GET    /api/drivers/earnings         (auth, driver-only)
```

**Middleware Applied:**
- `authMiddleware` - Verify JWT token
- `driverOnlyMiddleware` - Enforce driver role
- `riderOnlyMiddleware` - Enforce rider role

---

#### `payments.ts` (25 lines)
**Purpose:** Route definitions for payment endpoints

**Route Definitions:**
```
POST   /api/payments/initialize-paypal    (auth, rider-only)
POST   /api/payments/confirm              (auth, everyone)
GET    /api/payments/history              (auth, everyone)
POST   /api/payments/refund               (auth, rider-only)
GET    /api/payments/paypal/return        (public - webhook)
GET    /api/payments/paypal/cancel        (public - webhook)
```

---

### Configuration

#### `index.ts` (MODIFIED)
**Changes Made:**
- Added imports: `import ridesRoutes from './routes/rides'`
- Added imports: `import paymentsRoutes from './routes/payments'`
- Mounted routes: `app.use('/api/rides', ridesRoutes)`
- Mounted routes: `app.use('/api/payments', paymentsRoutes)`

---

### Documentation

#### `PHASE3_TESTING.md` (450+ lines)
**Purpose:** Comprehensive testing guide for all Phase 3 endpoints

**Sections:**
1. Setup & Prerequisites (database, test users, env vars)
2. Ride Booking Workflow Tests (fare estimation, request, accept, start, complete, cancel)
3. Payment Workflow Tests (initialize, confirm, history, refund)
4. Driver Earnings Tests
5. End-to-End Workflow Test (complete ride lifecycle)
6. Postman Collection Setup
7. Debugging & Common Issues
8. Performance Expectations

**Test Examples Provided:**
- cURL commands for each endpoint
- JSON request/response bodies
- Error cases (400, 403, 404 responses)
- Multiple test scenarios per endpoint

---

## Frontend Files

### Redux State Management

#### `redux/slices/rideSlice.ts` (MODIFIED)
**Purpose:** Redux state management for rides

**State Structure:**
```typescript
{
  currentRide: Ride | null,
  rideHistory: Ride[],
  availableRides: Ride[],
  fareEstimate: FareEstimate | null,
  isLoading: boolean,
  error: string | null
}
```

**Async Thunks (Redux to Backend):**
- `requestRide()` - POST ride request
- `getAvailableRides()` - GET nearby rides (driver)
- `acceptRide()` - Accept ride (driver)
- `startRide()` - Start ride (driver)
- `completeRide()` - Complete ride (driver)
- `cancelRide()` - Cancel ride
- `getRideDetails()` - Get ride by ID
- `getRideHistory()` - Get past rides
- `estimateFare()` - Get fare quote

**Synchronous Actions:**
- `setCurrentRide()` - Set active ride
- `updateCurrentRide()` - Update specific fields
- `clearCurrentRide()` - Reset to null
- `clearError()` - Clear error message

**Used By:** Ride screens (request, in-progress, history screens)

---

#### `redux/slices/paymentSlice.ts` (NEW)
**Purpose:** Redux state for payment operations

**State Structure:**
```typescript
{
  currentPayment: any | null,
  paymentHistory: Payment[],
  isLoading: boolean,
  error: string | null
}
```

**Async Thunks:**
- `initializePayPal()` - Start payment
- `confirmPayment()` - Complete payment
- `getPaymentHistory()` - Get payment list
- `requestRefund()` - Request refund

**Synchronous Actions:**
- `clearError()` - Clear error state
- `clearCurrentPayment()` - Reset to null

---

#### `redux/slices/locationSlice.ts` (EXISTING)
**Status:** Already implemented in Phase 2
**Purpose:** Track user location for ride pickup/dropoff
**State:** userLocation, driverLocation, isTracking

---

#### `redux/store.ts` (MODIFIED)
**Changes:**
- Imported `paymentReducer` from paymentSlice
- Added to store: `payment: paymentReducer`

**Updated Store Configuration:**
```typescript
configureStore({
  reducer: {
    auth: authReducer,
    ride: rideReducer,
    location: locationReducer,
    payment: paymentReducer,  // NEW
  },
})
```

---

### Services Layer

#### `services/ride.ts` (NEW)
**Purpose:** API client for ride operations (axios wrapper)

**Methods:**
- `requestRide(token, coordinates, addresses)`
- `getAvailableRides(token, latitude, longitude, radius)`
- `acceptRide(token, rideId)`
- `startRide(token, rideId)`
- `completeRide(token, rideId, distance, duration)`
- `cancelRide(token, rideId, reason)`
- `getRideDetails(token, rideId)`
- `getRideHistory(token, limit)`
- `estimateFare(coordinates)`

**Usage in Components:**
```typescript
import { RideService } from '../services/ride';

const { ride, fareEstimate } = await RideService.requestRide(
  token, 
  19.0760, 72.8738,  // pickup
  19.1136, 72.8326,  // dropoff
  'Location A', 'Location B'
);
```

---

#### `services/payment.ts` (NEW)
**Purpose:** API client for payment operations

**Methods:**
- `initializePayPal(token, rideId)` - Start payment
- `confirmPayment(token, rideId, transactionId)` - Complete payment
- `getPaymentHistory(token, limit)` - Get payments
- `requestRefund(token, rideId, amount, reason)` - Request refund

---

#### `services/auth.ts` (EXISTING)
**Status:** Implemented in Phase 2
**Purpose:** Authentication service (signup, login, OTP)

---

#### `services/api.ts` (EXISTING)
**Status:** Implemented in Phase 2
**Purpose:** Axios HTTP client with auth interceptors

---

#### `services/socket.ts` (EXISTING)
**Status:** Implemented in Phase 2
**Purpose:** Socket.io client wrapper (used in Phase 4)

---

### Type Definitions

#### `services/rideTypes.ts` (NEW)
**Purpose:** TypeScript interfaces for ride data

**Types:**
```typescript
interface Ride {
  id: number
  rider_id: number
  driver_id?: number
  pickup_latitude, pickup_longitude: number
  pickup_address?: string
  dropoff_latitude, dropoff_longitude: number
  dropoff_address?: string
  status: 'pending' | 'accepted' | 'in_progress' | 'completed' | 'cancelled'
  estimated_fare?: number
  actual_fare?: number
  distance_km?: number
  duration_minutes?: number
  payment_status: 'pending' | 'completed' | 'failed'
  created_at: string
  accepted_at?: string
  started_at?: string
  completed_at?: string
}

interface FareEstimate {
  distance_km: number
  estimated_duration_minutes: number
  base_fare: number
  distance_fare: number
  time_fare: number
  total_fare: number
  currency: string
}
```

---

#### `types/auth.ts` (EXISTING)
**Status:** Implemented in Phase 2
**Purpose:** Authentication types (User, Driver, AuthPayload)

---

### Entry Points

#### `App.tsx` (EXISTING)
**Status:** Splash screen only
**Purpose:** App initialization
**Next Phase:** Add navigation stacks (Phase 5)

---

## Project Root Files

### Documentation

#### `PHASE3_SUMMARY.md` (NEW - THIS FILE)
**Purpose:** Complete Phase 3 documentation
**Sections:**
- Overview of Phase 3 features
- Backend implementation details (models, services, controllers, routes)
- Frontend implementation (Redux, services, types)
- API endpoints summary (15 total)
- Testing information
- File statistics
- Security features
- Performance optimizations
- Phase 3 completion checklist
- What's ready for Phase 4

---

#### `README.md` (TOP LEVEL)
**Status:** Already created in Phase 1
**Purpose:** Project overview, tech stack, project structure
**Updates Needed:** Add Phase 3 endpoints list

---

#### `SETUP_INSTRUCTIONS.md` (TOP LEVEL)
**Status:** Created in Phase 2
**Purpose:** Prerequisites and setup guide
**Content:** Node.js, PostgreSQL, Redis installation

---

#### `PHASE2_SUMMARY.md` (TOP LEVEL)
**Status:** Completed in Phase 2
**Purpose:** Phase 2 implementation summary

---

#### `PHASE2_TESTING.md` (TOP LEVEL)
**Status:** Completed in Phase 2
**Purpose:** Phase 2 testing guide

---

#### `PHASE2_FILES_REFERENCE.md` (TOP LEVEL)
**Status:** Completed in Phase 2
**Purpose:** Phase 2 files navigation guide

---

## Quick Navigation by Role

### If You're a **Backend Developer**

**Start Here:**
1. `src/models/Ride.ts` - Understand ride data model
2. `src/services/RideService.ts` - Learn fare calculation
3. `src/controllers/RideController.ts` - See HTTP handlers
4. `src/routes/rides.ts` - Route definitions

**For Payments:**
1. `src/models/Payment.ts` - Payment data model
2. `src/services/PaymentService.ts` - Payment logic
3. `src/controllers/PaymentController.ts` - Payment handlers

**For Testing:**
- `PHASE3_TESTING.md` - Complete test guide with cURL examples

---

### If You're a **Frontend Developer**

**Start Here:**
1. `redux/slices/rideSlice.ts` - Redux ride state
2. `services/ride.ts` - API wrapper for rides
3. `services/payment.ts` - API wrapper for payments
4. `services/rideTypes.ts` - TypeScript types

**When Building Screens (Phase 5):**
- Use `RideService.requestRide()` to request rides
- Dispatch `requestRide()` thunk to Redux
- Subscribe to `state.ride.currentRide` for current ride
- Subscribe to `state.payment.currentPayment` for payment state

---

### If You're **Testing**

**Manual Testing:**
1. Read `PHASE3_TESTING.md` - Sections 1-7
2. Use Postman collection (provided in testing guide)
3. Or run cURL commands (provided in guide)

**Test Scenarios:**
- Fare estimation (various distances)
- Complete ride lifecycle (request → payment)
- Payment workflows
- Error cases

---

## File Structure Summary

```
backend/
├── src/
│   ├── models/
│   │   ├── Ride.ts              ← Ride model (NEW)
│   │   ├── Payment.ts           ← Payment model (NEW)
│   │   └── User.ts              (Existing)
│   ├── services/
│   │   ├── RideService.ts       ← Fare calculation (NEW)
│   │   ├── PaymentService.ts    ← Payment processor (NEW)
│   │   └── AuthService.ts       (Existing)
│   ├── controllers/
│   │   ├── RideController.ts    ← Ride handlers (NEW)
│   │   ├── PaymentController.ts ← Payment handlers (NEW)
│   │   └── AuthController.ts    (Existing)
│   ├── routes/
│   │   ├── rides.ts             ← Ride routes (NEW)
│   │   ├── payments.ts          ← Payment routes (NEW)
│   │   └── auth.ts              (Existing)
│   └── index.ts                 ← Updated with new routes
└── PHASE3_TESTING.md            ← Complete test guide (NEW)

frontend/
├── src/
│   ├── redux/
│   │   ├── slices/
│   │   │   ├── rideSlice.ts           ← Updated with thunks
│   │   │   ├── paymentSlice.ts        ← Payment state (NEW)
│   │   │   └── ...
│   │   └── store.ts                   ← Updated with payment reducer
│   ├── services/
│   │   ├── ride.ts                    ← Ride API wrapper (NEW)
│   │   ├── payment.ts                 ← Payment API wrapper (NEW)
│   │   └── ...
│   └── ...
└── ...

root/
├── PHASE3_SUMMARY.md                  ← This document (NEW)
└── ...
```

---

## Key Implementation Details

### Fare Calculation Example
```typescript
// Input
distance = 3.5 km
duration = 14 minutes

// Calculation
base = $2.00
distance_fare = 3.5 × $1.50 = $5.25
time_fare = 14 × $0.25 = $3.50
total = $2.00 + $5.25 + $3.50 = $10.75
final = max($10.75, $3.00) = $10.75

// Output
{
  "distance_km": 3.5,
  "estimated_duration_minutes": 14,
  "base_fare": 2.0,
  "distance_fare": 5.25,
  "time_fare": 3.5,
  "total_fare": 10.75,
  "currency": "USD"
}
```

### Ride Lifecycle State Diagram
```
pending (rider requests)
   ↓
accepted (driver accepts)
   ↓
in_progress (driver starts)
   ↓
completed (driver finishes)
   ↓
payment (rider pays)

At any point → cancelled (rider or driver cancels)
```

### API Payment Flow
```
1. POST /payments/initialize-paypal
   → Returns PayPal payment ID & approval URL

2. User approves payment on PayPal (or mock approves)

3. POST /payments/confirm
   → Confirms transaction, updates ride payment_status

4. Payment recorded in database
```

---

## What's Next?

**Phase 4 (Real-time & Notifications):**
- Socket.io integration for live updates
- Location streaming from driver to rider
- Real-time ride status notifications
- Notification service (in-app + push)

**Phase 5 (UI Screens):**
- Rider home screen (map, ride request)
- Driver home screen (available rides list)
- Ride in-progress screen
- Payment confirmation screen
- Ride history screen

---

## Support Quick Links

**Testing:** See `PHASE3_TESTING.md`
**API Docs:** All endpoints documented above and in controllers
**Types:** See `services/rideTypes.ts` for TypeScript interface definitions
**Services:** See `services/ride.ts` and `services/payment.ts` for API usage
**Redux:** See Redux slices for state management patterns

---

*Last Updated: January 15, 2024*
*Phase 3 Status: ✅ COMPLETE*
