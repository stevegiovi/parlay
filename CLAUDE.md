# Parlay — Development Instructions

## RELEASE CHECKLIST (MANDATORY)

Every time you commit changes to `index.html`, you MUST:

1. **Bump `VERSION`** at the top of `index.html` (around line 699)
   - Bug fix: increment patch (e.g., 2.0.5 → 2.0.6)
   - New feature: increment minor (e.g., 2.0.6 → 2.1.0)
   - Breaking change: increment major (e.g., 2.1.0 → 3.0.0)

2. **Update `BUILD_DATE`** to the current UTC date/time in ISO format
   - Format: `YYYY-MM-DDTHH:MM:SSZ`
   - Example: `2026-03-11T14:30:00Z`

These values are displayed in the UI footer, demo bar, and settings screen.
Forgetting to update them makes it impossible to tell which version is deployed.

## Architecture

- **Single-file app**: Everything lives in `index.html` (~5400 lines)
- **No build step**: Deployed directly via GitHub Pages from `main` branch
- **Dependencies**: Supabase JS (CDN), Google Fonts (CDN)
- **Demo mode**: Activated via `?demo` URL param or "Try Demo" button — uses mock data, no Supabase calls

## Key Sections in index.html

- **CONFIG / VERSION**: Lines ~694-698
- **DEMO**: Lines ~713-1012 (mock data, activation, demo bar)
- **Auth**: Lines ~1096-1300 (Supabase auth, bootstrap, profiles)
- **Store**: Lines ~1400-1700 (parlays, tasks, CRUD)
- **Demo Proxy**: Lines ~1703-1800 (intercepts Auth/Store in demo mode)
- **Router**: Lines ~2034-2103
- **AI**: Lines ~2108+ (Claude API calls via Supabase edge function)
- **Screens**: Lines ~2900+ (auth, home, plan, tasks, setup)
- **Boot**: Lines ~5275+ (initialization, dev mode, demo URL param)

## Testing

- Always test demo mode after changes: load with `?demo` URL parameter
- The demo button on the landing page calls `DEMO.activate('A')`
- Watch for `ago()` helper — it returns an ISO **string**, not a Date object
