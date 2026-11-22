# 📍 WorkNear

A free, mobile-first job tracker that helps students, jobseekers and people without a steady income find **part-time, full-time and gig work near them**, and apply by phone, WhatsApp or the app.

Built for India first (Kerala cities: Alappuzha, Kochi, Thiruvananthapuram, Kozhikode, plus any city you add). Works on low-end phones and slow connections: one HTML file, no frameworks, no build step, no external requests.

## Features

- **Jobs near me**: pick a city or auto-detect it, filter by distance, job type, category and pay
- **Job types**: part-time, full-time, internship, weekend only, work from home, one-day gig
- **Student mode**: only flexible, no-experience jobs; enter free hours (evenings / weekends / mornings) and get matches
- **Earn today**: same-day gigs in one tap
- **Job cards**: pay, hours, type, posted date, requirements, with Apply, Call, WhatsApp, Save
- **Fresh listings**: "posted X days ago", listings older than 30 days auto-hide, "Report this job", "Verified employer" badge
- **Employer portal**: post a job in under 2 minutes, no resume or registration
- **Job alerts**: by city, category and job type
- **Application tracker**: Saved, Applied, Interview, Offer, Rejected, with reminder dates
- **Simple profile**: name, phone or email, skills, availability, about me
- **Safety tips**: shown at first launch and on every job
- **Languages**: English and Malayalam (structure ready for Hindi and Tamil)
- **Accessible**: large tap targets, high contrast, screen-reader labels, light and dark mode

## Run it

No install needed.

1. Download or clone this repo
2. Open `index.html` in a browser (or on your phone)

Or serve locally:

```bash
python3 -m http.server 8000
# open http://localhost:8000
```

## Deploy free on GitHub Pages

1. Push to a GitHub repo
2. Settings → Pages → Source: *Deploy from a branch* → `main` / root
3. Your site appears at `https://<your-username>.github.io/<repo-name>/`

## Project structure

The app is currently one file for speed and simplicity. The code is already split into clearly commented sections, so it is easy to break up later:

```
index.html            # everything: styles, data, screens, logic
  ├─ T / TY / CA      # i18n strings, job types, categories   → src/i18n.js
  ├─ CITIES / ROWS    # city list + 32 seed jobs               → src/data/seed.js
  ├─ SOURCES / save() # data layer (swap for API/backend)      → src/db.js
  └─ *View() / card() # screens: Jobs, My jobs, Post, Profile  → src/screens/
```

## Data model

Stored in the browser (`localStorage` key `worknear`) for now. Maps directly to database tables later:

| Table | Fields |
|---|---|
| `jobs` | id, title, employer, city, area, category, type, pay, unit (hour/day/month), hours, days since posted, verified, slots (e/w/m/f/n), km, requirements, phone |
| `employers` | name, phone, verified |
| `users` | name, contact, skills, about, free hours, city |
| `saved_jobs` (tracker) | user, job id, status, reminder date |
| `reports` | job id, user, reason, date |

Seed row format (one line per job, `|` separated):
`title|employer|city|area|category|type|pay|unit|hours|daysAgo|verified|slots|km|requirements`

## Connect real job data later

Jobs come from the `SOURCES` array. Each source is a function returning jobs in the same shape as the seed data.

```js
// Example: Adzuna India (https://developer.adzuna.com)
async function adzunaSource(city) {
  const url = `/api/adzuna?where=${encodeURIComponent(city)}`; // your proxy keeps the API key secret
  const res = await fetch(url);
  const data = await res.json();
  return data.results.map(r => ({
    id: 'adz' + r.id, t: r.title, e: r.company.display_name,
    c: city, a: r.location.display_name, cat: 'other', ty: 'fulltime',
    p: Math.round(r.salary_min / 12) || 0, u: 'month', h: 'See listing',
    d: Math.floor((Date.now() - new Date(r.created)) / 864e5),
    v: 0, s: '', km: 0, req: r.description.slice(0, 200), ph: ''
  }));
}
SOURCES.push(adzunaSource);
```

Notes:
- Never put API keys in `index.html`. Use a tiny proxy (Cloudflare Workers, Netlify or Vercel functions, all have free tiers).
- Make `loadJobs()` async (`Promise.all(SOURCES.map(...))`) and set `S.updated = Date.now()` after each fetch.
- Government portals and RSS feeds work the same way: fetch, map to the job shape, push to `SOURCES`.
- For real accounts, alerts (email/SMS/push), and report moderation, replace `save()` with a backend. [Supabase](https://supabase.com) free tier is a good fit.

## Add a language (Hindi, Tamil)

1. Copy the `en` block in `T` and add `hi: {...}` or `ta: {...}`
2. Add a third label to each entry in `TY` and `CA`, and update `L()`
3. Add an `<option>` to the language selector in `render()`

Missing keys fall back to English automatically.

## Roadmap

- Map view of nearby jobs
- Resume builder and free skill tips
- Salary and cost-of-living hints per city
- Push / SMS / WhatsApp alerts
- Employer verification flow and moderation dashboard
- Hindi and Tamil

## Safety

Seed jobs and phone numbers are **sample data for demonstration**. Do not contact them. Real deployments must moderate listings. Users should never pay to get a job, should verify employers, and should meet in public places.

## License

MIT
