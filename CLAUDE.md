# Syfer Lead Intelligence System — CLAUDE.md
**Project:** Saratoga Hot Springs Resort | Group Sales Pipeline
**Owner:** Dylan Foster (dylan@coxgp.com)
**Last Updated:** April 3, 2026

---

## What This Is

Syfer is a weekly automated lead intelligence pipeline that surfaces 50 vetted, Apollo-enriched group sales prospects per run for Saratoga Hot Springs Resort in Saratoga, Wyoming. It targets Denver and Front Range Colorado organizations for group retreats, board offsites, staff trips, and team-building events.

The system scouts leads, scores them, enriches contact info via Apollo, writes personalized outbound emails, builds a password-protected dashboard, and pushes drafts to Gmail.

**Dashboard:** https://dylanfostercoxgp.github.io/Syfer-Dashboard/
**Password:** Saratogaleads
**Gmail account:** saratogagroupsales@gmail.com

---

## Trigger Phrases

| Phrase | Action |
|--------|--------|
| `Begin Manual Run` | Run the full 50-contact pipeline immediately |
| `Update email drafts` | Create Gmail drafts for all current leads |
| `Update email drafts` (after template change) | Rebuild all 50 emails in new format and re-draft |

---

## Deployment Rule — Always Update All 3 Targets

Every change to the dashboard or pipeline must be pushed to all three:

1. **Workspace folder:** `/Lead Dashboard/index.html`
2. **GitHub Pages:** `dylanfostercoxgp/Syfer-Dashboard` via `github_pusher.py`
3. **Scheduled task:** `syfer-dashboard-leads` (update prompt if email logic or pipeline changes)

Never skip any of the three. Never ask Dylan which ones to update.

---

## File Map

```
/Lead Dashboard/
  index.html                          # Main dashboard (single-file, deployed to GitHub Pages)
  CLAUDE.md                           # This file — project context for Claude
  Syfer_Process_Criteria.md           # Full search criteria, scoring, email rules, deployment targets
  Syfer_Production_Package_v3/
    syfer_config.json                 # GitHub token, repo owner/name, file path
    agents/
      email_writer.py                 # Email template generator (4 types: Corporate/Nonprofit/Medical/Outdoor)
      github_pusher.py                # Pushes index.html to GitHub Pages via API
      dashboard_gen.py                # Builds the HTML dashboard from the DB
      db_manager.py                   # SQLite insert/query helpers
      main.py                         # Pipeline entry point
      corporate_scout.py              # DuckDuckGo search for corporate/HR leads
      nonprofit_scout.py              # DuckDuckGo search for nonprofit/chamber leads
      medical_scout.py                # DuckDuckGo search for medical/practice leads
      outdoor_scout.py                # DuckDuckGo search for outdoor/adventure leads
    data/
      syfer.db                        # SQLite database (wiped clean April 3, 2026)
    output/
      dashboard.html                  # Local output copy of dashboard
```

---

## Key Technical Details

### Dashboard (index.html)
- Single-file HTML with embedded `const ALL_LEADS = [...]` JavaScript array
- Password overlay: `Saratogaleads` (element id: `pw-overlay`)
- Date selector filters leads by `found_date` field (format: `YYYY-MM-DD`)
- Each lead object requires: `org_name, org_type, city, state, score, status, email, phone, website, contact_name, found_date, description, full_notes, pitch_angle, draft_subjects (array), draft_email`
- VIEW button opens modal with: score, type, email, phone, website, contact, notes, pitch angle, subject lines, email body with COPY button
- Table columns: ORGANIZATION, NAME, PHONE, TYPE, PRIORITY, EMAIL, NOTES/EMAIL

### Apollo Enrichment
- Use `apollo_people_match` with the `id` parameter (NOT `apollo_people_bulk_match` — it fails)
- Run 10 parallel calls per batch for speed
- Phone number comes from `organization.primary_phone.number` (org phone, not personal mobile)
- Email field: `person.email` with `email_status: "verified"` required

### SQLite Database
- Path: `/Lead Dashboard/Syfer_Production_Package_v3/data/syfer.db`
- WARNING: disk I/O errors occur when writing directly to the mounted volume. Copy to /tmp first, work there, then copy back.
- Schema: `id, org_name, org_type, city, state, website, email, contact_name, source_query, description, score, status, found_date, first_seen, last_seen, run_id`
- No-repeat policy: deduplication by email address across all historical runs (starting April 3, 2026)

### GitHub Push
- Token stored in: `syfer_config.json` under `github.token`
- Push via: `python3 agents/github_pusher.py ../../index.html`
- Or call the `push()` function directly from `github_pusher.py`

### Gmail Drafts
- Account: `saratogagroupsales@gmail.com`
- Use `mcp__5a941daf-9cda-4281-9985-37dacb85b091__gmail_create_draft`
- contentType: `text/html` (emails use HTML with signature)
- Must have full OAuth scope (not read-only) — if create_draft fails, user needs to reconnect Gmail with write permissions

---

## Email Format (Current)

### Structure
```
Hey [First Name],

[Unique company/industry hook — 1-2 sentences specific to their role/org]

[Saratoga pitch — 2.5 hours from Denver, 2-3 property highlights with links, group program link, midweek special mention]

[Soft CTA: "If this sounds like a fit, just hit reply and our team will get back to you right away."]

Warm Regards,

[HTML Signature — Ted Roman, General Manager]
```

### Rules (Never Break)
- Opening: "Hey [First]," followed by two line breaks, then body
- Body must include the hook referencing their specific org/role
- Mention "2.5 hours from Denver" in the Saratoga paragraph
- Include exactly 3 links: hot-springs, groups, atv-rentals
- Midweek special: "(Monday through Thursday)"
- Close: "If this sounds like a fit, just hit reply and our team will get back to you right away."
- Sign-off: "Warm Regards,"
- Signature: Ted Roman, General Manager (HTML formatted with Saratoga brown #8E632F)
- NO em dashes
- NO group size numbers
- NO "I hope this finds you well" / "My name is..."

### Email Types by Org
| Type | Lead with | Key Highlights |
|------|-----------|----------------|
| Corporate | Team offsite angle, "something worth the drive" | ATV, hot springs, meeting room, brewery |
| Nonprofit | Board/leadership retreat, "earned rest" | Hot springs, meeting room, brewery, ATV option |
| Medical | Staff wellness, "real recovery" | Hot springs, spa, ATV option, brewery |
| Outdoor | Adventure + terrain match | ATV, North Platte River, hot springs recovery, brewery |

### Signature (HTML)
- Name: Ted Roman
- Title: General Manager
- Company: Saratoga Hot Springs Resort
- Address: 601 East Pic Pike Road, Saratoga, Wyoming 82331
- Phone: 307-326-5261
- Email: info@shsr.us
- Website: https://saratogahotspringsresort.com/
- Colors: Primary brown #8E632F, text #1a1714, background cream #f4f0e8

---

## Scoring System

Base score: 5. Add points per signal. Must reach 7+ to include in dashboard.

| Signal | Points |
|--------|--------|
| Board/leadership retreat language | +4-5 |
| Annual outing / group trip | +4 |
| Corporate offsite / team building | +3-4 |
| Outdoor activity match (ATV, hunting, fly fishing) | +3-4 |
| Planning / looking / seeking language | +2 |

---

## Weekly Run Schedule

**Automated:** Every Monday at 5:06 AM (task: `syfer-dashboard-leads`)
**Manual trigger:** "Begin Manual Run"
**Contacts per run:** 50
**No-repeat policy:** Started April 3, 2026. Deduplication by email at insert time.

### Pipeline Steps
1. Run 4 scouts (Corporate, Nonprofit, Medical, Outdoor) via DuckDuckGo
2. Classify each result (real company vs blog/directory — skip non-companies)
3. Score with signal-based system (base 5, must reach 7+)
4. Filter: verified email, real company, Front Range CO
5. Enrich via Apollo: `apollo_mixed_people_api_search` to find ID, then `apollo_people_match` with id
6. Generate personalized email per lead using current email format
7. Deduplicate against SQLite DB, insert new leads with today's date
8. Add date option to dashboard date selector, append to ALL_LEADS array
9. Deploy to all 3 targets (workspace, GitHub, scheduled task)
10. Create Gmail drafts in saratogagroupsales@gmail.com

---

## Property Highlights (Use 2-3 per email based on type)

- Natural hot springs (large pool + smaller soaking pools)
- Full spa on property
- ATV rentals + Wyoming Outdoor Adventures outfitter
- Snowy Mountain Brewery (private tours, private dining with round-table seating)
- Dedicated small-group meeting room
- 9-hole golf course (only for appropriate audiences)
- North Platte River (fly fishing, outdoor setting)

---

## Lead Intelligence Report

A styled HTML email report is created each run and drafted to:
- **To:** rodrick@coxgp.com
- **CC:** dylan@coxgp.com
- **Subject:** Saratoga Lead Intelligence Report [Date]
- Content: Top leads table with org, contact, type, score, email, and why qualified

---

## Common Issues and Fixes

| Issue | Fix |
|-------|-----|
| SQLite disk I/O error | Copy syfer.db to /tmp, work there, copy back |
| apollo_people_bulk_match returns 0 results | Use apollo_people_match individually with the id field |
| Gmail create_draft fails | Gmail connected with read-only OAuth — user must reconnect with full scope |
| index.html too large to read at once | Use offset/limit params, or grep for specific line numbers |
| ALL_LEADS array not updating | Check that regex replace targets the full array including newlines (re.DOTALL flag) |
