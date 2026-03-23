# Telimani — Setup Instructions

All 5 development phases are complete. Follow these steps to get the project running locally.

## Prerequisites

| Tool | Version | Check |
|---|---|---|
| Node.js | 18+ | `node --version` |
| npm | 9+ | `npm --version` |
| PostgreSQL | 12+ | `psql --version` |
| Redis | 6+ | `redis-cli ping` → `PONG` |
| Git | any | `git --version` |

---

## Option A — Docker Compose (Recommended)

Starts backend, PostgreSQL, and Redis automatically.

```bash
# From repo root:
docker compose up --build

# Once containers are healthy, run migrations:
docker compose exec backend npm run db:migrate
```

Services:
- **Backend API**: `http://localhost:5000`
- **PostgreSQL**: `localhost:5432` (user: `postgres`, password: `password`, db: `telimani`)
- **Redis**: `localhost:6379`

Then start the frontend separately (Expo needs the local network):
```bash
cd frontend
npm install
# Create .env (see below)
npx expo start
```

---

## Option B — Manual Setup

### 1. Backend

```bash
cd backend
npm install
cp .env.example .env
# Edit .env — fill in DATABASE_URL, REDIS_URL, JWT_SECRET
```

Create the database and run migrations:
```bash
createdb telimani
npm run db:migrate
```

Start dev server:
```bash
npm run dev         # Auto-reload via ts-node-dev
# → http://localhost:5000
```

Verify:
```bash
curl http://localhost:5000/health
# → {"status":"ok"}
```

### 2. Frontend

```bash
cd frontend
npm install
```

Create a `.env` file in `frontend/`:
```
EXPO_PUBLIC_API_URL=http://<your-machine-ip>:5000
EXPO_PUBLIC_SOCKET_URL=http://<your-machine-ip>:5000
# Optional — enables Google Maps on native:
EXPO_PUBLIC_GOOGLE_MAPS_API_KEY=your-key
```

> Use your LAN IP (e.g., `192.168.1.x`), not `localhost`, when testing on a real device.

Start Expo:
```bash
npx expo start       # QR code in terminal — scan with Expo Go
npx expo start -c    # Clear Metro cache if you hit bundling issues
```

Press `a` for Android emulator · `i` for iOS simulator · `w` for browser.

---

## Environment Reference

### Backend (`backend/.env`)

```
NODE_ENV=development
PORT=5000
DATABASE_URL=postgresql://postgres:password@localhost:5432/telimani
REDIS_URL=redis://localhost:6379
JWT_SECRET=change-me
JWT_EXPIRY=7d
# Twilio (leave blank to use mock OTP — code is printed to console)
TWILIO_ACCOUNT_SID=
TWILIO_AUTH_TOKEN=
TWILIO_PHONE_NUMBER=
# PayPal Sandbox
PAYPAL_MODE=sandbox
PAYPAL_CLIENT_ID=
PAYPAL_SECRET=
```

### Frontend (`frontend/.env`)

```
EXPO_PUBLIC_API_URL=http://localhost:5000
EXPO_PUBLIC_SOCKET_URL=http://localhost:5000
EXPO_PUBLIC_GOOGLE_MAPS_API_KEY=    # optional
```

---

## Useful Commands

### Backend
```bash
npm run dev          # Start with auto-reload
npm run build        # Compile TypeScript → dist/
npm start            # Run compiled output
npm test             # Jest tests
npm run db:migrate   # Apply Knex migrations
npm run db:rollback  # Revert last migration batch
npm run db:seed      # Load sample data
```

### Frontend
```bash
npx expo start           # Dev server
npx expo start -c        # Clear cache
npx tsc --noEmit         # TypeScript check (must be 0 errors)
npx expo export --platform web   # Web static export
npm run build:android    # EAS Android APK
```

---

## Troubleshooting

| Issue | Fix |
|---|---|
| `ECONNREFUSED` on backend startup | Ensure PostgreSQL + Redis are running |
| OTP not received | Leave Twilio env blank — OTP is printed to the backend console |
| App can't reach backend on phone | Use LAN IP in `.env`, not `localhost` |
| Metro bundling error: `codegenNativeCommands` | Press `w` for web is handled by `RideMap.web.tsx`; native builds are fine |
| `Attempted to navigate before mounting Root Layout` | Fixed in `app/_layout.tsx` — auth hydrates before rendering |
| TypeScript errors | Run `npx tsc --noEmit`; should be 0 errors in both projects |
