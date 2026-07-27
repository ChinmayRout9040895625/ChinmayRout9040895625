# GitHub Profile README — Design Spec

Date: 2026-07-27

## Purpose

Build a GitHub "special" profile README (repo name == username) for Chinmay Rout
(`ChinmayRout9040895625`), modeled on the cloned `zyh3699/zyh3699` repo's structure
and feature set, but with entirely different content, tech stack, and a custom
navy/cyan visual theme.

## Constraints

- GitHub sanitizes README markdown: no custom CSS/JS, no real glassmorphism/glow.
  The navy/cyan "premium AI product" look is approximated using hex-color params
  supported by the external badge/card-generating services (capsule-render,
  shields.io, github-readme-stats, skillicons.dev, github-profile-trophy,
  github-profile-3d-contrib), not literal CSS effects.
- The repo must be named exactly `ChinmayRout9040895625` for GitHub to render its
  README on the profile page. That repo does not exist yet — files are built
  locally first, then pushed.
- The 3D-contribution-graph workflow force-pushes to `main` using a secret named
  `TOKEN` (a classic PAT with `repo` scope), because the default `GITHUB_TOKEN`
  cannot do this. This must be created and added as a repo secret before that
  workflow will succeed.

## Color theme

Gradient: `#0F172A` (deep navy) → `#1E3A8A` (blue) → `#06B6D4` (cyan).
Card backgrounds: `#0D1117`/`#0F172A`. Accent/title/icon color: `#06B6D4`.
Text: `#C9D1D9`. Thin cyan borders on stat/language/streak cards. Minimal
animation — the typing tagline and the snake graph are the only animated
elements; decorative giphy icons next to section headings (present in the
cloned repo) are dropped in favor of plain headings or thin capsule-render
divider strips.

## Content

- **Name:** Chinmay Rout
- **Role/tagline (typing SVG cycle):** "AI/ML Engineer", "Generative AI", "Agentic AI Builder"
- **Contact badges:** GitHub, Email (chinmayrout2005.jaj@gmail.com), LinkedIn
  (https://www.linkedin.com/in/chinmay-rout-b8797429b/), X
  (https://x.com/chinmayrout2005) — restyled navy/cyan instead of default brand colors.
- **About Me (2-3 lines):** professional summary focused on AI/ML, Generative AI,
  Agentic AI, LLMs, RAG, and building intelligent systems.
- **Currently Working On (short bullets):** Multi-Agent Systems, RAG Pipelines,
  AI Automation, Open Source.
- **Skills icon row (skillicons.dev, theme=dark):** Python, C++, PyTorch,
  TensorFlow, Scikit-learn, Pandas, NumPy, OpenCV, HuggingFace/Transformers,
  LangChain, LangGraph, OpenAI API, Anthropic API, FAISS, ChromaDB, FastAPI,
  Flask, Streamlit, Docker, Git, Linux, SQL, PostgreSQL, React, TypeScript,
  JavaScript.
- **Stats row (github-readme-stats, custom hex theme):** GitHub Stats card,
  Top Languages card, Streak Stats card — side by side.
- **Contribution graphs:**
  - Snake animation (Platane/snk action) recolored to navy/cyan, generated to
    an `output` branch, embedded via `<picture>` light/dark source.
  - 3D contribution graph (yoshi389111/github-profile-3d-contrib action),
    `profile-3d-contrib/settings.json` recolored navy/cyan, committed to `main`.
- **Featured projects** (one block each: name, one-line description, tech badges):
  1. **MemCore** — Production-grade long-term memory infrastructure for AI agents.
     Python, FastAPI, PostgreSQL, Qdrant, Neo4j, Redis, Docker, Kubernetes, Celery, Linux.
  2. **SwarmAI** — Multi-agent personal assistant.
     Python, OpenAI GPT-4o API, Telegram Bot API, Google Calendar/Gmail APIs, Docker, pytest.
  3. **AI Media Intelligence Agent** — Conversational analytics, sentiment analysis,
     predictive forecasting, live social monitoring, RAG, interactive dashboards.
     Python, Streamlit, LangChain, OpenAI, ChromaDB, Pandas, Plotly.
  4. **Document-Aware RAG Chatbot** — Dockerized RAG chatbot answering strictly
     from provided documents. Python, LangChain, FAISS, Ollama, Streamlit, Docker.
- **Trophies:** github-profile-trophy, `theme=onedark`.
- **Footer:** capsule-render waving footer, same gradient as header.

## File layout

```
ChinmayRout9040895625/
  README.md
  .github/workflows/snake.yml
  .github/workflows/profile-3d.yml
  profile-3d-contrib/settings.json
```

The cloned repo's pre-generated SVGs under `profile-3d-contrib/` are not carried
over — they reflect the original author's contribution graph and would be
generated fresh by the workflow on first run. `LICENSE` is not carried over
either — out of scope for a profile README.

## Setup steps (user-performed, documented in a SETUP note)

1. Create a public GitHub repo named exactly `ChinmayRout9040895625`.
2. Push this local repo's `main` branch to it.
3. Repo Settings → Actions → General → Workflow permissions → "Read and write permissions".
4. Create a classic PAT with `repo` scope, add it as a repo secret named `TOKEN`.
5. Manually run both workflows once (Actions tab → Run workflow) so the snake
   animation and 3D graph exist before the first scheduled run.

## Out of scope

- No custom build tooling — pure markdown + external services + 2 GitHub Actions.
- No literal CSS glassmorphism/glow (not renderable in GitHub markdown).
- No portfolio-site badge (not requested).
