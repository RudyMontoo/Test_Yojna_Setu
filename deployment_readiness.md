# 🚀 Yojna Setu — Deployment Readiness Report

## TL;DR

> **Partially Ready.** The `deploy/` folder is pre-structured for Railway + Vercel deployment, but several blockers must be resolved before the app will work end-to-end for a shared URL.

---

## Architecture Overview

```mermaid
graph LR
  A[User Browser] --> B[React Frontend\nVercel]
  B --> C[Spring Boot Gateway\nRailway]
  C --> D[FastAPI AI Hub\nRailway]
  B --> E[Supabase\nAuth + DB]
```

The app has **3 tiers**:
| Service | Tech | Deploy Target |
|---------|------|--------------|
| Frontend | React + Vite | Vercel |
| API Gateway | Spring Boot (Java 17) | Railway |
| AI Hub | FastAPI (Python) | Railway |
| Auth/DB | Supabase | Supabase Cloud |

---

## ✅ What's Ready

| Item | Status | Notes |
|------|--------|-------|
| `deploy/frontend/` folder | ✅ | Vite build config, package.json present |
| `deploy/backend/ai-hub/` | ✅ | `Procfile` + `requirements.txt` present — Railway auto-detects |
| `deploy/backend/spring-gateway/` | ✅ | `Dockerfile` + `pom.xml` present |
| Deploy README | ✅ | Step-by-step Railway + Vercel guide at `deploy/README.md` |
| Frontend routing | ✅ | `react-router-dom` with 10 pages wired |
| Vite build script | ✅ | `npm run build` → `dist/` folder |
| API proxy (dev only) | ✅ | Proxies `/api` → `localhost:8000` in dev |

---

## ❌ Blockers — Must Fix Before Deploying

### 1. 🔑 Missing Supabase Environment Variables (CRITICAL)
The frontend reads two env vars that are **never set**:
```js
// frontend/src/lib/supabase.js
const SUPABASE_URL = import.meta.env.VITE_SUPABASE_URL       // ← undefined!
const SUPABASE_ANON_KEY = import.meta.env.VITE_SUPABASE_ANON_KEY  // ← undefined!
```
**Fix:** Create a `.env` file in `deploy/frontend/` (or set on Vercel dashboard):
```env
VITE_SUPABASE_URL=https://your-project.supabase.co
VITE_SUPABASE_ANON_KEY=your-anon-key-here
VITE_API_URL=https://your-gateway.railway.app
```

> [!CAUTION]
> Without `VITE_SUPABASE_URL` and `VITE_SUPABASE_ANON_KEY`, the app will crash on load — authentication will be completely broken.

---

### 2. 🔑 Missing GROQ API Key for AI Hub (CRITICAL)
The FastAPI AI Hub uses Groq's LLM. Without the key, all chat/agent features return errors.

**Fix:** Set this env var on Railway when deploying `ai-hub`:
```env
GROQ_API_KEY=your_groq_api_key   # Get free at https://console.groq.com
```

---

### 3. 🔑 Exposed Gemini API Key in `.env.example` (SECURITY RISK)
```
# ai_service/.env.example  ← Line 1
GEMINI_API_KEY=AIzaSyDHj7KYhltrfcU3rCR_VOiHg9xjFt3DJpc
```
> [!WARNING]
> A real API key is hardcoded in `.env.example` and committed to the repo. **Revoke this key immediately** at https://console.cloud.google.com/apis/credentials, then generate a new one.

---

### 4. ⚙️ Frontend Has No `.env` File for Production API URL
The Vite proxy (`/api` → `localhost:8000`) only works in local dev. In production, all API calls need `VITE_API_URL` pointing to the Railway gateway.

**Fix:** Ensure all API calls in the frontend use:
```js
const API_BASE = import.meta.env.VITE_API_URL || ''
```
Check that `ChatPage.jsx`, `StatusPage.jsx`, and others don't hardcode `localhost`.

---

### 5. ☁️ Supabase Project Must Be Created
The app uses Supabase for auth. You need to:
1. Create a free project at https://supabase.com
2. Get the **Project URL** and **anon/public key** from Settings → API
3. Enable Email/Password auth in Authentication settings

---

## ⚠️ Secondary Issues (Non-Blocking but Recommended)

| Issue | Risk | Fix |
|-------|------|-----|
| No `vercel.json` in deploy/frontend | Medium | Needed for React Router SPA — without it, direct URL navigation (e.g. `/chat`) returns 404 |
| Spring Boot JWT secret not set | Medium | Set `JWT_SECRET` on Railway |
| `SARVAM_API_KEY` not filled | Low | Multilingual features will silently fail |

### Fix: Add `vercel.json` for React Router SPA

Create `deploy/frontend/vercel.json`:
```json
{
  "rewrites": [{ "source": "/(.*)", "destination": "/index.html" }]
}
```

---

## 🗺️ Step-by-Step Deployment Path

Follow this exact order:

```
Step 1: Supabase
  → Create project → get URL + anon key

Step 2: Groq
  → Sign up at console.groq.com → get free API key

Step 3: Railway — Deploy AI Hub
  → New Project → GitHub → deploy/backend/ai-hub/
  → Set env: GROQ_API_KEY

Step 4: Railway — Deploy Spring Gateway
  → New Project → GitHub → deploy/backend/spring-gateway/
  → Set env: FASTAPI_URL, FRONTEND_URL, JWT_SECRET

Step 5: Vercel — Deploy Frontend
  → New Project → GitHub → deploy/frontend/
  → Set env: VITE_SUPABASE_URL, VITE_SUPABASE_ANON_KEY, VITE_API_URL
  → Add vercel.json for SPA routing
  → Deploy → get your public URL! 🎉
```

---

## 📋 Pre-Deploy Checklist

- [ ] Revoke the leaked Gemini API key
- [ ] Create Supabase project + get URL/key
- [ ] Get a free Groq API key
- [ ] Add `vercel.json` to `deploy/frontend/`
- [ ] Verify no `localhost` hardcoding in frontend API calls
- [ ] Set all env vars on Vercel and Railway dashboards
- [ ] Test locally first with `deploy/README.md` instructions

---

## 🏁 Verdict

**~70% deploy-ready.** The infrastructure scaffolding (Procfile, Dockerfile, Vite build) is all there. The main gaps are environment variable setup (Supabase, Groq keys) and the `vercel.json` SPA fix. Once those are addressed, this can be deployed and shared as a working link in under 1 hour.
