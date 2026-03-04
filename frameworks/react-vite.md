# Deploy React + Vite

> Deploy a React app built with Vite to GitHub Pages and Vercel.

This guide covers deploying a Vite-powered React app to two platforms: **GitHub Pages** (free static hosting) and **Vercel** (zero-config with preview deploys). Choose whichever fits your workflow.

## Prerequisites

- [ ] [Node.js 18+](https://nodejs.org/) installed
- [ ] [Git](https://git-scm.com/downloads) installed
- [ ] A [GitHub account](https://github.com/signup)
- [ ] (For Vercel) A [Vercel account](https://vercel.com/signup)

---

## Create the App

```bash
npm create vite@latest my-react-app -- --template react
cd my-react-app
npm install
```

Verify it runs locally:

```bash
npm run dev
```

Open `http://localhost:5173` to confirm it works.

---

## Option A: Deploy to GitHub Pages

### Step 1: Set the Base Path

Edit `vite.config.js`:

```js
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react()],
  base: '/my-react-app/',  // Must match your GitHub repo name
})
```

### Step 2: Create a GitHub Repository

```bash
git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/<username>/my-react-app.git
git push -u origin main
```

### Step 3: Add the Deploy Workflow

Create `.github/workflows/deploy.yml`:

```yaml
name: Deploy to GitHub Pages

on:
  push:
    branches: [main]

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: false

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: npm

      - run: npm ci
      - run: npm run build

      - uses: actions/upload-pages-artifact@v3
        with:
          path: dist

  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - id: deployment
        uses: actions/deploy-pages@v4
```

### Step 4: Enable GitHub Pages

1. Go to **Settings** > **Pages** in your repo
2. Under **Source**, select **GitHub Actions**
3. Push your changes:

```bash
git add .
git commit -m "Add deploy workflow"
git push origin main
```

Your app is live at `https://<username>.github.io/my-react-app/`.

### Handle Client-Side Routing on GitHub Pages

If you use React Router, create `public/404.html` to handle routing:

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <script>
      var pathSegmentsToKeep = 1;
      var l = window.location;
      l.replace(
        l.protocol + '//' + l.hostname + (l.port ? ':' + l.port : '') +
        l.pathname.split('/').slice(0, 1 + pathSegmentsToKeep).join('/') + '/?/' +
        l.pathname.slice(1).split('/').slice(pathSegmentsToKeep).join('/').replace(/&/g, '~and~') +
        (l.search ? '&' + l.search.slice(1).replace(/&/g, '~and~') : '') +
        l.hash
      );
    </script>
  </head>
  <body></body>
</html>
```

And configure React Router with the base path:

```jsx
import { BrowserRouter } from 'react-router-dom';

function App() {
  return (
    <BrowserRouter basename="/my-react-app">
      {/* your routes */}
    </BrowserRouter>
  );
}
```

---

## Option B: Deploy to Vercel

### Step 1: Push to GitHub

```bash
git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/<username>/my-react-app.git
git push -u origin main
```

> **No `base` path needed.** Vercel serves your app at the root domain, so you can leave `vite.config.js` without a `base` setting.

### Step 2: Import in Vercel

1. Go to [vercel.com/new](https://vercel.com/new)
2. Click **Import Git Repository**
3. Select your `my-react-app` repository
4. Vercel auto-detects Vite -- no config needed
5. Click **Deploy**

Your app is live in under a minute at `https://my-react-app.vercel.app`.

### Alternative: Deploy via CLI

```bash
npm i -g vercel
cd my-react-app
vercel          # First deploy: links project
vercel --prod   # Deploy to production
```

Vercel automatically gives you:
- Preview deployments on every PR
- Automatic production deploys on push to `main`
- Instant rollbacks from the dashboard

---

## Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `VITE_API_URL` | Backend API base URL | `https://api.example.com` |
| `VITE_APP_TITLE` | Application title | `My React App` |

> **Important:** Only variables prefixed with `VITE_` are exposed to the browser. Never store secrets in `VITE_` variables.

### For GitHub Pages

Set env vars in the GitHub Actions workflow:

```yaml
- run: npm run build
  env:
    VITE_API_URL: ${{ secrets.API_URL }}
```

Add secrets at **Settings** > **Secrets and variables** > **Actions**.

### For Vercel

Set env vars in the Vercel dashboard:

1. Go to your project **Settings** > **Environment Variables**
2. Add `VITE_API_URL` with the value
3. Choose environments: Production, Preview, Development

Access in your code:

```js
const API_URL = import.meta.env.VITE_API_URL;
```

---

## Custom Domain

### GitHub Pages

1. Create `public/CNAME` containing your domain: `www.yourdomain.com`
2. Configure DNS (see [GitHub Pages guide](../guides/github-pages.md#custom-domain))
3. Enable in **Settings** > **Pages** > **Custom domain**

### Vercel

1. Go to **Settings** > **Domains** in your Vercel project
2. Add your domain
3. Configure DNS as shown by Vercel (typically an A record to `76.76.21.21`)
4. SSL is automatic

---

## Troubleshooting

### Problem: Blank page on GitHub Pages

**Cause:** `base` not set in `vite.config.js`.

**Fix:**

```js
export default defineConfig({
  plugins: [react()],
  base: '/your-repo-name/',  // Trailing slash required
})
```

### Problem: Assets return 404 on GitHub Pages

**Cause:** Asset paths don't include the base prefix.

**Fix:** Always use relative imports for assets in your code. Vite handles the base path automatically for imports. For paths in HTML or CSS, use the `base` prefix.

### Problem: React Router routes return 404 on page refresh (GitHub Pages)

**Cause:** GitHub Pages doesn't support SPA routing.

**Fix:** Add the `404.html` redirect and set `basename` on `BrowserRouter` (see the routing section above).

### Problem: Environment variables are undefined

**Cause:** Variables are not prefixed with `VITE_`, or the app was not rebuilt after adding them.

**Fix:**

1. Ensure variables start with `VITE_`
2. Access them with `import.meta.env.VITE_YOUR_VAR` (not `process.env`)
3. Restart the dev server or trigger a new build

### Problem: Build fails on Vercel with "out of memory"

**Cause:** Large dependencies or too many assets.

**Fix:**

```bash
# Increase Node.js memory limit in Vercel build settings
# Override Build Command:
NODE_OPTIONS=--max-old-space-size=4096 npm run build
```
