# GitHub Profile README Refresh Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Refresh `README.md` from the current resume and replace broken live stats with Action-generated static SVGs (stats, top-langs, snake, 3D contrib).

**Architecture:** One GitHub Actions workflow regenerates all profile assets daily and on manual dispatch, then commits them into the profile repo. The README embeds only local paths — no `github-readme-stats.vercel.app`.

**Tech Stack:** GitHub Actions (`stats-organization/github-readme-stats-action@v2`, `Platane/snk/svg-only@v3`, `yoshi389111/github-profile-3d-contrib@v0.9.3`), Markdown/HTML profile README.

**Spec:** `docs/superpowers/specs/2026-07-13-github-profile-readme-design.md`

---

## File map

| File | Responsibility |
|------|----------------|
| `.github/workflows/update-readme-stats.yml` | Daily/manual generation of stats, langs, snake, 3D SVGs + commit |
| `README.md` | Profile content + embeds for local assets |
| `profile/` | Output dir for stats/langs/snake SVGs (created by Action) |
| `profile-3d-contrib/` | Output dir for 3D contrib SVGs (created by Action) |

No application source, tests, or package manager in this repo.

---

### Task 1: Add combined stats workflow

**Files:**
- Create: `.github/workflows/update-readme-stats.yml`

- [ ] **Step 1: Create the workflow directory**

```bash
mkdir -p ".github/workflows"
```

- [ ] **Step 2: Write the workflow file**

Create `.github/workflows/update-readme-stats.yml` with exactly:

```yaml
name: Update README stats

on:
  schedule:
    - cron: "0 0 * * *"
  workflow_dispatch:

permissions:
  contents: write

jobs:
  generate:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v5

      - name: Generate stats card
        uses: stats-organization/github-readme-stats-action@v2.0.1
        with:
          card: stats
          options: username=${{ github.repository_owner }}&show_icons=true&theme=tokyonight
          path: profile/stats.svg
          token: ${{ secrets.GITHUB_TOKEN }}

      - name: Generate top languages card
        uses: stats-organization/github-readme-stats-action@v2.0.1
        with:
          card: top-langs
          options: username=${{ github.repository_owner }}&theme=tokyonight&layout=compact
          path: profile/top-langs.svg
          token: ${{ secrets.GITHUB_TOKEN }}

      - name: Generate contribution snake
        uses: Platane/snk/svg-only@v3.5.0
        with:
          github_user_name: ${{ github.repository_owner }}
          outputs: |
            profile/github-snake.svg
            profile/github-snake-dark.svg?palette=github-dark

      - name: Generate 3D contribution graphs
        uses: yoshi389111/github-profile-3d-contrib@v0.9.3
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          USERNAME: ${{ github.repository_owner }}

      - name: Commit and push generated assets
        run: |
          git config user.name "github-actions[bot]"
          git config user.email "41898282+github-actions[bot]@users.noreply.github.com"
          git add profile/*.svg profile-3d-contrib/*.svg || true
          if git diff --staged --quiet; then
            echo "No asset changes to commit"
            exit 0
          fi
          git commit -m "chore: update README stats and graphs"
          git push
```

- [ ] **Step 3: Verify workflow file exists and has no Vercel host**

```bash
test -f .github/workflows/update-readme-stats.yml
rg -n "github-readme-stats\.vercel\.app|Platane/snk|github-profile-3d-contrib|github-readme-stats-action" .github/workflows/update-readme-stats.yml
```

Expected: file exists; matches include the three Actions; no `vercel.app` hit.

- [ ] **Step 4: Commit**

```bash
git add .github/workflows/update-readme-stats.yml
git commit -m "ci: add workflow to generate profile stats and graphs"
```

---

### Task 2: Rewrite profile README from resume

**Files:**
- Modify: `README.md`

- [ ] **Step 1: Replace `README.md` contents**

Overwrite `README.md` with:

```markdown
## :octocat: Hi, I'm Bharat Malik

**Senior Software Engineer · Full-Stack · 3+ years**

Full-stack engineer building and leading scalable web apps with React.js, Next.js, Node.js, TypeScript, and Nest.js. I own work end-to-end — architecture, delivery, code review, and production. Currently exploring agentic AI systems, RAG pipelines, and LangGraph workflows.

- 🌱 Exploring agentic AI, RAG, and LangGraph for real product workflows
- 👯 Open to collaborating on interesting web and AI projects
- 💬 Ask me about full-stack engineering, Shopify apps, or AI tooling

### Tech Stack

**Languages:** JavaScript · TypeScript · C++ · Python  
**Frontend:** React.js · Next.js · Redux · HTML5 · CSS3  
**Backend:** Node.js · Nest.js · PostgreSQL · REST · Microservices  
**AI / ML:** RAG · LangGraph · LLM integration · Prompt engineering  
**Other:** Shopify · CI/CD · System design · Mentoring

<details>
<summary><b>GitHub Stats</b></summary>
<br/>
<p align="center">
  <img src="./profile/stats.svg" alt="Bharat's GitHub stats" />
  <img src="./profile/top-langs.svg" alt="Top languages" />
</p>
<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="./profile/github-snake-dark.svg" />
    <source media="(prefers-color-scheme: light)" srcset="./profile/github-snake.svg" />
    <img alt="github contribution snake" src="./profile/github-snake.svg" />
  </picture>
</p>
<p align="center">
  <img src="./profile-3d-contrib/profile-green-animate.svg" alt="GitHub 3D contribution graph" />
</p>
</details>

[![LinkedIn](https://img.shields.io/badge/-BharatMalik-blue?style=flat-square&logo=Linkedin&logoColor=white)](https://www.linkedin.com/in/bharat1999/)
[![Website](https://img.shields.io/badge/-bharatmalik.cv-0A0A0A?style=flat-square&logo=safari&logoColor=white)](https://bharatmalik.cv)
[![Gmail](https://img.shields.io/badge/-Email-c14438?style=flat-square&logo=Gmail&logoColor=white)](mailto:bharatmalik1999@gmail.com)

<p align="left">
  <img src="https://komarev.com/ghpvc/?username=bharat1999&label=Profile%20Views&color=blue&style=plastic" alt="profile views" />
</p>
```

- [ ] **Step 2: Verify old content and broken host are gone**

```bash
rg -n "github-readme-stats\.vercel\.app|Mongo|Firebase|University School|ruby on rails" README.md || true
rg -n "profile/stats\.svg|github-snake|profile-green-animate|bharatmalik\.cv|Senior Software Engineer" README.md
```

Expected: first command finds no matches (or only exits 1 with no lines). Second command finds the new local embeds and resume-aligned copy.

- [ ] **Step 3: Commit**

```bash
git add README.md
git commit -m "docs: refresh profile README from current resume"
```

---

### Task 3: Verify readiness and hand off first Action run

**Files:**
- Verify: `.github/workflows/update-readme-stats.yml`, `README.md`

- [ ] **Step 1: Confirm no paused Vercel endpoint remains in tracked files**

```bash
rg -n "github-readme-stats\.vercel\.app" --glob '!docs/**' . || true
```

Expected: no matches outside historical docs (ideally none at all in non-doc files). Spec/plan docs may still mention the broken host as context — that is fine.

- [ ] **Step 2: Confirm workflow + README paths align**

```bash
rg -n "profile/stats\.svg|profile/top-langs\.svg|profile/github-snake|profile-green-animate" README.md .github/workflows/update-readme-stats.yml
```

Expected: both files reference the same asset paths from the spec.

- [ ] **Step 3: Push branch (only if user asked to push) and tell user to run the workflow**

Do **not** push unless the user explicitly asked. After files are committed locally, instruct the user:

1. Push to `origin` if needed.
2. Open **Actions → Update README stats → Run workflow**.
3. Confirm `profile/*.svg` and `profile-3d-contrib/profile-green-animate.svg` appear in the repo and render on the profile.

Until that first run succeeds, README image embeds may show broken until assets exist — this is expected per the spec bootstrap note.

- [ ] **Step 4: Final commit only if stray fixes were needed**

If Step 1–2 required small path/typo fixes, commit them:

```bash
git add README.md .github/workflows/update-readme-stats.yml
git commit -m "fix: align README asset paths with stats workflow"
```

Otherwise skip.

---

## Spec coverage (self-review)

| Spec requirement | Task |
|------------------|------|
| Resume-based intro / stack / contact + site | Task 2 |
| Remove vercel.app stats | Tasks 1–3 |
| Stats + top-langs Action SVGs | Task 1 |
| Snake light/dark + `<picture>` | Tasks 1–2 |
| 3D green animate embed | Tasks 1–2 |
| Valid details/summary stats section | Task 2 |
| Profile views badge kept | Task 2 |
| Combined daily + manual workflow | Task 1 |
| Bootstrap via first Action run | Task 3 |

## Out of scope (do not implement)

- Committing `Bharat_Malik_Resume_Updated.pdf` or `.DS_Store`
- Private-repo PAT setup
- Extra animations (Maeul, CRT, typing SVG, trophies)
