# MiniQR

Minimalistic QR generator UI built with **Nuxt** + **Tailwind CSS** + **TypeScript**.

> Add your screenshot below:
>
> `![MiniQR screenshot](./docs/screenshot.png)`

## Features

- Big QR code centered on the page
- Actions: Share / Print / Download (PNG)
- State in URL via `?s=...`

## Requirements

- Bun

## Local development

```bash
bun install
bun run dev
```

## Static build

```bash
bun run generate
```

Output is generated into `.output/public`.

## Deploy to GitHub Pages

This repo includes a GitHub Actions workflow that builds and deploys the static site to **GitHub Pages**:

- `.github/workflows/deploy-pages.yml`

Steps:

1. Push to the `main` branch.
2. In GitHub: **Settings → Pages → Build and deployment → Source: GitHub Actions**.
3. Wait for the `Deploy MiniQR UI to GitHub Pages` workflow to finish.

The workflow automatically sets `NUXT_APP_BASE_URL`:
- `/<repo>/` for normal repositories (`username.github.io/<repo>/`)
- `/` for `username.github.io` repositories

