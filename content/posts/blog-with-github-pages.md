---
title: Building This Blog with Astro and GitHub Pages
description: How this Astro blog is written, checked, and deployed to GitHub Pages with a custom domain.
publishDate: 2024-03-22T14:00:36+08:00
updatedDate: "2026-07-19T00:00:00+08:00"
tags:
  - Blog
---

# Why Astro and GitHub Pages?

This site started as a Hexo blog in 2024 and was later migrated to [Astro](https://astro.build/). Astro generates static HTML, while GitHub Pages hosts the generated site without requiring a database or a continuously running server.

The current source is based on the [Astro Cactus](https://github.com/chrismwilliams/astro-theme-cactus) theme. It also includes RSS, a sitemap, Pagefind search, syntax highlighting, KaTeX math, Mermaid diagrams, and Markdown/MDX support.

## Local development

The project uses the Node.js version recorded in `.nvmrc`.

```bash
npm install
npm run dev
```

The local development server runs at `http://localhost:4321`. Before pushing a change, I run:

```bash
npm run check
npm run build
```

`npm run check` validates the Astro and Biome configuration. `npm run build` generates the static site in `dist/` and builds the Pagefind search index.

## Writing posts

Regular posts live in `content/posts/`, while shorter notes live in `content/notes/`. A normal post is a Markdown file with frontmatter:

```yaml
---
title: Example post
description: A short summary used in post lists and search metadata.
publishDate: 2026-07-19
updatedDate: 2026-07-19
tags:
  - robotics
  - ros
draft: false
---
```

I use Markdown for most posts and reserve MDX for pages that need an Astro component. Images can be committed with the site for straightforward version control or hosted separately when repository size is a concern.

## Deployment to GitHub Pages

Pushes to `main` trigger `.github/workflows/pages.yml`. The workflow checks out the repository, installs the pinned dependencies with `npm ci`, runs the checks, builds the site, uploads `dist/`, and deploys that artifact to GitHub Pages.

In the repository's **Settings → Pages** page, the deployment source must be set to **GitHub Actions**. The workflow needs `pages: write` and `id-token: write` permissions for deployment.

## Custom domain and HTTPS

First add the custom domain in the repository's Pages settings. Configure the DNS records using GitHub's current [custom-domain documentation](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site). Use the complete address set GitHub recommends instead of relying on a single A or AAAA record.

After DNS has propagated and GitHub has provisioned the certificate, enable **Enforce HTTPS**. DNS changes and certificate issuance can take some time, so a temporary validation failure is not necessarily a deployment failure.
