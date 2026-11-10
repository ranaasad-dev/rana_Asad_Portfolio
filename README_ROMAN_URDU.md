# Rana Asad Portfolio Studio — GitHub Pages + Supabase

**Status:** Ready-to-configure website and admin dashboard. This ZIP does NOT connect to your private Supabase project until you complete the steps below. The frontend uses the existing Portfolio Studio black/gold design from the original FastAPI/SQLite CMS v2.

## Architecture
- GitHub Pages serves `index.html`, `project.html`, `admin/index.html`, CSS, and JS.
- Supabase Postgres stores `portfolio_profile`, `portfolio_settings`, `portfolio_items`, messages, comments, activity logs.
- Supabase Auth protects admin login. **Only an allowlisted Auth user UID** can modify data via RLS.
- Supabase Storage bucket `portfolio-media` holds newly uploaded images and PDF CVs.
- Old FastAPI/SQLite local backend is NOT deployed. Existing Windows CMS remains untouched.

## Step-by-step setup

**0) BACKUP:** Keep your old ZIP/SQLite database and local uploads safe. Do not overwrite your current live portfolio repository yet. Test this version in a NEW GitHub repository first, e.g. `portfolio-supabase-test`.

**1) Supabase project:** Visit https://supabase.com/dashboard. Make a separate project if your Free plan allows. You can also reuse your existing Supabase organization; do not mix this app's admin permissions with another website. Project Settings / API contains the Project URL and publishable key (older projects may say `anon` key).

**2) Run SQL:** In Supabase > SQL Editor, first run the entire `supabase/schema.sql` script. Then run `supabase/seed.sql` to insert the initial content from original CMS. Do this on a new, empty portfolio project. `seed.sql` is **NOT idempotent for items**: running it twice creates duplicate items.

**3) Create one admin:** Supabase > Authentication > Users > Add user (email/password). Create a NEW strong password (at least 12 characters); if email confirmations are enabled, make sure user is confirmed. Copy the Auth User UUID. In SQL Editor run:

```sql
insert into public.portfolio_admins(uid)
values ('PASTE-YOUR-REAL-AUTH-USER-UUID-HERE');
```

Use only your actual Auth user UUID. Don't copy this example literally. A normal Supabase login is NOT sufficient: the allowlist enforces the single administrator. If using multiple admins in the future, you must explicitly authorize each UUID (this version is designed for one main admin).

**4) Configure website:** Open `js/config.js` and set ONLY `SUPABASE_URL` and `SUPABASE_PUBLISHABLE_KEY` from your Supabase Project Settings. Example:

```js
export const SUPABASE_URL = 'https://YOURPROJECT.supabase.co';
export const SUPABASE_PUBLISHABLE_KEY = 'sb_publishable_YOUR_PUBLIC_KEY';
```

Your real public key might start with `eyJ...` on older projects. **Never put `service_role`, `sb_secret_...`, database passwords, email passwords or your admin password in source files or GitHub.** The publishable/anon key is intended to be public; database access is protected by RLS.

**5) Upload to GitHub:** Extract the ZIP and upload the CONTENTS of `AJTech_Portfolio_Supabase_CMS/` to the ROOT of your NEW repository: `index.html` must be directly in the repository root, not nested under the extracted folder. Do NOT upload `portfolio.db` / `.env` / backup JSON. The folder `supabase/` and the documentation may be committed; SQL contains no secrets. Set repository to Public if using free GitHub Pages.

**6) Enable Pages:** GitHub repo > Settings > Pages > Build and deployment > Deploy from a branch > `main` > `/(root)` > Save. After publishing:
- Public site: `https://ranaasad-dev.github.io/YOUR-REPO/`
- Admin login: `https://ranaasad-dev.github.io/YOUR-REPO/admin/`

**7) Test:** Sign in through `/admin/` using the Auth email/password. Add a test project (keep it as draft, then publish). Open public website and refresh. Try changing profile text, uploading photo/CV and viewing published content. Test contact form -> private inbox and comment -> pending approval. Sign out and confirm dashboard is inaccessible without login.

**8) Go live:** Once everything works on the test repo, back up your existing `rana_Asad_Portfolio` repository and then deploy the new files there if you want to keep the old portfolio address. The original separate AJTech digital business card at `ajtech-connect` can stay unchanged.

## Admin functionality

| Admin section | Can manage |
|---|---|
| Overview | Stats, quick edits, recent activity |
| Profile & media | Name, headline, about, email, social links, location, SEO text, optional logo, brand label, photo, downloadable CV, WhatsApp |
| Projects | Add/edit/delete, upload image, technologies, GitHub/demo URLs, status, publish/hide, sort order |
| Skills | Add/edit/delete, categories, details, badges, publish/hide |
| Experience | Add/edit/delete, company, duration, description, publish/hide |
| Education | Add/edit/delete, institution, dates, publish/hide |
| Certificates | Add/edit/delete, description, verification URL, image, publish/hide |
| Comments | Moderate visitor submissions (approve/reject/delete) |
| Private inbox | Read, reply through email, mark read/unread, archive, delete messages |
| Website settings | Section visibility, feedback/message toggles, accent highlight color |
| Security | Change password using Supabase Auth |
| Backups | Export JSON copy of content; separately back up Storage images/CV |
| Activity log | Records app edits on a best-effort basis |

### Difference from old FastAPI CMS
1. Local SQLite and password hashes do not work on GitHub Pages. Supabase Auth and Postgres replace them.
2. Site HTML/CSS is hosted by GitHub; data comes live from Supabase when page loads. You don't need to re-upload GitHub code after a content change.
3. Uploaded photos are served from Supabase Storage, no local Windows `uploads/` files.
4. Content in the included seed is from the older CMS **at the time of its ZIP**. Later changes in your local SQLite database are **not automatically migrated**.
5. Activity log records CMS changes, but is not a tamper-proof compliance log.
6. Old CMS backup/restore screens are replaced by private JSON export. Automated server snapshots/restore are a separate feature.
7. Client-side pages cannot generate unique server-rendered Open Graph data for every project. The main page has fixed social metadata; individual project pages have generic previews.

## Migrate your real local SQLite content instead of starter data

If you changed many details locally, run on your Windows machine **before** running `seed.sql`:

```powershell
py -3 tools/export_local_sqlite.py "D:\\downloadaj\\Rana-Asad-Portfolio-CMS\\data\\YOUR_DB_FILE.sqlite" --output migrated_content.sql
```

Find the actual DB filename in the old project's `data` folder (do not assume the sample filename). After running `schema.sql`, run **only** the generated SQL instead of `seed.sql` to import current profile and items. This script does not automatically migrate messages, comments, image files, password hashes or Supabase Auth accounts; those require separate review. Upload images and CV afresh through Admin.

## Security & limitations
- RLS is enabled for every portfolio table. Public can read published content, but only authorized Auth UID can edit. Comments/messages are private, and public comments come from a safe approved-only view that omits email.
- Supabase `portfolio-media` bucket is PUBLIC for published photos/CV; never upload private files to it. Validate your CV content before launch.
- Public message/comment forms have field validation and hidden honeypots but NO enterprise-grade spam protection. For production with heavy traffic, enable a rate-limited Supabase Edge Function with CAPTCHA/Turnstile, or disable forms in Settings.
- Update / regenerate assets for your website; don't upload private financial or identity documents.
- Never allow visitors to sign up for admin privileges. Only manually allowlisted Auth UUIDs have admin actions.
- Supabase Free plan has quotas and can pause after 7 days of insufficient activity. A paused backend means portfolio data won't load; review project usage and backups. See https://supabase.com/pricing and https://supabase.com/docs/guides/platform/free-project-pausing.
- The starter `assets/original-cv.pdf` comes from the prior CMS archive. **Review it before publishing**; replace via Admin after launch if old/outdated. You may delete it and clear the default CV value in seed JSON/SQL if you don't want it public.
- API requests require JavaScript and internet to Supabase; if you test `index.html` from a local file (`file://`) the ES modules may fail. Use GitHub Pages or a local HTTP server (example: `python -m http.server 8000`).
- The Supabase SDK is imported from jsDelivr CDN. If you operate in a network that blocks CDNs, self-host the package as a local JS asset.

## Files
- `index.html`: public portfolio; `project.html`: project details; `privacy.html`: privacy notice.
- `admin/index.html`: protected admin UI.
- `js/config.js`: your public Supabase URL/key.
- `js/public.js`, `js/project.js`, `js/admin.js`: frontend logic.
- `supabase/schema.sql`, `supabase/seed.sql`: database and RLS setup.
- `assets/portrait.webp`, `assets/original-cv.pdf`: initial original CMS media.
- `data/starter.json`: disconnected demo data until configuration is entered.
- `tools/export_local_sqlite.py`: optional migration of old locally changed content.

## If changes don't appear
1. Go to Admin, save the edit, check success message. Then reload the public portfolio in a new tab.
2. If you see an empty/error page, verify URL/key, SQL script and `portfolio_admins` UUID.
3. Verify browser Console errors and Supabase > Logs > API. A public API key alone does not grant admin rights.
4. Uploaded media may take a short time to update in browser cache. Upload creates a fresh file URL.
