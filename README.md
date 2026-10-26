# ChargeShare: "Stranded? Find a charge."

A peer-to-peer emergency EV charging platform built for community resilience. When EV drivers find themselves stranded with critical battery levels, ChargeShare connects them with nearby residents and businesses willing to share their verified EV charging points.

---

## ⚡ Architecture Overview

- **Frontend**: React 19 + Vite, Tailwind CSS + Vanilla CSS visual tokens, React Router v7, Leaflet & React-Leaflet dark basemap tiles, Lucide-React icons.
- **Database & Auth**: Supabase (PostgreSQL 15+, Row Level Security, Realtime subscriptions, PostGIS, Storage buckets).
- **AI Service**: Lightweight Express.js backend bridging to local Ollama running Google's `gemma4:latest` model on Windows.
- **Compatibility Engine**: Pure, deterministic, zero-dependency unit-tested JavaScript engine.
- **Privacy & Safety First**: Exact charger coordinates rounded to ~300-500 m in public views; contact details protected; conservative compatibility rules preventing unsafe connections; drivers must confirm safety disclaimer prior to request dispatch.
- **Verification Policy**: The "Verified" badge indicates an administrator reviewed the host's equipment nameplate photo and safety declaration. It does **not** constitute an electrical code inspection or regulatory certification.

---

## 🔒 Verification & Compliance Policy

> [!IMPORTANT]
> **What "Verified" Means:**
> In ChargeShare, the "Verified" badge means that an administrator inspected the host's submitted equipment nameplate photo and safety declaration in the admin portal (`/admin`). It does **not** replace a certified electrical inspection, local utility permission, or official equipment certification.

---

## 🌐 Geolocation & HTTPS Requirements

> [!NOTE]
> **Browser Geolocation Requirements:**
> High-accuracy GPS positioning (`navigator.geolocation`) works exclusively in secure contexts:
> - `http://localhost:5173` (Local development)
> - `https://...` (Production deployment with SSL/TLS)
> If geolocation is denied or unavailable, ChargeShare provides a manual place search powered by OpenStreetMap's Nominatim geocoding service (debounced ≥1s with full attribution).

---

---

## 🎛️ Mode Switch: Demo Mode vs Production Mode

ChargeShare includes a seamless mode switch allowing the application to be evaluated immediately without authentication, or connected to live Supabase backend infrastructure:

| Mode | Variable Setting | Behavior |
|---|---|---|
| **Demo Mode (Default)** | `VITE_DEMO_MODE=true` (client)<br>`DEMO_MODE=true` (server) | **Zero login required.** The app opens immediately. "Find charging" opens `/driver`; "Become a host" opens `/host`. Fictional chargers dynamically center on the user's location, emergency request lifecycle and messaging function locally, and Gemma AI runs without a token. Prominent `DEMO` badge and yellow banners identify simulated data. |
| **Production Mode** | `VITE_DEMO_MODE=false` (client)<br>`DEMO_MODE=false` (server) | **Strict Supabase Auth.** Protected routes (`/driver`, `/host`, `/charger/:id`, `/request/:id`) require authentication and redirect unauthenticated users to `/login`. Live database queries, RLS policies, storage buckets, and Bearer token verification on `/api/chat` are enforced. |

> [!TIP]
> **Default Behavior:** If `VITE_DEMO_MODE` or `DEMO_MODE` are unset in your `.env` files, ChargeShare defaults to **Demo Mode** (`true`), ensuring immediate hackathon usability out-of-the-box.
>
> Remember to restart the Vite and Express servers whenever you change `.env` values, as Vite only reads environment variables at launch.

---

## 🚀 Quickstart & Setup (Windows)

### Prerequisites
1. **Node.js**: v18+ or v20+ (`node --version`)
2. **Git**: Installed and available in PATH (`git --version`)
3. **Ollama**: Installed from [ollama.com](https://ollama.com) on Windows

### 1. Ollama & Gemma Setup
Verify Ollama is running and download the Gemma model:
```powershell
# Verify Ollama service
ollama list

# Pull the Gemma model (if not already downloaded)
ollama pull gemma4:latest

# Verify Ollama HTTP API
curl http://localhost:11434/api/tags
```

### 2. Environment Variables Configuration

ChargeShare uses environment files in `client/` and `server/`.

#### Client Configuration (`client/.env`)
```ini
# ChargeShare Mode: 'true' enables demo mode (defaults to true when unset)
VITE_DEMO_MODE=true

# Real Supabase credentials (REQUIRED when VITE_DEMO_MODE=false)
VITE_SUPABASE_URL=https://your-project.supabase.co
VITE_SUPABASE_ANON_KEY=your-anon-key-here
VITE_AI_API_URL=http://localhost:3001
VITE_MAP_TILE_URL=https://{s}.basemaps.cartocdn.com/dark_all/{z}/{x}/{y}{r}.png
VITE_MAP_ATTRIBUTION=&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors &copy; <a href="https://carto.com/attributions">CARTO</a>
```

#### Server Configuration (`server/.env`)
```ini
# Server Mode: 'true' enables demo mode (defaults to true when unset)
DEMO_MODE=true

PORT=3001
OLLAMA_HOST=http://localhost:11434
OLLAMA_MODEL=gemma4:latest
CLIENT_ORIGIN=http://localhost:5173
SUPABASE_URL=https://your-project.supabase.co
SUPABASE_ANON_KEY=your-anon-key-here
```

### 3. Database & Storage Setup (Supabase)
1. Open the SQL Editor in your Supabase dashboard.
2. Execute the schema from [supabase/schema.sql](file:///d:/ChargeSharehackathon/supabase/schema.sql).
3. This creates all tables (`profiles`, `chargers`, `vehicles`, `charging_requests`, `messages`), the `chargers_public` view, row-level security (RLS) policies, and the private `charger-evidence` storage bucket.

### 4. Admin Role Assignment (Manual SQL)
To designate an administrator who can approve or reject submitted chargers at `/admin`:
```sql
UPDATE profiles SET is_admin = true WHERE email = 'admin@example.com';
```

### 5. Optional Seed Data (Manual Only)
No fictional chargers appear in the app by default. To seed test chargers manually for local evaluation:
```powershell
cd client
npm run seed:dev
```

---

## 💻 Running the Application

### Start Backend AI Server
```powershell
cd server
npm install
npm start
# Server listens at http://localhost:3001
```

### Start Frontend Dev Server
```powershell
cd client
npm install
npm run dev -- --port 5173
# Web client opens at http://localhost:5173
```

### Run Automated Tests & Build
```powershell
cd client

# Run all 25 unit tests (Compatibility, Haversine, Search, Transitions)
npm test

# Run production build
npm run build
```

---

## 🗺️ Product Routes & Role Access

| Route | Role / Protection | Description |
|---|---|---|
| `/` | Public | Public landing page with working CTAs, nav, and floating AI chat |
| `/login` | Public | Sign in with `returnTo` redirect preservation |
| `/signup` | Public | Role selection card: Driver vs Host |
| `/signup/driver` | Public | 2-step driver onboarding: Account + EV vehicle specifications |
| `/signup/host` | Public | 5-step host onboarding wizard: Specs, suitability, photo evidence, declaration, location |
| `/forgot-password` | Public | Dispatch password recovery link via Supabase Auth |
| `/driver` | Protected (`driver` \| `both` \| `admin`) | Driver search dashboard: GPS location, Nominatim search, radius slider, compatibility legend |
| `/host` | Protected (`host` \| `both` \| `admin`) | Host dashboard: Listed chargers, verification statuses, availability toggle, incoming requests |
| `/charger/:id` | Protected | Public charger details, host claims vs conservative compatibility check, request form |
| `/request/:id` | Protected | Direct coordinator chat with realtime updates, terms negotiation, unlocked location upon acceptance |
| `/admin` | Protected (`profiles.is_admin = true`) | Admin verification queue: Inspect photo evidence, view declaration timestamp, approve or reject |
| `/ai` | Protected | Full-page standalone AI assistant (in addition to floating drawer) |

---

## 🧪 AI Assistant Verification
- **Model**: `gemma4:latest` running locally via Ollama on Windows.
- **Health Check**: `GET http://localhost:3001/api/health` reports status, Ollama liveness, and model availability.
- **Chat Endpoint**: `POST http://localhost:3001/api/chat` coordinates questions with user vehicle/host context, strictly governed by safety instructions preventing hallucination of non-existent chargers.
