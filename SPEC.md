<USER_REQUEST>
# ROLE
You are an autonomous senior full-stack engineer working in Antigravity IDE on Windows. Build, run, test, and document ChargeShare in this workspace. Don't stop at a plan or template. Ask me only for credentials, account authorizations, or scope-changing decisions.

# PRODUCT
ChargeShare: "Stranded? Find a charge."
A peer-to-peer emergency EV charging platform. A stranded driver finds nearby hosts, sees honest compatibility labels, chats with the host, and sends a charging request. Hosts list chargers and respond. A local AI assistant explains things. This is a 24-hour hackathon: a working core journey beats breadth.

# SCOPE TIERS
MUST (finish and test first): auth, vehicle profile, host charger listing, nearby search with radius, compatibility labels, charging request lifecycle, per-request chat with realtime updates, Ollama chatbot, demo mode, landing page with working CTA and footer.
SHOULD (only after MUST passes): filters and sorting, edit/pause/delete charger, request history, multiple saved vehicles, unread indicators, smoke video, entrance animation, hover polish.
COULD (skip unless time remains): notifications table, typing/read receipts, price models beyond free/fixed/per-kWh, rate limiting.

# STACK (fixed)
React + Vite + JavaScript, Tailwind, React Router, Leaflet/OpenStreetMap. Supabase (Postgres, Auth, RLS, Realtime). Express (Node), used only for AI endpoints. Ollama with Gemma 4. No MongoDB, no paid APIs.

# ARCHITECTURE (decided, don't redesign)
1. The frontend uses Supabase directly for CRUD and realtime, protected by RLS. The service-role key is never in the frontend or the Express runtime (dev seed script only).
2. Express exposes only GET /api/health and POST /api/chat. /api/chat verifies the user's Supabase access token, reads data using THAT TOKEN so RLS applies, and passes real results as context to Gemma.
3. Request transitions are enforced in Postgres (trigger or RPC): pending → accepted → completed, plus declined and cancelled. Negotiation happens in chat. The request has proposed_price and agreed_price (set when the host accepts); never conflate them. The AI never changes status.
4. Tables: profiles, vehicles, chargers, charging_requests, messages (one chat per request). RLS: owners edit only their own rows; messages are readable and writable only by the request's driver and the charger's owner; no broad public read. Public search uses a view with minimum fields and coordinates rounded to ~500 m. Never expose exact addresses, phone numbers, or emails by default.
5. The compatibility function is a pure, unit-tested JS module shared by the UI and the AI context. Gemma never decides compatibility.

# COMPATIBILITY ENGINE
Compare connector, AC/DC, standard, vehicle max charge power vs charger output, and availability. Output one of: Compatible / Potentially compatible (needs confirmation) / Not compatible / Insufficient information, plus a one-line reason. Never output Compatible if any known mismatch exists. Treat connector shape, protocol, and power limits as separate checks; never imply an adapter guarantees compatibility. Show charging time only when battery capacity and verified power are both known.
Vehicle presets (e.g. Tata Nexon EV, Tiago EV, MG ZS EV, Comet EV, Mahindra XUV400, Hyundai Kona, BYD Atto 3) provide model names only; all specs default to "unknown" and are user-editable. Do not invent specs from memory. Connector options: CCS2, Type 2, Bharat AC-001, GB/T, 15A 3-pin (light EVs), default unknown. Currency INR.

# PAGES
Landing, Auth, Driver dashboard (vehicle, search + map + list, requests), Host dashboard (listings, incoming requests), Request detail with chat, AI assistant panel. One account can be both driver and host. Every visible button must work. Include loading, empty, and error states, accessible forms, mobile-first.
Demo mode: seed script plus local fictional data when Supabase credentials are missing. Clearly label fictional hosts and chargers. Never show fictional data as real availability.

# VISUAL SYSTEM (landing page and app shell)
Pure black page, one framed CTA card filled with slowly swirling smoke, white headline, dark email pill with a lime button, minimal footer. Implement as React components (LandingCta.jsx, Footer.jsx) with one plain CSS file for scaling and motion; Tailwind elsewhere.

Tokens: page #000; cards #0B0B0C with a 1px border at 15% white; text white; placeholder #808081; dividers white at 20%; accent gradient #C6F432 → #84CC16 with button text #0A0A0A; card border a lime gradient from ~67% to ~29% opacity. Font: Inter (variable) with a system-ui fallback.

Media: copy these into client/public/media/ and do not hotlink. If you can't download them, say so and use a CSS-only smoke placeholder.
- Plate: https://d2ol7oe51mr4n9.cloudfront.net/user_38xzZboKViGWJOttwIXH07lWA1P/03c58212-3a59-4c80-b19d-30d05a174563.webp
- Video: https://d2ol7oe51mr4n9.cloudfront.net/user_38xzZboKViGWJOttwIXH07lWA1P/f6f4ec49-d5dd-4893-8eb4-fa0f932aceaf.mp4
They are coral. Recolor with `filter: hue-rotate(65deg)` as a starting point and tune by eye until the smoke reads lime, not yellow or muddy. Report them as recolored placeholders. Video: muted, playsInline, loop, aria-hidden, poster = plate, object-fit cover anchored right bottom, 80% opacity, fade in on loadeddata, hidden under prefers-reduced-motion.

Scale: desktop sizes in reference px of a 1440 frame (--u = 100cqw / 1440); the card also respects --v = min(100cqw / 1240, 100dvh / 720) so it fits one screen. Tablet (768) and phone (390) use their own reference boxes and stack the footer.

Hero: headline, one line of copy, "Find charging" and "Become a host" buttons over the still plate (no second video).

CTA card (below hero): ~1380 × 340u, 20u radius. Centered white headline ~34u semibold, "Stranded? Find a charge." Two-line sub: "Find nearby hosts, check charger compatibility, and send a request, all in one place." Email pill ~460u wide, placeholder "Enter your email to get started", inset lime button "Get started free" with a soft inner highlight. It must work: validate the email, then route to /signup with it prefilled. No fake waitlist, counts, or customer claims.

Footer: left, a simple ChargeShare SVG mark plus "Peer-to-peer emergency EV charging. Hosts share chargers, drivers get back on the road. Always confirm compatibility with your vehicle manufacturer and the host." Right, "Contact" with one placeholder email (hello@chargeshare.example, documented as a placeholder). No invented phone or address. Thin rule, then "© ChargeShare. 2026. Hackathon demo." and "Demo data is fictional." Use a slim footer on app pages.

Motion (CSS only, ~1.4s, once): card border draws, title rises, copy fades, form expands from center, button pops, footer mark fades. Hover: input border brightens; button lifts 1px with a warmer glow; footer links underline. prefers-reduced-motion: static, no video.

# SAFETY
Show a visible warning that compatibility must be confirmed with the vehicle manufacturer and the host. Require driver confirmation before a request is sent. Never suggest improvised wiring, bypassing protections, or unsafe adapters. Never guarantee charge duration or battery gain. If no safe option is verified, point to roadside assistance.

# AI CHATBOT
All answers must be generated by Gemma through Ollama. No canned, templated, or keyword-matched answers, and no fake fallback answers: when Ollama is down, show an app-level error outside the message history.
Verify the installed model with `ollama list` and the Ollama library; pull at most one (try gemma4:e2b, smaller if too slow); document the exact pull command.
System prompt: "You are ChargeShare AI, an EV roadside assistance and charging coordination assistant. Be concise and empathetic. Use only the context provided; say when information is missing. Never invent hosts, availability, prices, distances, specs, or compatibility. Never suggest unsafe electrical workarounds. Compatibility must be confirmed with the manufacturer and host. If no safe option is verified, recommend roadside assistance. Never claim a request was sent, accepted, or completed unless the backend confirmed it."
Backend: validate input (non-empty, ~1000 chars max), timeout, controlled errors, body-size limit, CORS to the frontend origin only, Ollama never public. UI: session history, loading indicator, Enter to send, send disabled while loading, retry, "runs locally" label, safe text rendering. Don't store chat content.

# CODE STYLE
Small readable functions (~12 lines or fewer when practical), no needless abstractions, structure: client/, server/, supabase/schema.sql, supabase/seed.sql. Security and correctness beat brevity.

# METHOD
Inspect the workspace and tools first (Windows, Node, Ollama, model, Git). Reuse existing code. Write a short checklist, then build. Pass each phase's CHECK before moving on and fix errors yourself.
P1 Scaffold + demo-mode UI + landing page (card, working CTA, footer). CHECK: build passes; landing, driver, and host views render at 1440, 768, and 390 widths with no horizontal scroll or console errors (screenshot them).
P2 schema.sql + RLS + transition trigger + seed. CHECK: SQL reviewed for policy gaps; live-applied only if credentials exist, else BLOCKED.
P3 Compatibility + nearby search. CHECK: unit tests for Haversine, radius filter, unavailable hosts excluded, each of the 4 labels, no false "Compatible".
P4 Requests + chat + realtime. CHECK: transition tests (legal and illegal); another user can't read my chat (policy review, plus live test if credentials exist); polling fallback works.
P5 Ollama + /api/chat + chatbot UI. CHECK: /api/health reports Ollama; real Gemma reply in the browser; empty and over-long input rejected; Ollama stopped gives a controlled error.
P6 SHOULD-tier polish (video, animation, hover), full-journey test, README, final report.
Never claim something works unless you ran it. Mark anything needing my credentials as BLOCKED with the minimum steps to unblock.

# GIT
Inspect first. Work on feature/chargeshare-platform with small commits. Never commit .env or keys; scan before each commit. Use GitHub CLI only if installed, authenticated, and I've approved it. No force-push. Never claim a push or PR exists unless verified.

# DELIVERABLES
README (Windows prerequisites, Ollama + Gemma setup and verification, Supabase setup and schema, env vars, run and test commands, API examples, demo steps, security notes, limitations, troubleshooting, real vs demo functionality, media recolor note, placeholder email note). .env.example files for client and server (placeholders only; client: VITE_SUPABASE_URL, VITE_SUPABASE_ANON_KEY, VITE_AI_API_URL; server: PORT, OLLAMA_HOST, OLLAMA_MODEL, CLIENT_ORIGIN, SUPABASE_URL, SUPABASE_ANON_KEY). CONTRIBUTING.md with the 4-person split: (1) frontend/map, (2) host/vehicle features, (3) schema/RLS/realtime, (4) compatibility/AI/testing.

# FINAL REPORT
Features completed per tier; files changed; start commands; Ollama/Gemma status; Supabase status (live or demo); tests actually run with results; screenshots checked; Git state actually verified; remaining steps needing my input; known limitations.
</USER_REQUEST>
<ADDITIONAL_METADATA>
The current local time is: 2026-10-26T10:43:21+05:30.
</ADDITIONAL_METADATA>
<USER_SETTINGS_CHANGE>
The user changed setting `Model Selection` from None to Claude Sonnet 4.6 (Thinking). No need to comment on this change if the user doesn't ask about it. If reporting what model you are, please use a human readable name instead of the exact string.
</USER_SETTINGS_CHANGE>