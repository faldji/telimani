# Telimani - Motorcycle Ride-Sharing Platform

A full-stack mobile app for motorcycle-based ride sharing, similar to Uber but optimized for motorcycle transport. Built with React Native (iOS/Android), Node.js/Express backend, PostgreSQL, and real-time Socket.io tracking.

## Project Structure

```
telimani/
├── backend/          # Node.js/Express API server
├── frontend/         # React Native mobile app (Expo)
└── .github/          # GitHub Actions & Copilot configs
```

## Quick Start

### Prerequisites
- Node.js 18+
- npm or yarn
- PostgreSQL 12+ (for backend)
- Redis (for backend real-time features)
- Expo CLI (for frontend): `npm install -g expo-cli`

### Backend Setup
```bash
cd backend
npm install
cp .env.example .env
npm run db:migrate    # Run database migrations
npm run dev          # Start development server
```

Backend runs on `http://localhost:5000`

### Frontend Setup
```bash
cd frontend
npm install
cp .env.example .env
npm start            # Start Expo dev server
```

Scan QR code with Expo Go app (iOS/Android) to run the app.

## Architecture Overview

### Backend
- **Express Server**: REST API + WebSocket (Socket.io)
- **Authentication**: JWT + Phone OTP
- **Database**: PostgreSQL for persistent data, Redis for real-time location caching
- **Real-time**: Socket.io for location streaming and ride notifications
- **Payment**: PayPal Sandbox integration

### Frontend
- **React Native**: Cross-platform iOS/Android via Expo
- **Navigation**: React Navigation (Auth, Rider, Driver stacks)
- **State Management**: Redux Toolkit
- **Real-time**: Socket.io client for live updates
- **Maps**: react-native-maps for location visualization

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
```bash
# Backend
cd backend
npm test              # Unit & integration tests

# Frontend
cd frontend
npm test              # Component tests
```

### Database Migrations
```bash
cd backend
npm run db:migrate    # Run pending migrations
npm run db:rollback   # Rollback last batch
npm run db:seed       # Seed test data
```

### Build for Production

**Backend Deployment:**
```bash
cd backend
npm run build
npm start             # Runs compiled JavaScript
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

## API Documentation

Detailed API endpoints are documented in [backend/README.md](./backend/README.md).

## Mobile App Guide

Frontend app flow and components are documented in [frontend/README.md](./frontend/README.md).

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

### Phase 1 ✅ Complete: Project Setup
- ✅ Directory structure created
- ✅ Environment templates ready

### Phase 2 🔄 In Progress: Backend Core
- [ ] User authentication (signup, OTP, login)
- [ ] Database schema for users, drivers, rides
- [ ] API endpoints for core flows

### Phase 3: Ride & Payment Logic
- [ ] Ride request and matching
- [ ] Fare calculation
- [ ] PayPal integration

### Phase 4: Real-Time Features
- [ ] Socket.io location streaming
- [ ] Ride notifications
- [ ] Real-time matching

### Phase 5: Frontend Build
- [ ] Auth screens
- [ ] Rider home and booking flow
- [ ] Driver mode and earnings
- [ ] Real-time tracking UI

### Phase 6: Integration & Testing
- [ ] End-to-end testing
- [ ] Device testing (Android/iOS)
- [ ] Performance optimization

### Phase 7: Deployment & Launch
- [ ] Production deployment
- [ ] App Store submission
- [ ] Post-launch monitoring

## Contributing

TBD - Contributing guidelines will be added as the team grows.

## License

TBD - License information will be added.

## Support

For issues or questions:
1. Check [backend/README.md](./backend/README.md) for backend-specific questions
2. Check [frontend/README.md](./frontend/README.md) for frontend-specific questions
3. Review project plan in `.github/copilot-instructions.md`

---

**Last Updated**: March 21, 2026
