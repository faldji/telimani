# Telimani — Motorcycle Ride-Sharing Platform

A production-grade full-stack mobile app for motorcycle ride sharing, similar to Uber. Built with React Native/Expo (iOS/Android/Web), Node.js/Express/TypeScript backend, PostgreSQL, Redis, and real-time Socket.io location tracking. **All 5 development phases are complete.**

## Project Structure

```
telimani/
├── backend/          # Node.js/Express/TypeScript API server
├── frontend/         # React Native/Expo app (iOS/Android/Web)
├── docker-compose.yml
└── .github/          # Copilot instructions & configs
```

## Quick Start

### Prerequisites
- Node.js 18+
- PostgreSQL 12+
- Redis 6+
- Docker + Docker Compose (optional, backend infra only: API + Postgres + Redis)

### Quickest Start — Docker Compose (Backend Only)
```bash
docker compose up --build
docker compose exec backend npm run db:migrate
```
- Backend: `http://localhost:5000`
- PostgreSQL: `localhost:5432` (postgres / password)
- Redis: `localhost:6379`

Then run frontend locally:
```bash
cd frontend
npm install
npx expo start
```

### Manual — Backend
```bash
cd backend
npm install
cp .env.example .env   # fill in DATABASE_URL, REDIS_URL, JWT_SECRET, etc.
npm run db:migrate
npm run dev            # http://localhost:5000 with auto-reload
```

### Manual — Frontend
```bash
cd frontend
npm install
# Create .env with EXPO_PUBLIC_API_URL and EXPO_PUBLIC_SOCKET_URL
npx expo start         # Scan QR with Expo Go on phone, or press 'a'/'w'
```

## Architecture Overview

### Backend (3-Layer: Routes → Controllers → Services)
- **Express 4 / TypeScript**: REST API + WebSocket (Socket.io 4)
- **Authentication**: JWT tokens + Phone OTP (stored in Redis, 5 min TTL)
- **Database**: PostgreSQL via Knex.js migrations
- **Cache/Real-time**: Redis for driver locations, OTP codes, pub-sub
- **Payment**: PayPal Sandbox via `paypal-rest-sdk`
- **Sockets**: `/rides` and `/notifications` Socket.io namespaces

### Frontend (expo-router file-based navigation)
- **React Native 0.81.5 / Expo SDK 54**: iOS · Android · Web
- **Navigation**: expo-router 6 (`app/` directory, file-based)
- **State Management**: Redux Toolkit 2 (6 slices)
- **Real-time**: Socket.io client (socket-ride + socket-notifications services)
- **Maps**: MapLibre + OpenStreetMap tiles, OSRM route polyline support

## MVP Features

### User Management
- Phone-based signup with OTP verification
- Role selection: Rider, Driver, or Both
- Driver verification (license, vehicle info)

### Rider Features
- Browse map and request rides (pickup → dropoff)
- Real-time driver tracking during ride
- Fare estimation and payment via PayPal
- Ride history and booking confirmation

### Driver Features
- Go online/offline availability
- View nearby ride requests
- Accept and navigate to pickup location
- Real-time navigation to destination
- Earnings dashboard

### Core Features
- Real-time location tracking (10-second intervals)
- Proximity-based ride matching (5km radius)
- Fare calculation: base + per-km + per-minute
- PayPal Sandbox payment integration
- Ride notifications and status updates

## Development

### Running Tests

#### Backend Tests
```bash
# Local development
cd backend
npm test              # Unit & integration tests
npm run test:watch    # Watch mode for development

# Docker (recommended for CI/CD)
npm run test:docker   # Run tests in isolated container
```

#### Frontend Tests
```bash
cd frontend
npm test              # Component tests
```

### Database Migrations
```bash
# Local development
cd backend
npm run db:migrate    # Run pending migrations
npm run db:rollback   # Rollback last batch
npm run db:seed       # Seed test data

# Docker (when using docker-compose)
npm run db:migrate:docker   # Run migrations in container
npm run db:rollback:docker  # Rollback in container
```

### Build for Production

**Backend Deployment:**
```bash
# Local build
cd backend
npm run build
npm start             # Runs compiled JavaScript

# Docker build
npm run build:docker  # Build TypeScript in container
```

**Frontend Build:**
```bash
cd frontend
eas build --platform android  # Build APK/AAB
eas build --platform ios      # Build IPA
```

## Environment Variables

### Backend (.env)
```
NODE_ENV=development
PORT=5000
DATABASE_URL=postgresql://user:password@localhost/telimani
REDIS_URL=redis://localhost:6379
JWT_SECRET=your-secret-key
TWILIO_ACCOUNT_SID=your-twilio-sid
TWILIO_AUTH_TOKEN=your-twilio-token
PAYPAL_MODE=sandbox
PAYPAL_CLIENT_ID=your-paypal-client-id
PAYPAL_SECRET=your-paypal-secret
```

### Frontend (.env)
```
EXPO_PUBLIC_API_URL=http://localhost:5000
EXPO_PUBLIC_SOCKET_URL=http://localhost:5000
```

## Documentation

Use these as the canonical docs for each topic:

- Setup and local environment: [SETUP_INSTRUCTIONS.md](./SETUP_INSTRUCTIONS.md)
- Backend API, architecture, and runtime: [backend/README.md](./backend/README.md)
- Frontend app structure and flows: [frontend/README.md](./frontend/README.md)
- Backend testing: [backend/TESTING.md](./backend/TESTING.md)
- Frontend testing: [frontend/TESTING.md](./frontend/TESTING.md)
- Agent guidance for this repo: [.github/copilot-instructions.md](./.github/copilot-instructions.md)

## Deployment

### Prerequisites
- GitHub account (for CI/CD)
- Heroku/Railway/DigitalOcean account (for backend hosting)
- PostgreSQL and Redis managed services
- PayPal Developer Account (already set up with sandbox)
- Expo Account (for frontend OTA updates)

### Steps
1. Deploy backend to Heroku/Railway
2. Configure environment variables in production
3. Run database migrations in production
4. Build and deploy frontend to Play Store/App Store or use Expo
5. Update frontend .env with production backend URL

## Project Phases

### Phase 1 ✅ Complete: Project Scaffolding
- Directory structure, TypeScript configs, environment templates, Docker Compose

### Phase 2 ✅ Complete: Backend Auth & User Management
- Phone OTP signup, JWT auth, user/driver models, 8 auth endpoints, Redis OTP storage

### Phase 3 ✅ Complete: Ride Booking & Payments
- Ride lifecycle (request → accept → start → complete), fare calculation, PayPal sandbox, driver earnings

### Phase 4 ✅ Complete: Real-Time Socket.io
- `/rides` and `/notifications` namespaces, live driver location streaming, Redux real-time slices

### Phase 5 ✅ Complete: Frontend UI (expo-router)
- Auth screens (Landing, Phone, OTP), role-aware home tab, ride tracking, notifications, profile, driver dashboard
- Map screen (`app/map.tsx`) with MapLibre/OpenStreetMap, route drawing, and real-time driver tracking via `ride:{rideId}` rooms
- Web bundle verified clean; Android bundled successfully

### Next: Integration Testing & Production Deployment
- End-to-end test suite, production environment config, Play Store / App Store release
- [ ] App Store submission
- [ ] Post-launch monitoring

## Contributing

TBD - Contributing guidelines will be added as the team grows.

## License

TBD - License information will be added.

## Support

For issues or questions:
1. Start with [SETUP_INSTRUCTIONS.md](./SETUP_INSTRUCTIONS.md) for local environment and startup issues
2. Check [backend/README.md](./backend/README.md) for backend-specific questions and [backend/TESTING.md](./backend/TESTING.md) for backend verification steps
3. Check [frontend/README.md](./frontend/README.md) for app-specific questions and [frontend/TESTING.md](./frontend/TESTING.md) for frontend verification steps
4. Review [.github/copilot-instructions.md](./.github/copilot-instructions.md) for repo-specific agent guidance

---

**Last Updated**: March 25, 2026
