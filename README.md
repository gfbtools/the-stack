```
 ________  ___  ___  _______           ________  _________  ________  ________  ___  __       
|\\   __  \\|\\  \\|\\  \\|\\  ___ \\         |\\   ____\\|\\___   ___\\\\   __  \\|\\   ____\\|\\  \\|\\  \\     
\\ \\  \\|\\  \\ \\  \\\\\\  \\ \\   __/|        \\ \\  \\___|\\|___ \\  \\_\\ \\  \\|\\  \\ \\  \\___|\\ \\  \\/  /|_   
 \\ \\   _  _\\ \\   __  \\ \\  \\_|/__       \\ \\_____  \\   \\ \\  \\ \\ \\   __  \\ \\  \\    \\ \\   ___  \\  
  \\ \\  \\\\  \\\\ \\  \\ \\  \\ \\  \\_|\\ \\       \\|____|\\  \\   \\ \\  \\ \\ \\  \\ \\  \\ \\  \\____\\ \\  \\\\ \\  \\ 
   \\ \\__\\\\ _\\\\ \\__\\ \\__\\ \\_______\\        ____\\_\\  \\   \\ \\__\\ \\ \\__\\ \\__\\ \\_______\\ \\__\\\\ \\__\\
    \\|__|\\|__|\\|__|\\|__|\\|_______|       |\\_________\\   \\|__|  \\|__|\\|__|\\|_______|\\|__| \\|__|
                                         \\|_________|                                          
```

<div align="center">

### Terminal → Vite/React/TypeScript → GitHub → Supabase → Cloudflare Workers → Vercel

**A complete field guide for building and shipping modern web apps**

[![Live Site](https://img.shields.io/badge/Live%20Site-Visit%20Guide-6366F1?style=for-the-badge)](https://gfbtools.github.io/the-stack)
[![PDF Download](https://img.shields.io/badge/PDF-Download%20Guide-10B981?style=for-the-badge)](./The_Stack_Guide.pdf)
[![License MIT](https://img.shields.io/badge/License-MIT-F59E0B?style=for-the-badge)](./LICENSE)

</div>

---

## What This Is

The Stack is a real deal developer reference guide covering the exact workflow used to build and deploy production web apps — from the first terminal command to a live URL with a real database, user auth, file storage, and protected API keys.

Not a course. Not a tutorial series. A field guide. The kind you keep open in a second tab while you're building.

Every section is one tool. Every step is actionable. Every code block is copy-paste ready. The SQL fixes, the Worker templates, the .gitignore you always forget — it's all here, in one place.

---

## The Stack

```
┌─────────────────────────────────────────────────────┐
│                                                     │
│   VS Code  +  Terminal                              │
│       ↓                                             │
│   Vite  +  React  +  TypeScript    ← your app       │
│       ↓                                             │
│   Supabase                         ← data + auth    │
│       ↓                                             │
│   Cloudflare Workers               ← API security   │
│       ↓                                             │
│   GitHub                           ← version ctrl   │
│       ↓                                             │
│   Vercel                           ← live deploy    │
│                                                     │
└─────────────────────────────────────────────────────┘
```

---

## Contents

| # | Section | What's Inside |
|:--|:--------|:--------------|
| `01` | **Terminal Basics** | 14 navigation commands, full npm reference, what never to run |
| `02` | **New Project Setup** | Vite + React + TS scaffold, project structure map, common packages |
| `03` | **GitHub** | First-time config, repo creation, 10-command daily workflow, .gitignore |
| `04` | **Supabase** | SDK setup, table SQL, auth, storage, RLS policies, trigger functions, SQL fixes |
| `05` | **Environment Variables** | .env format, key safety rules, adding secrets to every platform |
| `06` | **Cloudflare Workers** | Full proxy Worker template — manual dashboard, zero CLI |
| `07` | **Vercel** | Import, env vars, build settings, auto-deploy, custom domain |
| `08` | **Troubleshooting** | 13 specific problems with specific fixes — Supabase, Cloudflare, Vercel, Git |
| `09` | **Cheatsheet** | All 8 categories on one page, copy-paste ready |

---

## Formats

**🌐 Website** — [`index.html`](./index.html)
A single self-contained HTML file. No build step, no npm install, no framework required. Fixed sidebar nav on desktop. Fixed bottom tab bar on mobile. Copy it anywhere and it works.

**📄 PDF** — [`The_Stack_Guide.pdf`](./The_Stack_Guide.pdf)
24 pages. Every code block, SQL query, and fix included. Built for offline reference, printing, or sharing.

---

## What's Included

### Database — Full SQL Reference

The Supabase section covers real tables, real fixes, and the queries you actually need running in the SQL Editor.

```sql
-- Profiles table
CREATE TABLE public.profiles (
  id uuid REFERENCES auth.users(id) PRIMARY KEY,
  username text UNIQUE,
  full_name text,
  avatar_url text,
  created_at timestamptz DEFAULT now()
);

-- Fix: user can't log in (email confirmation block)
UPDATE auth.users
SET email_confirmed_at = now(), confirmed_at = now()
WHERE email = 'user@email.com';

-- Enable Row Level Security
ALTER TABLE public.tracks ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Users see own tracks"
  ON public.tracks FOR SELECT
  USING (auth.uid() = user_id);

-- Auto-create profile on signup
CREATE OR REPLACE FUNCTION public.handle_new_user()
RETURNS TRIGGER AS $$
BEGIN
  INSERT INTO public.profiles (id, full_name, avatar_url)
  VALUES (NEW.id,
    NEW.raw_user_meta_data->>'full_name',
    NEW.raw_user_meta_data->>'avatar_url');
  RETURN NEW;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;

CREATE TRIGGER on_auth_user_created
  AFTER INSERT ON auth.users
  FOR EACH ROW EXECUTE FUNCTION public.handle_new_user();
```

---

### API Security — Cloudflare Worker Template

Your frontend never holds a real API key. The Worker sits in between, holds the secret server-side, and forwards the request. Paste this directly into the Cloudflare dashboard — no Wrangler, no CLI, no config files.

```js
export default {
  async fetch(request, env) {

    // Handle CORS preflight
    if (request.method === 'OPTIONS') {
      return new Response(null, {
        headers: {
          'Access-Control-Allow-Origin': '*',
          'Access-Control-Allow-Methods': 'POST, OPTIONS',
          'Access-Control-Allow-Headers': 'Content-Type',
        }
      })
    }

    const body = await request.json()

    const response = await fetch('https://api.anthropic.com/v1/messages', {
      method: 'POST',
      headers: {
        'x-api-key': env.ANTHROPIC_API_KEY,   // key lives in Cloudflare, not your app
        'anthropic-version': '2023-06-01',
        'content-type': 'application/json',
      },
      body: JSON.stringify(body),
    })

    const data = await response.json()
    return new Response(JSON.stringify(data), {
      headers: {
        'Content-Type': 'application/json',
        'Access-Control-Allow-Origin': '*',
      }
    })
  }
}
```

Swap the endpoint URL for any API — OpenAI, a custom backend, anything.

---

### Troubleshooting — Real Problems, Real Fixes

```
SUPABASE
  ✗ Login fails silently           →  Email confirmation block — SQL fix included
  ✗ Queries return empty           →  RLS enabled, no policies — SQL fix included
  ✗ Column doesn't exist error     →  ALTER TABLE never ran — SQL fix included
  ✗ Storage upload 403             →  Missing bucket INSERT policy — SQL fix included
  ✗ Profile not created on signup  →  Trigger missing — full trigger SQL included

CLOUDFLARE
  ✗ CORS error in browser          →  Missing OPTIONS handler — full Worker template included
  ✗ env.KEY_NAME is undefined      →  Forgot Save and Deploy after adding variable
  ✗ Worker returns 500             →  Check Logs tab in dashboard

VERCEL
  ✗ Build fails, works locally     →  Missing env var or TypeScript error in strict mode
  ✗ Deploys but shows white screen →  VITE_ env vars not added to project settings
  ✗ Stale version after push       →  Force redeploy from Deployments tab

GIT
  ✗ Push rejected                  →  git pull first, then push
  ✗ Pushed .env by accident        →  git rm --cached .env + rotate every exposed key
  ✗ Pushed node_modules            →  git rm -r --cached node_modules + add to .gitignore
```

---

## Key Rules

**Environment Variables**
```js
// ❌ Never — key is exposed in your source code
const key = 'sk-ant-api03-...'

// ✓ Always — value comes from .env, never committed
const key = import.meta.env.VITE_API_KEY
```

**The .gitignore You Need on Every Project**
```
node_modules/
dist/
.env
.env.local
.env.production
.DS_Store
*.log
```

**The Daily Git Workflow**
```bash
git status                        # see what changed
git add .                         # stage all changes
git commit -m "what you did"      # save the snapshot
git push                          # trigger Vercel redeploy
```

---

## Repository Structure

```
the-stack/
├── index.html            ← complete website, single file
├── The_Stack_Guide.pdf   ← 24-page PDF
└── README.md             ← you are here
```

---

## Run It Locally

```bash
# macOS
open index.html

# Windows
start index.html

# VS Code — right-click index.html → Open with Live Server
```

No install. No build. No dependencies.

---

## Deploy Your Own Copy

**GitHub Pages**
```
Settings → Pages → Deploy from branch → main → / (root)
Live at: yourusername.github.io/the-stack
```

**Vercel**
```
Import repo → Framework: Other → Deploy
```

---

## Contributing

Something wrong, outdated, or missing? Open a PR.

1. Fork the repo
2. Edit `index.html` or `README.md`
3. Submit a PR with a clear description of what changed and why

---

## License

MIT — use it, share it, build on it.

---

<div align="center">

*Learn by doing. Ship before perfect.*

</div>
