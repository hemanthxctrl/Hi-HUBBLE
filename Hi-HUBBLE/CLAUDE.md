# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 📂 Project Overview
Hi-HUBBLE is a high-fidelity social media web application prototype featuring:
- Glassmorphism UI with custom CSS properties
- Modular frontend architecture (auth.js, main.js, style.css)
- Node.js/Express backend with Nodemailer for OTP email verification
- Vite dev server & build tool
- State management via vanilla JS objects and localStorage

## 🛠️ Development Commands

| Command | Description |
|---------|-------------|
| `npm run dev` | Starts both Vite dev server (frontend) and Express server (backend) concurrently via `concurrently`. Frontend: http://localhost:5173, Backend: http://localhost:3000 |
| `npm run server` | Starts only the Express backend server (port 3000) |
| `npm run build` | Builds production assets via `vite build`; output placed in `dist/` |
| `npm run preview` | Previews the production build locally using Vite preview server |

**Note**: There is no configured test runner or linting script. The project relies on manual verification and browser testing.

## 🏗️ Code Architecture & Structure

### Frontend (client-side)
- **Entry Point**: `index.html` contains all UI views (auth, home, search, explore, chats, profile, settings, etc.) as hidden `<div>` elements toggled via JavaScript.
- **Styles**: 
  - `src/style.css` – Global CSS variables, layout, glassmorphism, theme switching (dark/light).
  - `src/auth.css` – Auth‑specific UI (modals, forms, OTP inputs).
- **Modules**:
  - `src/auth.js` – Handles authentication UI flows (sign‑in, sign‑up, forgot password, OTP verification, profile picture capture via webcam, session management via `localStorage`). Exposes `initAuth()` and `updateAppUI()`.
  - `src/main.js` – Core application logic:
    - View switching (home, search, explore, chats, profile, settings) via `switchView()`.
    - Radial/floating navigation menu with drag‑to‑reposition.
    - Stories autoplay, heart‑explosion like animations, double‑tap to like.
    - Chat system for posts, reels, Ludo dice roller, emoji picker, camera capture for DMs, comment/modal handling, theme toggles, toast notifications.
    - Central `state` object holds UI state (theme, active view, chat thread, story progress, like states, etc.).
- **Assets**: Static images (mascot, etc.) stored in `public/`; Lucide icons loaded via CDN.
- **Persistence**: User session (`invibeUser`, `invibeIsLoggedIn`) and profile image (`invibeProfileImage`) stored in `localStorage`.

### Backend (server-side)
- **File**: `server.js`
- **Framework**: Express with CORS and JSON body parsing.
- **Environment**: Uses `dotenv`; expected variables in `.env`:
  ```
  PORT=3000
  EMAIL_USER=your-email@gmail.com
  EMAIL_PASS=your-app-specific-password
  SMTP_HOST=smtp.gmail.com
  SMTP_PORT=465
  ```
- **Routes**:
  - `POST /api/send-otp` – Generates a 6‑digit OTP, stores it in an in‑memory `Map` (5‑minute expiry), attempts to send via Nodemailer; if SMTP fails, falls back to a dev‑mode OTP returned in the response.
  - `POST /api/verify-otp` – Validates submitted OTP against stored record.
- **Server**: Listens on `process.env.PORT` or `3000`.

### Build Tool
- **Vite** (version 8) – Dev server with HMR; production build optimizes CSS/JS and outputs to `dist/`.

### Data Flow
1. User interacts with auth UI → `auth.js` collects credentials and calls `/api/send-otp` (frontend) → backend generates OTP, attempts email, returns success/failure.
2. Frontend OTP modal collects code → calls `/api/verify-otp` → on success, stores user data in `localStorage` and shows main app (`app-container`).
3. Subsequent UI interactions are handled entirely by `main.js` (state updates, DOM mutations) without further API calls (except for simulated features like camera capture which uses the browser's `getUserMedia`).

## 📑 Typical Development Workflow
1. Clone repo and navigate into `Hi-HUBBLE/`.
2. Run `npm install` to install dependencies.
3. Create `.env` file based on the example in the README (add your SMTP credentials for email verification; otherwise dev‑mode OTP will be shown in console).
4. Start development: `npm run dev`.
5. Make changes to frontend files (`src/*.js`, `src/*.css`, `index.html`) – Vite will hot‑reload.
6. For backend‑only changes (e.g., modifying OTP logic), restart the server via `npm run server` or restart the whole dev process.
7. When ready for production, run `npm run build` and serve the `dist/` folder with any static host (or the Express server can serve it – currently `server.js` is API‑only; you would need to add static serving if desired).

## 📝 Notes
- The project uses inline‑style modals and dynamic class toggling for UI state; there is no framework (React, Vue, etc.).
- All business logic is vanilla JavaScript; feel free to refactor toward modules or a framework as the app scales.
- No test suite exists; consider adding unit tests for `auth.js` utilities and integration tests for API endpoints if expanding the codebase.