# Contributing to ChargeShare: 4-Person Engineering Split

To maximize velocity during hackathons and parallel development, the ChargeShare codebase is partitioned into four clear, decoupled ownership domains with minimal merge conflict overlap:

---

## 👥 Module Ownership & Division of Responsibilities

### Role 1: Frontend & Interactive Map Engineer
- **Primary Files**:
  - `client/src/pages/LandingPage.jsx`
  - `client/src/pages/DriverDashboard.jsx` (MapContainer, TileLayer, radius circle, user geolocation)
  - `client/src/components/Footer.jsx`
  - `client/src/styles/` & `client/src/index.css` (Visual system tokens, glassmorphic cards, CSS animations, responsive layouts)
- **Core Responsibilities**:
  - Landing page CTA experience and responsive layouts across 1440px, 768px, and 390px viewports.
  - Leaflet / React-Leaflet integration, custom marker clusters, radius rings, and map recentering.
  - Mobile-first responsiveness, accessibility (ARIA attributes, keyboard navigation, focus states).
  - Micro-animations, CSS transitions, and smoke media presentation.

---

### Role 2: Host & Vehicle Lifecycle Engineer
- **Primary Files**:
  - `client/src/pages/HostDashboard.jsx`
  - `client/src/pages/ChargerDetail.jsx`
  - `client/src/components/VehicleForm.jsx`
  - `client/src/hooks/useAuth.jsx`
  - `client/src/pages/AuthPage.jsx`
- **Core Responsibilities**:
  - Host charger management: create, edit, pause, and delete charging stations.
  - Vehicle management: preset EV selection (Tata Nexon, MG ZS, Tiago EV, etc.) with custom editable specs.
  - Charging request submission flow with proposed price offers and safety confirmations.
  - Multi-vehicle support and host availability scheduling.

---

### Role 3: Database, Security & Realtime Architect
- **Primary Files**:
  - `supabase/schema.sql`
  - `supabase/seed.sql`
  - `client/src/lib/supabase.js`
  - `client/src/pages/RequestDetail.jsx` (Realtime channel subscription & fallback polling)
- **Core Responsibilities**:
  - PostgreSQL schema design, extensions (`uuid-ossp`, `postgis`), and indexes.
  - Strict Row-Level Security (RLS) policies protecting driver and host privacy.
  - Database triggers and functions (`trg_status_transition`, `handle_new_user`).
  - Supabase Realtime WebSocket subscriptions for multi-party chat with 5-second polling fallback.
  - Obfuscated public view (`chargers_public`) rounding GPS coordinates to ~111m resolution.

---

### Role 4: Compatibility Engine, Local AI & Quality Assurance
- **Primary Files**:
  - `client/src/lib/compatibility.js`
  - `client/tests/compatibility.test.js`
  - `server/index.js`
  - `client/src/pages/AIChatPage.jsx`
  - `client/src/components/CompatBadge.jsx`
- **Core Responsibilities**:
  - Deterministic compatibility engine: Connector verification, AC/DC current type checks, power ratings, and charging time estimates.
  - Strict physical connector standards enforcement (e.g. Type 2 vs Bharat AC-001 adapter requirements).
  - Express.js backend bridging to local Ollama running Gemma 4.
  - Grounded AI prompt engineering preventing hallucination of specs, hosts, or unsafe electrical advice.
  - Automated unit test suite with 100% pass rate covering compatibility, Haversine formulas, search filters, and status transitions.

---

## 🛠️ Code Conventions & Safety Rules

1. **Conservative Safety**: Never assume two connectors match unless physically verified. Never output `Compatible` if power or protocol are unverified.
2. **Documented Placeholders**: Never commit real API keys, secrets, or service-role keys.
3. **Small Readable Functions**: Functions should remain modular and concise (~12 lines or fewer when practical).
4. **Verified Code Only**: Always run `npm test` and `npm run build` before pushing to `feature/chargeshare-platform`.
