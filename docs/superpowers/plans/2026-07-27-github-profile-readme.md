# GitHub Profile README Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a GitHub special profile README repo (`ChinmayRout9040895625/ChinmayRout9040895625`) with Chinmay Rout's content, a navy/cyan visual theme, and the same feature set as the cloned `zyh3699/zyh3699` repo (header, stats, snake animation, 3D contribution graph, trophies).

**Architecture:** Pure markdown + external badge/card-generating services (capsule-render, shields.io, skillicons.dev, github-profile-summary-cards, streak-stats, github-trophies) stitched together via URL query params, plus 2 GitHub Actions workflows that regenerate the snake animation and 3D contribution graph on a schedule.

**Tech Stack:** Markdown, YAML (GitHub Actions), JSON (action config). No app code, no build step.

## Global Constraints

- GitHub username / repo name: `ChinmayRout9040895625` (must match exactly for the README to render on the profile page).
- Color theme: gradient `#0F172A` → `#1E3A8A` → `#06B6D4`; card backgrounds `#0F172A`; text `#C9D1D9`; accents `#06B6D4`. Minimal animation — only the typing tagline and snake graph animate; no decorative giphy icons on headings.
- Contact: Email `chinmayrout2005.jaj@gmail.com`, LinkedIn `https://www.linkedin.com/in/chinmay-rout-b8797429b/`, X `https://x.com/chinmayrout2005`.
- Every external service URL/icon slug used below has been verified live (via curl) at plan-writing time. `github-readme-stats.vercel.app` and `github-profile-trophy.vercel.app` are currently down/rate-limited — do not use them; use the verified replacements specified in each task instead.
- No Python is installed in this environment; JSON/YAML validation in this plan uses Node.js (`node`, confirmed v22.16.0) and `npx js-yaml` (confirmed working, network access confirmed available).

---

## Task 1: 3D contribution graph theme config

**Files:**
- Create: `profile-3d-contrib/settings.json`

**Interfaces:**
- Produces: a JSON file consumed by the `yoshi389111/github-profile-3d-contrib` action in Task 3's workflow (that action reads this file's colors to render the 3D graph).

- [ ] **Step 1: Create the settings file**

```json
{
  "type": "rainbow",
  "backgroundColor": "#0F172A",
  "foregroundColor": "#C9D1D9",
  "strongColor": "#ffffff",
  "weakColor": "#334155",
  "radarColor": "#06B6D4",
  "growingAnimation": true,
  "contribColors": [
    "#0F172A",
    "#1E3A8A",
    "#2563EB",
    "#0891B2",
    "#06B6D4"
  ]
}
```

- [ ] **Step 2: Validate JSON syntax**

Run: `node -e "JSON.parse(require('fs').readFileSync('profile-3d-contrib/settings.json','utf8')); console.log('valid')"`
Expected: `valid`

- [ ] **Step 3: Commit**

```bash
git add profile-3d-contrib/settings.json
git commit -m "Add navy/cyan theme config for 3D contribution graph"
```

---

## Task 2: Snake animation workflow

**Files:**
- Create: `.github/workflows/snake.yml`

**Interfaces:**
- Produces: a workflow that pushes `github-contribution-grid-snake.svg`, `github-contribution-grid-snake-dark.svg`, and `ocean.gif` to an `output` branch. Task 6's README references these exact filenames via `raw.githubusercontent.com/ChinmayRout9040895625/ChinmayRout9040895625/output/<filename>`.

- [ ] **Step 1: Create the workflow file**

```yaml
name: Generate Snake Animation

on:
  schedule:
    - cron: "0 0 * * *"
  workflow_dispatch:
  push:
    branches:
      - main
      - master

permissions:
  contents: write

jobs:
  generate:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v3

      - name: Generate Snake
        uses: Platane/snk@v3
        with:
          github_user_name: ${{ github.repository_owner }}
          outputs: |
            dist/github-contribution-grid-snake.svg
            dist/github-contribution-grid-snake-dark.svg?palette=github-dark&color_snake=#06B6D4&color_dots=#0F172A,#1E3A8A,#2563EB,#0891B2,#06B6D4
            dist/ocean.gif?color_snake=#06B6D4&color_dots=#0F172A,#1E3A8A,#2563EB,#0891B2,#06B6D4

      - name: Push to Output Branch
        uses: crazy-max/ghaction-github-pages@v3.1.0
        with:
          target_branch: output
          build_dir: dist
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

- [ ] **Step 2: Validate YAML syntax**

Run: `npx --yes js-yaml .github/workflows/snake.yml`
Expected: prints the parsed structure with no error (a YAML syntax error would throw and print `YAMLException`).

- [ ] **Step 3: Commit**

```bash
git add .github/workflows/snake.yml
git commit -m "Add snake animation workflow with navy/cyan palette"
```

---

## Task 3: 3D contribution graph workflow

**Files:**
- Create: `.github/workflows/profile-3d.yml`

**Interfaces:**
- Consumes: `profile-3d-contrib/settings.json` (Task 1) as the color source the embedded heredoc mirrors.
- Produces: commits `profile-3d-contrib/profile-night-rainbow.svg` (among other variants) to `main`. Task 6's README references this exact path.
- Requires a repo secret named `TOKEN` (classic PAT, `repo` scope) — documented in Task 8's SETUP.md.

- [ ] **Step 1: Create the workflow file**

```yaml
name: 3D Contribution Graph

on:
  schedule:
    - cron: "0 18 * * *"
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest
    name: Generate 3D Contribution Graph
    steps:
      - uses: actions/checkout@v3
        with:
          token: ${{ secrets.TOKEN }}
      - name: Write theme settings
        run: |
          mkdir -p profile-3d-contrib
          cat > profile-3d-contrib/settings.json << 'EOL'
          {
            "type": "rainbow",
            "backgroundColor": "#0F172A",
            "foregroundColor": "#C9D1D9",
            "strongColor": "#ffffff",
            "weakColor": "#334155",
            "radarColor": "#06B6D4",
            "growingAnimation": true,
            "contribColors": [
              "#0F172A",
              "#1E3A8A",
              "#2563EB",
              "#0891B2",
              "#06B6D4"
            ]
          }
          EOL
      - uses: yoshi389111/github-profile-3d-contrib@0.7.1
        env:
          GITHUB_TOKEN: ${{ secrets.TOKEN }}
          USERNAME: ChinmayRout9040895625
      - name: Commit and push
        run: |
          git config user.name github-actions
          git config user.email github-actions@github.com
          git add -A .
          git commit -m "Update 3D contribution graph" || exit 0
          git remote set-url origin https://x-access-token:${{ secrets.TOKEN }}@github.com/ChinmayRout9040895625/ChinmayRout9040895625.git
          git push origin HEAD:main --force
```

- [ ] **Step 2: Validate YAML syntax**

Run: `npx --yes js-yaml .github/workflows/profile-3d.yml`
Expected: prints the parsed structure with no error.

- [ ] **Step 3: Commit**

```bash
git add .github/workflows/profile-3d.yml
git commit -m "Add 3D contribution graph workflow with navy/cyan palette"
```

---

## Task 4: README — Header, About Me, Currently Working On

**Files:**
- Create: `README.md`

**Interfaces:**
- Produces: the start of `README.md`, ending with a closed `</div>` after the badge row and two complete `<h2>` sections. Task 5 appends directly after this.

- [ ] **Step 1: Write the file**

```markdown
<div align="center">

<img src="https://capsule-render.vercel.app/api?type=waving&color=0:0F172A,50:1E3A8A,100:06B6D4&height=200&section=header&text=Chinmay%20Rout&fontSize=60&fontAlignY=38&animation=fadeIn&fontColor=ffffff&desc=AI%2FML%20Engineer%20%C2%B7%20Generative%20AI%20%C2%B7%20Agentic%20AI&descAlignY=58&descSize=17" />

[![Typing SVG](https://readme-typing-svg.demolab.com?font=Fira+Code&weight=600&size=24&duration=2500&pause=800&color=06B6D4&center=true&vCenter=true&width=600&lines=AI%2FML+Engineer;Generative+AI;Agentic+AI+Builder)](https://git.io/typing-svg)

<p align="center">
  <a href="https://github.com/ChinmayRout9040895625"><img src="https://img.shields.io/badge/GitHub-0F172A?style=for-the-badge&logo=github&logoColor=06B6D4" alt="GitHub"/></a>
  <a href="mailto:chinmayrout2005.jaj@gmail.com"><img src="https://img.shields.io/badge/Email-0F172A?style=for-the-badge&logo=gmail&logoColor=06B6D4" alt="Email"/></a>
  <a href="https://www.linkedin.com/in/chinmay-rout-b8797429b/"><img src="https://img.shields.io/badge/LinkedIn-0F172A?style=for-the-badge&logo=linkedin&logoColor=06B6D4" alt="LinkedIn"/></a>
  <a href="https://x.com/chinmayrout2005"><img src="https://img.shields.io/badge/X-0F172A?style=for-the-badge&logo=x&logoColor=06B6D4" alt="X"/></a>
</p>

</div>

<h2 align="center">About Me</h2>

<p align="center">
I'm an AI/ML Engineer focused on Generative AI and Agentic AI systems — building LLM-powered agents,<br/>
Retrieval-Augmented Generation pipelines, and the infrastructure that lets models reason, retrieve, and act reliably.<br/>
I care about turning research-grade AI ideas into systems that run in production.
</p>

<h2 align="center">Currently Working On</h2>

<p align="center">

- Multi-Agent Systems
- RAG Pipelines
- AI Automation
- Open Source

</p>
```

- [ ] **Step 2: Verify required markers are present**

Run: `node -e "const s=require('fs').readFileSync('README.md','utf8'); ['Chinmay Rout','About Me','Currently Working On','chinmayrout2005.jaj@gmail.com','linkedin.com/in/chinmay-rout-b8797429b','x.com/chinmayrout2005'].forEach(m=>{if(!s.includes(m))throw new Error('missing: '+m)}); console.log('ok')"`
Expected: `ok`

- [ ] **Step 3: Commit**

```bash
git add README.md
git commit -m "Add README header, About Me, and Currently Working On sections"
```

---

## Task 5: README — Skills & Tools

**Files:**
- Modify: `README.md` (append)

**Interfaces:**
- Consumes: end of file from Task 4 (last line is `</p>` closing "Currently Working On").
- Produces: appended, self-contained "Skills & Tools" section. Task 6 appends directly after this.

- [ ] **Step 1: Append the section**

```markdown

<h2 align="center">Skills & Tools</h2>

<div align="center">
  <img src="https://skillicons.dev/icons?i=python,cpp,pytorch,tensorflow,sklearn,pandas,opencv,fastapi,flask,docker,git,linux,postgres,react,ts,js&theme=dark&perline=8" />
</div>

<p align="center">
  <img src="https://img.shields.io/badge/NumPy-0F172A?style=for-the-badge&logo=numpy&logoColor=06B6D4" alt="NumPy"/>
  <img src="https://img.shields.io/badge/Hugging%20Face-0F172A?style=for-the-badge&logo=huggingface&logoColor=06B6D4" alt="Hugging Face"/>
  <img src="https://img.shields.io/badge/LangChain-0F172A?style=for-the-badge&logo=langchain&logoColor=06B6D4" alt="LangChain"/>
  <img src="https://img.shields.io/badge/LangGraph-0F172A?style=for-the-badge&logo=langgraph&logoColor=06B6D4" alt="LangGraph"/>
  <img src="https://img.shields.io/badge/OpenAI%20API-0F172A?style=for-the-badge" alt="OpenAI API"/>
  <img src="https://img.shields.io/badge/Anthropic%20API-0F172A?style=for-the-badge&logo=anthropic&logoColor=06B6D4" alt="Anthropic API"/>
  <img src="https://img.shields.io/badge/FAISS-0F172A?style=for-the-badge" alt="FAISS"/>
  <img src="https://img.shields.io/badge/ChromaDB-0F172A?style=for-the-badge" alt="ChromaDB"/>
  <img src="https://img.shields.io/badge/Streamlit-0F172A?style=for-the-badge&logo=streamlit&logoColor=06B6D4" alt="Streamlit"/>
  <img src="https://img.shields.io/badge/SQL-0F172A?style=for-the-badge" alt="SQL"/>
</p>
```

- [ ] **Step 2: Verify markers**

Run: `node -e "const s=require('fs').readFileSync('README.md','utf8'); ['skillicons.dev/icons','Skills & Tools','LangGraph','ChromaDB'].forEach(m=>{if(!s.includes(m))throw new Error('missing: '+m)}); console.log('ok')"`
Expected: `ok`

- [ ] **Step 3: Commit**

```bash
git add README.md
git commit -m "Add Skills & Tools section to README"
```

---

## Task 6: README — Stats, Streak, and Contribution Graphs

**Files:**
- Modify: `README.md` (append)

**Interfaces:**
- Consumes: end of file from Task 5; also references `output` branch files from Task 2 and `profile-3d-contrib/profile-night-rainbow.svg` from Task 3 (both generated by their workflows, not present until those run — this is expected, see Task 8).
- Produces: appended "GitHub Stats" and "Contribution Activity" sections.

- [ ] **Step 1: Append the section**

```markdown

<h2 align="center">GitHub Stats</h2>

<div align="center">
  <img src="https://github-profile-summary-cards.vercel.app/api/cards/stats?username=ChinmayRout9040895625&theme=tokyonight" width="32%" />
  <img src="https://github-profile-summary-cards.vercel.app/api/cards/repos-per-language?username=ChinmayRout9040895625&theme=tokyonight" width="32%" />
  <img src="https://github-profile-summary-cards.vercel.app/api/cards/most-commit-language?username=ChinmayRout9040895625&theme=tokyonight" width="32%" />
</div>

<div align="center">
  <img src="https://streak-stats.demolab.com/?user=ChinmayRout9040895625&background=0F172A&ring=06B6D4&fire=06B6D4&currStreakLabel=06B6D4&sideLabels=C9D1D9&currStreakNum=C9D1D9&sideNums=C9D1D9&dates=8B96A5&border=1E3A8A&border_radius=8" width="100%" />
</div>

<h2 align="center">Contribution Activity</h2>

<div align="center">

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/ChinmayRout9040895625/ChinmayRout9040895625/output/github-contribution-grid-snake-dark.svg">
  <source media="(prefers-color-scheme: light)" srcset="https://raw.githubusercontent.com/ChinmayRout9040895625/ChinmayRout9040895625/output/github-contribution-grid-snake.svg">
  <img alt="github contribution grid snake animation" src="https://raw.githubusercontent.com/ChinmayRout9040895625/ChinmayRout9040895625/output/github-contribution-grid-snake-dark.svg" width="100%">
</picture>

<img src="https://raw.githubusercontent.com/ChinmayRout9040895625/ChinmayRout9040895625/main/profile-3d-contrib/profile-night-rainbow.svg" width="100%" />

</div>
```

- [ ] **Step 2: Verify markers**

Run: `node -e "const s=require('fs').readFileSync('README.md','utf8'); ['GitHub Stats','streak-stats.demolab.com','github-contribution-grid-snake-dark.svg','profile-night-rainbow.svg'].forEach(m=>{if(!s.includes(m))throw new Error('missing: '+m)}); console.log('ok')"`
Expected: `ok`

- [ ] **Step 3: Commit**

```bash
git add README.md
git commit -m "Add GitHub stats, streak, and contribution graph sections to README"
```

---

## Task 7: README — Featured Projects, Trophies, Footer

**Files:**
- Modify: `README.md` (append, finalize)

**Interfaces:**
- Consumes: end of file from Task 6.
- Produces: the complete, final `README.md`.

- [ ] **Step 1: Append the section**

```markdown

<h2 align="center">Featured Projects</h2>

<img src="https://capsule-render.vercel.app/api?type=rect&color=0:0F172A,100:06B6D4&height=3&section=header" width="100%"/>

**MemCore** — Production-grade long-term memory infrastructure for AI agents

<img src="https://img.shields.io/badge/Python-0F172A?style=flat-square&logo=python&logoColor=06B6D4"/> <img src="https://img.shields.io/badge/FastAPI-0F172A?style=flat-square&logo=fastapi&logoColor=06B6D4"/> <img src="https://img.shields.io/badge/PostgreSQL-0F172A?style=flat-square&logo=postgresql&logoColor=06B6D4"/> <img src="https://img.shields.io/badge/Qdrant-0F172A?style=flat-square&logo=qdrant&logoColor=06B6D4"/> <img src="https://img.shields.io/badge/Neo4j-0F172A?style=flat-square&logo=neo4j&logoColor=06B6D4"/> <img src="https://img.shields.io/badge/Redis-0F172A?style=flat-square&logo=redis&logoColor=06B6D4"/> <img src="https://img.shields.io/badge/Docker-0F172A?style=flat-square&logo=docker&logoColor=06B6D4"/> <img src="https://img.shields.io/badge/Kubernetes-0F172A?style=flat-square&logo=kubernetes&logoColor=06B6D4"/> <img src="https://img.shields.io/badge/Celery-0F172A?style=flat-square&logo=celery&logoColor=06B6D4"/> <img src="https://img.shields.io/badge/Linux-0F172A?style=flat-square&logo=linux&logoColor=06B6D4"/>

<img src="https://capsule-render.vercel.app/api?type=rect&color=0:0F172A,100:06B6D4&height=3&section=header" width="100%"/>

**SwarmAI** — Multi-agent personal assistant

<img src="https://img.shields.io/badge/Python-0F172A?style=flat-square&logo=python&logoColor=06B6D4"/> <img src="https://img.shields.io/badge/OpenAI%20GPT--4o%20API-0F172A?style=flat-square"/> <img src="https://img.shields.io/badge/Telegram%20Bot%20API-0F172A?style=flat-square&logo=telegram&logoColor=06B6D4"/> <img src="https://img.shields.io/badge/Google%20Calendar-0F172A?style=flat-square&logo=googlecalendar&logoColor=06B6D4"/> <img src="https://img.shields.io/badge/Gmail%20API-0F172A?style=flat-square&logo=gmail&logoColor=06B6D4"/> <img src="https://img.shields.io/badge/Docker-0F172A?style=flat-square&logo=docker&logoColor=06B6D4"/> <img src="https://img.shields.io/badge/pytest-0F172A?style=flat-square&logo=pytest&logoColor=06B6D4"/>

<img src="https://capsule-render.vercel.app/api?type=rect&color=0:0F172A,100:06B6D4&height=3&section=header" width="100%"/>

**AI Media Intelligence Agent** — Conversational analytics, sentiment analysis, predictive forecasting, live social media monitoring, RAG, and interactive dashboards

<img src="https://img.shields.io/badge/Python-0F172A?style=flat-square&logo=python&logoColor=06B6D4"/> <img src="https://img.shields.io/badge/Streamlit-0F172A?style=flat-square&logo=streamlit&logoColor=06B6D4"/> <img src="https://img.shields.io/badge/LangChain-0F172A?style=flat-square&logo=langchain&logoColor=06B6D4"/> <img src="https://img.shields.io/badge/OpenAI-0F172A?style=flat-square"/> <img src="https://img.shields.io/badge/ChromaDB-0F172A?style=flat-square"/> <img src="https://img.shields.io/badge/Pandas-0F172A?style=flat-square&logo=pandas&logoColor=06B6D4"/> <img src="https://img.shields.io/badge/Plotly-0F172A?style=flat-square&logo=plotly&logoColor=06B6D4"/>

<img src="https://capsule-render.vercel.app/api?type=rect&color=0:0F172A,100:06B6D4&height=3&section=header" width="100%"/>

**Document-Aware RAG Chatbot** — Dockerized RAG chatbot that answers strictly from provided documents

<img src="https://img.shields.io/badge/Python-0F172A?style=flat-square&logo=python&logoColor=06B6D4"/> <img src="https://img.shields.io/badge/LangChain-0F172A?style=flat-square&logo=langchain&logoColor=06B6D4"/> <img src="https://img.shields.io/badge/FAISS-0F172A?style=flat-square"/> <img src="https://img.shields.io/badge/Ollama-0F172A?style=flat-square&logo=ollama&logoColor=06B6D4"/> <img src="https://img.shields.io/badge/Streamlit-0F172A?style=flat-square&logo=streamlit&logoColor=06B6D4"/> <img src="https://img.shields.io/badge/Docker-0F172A?style=flat-square&logo=docker&logoColor=06B6D4"/>

<h2 align="center">Trophies</h2>

<div align="center">
  <img src="https://github-trophies.vercel.app/?username=ChinmayRout9040895625&theme=onedark&no-frame=true&row=1&column=7" />
</div>

<p align="center"><i>Thanks for visiting — let's build something intelligent together.</i></p>

<img src="https://capsule-render.vercel.app/api?type=waving&color=0:0F172A,50:1E3A8A,100:06B6D4&height=120&section=footer" width="100%"/>
```

- [ ] **Step 2: Verify markers**

Run: `node -e "const s=require('fs').readFileSync('README.md','utf8'); ['MemCore','SwarmAI','AI Media Intelligence Agent','Document-Aware RAG Chatbot','github-trophies.vercel.app','section=footer'].forEach(m=>{if(!s.includes(m))throw new Error('missing: '+m)}); console.log('ok')"`
Expected: `ok`

- [ ] **Step 3: Commit**

```bash
git add README.md
git commit -m "Add Featured Projects, Trophies, and footer to README"
```

---

## Task 8: Setup instructions

**Files:**
- Create: `SETUP.md`

**Interfaces:**
- None (documentation only, not read by any other task).

- [ ] **Step 1: Write the file**

```markdown
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
```

- [ ] **Step 2: Verify the file exists and is non-empty**

Run: `node -e "const s=require('fs').readFileSync('SETUP.md','utf8'); if(s.length<200) throw new Error('too short'); console.log('ok', s.length)"`
Expected: `ok <some number > 200>`

- [ ] **Step 3: Commit**

```bash
git add SETUP.md
git commit -m "Add setup instructions for repo, secrets, and workflow permissions"
```

---

## Task 9: Final validation pass

**Files:**
- None created/modified — validation only.

**Interfaces:**
- Consumes: all files from Tasks 1-8.

- [ ] **Step 1: Re-validate the JSON config**

Run: `node -e "JSON.parse(require('fs').readFileSync('profile-3d-contrib/settings.json','utf8')); console.log('json ok')"`
Expected: `json ok`

- [ ] **Step 2: Re-validate both workflow YAML files**

Run: `npx --yes js-yaml .github/workflows/snake.yml > /dev/null && npx --yes js-yaml .github/workflows/profile-3d.yml > /dev/null && echo "yaml ok"`
Expected: `yaml ok`

- [ ] **Step 3: Check README for balanced container tags**

Run: `node -e "const s=require('fs').readFileSync('README.md','utf8'); const open=(s.match(/<div align=\"center\">/g)||[]).length; const close=(s.match(/<\/div>/g)||[]).length; if(open!==close) throw new Error('unbalanced div: '+open+' vs '+close); console.log('div balance ok', open, close)"`
Expected: `div balance ok <N> <N>` (equal counts)

- [ ] **Step 4: List every external URL referenced in the README for a final manual skim**

Run: `node -e "const s=require('fs').readFileSync('README.md','utf8'); const m=s.match(/https?:\/\/[^\s\"')]+/g); [...new Set(m)].forEach(u=>console.log(u))"`
Expected: prints the full list of distinct URLs; skim for the correct username (`ChinmayRout9040895625`) and no leftover `zyh3699`/`yuanhao` references.

- [ ] **Step 5: Confirm no leftover references to the original repo's owner**

Run: `node -e "const fs=require('fs'); const files=['README.md','.github/workflows/snake.yml','.github/workflows/profile-3d.yml','profile-3d-contrib/settings.json','SETUP.md']; const hits=files.filter(f=>fs.readFileSync(f,'utf8').match(/zyh3699|Yuanhao|zephyrzhong/i)); if(hits.length) throw new Error('leftover refs in: '+hits.join(', ')); console.log('clean')"`
Expected: `clean`

- [ ] **Step 6: Final commit (if anything changed)**

```bash
git status
git add -A
git commit -m "Final validation pass"
```

If `git status` shows nothing to commit, skip this step.
