# Vercel

> Deploy frontend apps and serverless functions with zero configuration on Vercel.

Vercel is the company behind Next.js and provides a platform optimized for frontend frameworks. It supports automatic deployments from Git, serverless functions, edge functions, and preview deployments for every pull request.

## Prerequisites

- [ ] A [Vercel account](https://vercel.com/signup) (sign up with GitHub for easiest setup)
- [ ] [Git](https://git-scm.com/downloads) installed locally
- [ ] [Node.js 18+](https://nodejs.org/)
- [ ] (Optional) [Vercel CLI](https://vercel.com/docs/cli): `npm i -g vercel`

## Deploy Next.js

### Step 1: Create a Next.js App

```bash
npx create-next-app@latest my-nextjs-app
cd my-nextjs-app
```

### Step 2: Push to GitHub

```bash
git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/<username>/my-nextjs-app.git
git push -u origin main
```

### Step 3: Import in Vercel

1. Go to [vercel.com/new](https://vercel.com/new)
2. Select **Import Git Repository**
3. Choose your `my-nextjs-app` repository
4. Vercel auto-detects Next.js -- no configuration needed
5. Click **Deploy**

Your app is live in under a minute. Every push to `main` triggers a new production deploy. Every pull request gets a unique preview URL.

### Alternative: Deploy via CLI

```bash
cd my-nextjs-app
vercel          # First time: links to your Vercel project
vercel --prod   # Deploy to production
```

---

## Deploy React (Vite)

### Step 1: Create a Vite React App

```bash
npm create vite@latest my-react-app -- --template react
cd my-react-app
npm install
```

### Step 2: Push to GitHub and Import

Push to GitHub (same steps as above), then import in Vercel. Vercel auto-detects Vite and configures the build.

Or deploy via CLI:

```bash
vercel
```

Vercel will ask:
- **Framework Preset:** Vite
- **Build Command:** `npm run build`
- **Output Directory:** `dist`

That's it. No `base` path configuration needed (unlike GitHub Pages).

---

## Serverless Functions

Vercel supports serverless functions out of the box. Create files in the `api/` directory at your project root.

### Example: Node.js API Route

Create `api/hello.js`:

```js
export default function handler(req, res) {
  const { name } = req.query;
  res.status(200).json({ message: `Hello, ${name || 'World'}!` });
}
```

This is automatically deployed at `https://your-app.vercel.app/api/hello?name=Sagar`.

### Example: Using Environment Variables in Functions

Create `api/data.js`:

```js
export default async function handler(req, res) {
  const response = await fetch('https://api.example.com/data', {
    headers: {
      Authorization: `Bearer ${process.env.API_SECRET_KEY}`,
    },
  });
  const data = await response.json();
  res.status(200).json(data);
}
```

### Next.js API Routes

In Next.js (App Router), create `app/api/hello/route.js`:

```js
import { NextResponse } from 'next/server';

export async function GET(request) {
  return NextResponse.json({ message: 'Hello from Next.js!' });
}
```

These are deployed automatically as serverless functions on Vercel.

---

## Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `DATABASE_URL` | Database connection string | `postgresql://user:pass@host/db` |
| `API_SECRET_KEY` | Secret API key (server only) | `sk_live_abc123` |
| `NEXT_PUBLIC_API_URL` | Public API URL (Next.js) | `https://api.example.com` |
| `VITE_API_URL` | Public API URL (Vite) | `https://api.example.com` |

### Set via Dashboard

1. Go to your project on [vercel.com](https://vercel.com)
2. Navigate to **Settings** > **Environment Variables**
3. Add your variables
4. Choose which environments they apply to: **Production**, **Preview**, **Development**

### Set via CLI

```bash
# Add a secret (prompted for value)
vercel env add DATABASE_URL production

# Pull env vars to local .env file
vercel env pull .env.local

# List all env vars
vercel env ls
```

### Important Notes

- Variables prefixed with `NEXT_PUBLIC_` (Next.js) or `VITE_` (Vite) are exposed to the browser. Never put secrets in these.
- Server-side environment variables (no prefix) are only available in API routes and serverless functions.
- After changing env vars, you must redeploy for changes to take effect.

---

## Custom Domain

### Step 1: Add Domain in Vercel

1. Go to your project on Vercel
2. Navigate to **Settings** > **Domains**
3. Enter your domain (e.g., `yourdomain.com`) and click **Add**

### Step 2: Configure DNS

Vercel will show you the required DNS records. Typically:

**For apex domain (`yourdomain.com`):**

| Type | Name | Value |
|------|------|-------|
| A | @ | 76.76.21.21 |

**For subdomain (`www.yourdomain.com`):**

| Type | Name | Value |
|------|------|-------|
| CNAME | www | cname.vercel-dns.com |

### Step 3: Verify and SSL

Vercel automatically provisions an SSL certificate once DNS propagates. No manual configuration needed.

You can also set up a redirect so `www.yourdomain.com` redirects to `yourdomain.com` (or vice versa) in the Domains settings.

---

## Troubleshooting

### Problem: Build fails with "Module not found"

**Cause:** A dependency is missing or not installed.

**Fix:**

```bash
# Make sure all dependencies are in package.json (not just installed globally)
npm install
git add package.json package-lock.json
git commit -m "Fix missing dependencies"
git push
```

### Problem: Environment variable is undefined

**Cause:** The variable is not set for the correct environment, or the app needs a redeploy.

**Fix:**

1. Check **Settings** > **Environment Variables** -- ensure it is set for **Production**
2. Trigger a redeploy: **Deployments** > click the three dots on the latest deploy > **Redeploy**
3. For client-side variables, make sure they have the `NEXT_PUBLIC_` or `VITE_` prefix

### Problem: API route returns 404

**Cause:** The file is not in the correct directory or has a syntax error.

**Fix:**

1. Ensure the file is in `api/` (root-level) for plain Vercel projects, or `app/api/` for Next.js App Router
2. Check the function exports a default handler (for `api/` directory) or named HTTP method exports (for Next.js)
3. Check Vercel deployment logs for errors: **Deployments** > select deploy > **Functions** tab

### Problem: Preview deployment shows old code

**Cause:** Branch is behind `main` or caching issue.

**Fix:**

```bash
git pull origin main
git push origin your-branch
```

Or trigger a manual redeploy from the Vercel dashboard.

### Problem: Custom domain shows "Invalid Configuration"

**Cause:** DNS records are incorrect or haven't propagated yet.

**Fix:**

1. Double-check DNS records match what Vercel shows in **Settings** > **Domains**
2. Use `dig yourdomain.com +short` to verify DNS resolution
3. Wait up to 48 hours for propagation (usually much faster)
4. Try removing and re-adding the domain in Vercel

### Problem: Serverless function timeout

**Cause:** Function exceeds the execution time limit (10s on Hobby plan, 60s on Pro).

**Fix:**

1. Optimize the function -- reduce external API calls, use caching
2. For long-running tasks, consider using Vercel's background functions or a dedicated backend
3. Upgrade to Pro for longer timeouts if needed
