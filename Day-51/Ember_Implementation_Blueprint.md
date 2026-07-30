# Ember — Implementation Blueprint (Day 2 → Day 10)

**Project:** Ember — AI-Powered Personal CRM for Founders & Freelancers
**Challenge:** AB Talks 60-Day Claude AI Challenge — 10-Day Capstone
**Stack:** React (frontend) + Node.js/Express (backend) + MongoDB (database) + Claude API (AI feature)
**Builder availability:** ~3-4 focused hours/day

> **This document is the single source of truth for the rest of the capstone.** Each day's AI conversation starts fresh — paste that day's section (plus the "Project Context" block below) at the start of the conversation so the assistant has full context without re-planning anything.

---

## 🧭 Project Context (paste this at the start of every new daily conversation)

```
Project: Ember — AI-powered personal CRM for founders/freelancers.
Core value prop: Helps users build stronger relationships consistently, stay
top-of-mind with contacts, and turn relationships into clients/partners/opportunities
without forgetting to follow up.

Stack: React + Node.js/Express + MongoDB + Claude API for AI suggestions.
Multi-user SaaS with auth and per-user data isolation.

Signature AI feature: weekly "who to reach out to and why" suggestions,
shown in-app and delivered via an automated weekly email digest (Pro tier).

Freemium model (UI-only, no real billing in v1.0):
- Free: up to 25 contacts, in-app AI suggestions only
- Pro (UI-gated, not actually charged): unlimited contacts + automated email digest

Explicitly OUT of scope for v1.0: real Stripe billing, mobile app, browser
extension, team workspaces, AI message drafting, AI note summarization, SMS/push.

We are following a fixed 10-day Implementation Blueprint. Do not redesign the
architecture, rename the product, change the stack, or add features not listed
in today's section. Follow the day's plan step by step, explaining each manual
action, and wait for my confirmation/screenshot before moving to the next step.
```

---

## 📌 How to Use This Blueprint Every Day

1. Open a fresh AI conversation.
2. Paste the **Project Context** block above.
3. Paste that day's full section below.
4. Say "Let's start Day N."
5. Follow the plan step by step. Confirm each manual step with a screenshot before moving on.
6. At the end of the day, complete the **End-of-Day Checklist** and capture the listed screenshots before closing the conversation — the next day's AI will need to know you actually finished.

---

## 🗺️ 9-Day Roadmap Overview

| Day | Theme | Outcome |
|---|---|---|
| 2 | Environment Setup & Project Scaffolding | Empty but running full-stack app (React + Express + MongoDB connected) |
| 3 | Auth & Multi-User Foundation | Working sign-up/login, protected routes, per-user data isolation |
| 4 | Contact Management Core | Full CRUD for contacts + interaction timeline |
| 5 | Dashboard & Free/Pro Limits | Dashboard UI, 25-contact cap logic, pricing page UI |
| 6 | AI Suggestions Engine (Part 1) | Claude API integration producing real "reach out this week" suggestions |
| 7 | Automated Email Digest (Part 2) | Scheduled job emailing Pro users their AI suggestions |
| 8 | Polish, Edge Cases & Styling | Production-quality UI/UX pass, error handling, empty states |
| 9 | Testing & Deployment | Full app deployed live with a public URL |
| 10 | Final QA, Demo Video & Submission | Live app verified + demo video + deliverables packaged |

---

## Day 2 — Environment Setup & Project Scaffolding

### 🎯 Objective
Stand up the full technical foundation: repo structure, React frontend, Express backend, MongoDB connection — all running locally and talking to each other — with nothing feature-specific built yet.

### 📖 What I'll Learn
- Structuring a full-stack MERN monorepo cleanly
- Setting up environment variables safely
- Establishing frontend↔backend↔database connectivity end to end

### 🛠 Features to Build
- No user-facing features yet — this is pure scaffolding.

### 📝 Step-by-Step Implementation Plan
1. Install prerequisites if missing: Node.js (LTS), npm, MongoDB (local install OR MongoDB Atlas free-tier cluster — Atlas recommended since Day 9 deployment needs a cloud DB anyway).
2. Create the project root folder `ember/` with two subfolders: `client/` (React) and `server/` (Express).
3. Scaffold the React app inside `client/` using Vite (`npm create vite@latest client -- --template react`) — faster dev server than CRA.
4. Scaffold the Express app inside `server/`: `npm init -y`, install `express`, `mongoose`, `dotenv`, `cors`, `nodemon` (dev dependency).
5. Create `server/.env` with `MONGO_URI`, `PORT=5000`, and a placeholder `JWT_SECRET` (real value later).
6. Create `server/index.js`: basic Express app, `cors()` middleware, JSON body parsing, a single test route `GET /api/health` returning `{status: "ok"}`, and Mongoose connection to `MONGO_URI`.
7. Add `"dev": "nodemon index.js"` script in `server/package.json`.
8. In `client/`, configure a `.env` with `VITE_API_URL=http://localhost:5000`.
9. In `client/src/App.jsx`, add a simple fetch to `/api/health` on load and display the result — this proves the full chain works.
10. Create a root-level `.gitignore` covering `node_modules/`, `.env`, `dist/`.
11. Initialize git in the project root, make the first commit ("Initial scaffolding").

### 📂 Files/Folders to Create
```
ember/
  client/                (Vite React app)
  server/
    index.js
    .env
    package.json
  .gitignore
  README.md
```

### 🔗 Tools/Services to Integrate
- MongoDB Atlas (free-tier cluster) — recommended over local Mongo for zero migration pain on deployment day
- Vite, Express, Mongoose, dotenv, cors, nodemon

### 🧪 Testing Tasks
- `npm run dev` in `server/` — confirm console logs "MongoDB connected" and "Server running on port 5000"
- `npm run dev` in `client/` — confirm the health-check message renders in the browser
- Hit `http://localhost:5000/api/health` directly in browser — confirm JSON response

### 🐞 Common Issues & Debugging Tips
- **CORS errors in browser console** → confirm `cors()` middleware is registered before routes in `index.js`
- **MongoDB connection timeout** → check Atlas Network Access allows your current IP (or `0.0.0.0/0` for dev)
- **"Cannot find module" errors** → confirm you ran `npm install` inside the correct subfolder (`client/` vs `server/`), not the root
- **Vite env vars not loading** → Vite requires the `VITE_` prefix; restart dev server after editing `.env`

### ✅ End-of-Day Checklist
- [ ] `client/` and `server/` both run without errors
- [ ] Browser shows successful health-check response from backend
- [ ] MongoDB Atlas cluster created and connected
- [ ] First git commit made
- [ ] `.env` files are NOT committed (verify via `git status`)

### 📸 Expected Project State & Screenshots
- Screenshot: terminal showing both servers running without errors
- Screenshot: browser showing the health-check message
- Screenshot: MongoDB Atlas dashboard showing the created cluster

### ➡️ Handoff Notes for Day 3
"Scaffolding is complete: React (Vite) frontend and Express backend are running and connected to MongoDB Atlas. No auth, no models, no UI beyond a health check exist yet. Day 3 builds the User model, JWT-based signup/login, and protected route middleware."

---

## Day 3 — Auth & Multi-User Foundation

### 🎯 Objective
Build secure multi-user authentication so each user can sign up, log in, and only ever see their own data.

### 📖 What I'll Learn
- Password hashing and secure credential storage
- JWT-based authentication and protected routes
- React auth state management (context or a lightweight store)

### 🛠 Features to Build
- Sign-up form, login form, logout
- JWT issuance and verification middleware
- Protected route pattern on both frontend and backend

### 📝 Step-by-Step Implementation Plan
1. Backend: install `bcryptjs` and `jsonwebtoken`.
2. Create `server/models/User.js` (Mongoose schema): `email` (unique), `passwordHash`, `name`, `tier` (enum: `free`/`pro`, default `free`), `createdAt`.
3. Create `server/routes/auth.js`: `POST /api/auth/signup` (hash password with bcrypt, create user, return JWT), `POST /api/auth/login` (verify password, return JWT).
4. Create `server/middleware/auth.js`: verifies `Authorization: Bearer <token>` header, attaches `req.userId`, rejects with 401 if invalid/missing.
5. Wire `auth.js` routes into `index.js` under `/api/auth`.
6. Frontend: create an `AuthContext` (React Context) storing `token` and `user`, persisted to `localStorage`.
7. Build `SignupPage` and `LoginPage` components with simple forms; on success, store token in context + localStorage, redirect to dashboard placeholder.
8. Build a `ProtectedRoute` wrapper component that redirects to `/login` if no token exists.
9. Add a basic top navbar with app name "Ember", and a Logout button that clears context/localStorage.
10. Test the full loop manually: sign up a user → confirm it appears in MongoDB Atlas → log out → log back in.

### 📂 Files/Folders to Create or Modify
```
server/models/User.js
server/routes/auth.js
server/middleware/auth.js
server/index.js                 (modify: mount /api/auth)
client/src/context/AuthContext.jsx
client/src/pages/SignupPage.jsx
client/src/pages/LoginPage.jsx
client/src/components/ProtectedRoute.jsx
client/src/components/Navbar.jsx
```

### 🔗 Tools/Libraries to Integrate
- `bcryptjs`, `jsonwebtoken` (backend)
- `react-router-dom` (frontend routing — install now if not already)

### 🧪 Testing Tasks
- Sign up with a new email → verify JWT is returned and user document exists in MongoDB
- Attempt signup with a duplicate email → confirm proper error message, not a server crash
- Log in with correct/incorrect password → confirm correct success/failure behavior
- Try accessing a protected route without logging in → confirm redirect to login
- Refresh the page while logged in → confirm session persists (token read from localStorage)

### 🐞 Common Issues & Debugging Tips
- **Passwords stored in plain text** → double check `bcrypt.hash()` is called before saving, never save `req.body.password` directly
- **Token not attaching to requests** → confirm frontend sets the `Authorization` header on every authenticated API call (consider a shared `fetch` wrapper or axios instance with an interceptor)
- **401 errors after login** → check JWT secret matches between issuing and verifying, and token hasn't expired
- **State lost on refresh** → confirm `AuthContext` reads from `localStorage` on initial mount, not just on login

### ✅ End-of-Day Checklist
- [ ] User can sign up, log in, log out
- [ ] Passwords are hashed in the database (visually confirm in Atlas — should be a bcrypt hash, not plaintext)
- [ ] Protected routes redirect unauthenticated users
- [ ] JWT persists across page refresh

### 📸 Expected Project State & Screenshots
- Screenshot: signup form filled and submitted successfully
- Screenshot: MongoDB Atlas showing the new user document with hashed password
- Screenshot: logged-in state showing navbar with Logout button

### ➡️ Handoff Notes for Day 4
"Auth is fully working: signup, login, logout, protected routes, JWT middleware on backend. `req.userId` is available in any protected route. Day 4 builds the Contact model and full CRUD UI, scoped to `req.userId` so each user only sees their own contacts."

---

## Day 4 — Contact Management Core

### 🎯 Objective
Build the heart of the product: full contact management with the extended field set and interaction timeline, fully scoped per user.

### 📖 What I'll Learn
- Designing a Mongoose schema with a nested sub-document array (interaction timeline)
- Building full CRUD REST endpoints
- Building a real, usable form-heavy React UI

### 🛠 Features to Build
- Add / view / edit / delete contacts
- Contact fields: name, how you met, relationship type, last contact date, notes, company, role, email, phone, LinkedIn URL
- Interaction timeline: add a log entry (type: call/email/meeting, date, short note) per contact, displayed chronologically

### 📝 Step-by-step Implementation Plan
1. Create `server/models/Contact.js`: `userId` (ref, indexed), `name`, `howMet`, `relationshipType` (enum: client/partner/investor/friend/other), `lastContactDate`, `notes`, `company`, `role`, `email`, `phone`, `linkedinUrl`, `interactions` (array of sub-documents: `{type, date, note}`), `createdAt`.
2. Create `server/routes/contacts.js` (all routes protected by auth middleware, all queries filtered by `userId: req.userId`):
   - `GET /api/contacts` (list all for user)
   - `POST /api/contacts` (create)
   - `GET /api/contacts/:id` (single contact detail, verify ownership)
   - `PUT /api/contacts/:id` (edit, verify ownership)
   - `DELETE /api/contacts/:id` (verify ownership)
   - `POST /api/contacts/:id/interactions` (add a timeline entry, also update `lastContactDate` to the interaction date)
3. Mount `contacts.js` in `index.js` under `/api/contacts`.
4. Frontend: build `ContactsListPage` — table/card view of all contacts with name, relationship type badge, last contact date, and a "days since contact" indicator.
5. Build `AddContactForm` / `EditContactForm` (can share one component) with all fields.
6. Build `ContactDetailPage`: shows full profile + chronological interaction timeline + an "Add interaction" quick form.
7. Add relationship-type color badges (client/partner/investor/friend/other) for quick visual scanning.
8. Wire up delete with a confirmation prompt.
9. Add simple client-side validation (required: name, relationship type).

### 📂 Files/Folders to Create or Modify
```
server/models/Contact.js
server/routes/contacts.js
server/index.js                        (modify: mount /api/contacts)
client/src/pages/ContactsListPage.jsx
client/src/pages/ContactDetailPage.jsx
client/src/components/ContactForm.jsx
client/src/components/InteractionTimeline.jsx
client/src/components/RelationshipBadge.jsx
```

### 🔗 Tools/Libraries to Integrate
- None new — this is core CRUD on the existing stack

### 🧪 Testing Tasks
- Create a contact with all fields filled → confirm it appears correctly in the list and detail view
- Create a contact with only required fields → confirm no crash on missing optional fields
- Add 3+ interactions to one contact → confirm they display in correct chronological order and `lastContactDate` updates
- Edit a contact → confirm changes persist
- Delete a contact → confirm it's removed from the list and can't be accessed by ID anymore
- Log in as a second test user → confirm they see zero contacts (data isolation working)

### 🐞 Common Issues & Debugging Tips
- **Contacts from all users showing up** → you forgot to filter the Mongoose query by `userId: req.userId` — check every route
- **Interaction timeline out of order** → sort by date descending (or ascending, pick one and be consistent) on the frontend or via `.sort()` in the API response
- **Ownership bypass via URL** → always verify `contact.userId.toString() === req.userId` before returning/editing/deleting on `:id` routes, not just checking the contact exists
- **Date fields showing as ugly ISO strings** → format with `toLocaleDateString()` or a small date-formatting helper on the frontend

### ✅ End-of-Day Checklist
- [ ] Full CRUD works for contacts
- [ ] Interaction timeline works and updates last contact date
- [ ] Data isolation verified with a second test user
- [ ] Relationship type badges render correctly

### 📸 Expected Project State & Screenshots
- Screenshot: contacts list with 3-5 sample contacts and relationship badges
- Screenshot: a contact detail page showing a populated interaction timeline
- Screenshot: second test user's empty contacts list (proving isolation)

### ➡️ Handoff Notes for Day 5
"Contact CRUD and interaction timeline are fully working and correctly scoped per user. Sample data exists for testing (keep at least 5-6 contacts with varied last-contact dates for Day 6's AI feature to have realistic data to reason about). Day 5 builds the dashboard, the 25-contact free-tier cap, and the pricing/paywall UI."

---

## Day 5 — Dashboard & Free/Pro Limits

### 🎯 Objective
Build the at-a-glance dashboard and implement the freemium UI: the 25-contact cap for free users and the Pro plan pricing page (UI-only, no real billing).

### 📖 What I'll Learn
- Aggregating and summarizing data for a dashboard view
- Implementing soft feature-gating logic
- Designing a pricing page that reads as a real product

### 🛠 Features to Build
- Dashboard: total contacts, contacts going cold (30+/60+ days since last contact), placeholder for "this week's AI suggestions" (real data arrives Day 6), recent interactions feed
- 25-contact cap enforcement for free-tier users, with an "Upgrade to Pro" prompt when hit
- Pricing page comparing Free vs Pro
- A (fake, local-only) "Upgrade to Pro" button that flips the user's `tier` field to `pro` in the database — simulating the upgrade without real payment

### 📝 Step-by-Step Implementation Plan
1. Backend: add `GET /api/dashboard/summary` route — returns total contact count, count of contacts with `lastContactDate` older than 30/60 days, and the 5 most recent interactions across all contacts.
2. Backend: add a check in `POST /api/contacts` — if `user.tier === 'free'` and contact count is already 25, return a 403 with a clear error message (e.g., `{error: "FREE_TIER_LIMIT"}`).
3. Backend: add `POST /api/users/upgrade` — sets `req.userId`'s `tier` to `pro` (this simulates a purchase; no payment processor involved).
4. Frontend: build `DashboardPage` with summary cards (total contacts, cold relationships, recent activity) and a placeholder "AI Suggestions" card (static "Coming online tomorrow!" text for now — replaced Day 6).
5. Frontend: build `PricingPage` — a clean two-column Free vs Pro comparison table matching the PRD's feature table, with an "Upgrade to Pro" button on the Pro column.
6. Frontend: when `POST /api/contacts` returns the `FREE_TIER_LIMIT` error, show a modal: "You've reached the Free plan's 25-contact limit" with a link to `PricingPage`.
7. Frontend: after clicking "Upgrade to Pro," call `POST /api/users/upgrade`, refresh user context, show a success state ("You're now on Pro!").
8. Add a small "Free" / "Pro" badge in the navbar reflecting current tier.

### 📂 Files/Folders to Create or Modify
```
server/routes/dashboard.js
server/routes/users.js
server/index.js                    (modify: mount new routes)
server/routes/contacts.js          (modify: add tier-limit check)
client/src/pages/DashboardPage.jsx
client/src/pages/PricingPage.jsx
client/src/components/UpgradeModal.jsx
client/src/components/SummaryCard.jsx
```

### 🔗 Tools/Libraries to Integrate
- None new

### 🧪 Testing Tasks
- With a free-tier test user, add contacts up to 25 → confirm the 26th attempt is blocked with the upgrade modal
- Click "Upgrade to Pro" → confirm tier flips in the database and the cap no longer applies
- Confirm dashboard summary numbers match actual contact data (spot-check counts manually)
- Confirm cold-relationship counts use sensible thresholds (test with a contact whose `lastContactDate` is manually set 40+ days in the past)

### 🐞 Common Issues & Debugging Tips
- **Cap check bypassed** → make sure the count check happens server-side in the route, not just hidden in the UI (UI hiding alone isn't real enforcement)
- **Dashboard numbers wrong** → double-check date math (use a library like `date-fns` if manual day-difference math gets error-prone) — install `date-fns` if needed
- **Upgrade button doesn't reflect immediately** → make sure the frontend re-fetches or updates the user object in context after the upgrade call, not just showing a toast

### ✅ End-of-Day Checklist
- [ ] Dashboard shows accurate live summary data
- [ ] Free tier cap blocks the 26th contact with a clear upgrade prompt
- [ ] Pricing page clearly compares Free vs Pro
- [ ] Upgrade flow works (simulated, no real payment)

### 📸 Expected Project State & Screenshots
- Screenshot: dashboard with summary cards populated
- Screenshot: the upgrade modal triggered at the 25-contact limit
- Screenshot: pricing page

### ➡️ Handoff Notes for Day 6
"Dashboard, free/pro tier logic, and pricing UI are complete. `user.tier` field reliably controls the 25-contact cap. The dashboard has a placeholder AI Suggestions card ready to be wired to real data. Day 6 builds the actual Claude API integration that generates the weekly outreach suggestions and replaces that placeholder."

---

## Day 6 — AI Suggestions Engine (Part 1)

### 🎯 Objective
Build Ember's signature feature: real, Claude-powered "who to reach out to this week and why" suggestions, replacing the Day 5 placeholder.

### 📖 What I'll Learn
- Calling the Claude API from a Node backend
- Prompt design for structured, reliable output
- Turning unstructured contact/interaction data into a useful AI input

### 🛠 Features to Build
- Backend endpoint that gathers a user's contact data, sends it to Claude with a carefully designed prompt, and returns 3-5 ranked outreach suggestions with reasons
- Dashboard AI Suggestions card now shows real, live suggestions
- A "Refresh suggestions" action (manual regeneration, not yet automated — automation is Day 7)

### 📝 Step-by-step Implementation Plan
1. Get familiar with Claude API basics (model choice, `messages` format, structured output via JSON) — install the Anthropic SDK (`@anthropic-ai/sdk`) in `server/`.
2. Store the Claude API key in `server/.env` as `CLAUDE_API_KEY` (never commit this file).
3. Create `server/services/aiSuggestions.js`: a function that takes a user's contact list (filtered to relevant fields: name, relationshipType, lastContactDate, days-since-contact, last 2-3 interaction notes) and builds a prompt.
4. Design the prompt to explicitly request **strict JSON output**: an array of `{contactId, contactName, reason}` objects, ranked by outreach priority, max 5 items. Include instructions to prioritize: (a) longest time since contact for high-value relationship types, (b) any unresolved thread implied by recent notes (e.g., "offered to make an intro").
5. Parse the JSON response defensively (strip markdown code fences if present, `try/catch` with a fallback of an empty array + logged error).
6. Create `GET /api/suggestions` route: fetches the user's contacts, calls `aiSuggestions.js`, returns the ranked list. Cache the result on the user document with a `suggestionsGeneratedAt` timestamp so it's not re-calling Claude on every dashboard load (reuse cached result if generated within the last 24 hours).
7. Add `POST /api/suggestions/refresh` — force-regenerates regardless of cache.
8. Frontend: replace the Dashboard's placeholder AI card with real data — each suggestion shows contact name, the AI-generated reason, and a "View contact" link plus a "Log interaction" quick action.
9. Add a "Refresh suggestions" button with a loading state (Claude calls take a few seconds).
10. Handle the empty state gracefully (fewer than 3 contacts, or no contacts old enough to flag — show an encouraging message, not an error).

### 📂 Files/Folders to Create or Modify
```
server/services/aiSuggestions.js
server/routes/suggestions.js
server/index.js                     (modify: mount /api/suggestions)
server/.env                         (add CLAUDE_API_KEY)
server/models/User.js               (modify: add cachedSuggestions, suggestionsGeneratedAt fields)
client/src/components/AISuggestionsCard.jsx
client/src/pages/DashboardPage.jsx  (modify: wire real data)
```

### 🔗 APIs/Libraries to Integrate
- **Claude API** (`@anthropic-ai/sdk`) — this is the core AI integration for the entire product
- Ensure `CLAUDE_API_KEY` is never exposed to the frontend — all Claude calls happen server-side only

### 🧪 Testing Tasks
- With your Day 4 sample contacts (varied last-contact dates), call `/api/suggestions` → confirm the response is valid JSON with sensible, non-generic reasons referencing actual notes/dates
- Test with a contact who has no interaction history yet → confirm it doesn't crash the prompt/parsing
- Test the 24-hour cache — call twice in a row, confirm the second call doesn't re-hit the Claude API (check server logs/timestamps)
- Test "Refresh" button → confirm it forces a new generation
- Test with fewer than 3 contacts → confirm graceful empty/low-data state

### 🐞 Common Issues & Debugging Tips
- **Claude returns JSON wrapped in ```json fences** → strip fences before `JSON.parse()`; consider explicitly instructing "respond with raw JSON only, no markdown" in the prompt
- **Suggestions feel generic ("reach out to stay in touch")** → your prompt likely isn't passing enough real context (interaction notes, specific dates) — feed it more specific data
- **Slow dashboard loads** → confirm caching is actually working; Claude calls should not block every page load
- **API key exposed in frontend network tab** → if you see the key in browser dev tools, the call is happening client-side by mistake — move it to the backend immediately

### ✅ End-of-Day Checklist
- [ ] Real Claude-generated suggestions appear on the dashboard
- [ ] Reasons are specific and reference actual contact data, not generic text
- [ ] Caching prevents redundant API calls
- [ ] Refresh button works
- [ ] API key is never exposed to the frontend

### 📸 Expected Project State & Screenshots
- Screenshot: dashboard showing 3-5 real AI suggestions with specific reasons
- Screenshot: server terminal log showing a successful Claude API call
- Screenshot: the empty/low-data state

### ➡️ Handoff Notes for Day 7
"The AI suggestions engine is fully working: `/api/suggestions` returns real, cached, Claude-generated outreach suggestions and the dashboard displays them. `aiSuggestions.js` is the reusable service function — Day 7 will call this same function from a scheduled backend job to power the automated weekly email digest, so it should not need to change, only be invoked differently (for all Pro users, not just the requesting user)."

---

## Day 7 — Automated Weekly Email Digest (Part 2)

### 🎯 Objective
Automate the delivery of AI suggestions via a scheduled weekly email to Pro-tier users — Ember's most "wow" feature for the pitch/demo.

### 📖 What I'll Learn
- Scheduling recurring backend jobs (cron-style) in Node
- Sending transactional emails from a backend service
- Designing a clean HTML email template

### 🛠 Features to Build
- A scheduled job that runs weekly, generates fresh AI suggestions for every Pro-tier user, and emails them a digest
- An HTML email template with Ember branding, the ranked suggestions, and reasons
- A manual "Send me a test digest now" button (for demoing/testing without waiting a full week)

### 📝 Step-by-step Implementation Plan
1. Choose and set up a free-tier-eligible transactional email service (e.g., a provider with a generous free tier — confirm current options since offerings change; do not select a paid plan).
2. Install the email service's Node SDK (or use `nodemailer` if going the SMTP route) in `server/`.
3. Store email service credentials in `server/.env` (never commit).
4. Install `node-cron` in `server/` for job scheduling.
5. Create `server/services/emailDigest.js`: a function that (a) fetches all `tier: 'pro'` users, (b) for each, calls the existing `aiSuggestions.js` service to get fresh suggestions, (c) renders an HTML email template with those suggestions, (d) sends via the email service.
6. Create a simple HTML email template (can be a template string or a small templating helper) — Ember logo/wordmark, greeting with user's name, ranked list of suggestions with reasons, a link back to the dashboard.
7. In `server/index.js` (or a new `server/jobs/scheduler.js`), schedule the job with `node-cron` (e.g., weekly on a fixed day/time) to call `emailDigest.js`.
8. Add `POST /api/suggestions/send-test-digest` — an on-demand route that triggers the digest immediately for the logged-in user only (for demo purposes, doesn't wait for the cron schedule).
9. Frontend: add a "Send me a test digest" button on the Dashboard (visible only to Pro users) that calls this route and shows a confirmation toast.
10. Test end-to-end by upgrading a test account to Pro and triggering the test digest — confirm a real email arrives.

### 📂 Files/Folders to Create or Modify
```
server/services/emailDigest.js
server/jobs/scheduler.js
server/routes/suggestions.js          (modify: add send-test-digest route)
server/index.js                       (modify: initialize scheduler on server start)
server/.env                           (add email service credentials)
client/src/pages/DashboardPage.jsx    (modify: add test digest button for Pro users)
```

### 🔗 APIs/Libraries to Integrate
- A transactional email service (free tier) + its Node SDK, or `nodemailer`
- `node-cron`

### 🧪 Testing Tasks
- Trigger the manual test digest → confirm a real email arrives in your inbox within a minute, correctly formatted, with real suggestion data
- Confirm free-tier users cannot trigger or receive the digest
- Confirm the cron schedule is correctly configured (verify the schedule expression logs the expected next run time on server start)
- Test with a Pro user who has too few contacts for suggestions → confirm the email handles this gracefully (e.g., skips sending or sends an encouraging "add more contacts" email instead of a broken one)

### 🐞 Common Issues & Debugging Tips
- **Emails landing in spam** → check the email service's sender verification/domain setup steps; this is common with free-tier transactional email and worth testing early, not on Day 9
- **Cron job runs but no email arrives** → check the email service dashboard for delivery logs/errors, not just server console logs
- **Job runs multiple times unexpectedly** → confirm the scheduler is only initialized once (guard against `nodemon` hot-reload double-registering the cron job in dev)
- **HTML email looks broken in real inboxes** → keep the template simple (inline styles, no complex CSS/flexbox — email clients render HTML inconsistently)

### ✅ End-of-Day Checklist
- [ ] Manual test digest successfully sends and arrives in inbox
- [ ] Email content matches real, current AI suggestions
- [ ] Only Pro users can trigger/receive digests
- [ ] Cron schedule is correctly configured and logged on server start

### 📸 Expected Project State & Screenshots
- Screenshot: a real received digest email, opened, showing formatted suggestions
- Screenshot: server terminal showing the scheduled job registered with its next run time
- Screenshot: the "Send test digest" button and success confirmation in the app

### ➡️ Handoff Notes for Day 8
"All core features are functionally complete: auth, contacts, dashboard, tiered limits, AI suggestions, and the automated email digest. The app works end-to-end but has not had a dedicated UI/UX polish pass yet — spacing, empty states, loading states, mobile responsiveness, and error handling are inconsistent. Day 8 is entirely about polish and robustness, not new features."

---

## Day 8 — Polish, Edge Cases & Styling

### 🎯 Objective
Take the functionally-complete app and make it look and feel like a real, launchable product — this is the day that turns "working" into "impressive."

### 📖 What I'll Learn
- Systematic UI/UX polish passes
- Defensive error handling patterns across a full-stack app
- Responsive design basics for a dashboard-style SaaS product

### 🛠 Features to Build
- No new features — hardening and polish only
- Consistent design system: colors, spacing, typography applied across every page
- Loading states, empty states, and error states for every async action
- Basic mobile/tablet responsiveness

### 📝 Step-by-step Implementation Plan
1. Define a small design system: pick a primary color palette (align with "Ember" branding — warm oranges/reds work thematically), consistent spacing scale, and font choices. Apply as CSS variables or a shared stylesheet/Tailwind config (if not already using a CSS approach, plain CSS with variables is fine — don't introduce a new framework this late).
2. Pass through every page (Signup, Login, Dashboard, Contacts List, Contact Detail, Pricing) and apply consistent spacing, alignment, and color usage.
3. Add loading spinners/skeletons for: fetching contacts, generating AI suggestions, sending test digest.
4. Add empty states with helpful copy: no contacts yet ("Add your first contact to get started"), no AI suggestions yet, no interactions logged yet.
5. Add error states: failed API calls should show a user-friendly message, never a blank screen or raw error/stack trace.
6. Add form validation feedback (inline errors, not just silent failures) on signup/login/contact forms.
7. Test the app at mobile width (browser dev tools responsive mode) — fix any obviously broken layouts (overflow, unreadable text, unusable nav).
8. Add a simple favicon and a browser tab title ("Ember — Personal CRM").
9. Do a full manual click-through of every single feature built Days 3-7 as a real end user would, fixing anything broken or confusing along the way.

### 📂 Files/Folders to Create or Modify
```
client/src/styles/            (design system variables/shared styles)
(nearly every existing component/page file — polish pass, not new files)
client/public/favicon.ico
client/index.html             (modify: title tag)
```

### 🔗 Tools/Libraries to Integrate
- None required — CSS-only polish. (Optional: a small icon set if desired, but not required for a strong result.)

### 🧪 Testing Tasks
- Full click-through of: signup → login → add contact → log interaction → view dashboard → hit free-tier cap → upgrade → view AI suggestions → refresh suggestions → send test digest → logout
- Test every form with invalid/missing input — confirm friendly errors, not crashes
- Test on a narrow (mobile-width) browser window for every page
- Test what happens on slow network (throttle in dev tools) — confirm loading states appear, not a frozen UI

### 🐞 Common Issues & Debugging Tips
- **Inconsistent spacing "feels off" but hard to pinpoint** → audit against a fixed spacing scale (e.g., 8px/16px/24px/32px) rather than eyeballing each page individually
- **Mobile nav unusable** → a simple hamburger menu or stacked nav is enough; don't over-engineer
- **Error boundary missing** → an uncaught frontend error can blank the whole page; wrap the app in a basic React error boundary as a safety net
- **Forms don't show validation until submit** → acceptable for v1.0, but at minimum ensure submit-time validation errors are visible and clear

### ✅ End-of-Day Checklist
- [ ] Every page has consistent styling
- [ ] Every async action has a loading state
- [ ] Every empty/error scenario has a friendly message
- [ ] Full click-through completed with no crashes or dead ends
- [ ] App is usable (not necessarily perfect) at mobile width

### 📸 Expected Project State & Screenshots
- Screenshot: polished dashboard (desktop)
- Screenshot: polished dashboard or contacts page (mobile width)
- Screenshot: an example empty state and an example error state

### ➡️ Handoff Notes for Day 9
"The app is feature-complete and polished, tested thoroughly in local development. Nothing has been deployed yet — everything still runs on localhost. Day 9 is entirely focused on deployment: getting the frontend, backend, database, environment variables, and the scheduled email job all running reliably in production."

---

## Day 9 — Testing & Deployment

### 🎯 Objective
Get Ember live on the public internet at a real, shareable URL — fully functional, including the scheduled email job.

### 📖 What I'll Learn
- Deploying a MERN app's frontend and backend separately
- Managing production environment variables securely
- Keeping a scheduled job alive in a hosted environment

### 🛠 Features to Build
- No new features — deployment only

### 📝 Step-by-step Implementation Plan
1. Choose free-tier-eligible hosting: a static host for the React build (e.g., a service like Vercel or Netlify) and a Node-friendly host for the Express backend (e.g., a service like Render or Railway) — confirm current free-tier availability before choosing, since offerings change over time; do not select a paid tier.
2. Push the full project to GitHub (create the repo if not already done) — confirm `.env` files are excluded via `.gitignore`.
3. Deploy the backend first: connect the GitHub repo to the backend host, set the build/start commands, and add all production environment variables (`MONGO_URI`, `JWT_SECRET`, `CLAUDE_API_KEY`, email service credentials) via the host's dashboard — never commit these values.
4. Confirm the backend's production `/api/health` endpoint responds correctly at its live URL.
5. Update MongoDB Atlas Network Access to allow connections from the backend host (or `0.0.0.0/0` if the host uses dynamic IPs — common for free tiers).
6. Deploy the frontend: connect the same GitHub repo (or the `client/` subfolder) to the static host, set the production `VITE_API_URL` environment variable to point at the live backend URL, trigger the build.
7. Confirm the scheduled cron job initializes correctly in the hosted environment — check the backend host's logs on startup for the "next run time" log added on Day 7. (Note: some free-tier hosts spin down idle backends, which can interrupt scheduled jobs — verify this behavior for your chosen host and note any limitation honestly in the pitch deck's "known limitations," rather than overpromising reliability.)
8. Do a full click-through of the live production app, identical to Day 8's local test pass.
9. Fix any environment-specific bugs (CORS misconfiguration for the production domain, incorrect API URLs, etc.).

### 📂 Files/Folders to Create or Modify
```
(no new app code expected — configuration only)
.github/ (optional, only if setting up CI)
README.md    (modify: add live URL, deployment notes)
```

### 🔗 Tools/Services to Integrate
- A static frontend host (free tier)
- A Node backend host (free tier)
- GitHub (for connecting both hosts to your repo)

### 🧪 Testing Tasks
- Full click-through on the live URL: signup → contacts → dashboard → AI suggestions → pricing/upgrade → test digest email
- Confirm the live email digest test actually sends and arrives
- Test the live app on both desktop and mobile browsers
- Share the live URL with one other person (friend/mentor) and watch them use it fresh, without guidance — note anything confusing

### 🐞 Common Issues & Debugging Tips
- **CORS errors in production but not local dev** → the deployed frontend's domain must be added to the backend's CORS allowed-origins list
- **"Cannot connect to database" in production** → check MongoDB Atlas Network Access rules and confirm the production `MONGO_URI` environment variable is set correctly on the host
- **Environment variables "not defined" in production** → confirm they were added in the hosting dashboard, not just the local `.env` (which never gets deployed)
- **Free-tier backend "cold starts" (slow first request)** → normal behavior for many free hosting tiers; mention this honestly if relevant rather than treating it as a bug to fully solve today

### ✅ End-of-Day Checklist
- [ ] Frontend and backend both live at public URLs
- [ ] Full click-through works identically to local testing
- [ ] Scheduled email job confirmed working (or limitation documented if the host constrains it)
- [ ] Live URL tested by at least one other person

### 📸 Expected Project State & Screenshots
- Screenshot: live app loaded at its public URL (address bar visible)
- Screenshot: successful signup/login on the live app
- Screenshot: hosting dashboard showing both services deployed and running

### ➡️ Handoff Notes for Day 10
"Ember is fully deployed and live at [public URL]. All features confirmed working in production. Day 10 is the final day: last QA pass, recording the demo video backup, and packaging all deliverables (live app + PRD + this blueprint + pitch deck + demo video) for submission."

---

## Day 10 — Final QA, Demo Video & Submission

### 🎯 Objective
Confirm Ember is genuinely done, record a polished backup demo video, and package everything for submission.

### 📖 What I'll Learn
- Structuring a compelling product demo narrative
- Final-day QA discipline (verify, don't build)
- Packaging a complete project for presentation/submission

### 🛠 Features to Build
- None — Day 10 is verification and presentation only. Resist any urge to add features today.

### 📝 Step-by-step Implementation Plan
1. Do one final full click-through of the live app exactly as a brand-new user would, on both desktop and mobile.
2. Fix only genuinely critical bugs found (anything that blocks a core flow). Do not chase cosmetic nitpicks today.
3. Prepare a short demo script (60-90 seconds) covering: the problem (relationships going cold), the solution (Ember), a live walkthrough (sign up or log in → add/view a contact → show the AI "reach out this week" suggestions with real reasons → show the pricing/Pro paywall → show a received digest email), and the vision close.
4. Record the demo video using free built-in screen recording (e.g., your OS's native screen recorder, or a free browser-based recorder) — no paid tools.
5. Watch the recording back once — confirm audio/video are clear and the full flow is visible without dead air or confusing cuts.
6. Gather final deliverables into one place: live app URL, GitHub repo link, PRD (docx), Implementation Blueprint (this document), Pitch Deck (pptx), demo video file.
7. Do a final read-through of the PRD and pitch deck to confirm they accurately reflect what was actually built (update any detail that drifted during the 9 days).
8. Submit according to the AB Talks Challenge submission instructions.

### 📂 Files/Folders to Create or Modify
```
demo-video.mp4 (or similar, saved locally/uploaded per submission instructions)
README.md   (modify: final version with live URL, video link, summary)
```

### 🔗 Tools/Services to Integrate
- Free native screen recording tool only

### 🧪 Testing Tasks
- Full final click-through on live production app (desktop + mobile)
- Confirm the demo video plays back correctly and covers every key feature
- Confirm all deliverable links/files are accessible (not broken links, not local-only files forgotten on your machine)

### 🐞 Common Issues & Debugging Tips
- **Live app breaks right before submission** → this is why the demo video exists as a backup; don't panic-refactor, just document the known issue honestly if it can't be fixed in time
- **Video too long/rambling** → script and rehearse once before recording; a tight 60-90 seconds beats a meandering 4 minutes
- **Forgot to show the signature AI feature clearly** → this is the single most important 15 seconds of the demo — make sure the AI suggestions with real reasons are clearly visible and readable on screen

### ✅ End-of-Day Checklist
- [ ] Final live click-through passed with no critical bugs
- [ ] Demo video recorded and reviewed
- [ ] PRD, Blueprint, and Pitch Deck all reflect the actual final product
- [ ] All deliverables gathered and submitted per challenge instructions

### 📸 Expected Project State & Screenshots
- Screenshot: final live app homepage/dashboard
- Screenshot/thumbnail: the demo video file ready for submission
- Screenshot: submission confirmation (if applicable)

### ➡️ Handoff Notes (Post-Capstone)
"Ember v1.0 is complete, deployed, and submitted. Documented v1.1 future scope (from the PRD): real Stripe billing, AI-drafted follow-up messages, AI note summarization, mobile app, browser/LinkedIn contact import, team workspaces. Any future work should start from this blueprint's final state, not a redesign."

---

## 📋 Full-Capstone Feature Traceability (Quick Reference)

| PRD Requirement | Built On | Verified On |
|---|---|---|
| Multi-user auth & data isolation | Day 3 | Day 3, 4 |
| Contact management + interaction timeline | Day 4 | Day 4 |
| Dashboard summary | Day 5 | Day 5, 8 |
| Free tier 25-contact cap | Day 5 | Day 5 |
| Pricing / Pro paywall UI | Day 5 | Day 5, 8 |
| AI weekly outreach suggestions | Day 6 | Day 6 |
| Automated weekly email digest | Day 7 | Day 7, 9 |
| Production-quality polish | Day 8 | Day 8 |
| Live deployment | Day 9 | Day 9, 10 |
| Demo video + submission package | Day 10 | Day 10 |
