# THE STACK
### A Step-by-Step Developer Guide

> Terminal → GitHub → Vite/React/TypeScript → Supabase → Cloudflare Workers → Vercel

A practical, no-nonsense reference guide for developers building and deploying modern web apps. Covers the full workflow from opening a terminal for the first time to shipping a live production app — including the SQL fixes and API security patterns you actually hit in real projects.

---

## What's Inside

| # | Section | What It Covers |
|---|---------|----------------|
| 01 | **Terminal Basics** | Navigation, file commands, full NPM reference |
| 02 | **New Project Setup** | Vite + React + TypeScript from scratch, project structure |
| 03 | **GitHub** | First-time setup, everyday push workflow, .gitignore |
| 04 | **Supabase** | Database, auth, storage — full SQL reference included |
| 05 | **Environment Variables** | Key safety, .env setup, deployment platform config |
| 06 | **Cloudflare Workers** | API proxy setup — manual dashboard method, no Wrangler |
| 07 | **Vercel Deployment** | Connect GitHub, env vars, auto-deploy workflow |
| 08 | **Troubleshooting** | Common fixes for Supabase, Cloudflare, Vercel, and Git |
| 09 | **Cheatsheet** | Everything on one page |

---

## Formats

- **Website** — live at [https://gfbtools.github.io/the-stack/](#) *(update with your URL)*
- **PDF** — download [`The_Stack_Guide.pdf`](./The_Stack_Guide.pdf) for offline reference

---

## SQL Reference Included

The Supabase section includes copy-paste SQL for real scenarios:

```sql
-- Fix login blocked by email confirmation
UPDATE auth.users
SET email_confirmed_at = now(), confirmed_at = now()
WHERE email = 'user@email.com';

-- Enable Row Level Security
ALTER TABLE public.tracks ENABLE ROW LEVEL SECURITY;

-- Auto-create profile on signup trigger
CREATE TRIGGER on_auth_user_created
  AFTER INSERT ON auth.users
  FOR EACH ROW EXECUTE FUNCTION public.handle_new_user();
```

Plus table creation for profiles, tracks, playlists, play logs, and PRO publishing fields.

---

## Cloudflare Worker Template

Full API proxy Worker included — no Wrangler, no CLI, no config files. Paste it directly into the Cloudflare dashboard editor:

```js
export default {
  async fetch(request, env) {
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
        'x-api-key': env.ANTHROPIC_API_KEY,
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

---

## Tech Stack Covered

```
Terminal / VS Code
├── Node.js + npm
├── Vite + React + TypeScript
│   └── Tailwind CSS
├── Supabase
│   ├── PostgreSQL (database)
│   ├── Auth
│   └── Storage
├── Cloudflare Workers (API proxy)
└── Vercel (deployment)
    └── GitHub (version control)
```

---

## Who This Is For

Developers who learn by building — not by reading docs for hours before writing a line of code. The guide assumes you're starting from zero on each tool and walks through every step, including the things that break and how to fix them.

---

## Repository Structure

```
the-stack/
├── index.html          # Full website (single file, no dependencies)
├── The_Stack_Guide.pdf # Downloadable PDF version
└── README.md           # This file
```

The website is a single self-contained HTML file. No build step, no npm install, no framework. Drop it anywhere and it works.

---

## Deploy Your Own Copy

**GitHub Pages (free):**
1. Fork or clone this repo
2. Go to Settings → Pages
3. Source: Deploy from branch → `main` → `/` (root)
4. Live at `yourusername.github.io/the-stack`

**Vercel (free):**
1. Import the repo at vercel.com
2. Framework: Other
3. Deploy — done

---

## Local Preview

No build step needed. Just open `index.html` in a browser:

```bash
# macOS
open index.html

# Windows
start index.html

# Or with VS Code Live Server extension
# Right-click index.html → Open with Live Server
```

---

## Contributing

Found something wrong, outdated, or missing? Open an issue or pull request. The goal is to keep this accurate and useful for developers who are actively building with this stack.

---

## License

MIT — use it, share it, build on it.

---

*Built for developers who learn by doing. Ship before perfect.*
