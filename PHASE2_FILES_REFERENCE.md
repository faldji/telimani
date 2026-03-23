> Historical implementation reference — see root `README.md` for current project status.

# Phase 2 Files Reference Guide

## Quick Navigation

### Backend Core Authentication Files

**Database Configuration:**
- [backend/src/config/database.ts](backend/src/config/database.ts) - PostgreSQL connection pool
- [backend/src/config/redis.ts](backend/src/config/redis.ts) - Redis client for OTP storage

**Models (Database Layer):**
- [backend/src/models/User.ts](backend/src/models/User.ts) - User and Driver database models with CRUD operations

**Services (Business Logic):**
- [backend/src/services/AuthService.ts](backend/src/services/AuthService.ts) - Core authentication logic (OTP, JWT, password hashing)

**Controllers (HTTP Handlers):**
- [backend/src/controllers/AuthController.ts](backend/src/controllers/AuthController.ts) - Request handlers for all auth endpoints

**Routes:**
- [backend/src/routes/auth.ts](backend/src/routes/auth.ts) - Auth API route definitions

**Middleware:**
- [backend/src/middleware/auth.ts](backend/src/middleware/auth.ts) - JWT verification and role-based access control
- [backend/src/middleware/errorHandler.ts](backend/src/middleware/errorHandler.ts) - Error handling and 404 responses
- [backend/src/middleware/validation.ts](backend/src/middleware/validation.ts) - Request validation rules

**Utilities:**
- [backend/src/utils/logger.ts](backend/src/utils/logger.ts) - Winston logger configuration
- [backend/src/utils/response.ts](backend/src/utils/response.ts) - Standard API response formatting

**Main Entry Point:**
- [backend/src/index.ts](backend/src/index.ts) - Express server setup with all middleware and routes

**Configuration:**
- [backend/knexfile.js](backend/knexfile.js) - Knex.js migration configuration
- [backend/.env.example](backend/.env.example) - Environment variables template

**Database:**
- [backend/migrations/001_initial_schema.js](backend/migrations/001_initial_schema.js) - Database schema migration

### Frontend Authentication Files

**Services:**
- [frontend/src/services/auth.ts](frontend/src/services/auth.ts) - AuthService wrapper for API calls from React Native

**Redux Store:**
- [frontend/src/redux/slices/authSlice.ts](frontend/src/redux/slices/authSlice.ts) - Auth state management with Redux Toolkit

**Type Definitions:**
- [frontend/src/types/auth.ts](frontend/src/types/auth.ts) - TypeScript interfaces for auth (User, Driver, AuthPayload, etc.)

**Configuration:**
- [frontend/.env.example](frontend/.env.example) - Environment variables template

### Documentation & Testing

**Testing & Documentation:**
- [backend/PHASE2_TESTING.md](backend/PHASE2_TESTING.md) - Comprehensive Phase 2 testing guide with cURL examples, Postman instructions
- [PHASE2_SUMMARY.md](PHASE2_SUMMARY.md) - Complete Phase 2 implementation summary in root

---

## File Count Summary

| Category | Count |
|----------|-------|
| Backend Services | 1 |
| Backend Models | 1 |
| Backend Controllers | 1 |
| Backend Routes | 1 |
| Backend Middleware | 3 |
| Backend Utils | 2 |
| Backend Config | 2 |
| Backend Migrations | 1 |
| Backend Main | 1 |
| Frontend Services | 1 |
| Frontend Redux | 1 |
| Frontend Types | 1 |
| Documentation | 3 |
| **Total Files** | **23** |

---

## Key Implementation Details

### Authentication Flow (Frontend → Backend)

```
1. User enters phone number
   ↓
2. POST /api/auth/signup (phone)
   → Returns: OTP (in dev mode)
   ↓
3. User enters OTP
   ↓
4. POST /api/auth/verify-otp (phone, otp, role)
   → Returns: User object + JWT token
   ↓
5. Store token in AsyncStorage
   ↓
6. All subsequent requests include: Authorization: Bearer <token>
```

### Database Relationships

```
users (1) ━━━ (1) drivers
  │                │
  ├─── id ────────► user_id
  └─── phone
       email
       role: 'rider' | 'driver'
       ...
  
drivers
  ├─── license_number
  ├─── vehicle_model, color, plate
  ├─── is_verified
  ├─── is_online
  ├─── rating
  ├─── total_earnings, today_earnings
  └─── created_at, updated_at
```

### Middleware Chain

```
Request
  ↓
[CORS Middleware]
  ↓
[Helmet Security Headers]
  ↓
[Body Parser - JSON/URL-encoded]
  ↓
[Routes]
  ├─ Public routes (signup, verify-otp, login)
  │  └─ No auth middleware
  │
  └─ Protected routes
     └─ [authMiddleware - JWT verification]
        ├─ [Optional: roleMiddleware - driver/rider check]
        └─ [Handler]
  ↓
[404 Handler]
  ↓
[Error Handler]
  ↓
Response
```

---

## Testing Quick Reference

### Via cURL

```bash
# 1. Signup
curl -X POST http://localhost:5000/api/auth/signup \
  -H "Content-Type: application/json" \
  -d '{"phone": "+12345678900"}'

# 2. Verify OTP
curl -X POST http://localhost:5000/api/auth/verify-oTP \
  -H "Content-Type: application/json" \
  -d '{"phone": "+12345678900", "otp": "123456", "role": "rider"}'

# 3. Get Profile (use token from step 2)
TOKEN="eyJ..." # from verify-otp response
curl -X GET http://localhost:5000/api/auth/profile \
  -H "Authorization: Bearer $TOKEN"
```

### Via Frontend AuthService

```typescript
import authService from './services/auth';

// Signup
await authService.signup('+12345678900');

// Verify OTP
const { user, token } = await authService.verifyOTP(
  '+12345678900',
  '123456',
  'rider',
  'John',
  'Doe'
);

// Get profile
const profile = await authService.getProfile(token);

// Logout  
await authService.logout();
```

---

## Phase 2 Dependencies

Backend dependencies that were added to `package.json`:
- `jsonwebtoken` - JWT token generation and verification
- `bcryptjs` - Password hashing
- `redis` - Redis client for OTP storage
- `pg` - PostgreSQL database driver
- `knex` - Database migration tool
- `winston` - Logging
- Additional: `express`, `cors`, `helmet`, etc.

Frontend dependencies:
- `@react-redux` - Redux integration for React Native
- `@reduxjs/toolkit` - Redux state management
- `axios` - HTTP client
- `@react-native-async-storage/async-storage` - Local storage

---

## Next Steps

After Phase 2, implement Phase 3: **Ride & Payment Logic**

Key files to create in Phase 3:
- `backend/src/models/Ride.ts` - Ride model
- `backend/src/services/RideService.ts` - Ride business logic
- `backend/src/controllers/RideController.ts` - Ride handlers
- `backend/src/routes/rides.ts` - Ride endpoints
- Payment integration with PayPal

---

## Quick Start Checklist

- [ ] Install Node.js 18+
- [ ] Install PostgreSQL
- [ ] Install Redis
- [ ] Create database: `createdb telimani`
- [ ] Run migrations: `npm run db:migrate` (from backend directory)
- [ ] Update `.env` files with actual credentials
- [ ] Start backend: `npm run dev` (from backend directory)
- [ ] Test endpoints: See [backend/PHASE2_TESTING.md](backend/PHASE2_TESTING.md)
- [ ] Review code in VS Code

---

Generated: March 21, 2026
Phase Complete: ✅ Phase 2 - Backend Authentication
