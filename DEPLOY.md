# Tymr Website — Deployment Guide

This is the static marketing site for `tymrlogic.com`. Deploy-ready for Vercel, Netlify, or any static host.

---

## What's in this folder

| File | Purpose |
|---|---|
| `index.html` | Landing — hero, floor preview, features, AI agents section, cost tracking, "Notify Me at Launch" CTA |
| `pricing.html` | 3-tier pricing (Design Partner / Growth / Enterprise) + payback math |
| `security.html` | Compliance, infrastructure, access controls, subprocessors, vuln disclosure |
| `privacy.html` | GDPR/CCPA-aware privacy policy template |
| `terms.html` | Pragmatic SaaS terms template |
| `_shared.css` | Common styles for the 4 sub-pages (index has its own inline styles) |
| `favicon.png` | Tymr mark, cropped from your real logo PNG |
| `apple-touch-icon.png` | 180×180 mark for iOS home-screen |
| `vercel.json` | Clean URLs, security headers, `/signup` + `/login` redirects to app subdomain |
| `Branding and Marketing /` | Brand brief, original logo PNG, original landing (kept for reference) |

---

## Step 1 — Spot-check locally

You already have a server running on port 8091 serving this folder. Visit:

- http://localhost:8091/index.html — the landing
- http://localhost:8091/pricing.html
- http://localhost:8091/security.html
- http://localhost:8091/privacy.html
- http://localhost:8091/terms.html

(Clean URLs like `/pricing` only work once deployed to Vercel — locally you need `.html`.)

Confirm:
- [ ] Pricing/Security links in the landing's top nav go to the new pages
- [ ] All pages render with the light-theme palette (white bg, gold/cyan accents, green "mr")
- [ ] Footer on every page has Pricing · Security · Privacy · Terms
- [ ] Logo is the new Tymr mark (yellow square, black T)

---

## Step 2 — Initialize as a git repo and push to GitHub

This folder isn't a git repo yet. Make it one, push to a new `tymr-website` repo on GitHub:

```bash
cd /Users/kaluejibe/Documents/Tymr-website

# .gitignore
cat > .gitignore <<'EOF'
.DS_Store
.cursor/
node_modules/
*.log
EOF

git init
git add .
git commit -m "Initial website"

# Create the empty GitHub repo first at https://github.com/new — name it "tymr-website"
git branch -M main
git remote add origin https://github.com/YOUR_GITHUB/tymr-website.git
git push -u origin main
```

---

## Step 3 — Deploy to Vercel

1. Go to https://vercel.com/new
2. Click **Import** on the `tymr-website` repo
3. Framework preset: **Other** — Vercel auto-detects `vercel.json`
4. Root directory: leave blank (HTML files are at repo root)
5. Click **Deploy**

Within 30 seconds you'll get a preview URL like `tymr-website-abc.vercel.app`. Verify:
- [ ] Homepage loads
- [ ] `/pricing` (no `.html`) loads — clean URLs working
- [ ] `/security`, `/privacy`, `/terms` all load
- [ ] All "Notify Me at Launch" buttons open a mailto: compose
- [ ] Nav + footer cross-links work on every page
- [ ] Favicon shows the yellow Tymr mark in the browser tab

---

## Step 4 — Point tymrlogic.com at Vercel

In the Vercel dashboard for your project:

1. **Settings → Domains**
2. Add `tymrlogic.com` and `www.tymrlogic.com`
3. At your domain registrar, follow Vercel's DNS instructions:
   - Delete any existing `A`, `AAAA`, or `CNAME` records for `@` and `www` that point elsewhere
   - Add the `A` record Vercel shows (typically `76.76.21.21` for apex)
   - Add the `CNAME` record for `www` (typically `cname.vercel-dns.com`)
4. Vercel auto-provisions an SSL cert via Let's Encrypt once DNS propagates (5–30 min typical; up to 48h on slow registrars)

Check propagation:
```bash
dig tymrlogic.com +short
curl -sI https://tymrlogic.com | head -3
```

---

## Step 5 — (When ready) Switch to live demo CTAs

The landing currently uses `mailto:hello@tymrlogic.com?subject=Notify%20me%20at%20launch` for its hero CTA — that's a smart pre-launch pattern (collect email, qualify, follow up). When you're ready to book live demos:

1. Set up Calendly at `calendly.com/your-handle/demo`
2. Replace the mailto hrefs across all pages with your Calendly URL:

```bash
cd /Users/kaluejibe/Documents/Tymr-website

# Replace with your real Calendly handle:
sed -i '' 's|mailto:hello@tymrlogic.com?subject=Book%20a%20demo|https://calendly.com/YOUR_HANDLE/demo|g' *.html
sed -i '' 's|mailto:hello@tymrlogic.com?subject=Growth%20tier%20inquiry|https://calendly.com/YOUR_HANDLE/demo|g' *.html

# Add /demo short redirect to Calendly (optional but nice)
```

Or keep mailto: for the landing hero and just use Calendly on the pricing page. Mix as you prefer.

---

## Step 6 — Email sending DNS (before outbound)

Set up SPF / DKIM / DMARC on `tymrlogic.com` before sending any cold emails. Whether you use Google Workspace, Resend, Postmark, Instantly, or Smartlead — the shape is the same:

```
TXT  tymrlogic.com        "v=spf1 include:_spf.YOUR_SENDER ~all"
TXT  selector._domainkey.tymrlogic.com   "<DKIM key from your sender>"
TXT  _dmarc.tymrlogic.com "v=DMARC1; p=quarantine; rua=mailto:dmarc@tymrlogic.com"
```

Then warm the domain for 2 weeks:
- Week 1: 10 emails/day to friendly addresses
- Week 2: 30/day mix of friendly + prospects
- Week 3+: scale to 50–100/day, monitor reply + bounce rates

Don't skip warmup. A cold domain sending 30 unsolicited emails on day one lands in spam permanently.

---

## Step 7 — Point the app at the website

Once live, update the Flutter app (in `VirtualFloor` repo) to link to marketing pages:

- Settings → Org section: "Upgrade" button links to `https://tymrlogic.com/pricing`
- Empty-state "Learn more" buttons: link to relevant feature sections on tymrlogic.com
- Email footers from the app include the tymrlogic.com URL

I'll do these edits in the next session.

---

## Troubleshooting

### "Clean URLs aren't working — /pricing returns 404"
`vercel.json` must be at the repo root (not in a subfolder). Check Vercel build logs.

### "DNS points at Vercel but SSL cert fails"
Check for CAA records: `dig CAA tymrlogic.com`. If any exist, add `letsencrypt.org` to the allowlist at your registrar.

### "The favicon isn't updating"
Browsers cache favicons aggressively. Open devtools → Application → Clear Storage. Or visit `https://tymrlogic.com/favicon.png?v=2` to confirm the file is the new Tymr mark.

### "Branding and Marketing / folder is deployed too"
Harmless but messy. Add this to `.gitignore` if you don't want it public:
```
Branding and Marketing/
```
(Note the trailing space — your folder has one.) Or rename the folder to `_brand/` (underscore prefix) so it's obvious it's internal.

---

## When to ping me

- Landing is live on `tymrlogic.com` → I'll do a polish pass from the deployed URL
- You set up Calendly → I'll rewrite CTAs to use it + draft outbound email templates
- Anything in Steps 1–6 fails with a non-obvious error

Total time for Steps 1–4 (without DNS wait): **about 90 minutes**.
