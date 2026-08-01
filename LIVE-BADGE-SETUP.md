# Making the TryHackMe activity panel live

This wires `assets/thm-activity.svg` up to your **real, current** TryHackMe
stats (rank/level, points, streak) instead of a snapshot — no login,
password, or paid API needed. It works because TryHackMe's own badge embed
is served from a public, unauthenticated URL:

```
https://tryhackme.com/api/v2/badges/public-profile?userPublicId=<your id>
```

## One-time setup

1. Log into TryHackMe → go to your profile → **Settings → Badge**.
2. Copy the iframe embed code they give you. It looks like:
   `<iframe src="https://tryhackme.com/api/v2/badges/public-profile?userPublicId=1234567">`
3. Grab the number after `userPublicId=` — that's your `THM_USER_PUBLIC_ID`.
4. In your `goyalpranav2008` repo on GitHub: **Settings → Secrets and
   variables → Actions → Variables tab → New repository variable**
   - Name: `THM_USER_PUBLIC_ID`
   - Value: the number from step 3
5. Push these files to the repo, keeping the folder structure:
   - `.github/workflows/update-thm-activity.yml`
   - `scripts/update_thm_activity.py`
   - `assets/thm-activity.svg` (already in the README, just gets overwritten)
6. Go to the **Actions** tab → run **"Update TryHackMe Live Activity"**
   manually once (`Run workflow` button) to generate the first live version.
   After that it refreshes itself every 6 hours automatically.

## What it actually does

- Fetches the small public HTML fragment TryHackMe serves for the badge
- Parses out your username, level, points, and streak with BeautifulSoup
- Rebuilds `assets/thm-activity.svg` — same radar + typing-terminal look —
  with the real numbers baked into the text
- Commits the updated SVG back to the repo if anything changed

## Heads up

TryHackMe hasn't published a schema for that badge fragment, so the script
maps the four text values by position (`username, level, points, streak`).
If TryHackMe ever changes that page's layout, the mapping in
`scripts/update_thm_activity.py` (the `fetch_stats()` function) may need a
quick tweak — the raw scraped values are also saved to `data/thm-stats.json`
each run so you can check what actually came back.
