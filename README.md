# SurgiData prototype

Open `index.html` in a modern browser to try the interactive, static UI concept. The demo has no backend, does not upload or retain video, and uses synthetic case data. AI confidence values are illustrative only.

## Included

- `index.html` — responsive case dashboard, annotation timeline, clinical-review screen, JSON/CSV demo exports, and in-page ML roadmap.
- `ML-ROADMAP.md` — feasibility assessment, suggested modeling path, evaluation criteria, and 16-week plan based on the supplied papers.

## Deploy to Vercel

The Vercel CLI is available in the development environment. From this folder, run:

```powershell
vercel --prod
```

This is a static HTML site; no build step or application secrets are required. `.vercelignore` excludes the supplied papers, reference image, and roadmap from the deployment upload. For a public production deployment, confirm that your Vercel account/team and project name are the intended destination.

## Deploy to GitHub Pages

Create or choose a repository, add these project files, and enable Pages for the branch/root containing `index.html`. The PDF and image references are excluded from Git by `.gitignore`. Do not check in clinical videos, patient information, credentials, or consent documents.

## Before production use

This mockup is not a clinical product. A real annotation service needs an authenticated backend, secure video storage, audit trails, access controls, backup and retention policy, and reviewed consent/privacy procedures. See `ML-ROADMAP.md` before starting model work.
