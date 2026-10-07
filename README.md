# VISTAARA 2026 – Registration System

Static website (GitHub Pages) + Supabase (free tier) for email OTP, database, private file storage and admin login.
Files: `index.html` (registration wizard), `admin.html` (dashboard + CSV export), `js/config.js` (all settings), `js/app.js`, `css/style.css`, `supabase/schema.sql` (database, security rules, storage).

## 1. Backend setup (about 15 minutes)
1. Create a free account at supabase.com and a new project.
2. Open **SQL Editor**, paste all of `supabase/schema.sql`, click **Run**. This creates the tables, security rules, the registration function and the private `payment-proofs` bucket.
3. **Authentication → Email Templates → Magic Link**: replace the body with `<h2>VISTAARA 2026</h2><p>Your verification code is: <b>{{ .Token }}</b></p>` so users receive a code, not a link.
4. **Authentication → Providers → Email**: keep "Confirm email" on and set OTP expiry to 600 seconds.
5. **Authentication → SMTP Settings**: add a custom SMTP service (Gmail app password, Brevo, Resend...). Supabase's built-in email sender allows only a few emails per hour and is not enough for a public event.
6. Add yourself as admin: in SQL Editor run `insert into admins(email) values ('your@email');`

## 2. Configuration
Edit `js/config.js`: paste **Project URL** and **anon public key** (Supabase → Project Settings → API), payment details, UPI QR image path (put image in `assets/qr/`), contact details, fees, departments. Never paste the `service_role` key anywhere.
If you change fees or the mixed-team rule, change the same values at the top of `submit_registration` in `schema.sql` and run it again. The server calculates the real fee; the browser value is only for display.

## 3. Test locally
Run `python3 -m http.server 8000` in this folder, open http://localhost:8000. In Supabase → Authentication → URL Configuration add that URL (and later your GitHub Pages URL) as allowed Site URLs.

## 4. Publish on GitHub Pages
1. github.com → New repository (public) → upload all files of this folder (drag and drop, keep the folder structure).
2. Settings → Pages → Source: *Deploy from a branch*, branch `main`, folder `/ (root)` → Save.
3. After a minute the site is live at `https://USERNAME.github.io/REPO/`. Admin: `.../admin.html`.
4. Custom domain: Settings → Pages → Custom domain, then add the DNS records GitHub shows.

## 5. Excel export
Open `admin.html`, sign in with the admin email code, apply filters if needed, click **Export Excel (CSV)**. Excel opens the file directly. Payment proofs open via short-lived private links.

## 6. Troubleshooting
- *Code not received*: check spam; set up custom SMTP (step 5).
- *Email contains a link, not a code*: edit the template (step 3).
- *"Could not send the code"*: URL/key in config.js wrong, or email rate limit reached.
- *Upload or submit fails*: confirm schema.sql ran without errors.
- *Admin says not administrator*: run the insert in step 6 with the exact email.

## 7. Security notes
GitHub Pages is static and gives no backend security. Protection comes from Supabase: row-level security, a server-side registration function that re-validates everything and computes the fee, a private storage bucket with size/type limits, OTP expiry and Supabase rate limits. Only public anon keys are in the code. Limitation: the registration form can only be rate-limited by Supabase's own limits; add CAPTCHA (Authentication → Attack Protection) for heavy public traffic.
