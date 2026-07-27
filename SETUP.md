# Setup

This repo only renders on your GitHub profile page if it lives at
`github.com/ChinmayRout9040895625/ChinmayRout9040895625` (repo name must exactly
match your username).

## 1. Create and push the repo

1. On GitHub, create a new **public** repository named exactly `ChinmayRout9040895625`.
2. From this local folder:
   ```bash
   git remote add origin https://github.com/ChinmayRout9040895625/ChinmayRout9040895625.git
   git branch -M main
   git push -u origin main
   ```

## 2. Allow workflows to write

Repo → Settings → Actions → General → Workflow permissions → select
**"Read and write permissions"** → Save. Both workflows commit/push content and
will fail without this.

## 3. Add the TOKEN secret

The 3D-contribution-graph workflow force-pushes to `main`, which the default
`GITHUB_TOKEN` cannot do.

1. GitHub → Settings (your account) → Developer settings → Personal access
   tokens → Tokens (classic) → Generate new token → scope: `repo`.
2. Repo → Settings → Secrets and variables → Actions → New repository secret →
   name it exactly `TOKEN`, paste the token value.

## 4. Run both workflows once

Repo → Actions tab → select "Generate Snake Animation" → Run workflow → select
"3D Contribution Graph" → Run workflow. This generates the initial assets the
README already links to; after this they self-update on their schedules
(daily for both).

## Known service reliability notes

The stats/trophy images are served by free, community-run tools, not by us.
As of this writing:
- `github-readme-stats.vercel.app` is returning `503` (rate-limited) — this is
  why the Stats section uses `github-profile-summary-cards.vercel.app`
  instead. If you'd rather have exact-hex-colored stats cards, that project
  can be self-hosted on your own free Vercel account in a few minutes (fork
  + import to Vercel) once the public instance recovers.
- `github-profile-trophy.vercel.app` is fully disabled — the Trophies section
  uses the `github-trophies.vercel.app` mirror instead, which was confirmed
  working.
- If any image on your live profile ever shows broken, it's almost always
  one of these free services being temporarily down, not a bug in this repo.
