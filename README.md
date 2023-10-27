# snowball-pwa-vitepwa-template

VitePWA pwa template with routes. deployed using github pages to staticrouterpwa.snowballtools.xyz

## Installation

```sh
npm install
```

## Development

```sh
npm start
```

## Deploying with Github Pages?

- using a custom domain? Deploy PWA to Github Pages Action (defined in [main.yml](https://github.com/snowball-tools/snowball-pwa-vitepwa-template/blob/main/.github/workflows/main.yml)) will deploy to the custom domain set in settings
- using a github.io domain? (default domain created by github for repo) update [.github/workflows/main.yml](https://github.com/snowball-tools/snowball-pwa-vitepwa-template/blob/main/.github/workflows/main.yml) to

why? github pages deploys to <org-name/username>.github.io/<repo-name> by default breaking the routes

```yml
name: Deploy PWA to GitHub Pages

on:
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3

      - name: Setup Node
        uses: actions/setup-node@v1
        with:
          node-version: '16.13'

      - name: Install Dependencies
        run: npm install

      - name: Rewrite Manifest Path in Index
        run: sed -i 's@/manifest.json@${{ github.event.repository.name }}/manifest.json@g' ./index.html && cat index.html

      - name: Rewrite Service Worker Path in Index
        run: sed -i 's@/sw.js@${{ github.event.repository.name }}/sw.js@g' ./index.html && cat ./index.html

      - name: Replace Paths in Manifest
        run: sed -i 's@"/"@"/${{ github.event.repository.name }}/"@g' ./public/manifest.json && cat ./public/manifest.json

      - name: Vite Build
        run: npm run build -- --base=${{ github.event.repository.name }}

      - name: Redirect 404 to Index for SPA
        run: cp dist/index.html dist/404.html

      - name: Upload Artifact
        uses: actions/upload-pages-artifact@v1
        with:
          path: dist/

  deploy:
    needs: build
    runs-on: ubuntu-latest
    permissions:
      pages: write
      id-token: write
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - id: deployment
        uses: actions/deploy-pages@v1
```
