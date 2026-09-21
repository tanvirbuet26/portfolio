# Md. Tanvir Ahmmed — Academic & Consulting Website

A portable full-stack Next.js website for environmental and water research, civil engineering, and research & WASH consulting. The design uses the maroon, white and slate palette of BUET-JIDPUS, with a restrained teal environmental accent. The layout and identity are original.

**Start here:** the application is implemented, but no hosted Supabase project or live administrator account has been provisioned for you. Connect your own project using the steps below. Until then, an explicitly enabled read-only content preview is available. There are no demo passwords, browser-only CMS saves, or fake inquiry success messages. See `docs/TEST-REPORT.md` for the exact validation boundary.

## What is included

- Public homepage, About, research portfolio, publication catalogue and detail pages, engineering projects, experience, consulting, leadership, insights, gallery, skills, HTML CV, PDF CV and contact pages.
- A protected `/admin` CMS with labelled forms, rich-text editing, draft/publish/archive, visibility, featured content, duplication, drag-and-drop and keyboard ordering, private saved-draft previews, Trash and restore.
- Media upload, image optimisation, crop/reposition, reusable files, captions, credits, ALT text, stable file replacement, PDF support and a single current CV.
- Supabase PostgreSQL schema, RLS policies, private storage buckets, email/password authentication, session refresh, reset-password flow and admin allowlisting.
- Private inquiry storage, attachment downloads, unread/read/replied/archived states, Turnstile, honeypot and database-backed rate limiting.
- Site search, publication/year/topic filters, citation copy, BibTeX export, print styles, sitemap, robots, page SEO, scholarly citation metadata, structured data and sharing metadata.
- Optional dark mode, optional analytics with visitor consent, Search Console verification, JSON/CSV export and a media backup script.
- Verified CV-based initial content, genuine owner-published photographs, source notes, and explicit review flags for conflicts.

## Stack and requirements

Node.js **22 or newer**, npm, Next.js 16 App Router, React 19, TypeScript, Tailwind CSS 4, Supabase, Tiptap, dnd-kit and Sharp. Use the supplied `package-lock.json` with `npm ci` to reproduce the tested dependency versions.

The application needs a Node/Next.js server. It is **not** a static export and cannot be hosted directly on GitHub Pages. GitHub can store the repository; Vercel can run the application. A custom domain is optional.

## 1. Run the visual preview

Unzip the project, open a terminal in this folder, and run:

```bash
npm ci
cp .env.example .env.local
npm run dev
```

On Windows PowerShell, replace the copy command with:

```powershell
Copy-Item .env.example .env.local
```

Open `http://localhost:3000`. The example environment enables `CONTENT_PREVIEW=true`, which displays bundled content without a database. `/preview` provides mobile, tablet and desktop frames of the **actual public application**. `/admin/login` explains setup until Supabase is connected.

Preview mode never accepts mutations or stores inquiries. Production should use `CONTENT_PREVIEW=false`.

## 2. Create the Supabase backend

1. Create your own project in Supabase and save its database password securely.
2. Open the SQL editor. Run the complete contents of `supabase/migrations/001_initial.sql` **once**, in a new project. Alternatively apply it with your normal Supabase CLI migration workflow.
3. Find the project URL and publishable key in the project's Connect/API settings. Add them to `.env.local`.
4. Add the server-only service-role key. It must **never** be prefixed `NEXT_PUBLIC_`, committed, or pasted into public content.
5. Keep both `site-media` and `inquiry-attachments` buckets **private**. The migration creates their policies. Public media is delivered through an authorised application route and short-lived signed URLs.

```dotenv
NEXT_PUBLIC_SITE_URL=http://localhost:3000
NEXT_PUBLIC_SUPABASE_URL=https://YOUR_PROJECT.supabase.co
NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY=YOUR_PUBLISHABLE_KEY
SUPABASE_SERVICE_ROLE_KEY=YOUR_SERVER_ONLY_SERVICE_ROLE_KEY
ADMIN_EMAIL=YOUR_ADMINISTRATOR_EMAIL
CONTENT_PREVIEW=false
```

The schema uses one typed `content_items` table for the many CMS sections rather than dozens of nearly identical tables. The `collection` field identifies the section; its structured `data` holds section-specific fields. This lets all collections share reliable ordering, publishing, trash and export behaviour. The administration forms never expose database terminology or require JSON editing.

## 3. Create your administrator

1. In Supabase **Authentication → Users**, create the owner's email/password account securely. Use a strong unique password. This project has no public signup page.
2. Disable public signups in Supabase Authentication settings if you do not need them.
3. Set `ADMIN_EMAIL` to that exact email in `.env.local`.
4. Run:

```bash
npm run seed
```

The seed command finds the existing Auth user, adds that user to the private admin allowlist, uploads the bundled media and public CV, and adds initial content and review notes. It does not create or embed a password. It preserves existing records rather than overwriting later edits.

Restart the app, then open `/admin/login`. A valid Supabase account alone is insufficient: it must also appear in `admin_users`. To revoke administrator access, remove its allowlist row from the Supabase SQL editor.

A one-time SQL alternative for allowlisting an existing user is:

```sql
insert into public.admin_users (user_id)
select id from auth.users where email = 'YOUR_ADMINISTRATOR_EMAIL'
on conflict do nothing;
```

Use the seed command to load the initial website content even if you allowlisted the account manually.

## 4. Configure password reset

In Supabase Authentication URL Configuration:

- Site URL: your exact application origin.
- Allowed redirect URL: `http://localhost:3000/auth/callback` during local setup.
- Production redirect URL: `https://YOUR_DOMAIN/auth/callback`.

Configure a production SMTP provider in Supabase before relying on password recovery. The reset link opens `/auth/callback`, exchanges the one-time code, and takes the authorised administrator to `/admin/reset-password`. Reset sessions must belong to an allowlisted admin. Completing a reset signs out all sessions. The owner opens the email link in the same browser where the request began, because the PKCE verifier is stored in that browser.

The default Supabase email service is unsuitable as a general production mail-delivery guarantee. See [Supabase SMTP documentation](https://supabase.com/docs/guides/auth/auth-smtp).

## 5. Enable contact inquiries

Create a Cloudflare Turnstile widget for your actual host, and set:

```dotenv
NEXT_PUBLIC_TURNSTILE_SITE_KEY=YOUR_PUBLIC_WIDGET_KEY
TURNSTILE_SECRET_KEY=YOUR_SERVER_ONLY_SECRET
RATE_LIMIT_SALT=YOUR_LONG_RANDOM_SECRET
```

Generate the salt with a password manager or `openssl rand -hex 32`. Add your deployment domain and localhost to Turnstile as appropriate. Restart/redeploy after changing public environment variables.

The form requires a successful server-verified challenge with the expected hostname, validates field lengths, checks a honeypot and timing, and limits submissions by hashed IP and email. Requests go through a server route; anonymous clients cannot insert messages directly. Inquiry attachments remain private. The form fails clearly if the backend or security keys are missing.

Messages are available at **Admin → Messages**. Automatic notification emails are not enabled; the dashboard is the inbox. “Open email reply” opens your email client and does not send an email automatically.

## 6. Everyday content editing

| Task                                              | Where to go                      |
| ------------------------------------------------- | -------------------------------- |
| Name, role, intro and photograph                  | Profile → Edit                   |
| Biography and research philosophy                 | About                            |
| Homepage sections and their order                 | Homepage                         |
| Numbers in the impact strip                       | Research Impact                  |
| Research methodology, galleries and status        | Research Projects                |
| Journal or conference paper                       | Publications / Conference Papers |
| Engineering project                               | Academic Projects                |
| Consulting offering                               | Consulting Services              |
| Colours, fonts, navigation, email and footer      | Website Settings                 |
| External academic profiles                        | Social Links                     |
| Page titles, descriptions, canonical and indexing | SEO Settings                     |
| Photos, figures, credits, ALT and replacements    | Media Library                    |
| Current downloadable CV                           | CV                               |
| Private inquiries                                 | Messages                         |
| Portable content copies                           | Backup / Export                  |

A new entry starts as a draft. **Publish** makes it publicly visible if “Show on website” is enabled. “Featured” adds eligible research, publications, services, projects and insights to their homepage sections. Homepage sections can be hidden or reordered. Empty sections do not display invented content.

Drag a handle to reorder entries, or use **More → Move up / Move down** with a keyboard. Clear search/status filters before reordering. Use **Save draft**, then **Preview saved draft** to inspect private content. A duplicate becomes a new draft. Trash is reversible; permanent deletion is available only after moving an item to Trash.

Content is read from Supabase on each server request, so edits appear after the next page load without rebuilding the website. Open pages do not live-push updates; refresh them to see changes.

### Add a publication

Open **Publications → Add Publication**. Enter the title, ordered authors, year, journal, DOI, and abstract. Choose a PDF and featured image in the media controls. Enable “Featured” if required, then click **Publish**. It appears in the catalogue, its detail page, search, the sitemap, and the homepage if featured and the section is enabled. Citation and BibTeX defaults are generated only from the supplied fields; override them when precise journal formatting is required.

### Replace your photograph

Open **Profile → Edit → Media → Profile photograph → Choose or upload file**. Upload the image, enter ALT text and credit, use **Crop / reposition**, then publish the profile. Set portrait zoom to **1** for a freshly cropped photograph; adjust horizontal/vertical focus if needed. To replace an existing file everywhere, use **Media Library → Replace**. The media ID remains stable; public image URLs include the file update timestamp so optimized images refresh too.

### Replace your CV

Open **CV → Add CV Version**, choose a PDF, enable **Set as current CV**, and publish. All Download CV links resolve the active record. Old versions stay available for your records. The default public CV removes personal phone numbers, reference contacts and membership identifiers; inspect replacement PDFs before publishing.

### LinkedIn drafts

Under **News / Insights**, add an entry, paste the owner-authorised public post text, and enter the Original source URL. Save as a draft and review it. This deliberately avoids scraping private/login-protected LinkedIn pages or bulk-publishing unreviewed posts. External profile links require the “URL checked and identity confirmed” checkbox before publication.

## 7. Deploy to GitHub + Vercel

1. Create a private Git repository for this project. Do not include `.env.local`, `node_modules`, `.next`, backups, or private test credentials; `.gitignore` excludes them.
2. Import that repository into Vercel as a Next.js project. Use Node 22 or newer, `npm ci`, and the normal `npm run build` command.
3. Add the environment variables from `.env.example` in Vercel project settings. Set `CONTENT_PREVIEW=false`. Keep the service-role key and Turnstile secret server-only.
4. Set `NEXT_PUBLIC_SITE_URL` to the final HTTPS origin, with no path or trailing slash. Add the matching callback URL in Supabase and hostname in Turnstile.
5. Deploy and complete the launch checks in `docs/LAUNCH-CHECKLIST.md`.

The normal Vercel project URL works without buying a domain. For a custom domain, use Vercel **Project → Settings → Domains**, add your domain, and apply the exact DNS records Vercel shows. After verification, update `NEXT_PUBLIC_SITE_URL`, Supabase Auth URLs and Turnstile hostnames, then redeploy. No application source edit is required.

Do not assume a hosting plan permits commercial consulting use; check the provider's current terms and choose an appropriate plan. No paid service was purchased or activated during development. [Next.js on Vercel](https://vercel.com/docs/frameworks/full-stack/nextjs).

## 8. Backups and portability

Admin JSON export includes content, settings, media metadata, review notes and private inquiries. CSV exports content records in spreadsheet form and neutralises formula-leading cells. File bytes are separate. For content plus media and attachments:

```bash
npm run backup
```

Backups are saved in the ignored `backups/` directory. Keep encrypted offsite copies, and retain Supabase database backups. See `docs/BACKUP.md` for recovery and moving providers. No proprietary CMS service is required.

## 9. Validation

```bash
npm run typecheck
npm test
npm run build
npm run test:smoke
```

The database tests use an in-process PostgreSQL engine and Supabase-shaped Auth/Storage fixtures. They test SQL syntax, RLS visibility, non-admin denial, admin CRUD, ordering, CV selection and audit triggers. These do not replace live Supabase Auth/Storage tests.

Optional browser acceptance tests are in `tests/acceptance.spec.ts`. Install a local test browser (`npx playwright install chromium`), then run `npm run test:e2e`. Live-admin tests require a disposable staging Supabase project and an authenticated administrator storage-state file. Set `E2E_BASE_URL` to that staging origin, `E2E_ALLOW_MUTATIONS=staging` and `E2E_ADMIN_STATE` to the private storage-state file path. Never commit that file. Do not use a production site for those tests.

## File map

- `app/(public)/` — public pages and detail routes
- `app/admin/` — login, reset and protected administration
- `app/api/` — authenticated content/media APIs and validated inquiry delivery
- `components/` — public UI, editors, cropping, ordering and message tools
- `lib/collections.ts` — labelled CMS fields for all content categories
- `lib/seed-data.ts` — initial content; deployed edits live in the database
- `lib/supabase/` — browser/server clients and administrator checks
- `supabase/migrations/001_initial.sql` — tables, policies, private storage and functions
- `scripts/` — initial setup and complete media backup
- `docs/` — deployment, data sources, security, tests and launch instructions

## Practical limits

- Uploaded images are reduced to a maximum 2400-pixel edge, converted to WEBP and stripped of metadata. Safe SVG inputs are rasterised; active/external SVG content is rejected. Raster inputs are limited to 40 megapixels server-side.
- Admin uploads are capped at 4 MB after optimisation; inquiry attachments at 3 MB to fit typical serverless request limits. PDFs are identified by file signature; this is not malware scanning. Inquiries download PDFs as attachments.
- Rich text supports headings, links, lists, images, captions, quotes, tables and code. A specialised mathematical equation editor is not included.
- The CMS is a single-owner/admin-allowlist system, without multi-editor approval workflows or per-role permissions. Concurrent editing of the same content record is detected through its update timestamp.
- Previously published media remains public until you explicitly change its visibility, because it may be reused elsewhere. Private drafts do not make new media public until publication. Signed URLs expire after 60 seconds, but previously downloaded files cannot be recalled.
- Visitor statistics are not fabricated. Optional Google Analytics runs only after visitor consent. Core Web Vitals depend on the deployed server, database region, images and traffic; no numerical score is promised.

Official implementation references: [Supabase server-side clients](https://supabase.com/docs/guides/auth/server-side/creating-a-client?queryGroups=framework&framework=nextjs), [Next.js proxy/session refresh](https://nextjs.org/docs/app/api-reference/file-conventions/proxy), [Cloudflare Turnstile](https://developers.cloudflare.com/turnstile/get-started/client-side-rendering/).

The HTTP smoke test starts its own local production server and checks the bundled preview. Run it after a preview-mode build. If the build used a different `NEXT_PUBLIC_SITE_URL`, set `SMOKE_SITE_ORIGIN` to that same origin.
