# Netlify

> Deploy static sites, SPAs, and serverless functions with Git-based continuous deployment on Netlify.

Netlify is a platform for building and deploying modern web projects. It excels at static sites, single-page applications (SPAs), and JAMstack architectures. Netlify provides serverless functions, edge functions, form handling, split testing, and deploy previews for every pull request -- all with zero server management.

## Prerequisites

- [ ] A [Netlify account](https://app.netlify.com/signup) (sign up with GitHub recommended)
- [ ] [Git](https://git-scm.com/downloads) installed locally
- [ ] [Node.js 18+](https://nodejs.org/)
- [ ] (Optional) [Netlify CLI](https://docs.netlify.com/cli/get-started/): `npm i -g netlify-cli`

---

## Deploy a Static Site

### Step 1: Prepare Your Project

Any folder with HTML, CSS, and JavaScript files can be deployed. For a simple example:

```
my-static-site/
  index.html
  style.css
  script.js
```

Create `index.html`:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>My Netlify Site</title>
  <link rel="stylesheet" href="style.css">
</head>
<body>
  <h1>Hello from Netlify!</h1>
  <script src="script.js"></script>
</body>
</html>
```

### Step 2: Push to GitHub

```bash
git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/<username>/my-static-site.git
git push -u origin main
```

### Step 3: Import in Netlify

1. Go to [app.netlify.com](https://app.netlify.com/)
2. Click **Add new site** > **Import an existing project**
3. Select **GitHub** and authorize Netlify
4. Choose your `my-static-site` repository
5. Configure build settings:
   - **Branch to deploy:** `main`
   - **Build command:** (leave empty for plain HTML)
   - **Publish directory:** `.` (or your output folder)
6. Click **Deploy site**

Your site is live in seconds at a URL like `https://random-name-123.netlify.app`.

### Alternative: Deploy via CLI

```bash
# Install Netlify CLI
npm i -g netlify-cli

# Login to Netlify
netlify login

# Initialize and link to a Netlify site
netlify init

# Deploy a preview (draft URL)
netlify deploy

# Deploy to production
netlify deploy --prod

# Local development server with Netlify features
netlify dev
```

---

## Deploy a React / Vite SPA

### Step 1: Create a Vite React App

```bash
npm create vite@latest my-react-app -- --template react
cd my-react-app
npm install
```

### Step 2: Push to GitHub and Import

Push to GitHub (same steps as above), then import in Netlify. Netlify auto-detects Vite and configures:

- **Build command:** `npm run build`
- **Publish directory:** `dist`

Or deploy via CLI:

```bash
cd my-react-app
netlify init
# Select: Create & configure a new site
# Build command: npm run build
# Publish directory: dist
netlify deploy --prod
```

### SPA Redirect Configuration

SPAs use client-side routing, so all routes must serve `index.html`. Without this, direct URL access (e.g., `/about`) returns a 404.

**Option 1: `_redirects` file**

Create `public/_redirects` (so it is copied to the build output):

```
/*    /index.html   200
```

**Option 2: `netlify.toml`**

```toml
[[redirects]]
  from = "/*"
  to = "/index.html"
  status = 200
```

> **Important:** This is the single most common issue with SPA deployments on Netlify. Always add the redirect rule.

---

## netlify.toml Configuration

Create `netlify.toml` in your project root for full build and deploy configuration:

```toml
[build]
  command = "npm run build"
  publish = "dist"
  functions = "netlify/functions"

[build.environment]
  NODE_VERSION = "20"

# Production context
[context.production]
  command = "npm run build"

# Deploy preview context (pull requests)
[context.deploy-preview]
  command = "npm run build"

# Branch-specific context
[context.staging]
  command = "npm run build:staging"

# Dev server settings
[dev]
  command = "npm run dev"
  port = 3000
  targetPort = 5173

# Headers
[[headers]]
  for = "/*"
  [headers.values]
    X-Frame-Options = "DENY"
    X-XSS-Protection = "1; mode=block"
    X-Content-Type-Options = "nosniff"
    Referrer-Policy = "strict-origin-when-cross-origin"

# Redirects
[[redirects]]
  from = "/api/*"
  to = "/.netlify/functions/:splat"
  status = 200

[[redirects]]
  from = "/*"
  to = "/index.html"
  status = 200
```

---

## Serverless Functions

Netlify Functions run AWS Lambda functions without managing infrastructure. They are perfect for API endpoints, form handlers, and server-side logic.

### Step 1: Create a Function

Create `netlify/functions/hello.js`:

```js
export default async (req, context) => {
  const name = new URL(req.url).searchParams.get('name') || 'World';

  return new Response(
    JSON.stringify({ message: `Hello, ${name}!` }),
    {
      headers: { 'Content-Type': 'application/json' },
    }
  );
};
```

This function is automatically deployed at `https://your-site.netlify.app/.netlify/functions/hello?name=Sagar`.

### Step 2: API-Style Routing

Use a redirect to create clean API routes:

In `netlify.toml`:

```toml
[[redirects]]
  from = "/api/*"
  to = "/.netlify/functions/:splat"
  status = 200
```

Now `https://your-site.netlify.app/api/hello?name=Sagar` routes to the `hello` function.

### Example: Database Query Function

Create `netlify/functions/users.js`:

```js
import pg from 'pg';

const pool = new pg.Pool({
  connectionString: process.env.DATABASE_URL,
});

export default async (req, context) => {
  try {
    const result = await pool.query('SELECT id, name FROM users LIMIT 10');
    return new Response(JSON.stringify(result.rows), {
      headers: { 'Content-Type': 'application/json' },
    });
  } catch (error) {
    return new Response(JSON.stringify({ error: error.message }), {
      status: 500,
      headers: { 'Content-Type': 'application/json' },
    });
  }
};
```

### Example: Scheduled Function (Cron)

Create `netlify/functions/daily-cleanup.js`:

```js
import { schedule } from '@netlify/functions';

const handler = async (event) => {
  console.log('Running daily cleanup...');
  // Your cleanup logic here
  return { statusCode: 200 };
};

// Run every day at midnight UTC
export default schedule('0 0 * * *', handler);
```

---

## Edge Functions

Edge Functions run on Netlify's edge network (Deno-based), close to users for ultra-low latency. Use them for personalization, A/B testing, geolocation, and request/response transformation.

### Create an Edge Function

Create `netlify/edge-functions/geolocation.js`:

```js
export default async (request, context) => {
  const country = context.geo.country?.code || 'unknown';
  const city = context.geo.city || 'unknown';

  return new Response(
    JSON.stringify({
      country,
      city,
      message: `Hello from ${city}, ${country}!`,
    }),
    {
      headers: { 'Content-Type': 'application/json' },
    }
  );
};

export const config = {
  path: '/api/location',
};
```

### Edge Function for Response Transformation

Create `netlify/edge-functions/transform.js`:

```js
export default async (request, context) => {
  const response = await context.next();
  const html = await response.text();

  // Inject a banner into every HTML response
  const modified = html.replace(
    '</body>',
    '<div id="banner" style="position:fixed;bottom:0;width:100%;background:#333;color:#fff;text-align:center;padding:8px;">Preview Environment</div></body>'
  );

  return new Response(modified, response);
};

export const config = {
  path: '/*',
  // Only run on deploy previews
  excludedPath: ['/api/*', '/_next/*'],
};
```

---

## Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `DATABASE_URL` | Database connection string | `postgresql://user:pass@host/db` |
| `API_SECRET_KEY` | Secret API key (server only) | `sk_live_abc123` |
| `VITE_API_URL` | Public API URL (Vite apps) | `https://api.example.com` |
| `NEXT_PUBLIC_API_URL` | Public API URL (Next.js) | `https://api.example.com` |
| `NODE_VERSION` | Node.js version for builds | `20` |
| `NPM_FLAGS` | Custom npm flags | `--legacy-peer-deps` |
| `SITE_URL` | Site URL (auto-set by Netlify) | `https://your-site.netlify.app` |

### Set via Dashboard

1. Go to your site on [app.netlify.com](https://app.netlify.com/)
2. Navigate to **Site configuration** > **Environment variables**
3. Click **Add a variable**
4. Enter the key and value
5. Choose scopes: **All**, **Builds**, **Functions**, **Post processing**
6. Choose contexts: **All deploy contexts**, **Production**, **Deploy previews**, **Branch deploys**

### Set via CLI

```bash
# Set a variable
netlify env:set API_SECRET_KEY "my-secret-value"

# Set a variable for a specific context
netlify env:set API_URL "https://staging-api.example.com" --context deploy-preview

# List all variables
netlify env:list

# Get a specific variable
netlify env:get API_SECRET_KEY

# Delete a variable
netlify env:unset API_SECRET_KEY

# Import from a .env file
netlify env:import .env
```

### Important Notes

- Variables prefixed with `VITE_` (Vite) or `NEXT_PUBLIC_` (Next.js) are embedded into the build output and **exposed to the browser**. Never put secrets in these.
- Serverless functions have access to all environment variables.
- After changing env vars, trigger a new deploy for changes to take effect in the build.

---

## Netlify Forms

Netlify Forms handles form submissions without any server-side code. Add a `netlify` attribute to any HTML form and Netlify captures submissions automatically.

### Basic Form

```html
<form name="contact" method="POST" data-netlify="true">
  <input type="hidden" name="form-name" value="contact" />
  <p>
    <label>Name: <input type="text" name="name" required /></label>
  </p>
  <p>
    <label>Email: <input type="email" name="email" required /></label>
  </p>
  <p>
    <label>Message: <textarea name="message" required></textarea></label>
  </p>
  <p>
    <button type="submit">Send</button>
  </p>
</form>
```

### Form with Spam Protection (Honeypot)

```html
<form name="contact" method="POST" data-netlify="true" netlify-honeypot="bot-field">
  <input type="hidden" name="form-name" value="contact" />
  <p class="hidden">
    <label>Don't fill this out: <input name="bot-field" /></label>
  </p>
  <p>
    <label>Name: <input type="text" name="name" required /></label>
  </p>
  <p>
    <button type="submit">Send</button>
  </p>
</form>
```

### Form in React SPA

For SPAs, the form must also exist in a static HTML file so Netlify can detect it at build time.

Add to `public/index.html`:

```html
<!-- Hidden form for Netlify detection -->
<form name="contact" data-netlify="true" hidden>
  <input type="text" name="name" />
  <input type="email" name="email" />
  <textarea name="message"></textarea>
</form>
```

Then submit via JavaScript in your React component:

```jsx
function ContactForm() {
  const handleSubmit = async (e) => {
    e.preventDefault();
    const formData = new FormData(e.target);

    await fetch('/', {
      method: 'POST',
      headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
      body: new URLSearchParams(formData).toString(),
    });

    alert('Form submitted!');
  };

  return (
    <form name="contact" method="POST" onSubmit={handleSubmit}>
      <input type="hidden" name="form-name" value="contact" />
      <input type="text" name="name" required placeholder="Name" />
      <input type="email" name="email" required placeholder="Email" />
      <textarea name="message" required placeholder="Message"></textarea>
      <button type="submit">Send</button>
    </form>
  );
}
```

### View Submissions

1. Go to your site on Netlify
2. Navigate to **Forms** in the sidebar
3. View, export, or delete submissions
4. Set up email notifications under **Site configuration** > **Forms** > **Form notifications**

---

## `_redirects` and `_headers` Files

### `_redirects`

Place a `_redirects` file in your publish directory for URL redirects and rewrites:

```
# Redirect old paths
/old-page    /new-page    301

# Proxy API calls to an external backend
/api/*       https://api.example.com/:splat    200

# SPA fallback
/*           /index.html                       200

# Country-based redirect
/store       /store/us    302    Country=us
/store       /store/eu    302    Country=de,fr,it,es

# Redirect with query parameters
/search      /search-results    301    Query=q=:q
```

### `_headers`

Place a `_headers` file in your publish directory for custom HTTP headers:

```
# Headers for all paths
/*
  X-Frame-Options: DENY
  X-XSS-Protection: 1; mode=block
  X-Content-Type-Options: nosniff
  Referrer-Policy: strict-origin-when-cross-origin

# Cache static assets
/assets/*
  Cache-Control: public, max-age=31536000, immutable

# CORS headers for API functions
/.netlify/functions/*
  Access-Control-Allow-Origin: *
  Access-Control-Allow-Methods: GET, POST, OPTIONS
  Access-Control-Allow-Headers: Content-Type
```

---

## Custom Domain

### Step 1: Add Domain in Netlify

1. Go to your site on Netlify
2. Navigate to **Domain management** > **Add a domain**
3. Enter your domain (e.g., `yourdomain.com`)
4. Click **Verify** and **Add domain**

### Step 2: Configure DNS

Netlify shows the required DNS records. You have two options:

**Option A: Use Netlify DNS (recommended)**

1. In **Domain management**, click **Set up Netlify DNS**
2. Netlify provides nameservers (e.g., `dns1.p01.nsone.net`)
3. Update your domain registrar to use Netlify's nameservers:

| Nameserver |
|------------|
| `dns1.p01.nsone.net` |
| `dns2.p01.nsone.net` |
| `dns3.p01.nsone.net` |
| `dns4.p01.nsone.net` |

**Option B: External DNS**

Add these records at your DNS provider:

**For apex domain (`yourdomain.com`):**

| Type | Name | Value |
|------|------|-------|
| A | @ | `75.2.60.5` |

**For subdomain (`www.yourdomain.com`):**

| Type | Name | Value |
|------|------|-------|
| CNAME | www | `your-site.netlify.app` |

### Step 3: SSL Certificate

Netlify automatically provisions a free SSL certificate via Let's Encrypt once DNS propagates. You can check the status under **Domain management** > **HTTPS**.

To force HTTPS:

1. Navigate to **Domain management** > **HTTPS**
2. Ensure **Force HTTPS** is enabled (it is by default)

---

## Free Tier

Netlify's free tier is generous for personal and small projects:

| Feature | Free (Starter) | Pro ($19/mo per member) |
|---------|-----------------|-------------------------|
| **Bandwidth** | 100 GB/month | 1 TB/month |
| **Build minutes** | 300 min/month | 25,000 min/month |
| **Serverless functions** | 125,000 requests/month | 2M requests/month |
| **Function runtime** | 10 seconds max | 26 seconds max |
| **Edge functions** | 3M invocations/month | 3M invocations/month |
| **Forms** | 100 submissions/month | 1,000 submissions/month |
| **Identity** | 1,000 active users | 1,000 active users |
| **Analytics** | Not included | Included |
| **Deploy previews** | Unlimited | Unlimited |
| **Concurrent builds** | 1 | 3 |
| **Team members** | 1 | Unlimited |

### Key things to know:

- **The free tier is always-on.** Static sites do not spin down or sleep.
- **Deploy previews are unlimited.** Every pull request gets a unique preview URL at no cost.
- **Build minutes are shared** across all sites in your account. 300 minutes goes quickly with frequent deploys.
- **Serverless functions have a 10-second timeout** on the free tier. Optimize long-running operations.
- **Forms are limited to 100 submissions/month.** Use a third-party service (e.g., Formspree) if you need more.
- **Bandwidth overages** are billed at $55 per 100 GB on the free plan. Monitor usage under **Site configuration** > **Usage**.

---

## Troubleshooting

### Problem: Build fails with "command not found" or dependency errors

**Cause:** The build environment does not have the correct Node.js version or dependencies are missing.

**Fix:**

1. Set the Node.js version in `netlify.toml`:

```toml
[build.environment]
  NODE_VERSION = "20"
```

Or create a `.node-version` file:

```
20
```

2. Clear the build cache and redeploy:
   - Go to **Deploys** > **Trigger deploy** > **Clear cache and deploy site**

3. Make sure all dependencies are in `package.json`:

```bash
npm install
git add package.json package-lock.json
git commit -m "Fix dependencies"
git push
```

4. If using npm peer dependencies, set:

```toml
[build.environment]
  NPM_FLAGS = "--legacy-peer-deps"
```

### Problem: SPA routes return 404 on direct access

**Cause:** Netlify serves static files, so `/about` looks for an `about/index.html` file that does not exist. The SPA redirect rule is missing.

**Fix:**

Create `public/_redirects` (for Vite/React) or add to `netlify.toml`:

```
/*    /index.html   200
```

Or in `netlify.toml`:

```toml
[[redirects]]
  from = "/*"
  to = "/index.html"
  status = 200
```

> **Important:** The SPA redirect must be the LAST redirect rule because Netlify processes redirects in order and stops at the first match.

### Problem: Serverless function timeout (10-second limit)

**Cause:** The function exceeds the 10-second execution limit on the free tier.

**Fix:**

1. Optimize the function to complete within the time limit:

```js
// Use connection pooling for database queries
import pg from 'pg';

// Create pool OUTSIDE the handler (reused across invocations)
const pool = new pg.Pool({
  connectionString: process.env.DATABASE_URL,
  max: 3,
  connectionTimeoutMillis: 3000,
});

export default async (req, context) => {
  const result = await pool.query('SELECT id, name FROM users LIMIT 10');
  return new Response(JSON.stringify(result.rows), {
    headers: { 'Content-Type': 'application/json' },
  });
};
```

2. For long-running operations, use Netlify Background Functions (Pro plan):

```js
// netlify/functions/long-task-background.js
// The "-background" suffix makes it a background function (up to 15 minutes)
export default async (req, context) => {
  // Long-running task here
  await processLargeDataset();
  return { statusCode: 200 };
};
```

3. Move heavy computation to an external backend (e.g., Railway, Fly.io) and call it from your function

### Problem: Netlify Forms submissions not appearing

**Cause:** The form is not detected during the build, the `data-netlify="true"` attribute is missing, or the hidden `form-name` input is missing.

**Fix:**

1. Ensure the form has `data-netlify="true"`:

```html
<form name="contact" method="POST" data-netlify="true">
```

2. For SPAs (React, Vue), add a hidden HTML form in `public/index.html` so Netlify can detect it at build time:

```html
<form name="contact" data-netlify="true" hidden>
  <input type="text" name="name" />
  <input type="email" name="email" />
  <textarea name="message"></textarea>
</form>
```

3. Include the hidden `form-name` field in your JavaScript form submission:

```js
const formData = new URLSearchParams({
  'form-name': 'contact',
  name: 'John',
  email: 'john@example.com',
  message: 'Hello!',
});

await fetch('/', {
  method: 'POST',
  headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
  body: formData.toString(),
});
```

4. Check form submissions in the Netlify dashboard under **Forms**

### Problem: Deploy previews show wrong content or fail

**Cause:** The deploy preview uses a different branch context, environment variables are not set for previews, or the build command differs per context.

**Fix:**

1. Ensure environment variables are available in the deploy-preview context:
   - Go to **Site configuration** > **Environment variables**
   - Set the variable scope to include **Deploy previews**

2. Use `netlify.toml` to set context-specific build commands:

```toml
[context.deploy-preview]
  command = "npm run build"
  [context.deploy-preview.environment]
    VITE_API_URL = "https://staging-api.example.com"
```

3. Check the deploy preview build log:
   - Go to **Deploys** > find the PR deploy > click to see the build log

4. Ensure the publish directory is correct for all build contexts

### Problem: Redirect rules not working as expected

**Cause:** Redirect rules conflict with each other, the order is wrong, or actual files take precedence.

**Fix:**

1. Remember that **Netlify checks for actual files first**. If a file exists at the path, the redirect is skipped. Use `status = 200!` (with `!`) to force the redirect even when a file exists:

```
/old-page    /new-page    301!
```

2. Redirects are processed **top to bottom, first match wins**. Put more specific rules above general ones:

```
# Specific rules first
/api/*        /.netlify/functions/:splat    200
/blog/:slug   /posts/:slug                 301

# SPA fallback LAST
/*            /index.html                  200
```

3. Use the Netlify [redirect playground](https://play.netlify.com/redirects) to test your rules

4. Check redirect behavior with the CLI:

```bash
netlify dev
# Visit http://localhost:8888 and test your redirects locally
```

### Problem: Large site exceeds bandwidth on the free tier

**Cause:** The 100 GB monthly bandwidth limit is reached, often due to large assets or high traffic.

**Fix:**

1. Check bandwidth usage under **Site configuration** > **Usage**

2. Optimize assets:

```bash
# Compress images
npx sharp-cli --input "src/images/*.{jpg,png}" --output "src/images/optimized" --resize 1200

# Use modern image formats (WebP, AVIF) in your build
```

3. Add cache headers for static assets:

```
# _headers file
/assets/*
  Cache-Control: public, max-age=31536000, immutable

/*.js
  Cache-Control: public, max-age=31536000, immutable

/*.css
  Cache-Control: public, max-age=31536000, immutable
```

4. Use a CDN like Cloudflare in front of Netlify to reduce bandwidth from Netlify's perspective

5. Upgrade to the Pro plan ($19/month) for 1 TB bandwidth if needed
