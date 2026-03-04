# Railway

> Deploy backend apps and databases instantly with Railway's Git-based workflow and managed infrastructure.

Railway is a modern cloud platform that simplifies deploying web applications and provisioning databases. It supports automatic builds via Nixpacks, GitHub-based deployments, built-in PostgreSQL and Redis add-ons, and a usage-based billing model. Railway handles Dockerfiles, Nixpacks auto-detection, and `railway.toml` configuration for fine-grained control.

## Prerequisites

- [ ] A [Railway account](https://railway.app/) (sign up with GitHub recommended)
- [ ] [Git](https://git-scm.com/downloads) installed locally
- [ ] [Node.js 18+](https://nodejs.org/) (for Node.js/Express projects)
- [ ] [Python 3.9+](https://www.python.org/downloads/) (for FastAPI projects)
- [ ] (Optional) [Railway CLI](https://docs.railway.app/guides/cli): `npm i -g @railway/cli`

---

## Deploy Node.js / Express

### Step 1: Prepare Your Express App

Create `server.js`:

```js
import express from 'express';

const app = express();
const PORT = process.env.PORT || 3000;

app.get('/', (req, res) => {
  res.json({ message: 'Hello from Railway!' });
});

app.get('/health', (req, res) => {
  res.json({ status: 'ok' });
});

app.listen(PORT, '0.0.0.0', () => {
  console.log(`Server running on port ${PORT}`);
});
```

Create `package.json`:

```json
{
  "name": "my-express-app",
  "type": "module",
  "scripts": {
    "start": "node server.js"
  },
  "dependencies": {
    "express": "^4.18.0"
  }
}
```

> **Important:** Always bind to `0.0.0.0` and use `process.env.PORT`. Railway assigns the port dynamically.

### Step 2: Push to GitHub

```bash
git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/<username>/my-express-app.git
git push -u origin main
```

### Step 3: Deploy via Dashboard

1. Go to [railway.app/new](https://railway.app/new)
2. Click **Deploy from GitHub repo**
3. Select your `my-express-app` repository
4. Railway auto-detects Node.js via Nixpacks -- no configuration needed
5. Click **Deploy Now**

Railway builds and deploys your app. Click **Generate Domain** under the service settings to get a public URL like `https://my-express-app-production.up.railway.app`.

### Alternative: Deploy via CLI

```bash
# Authenticate with Railway
railway login

# Initialize a new Railway project in the current directory
railway init

# Link to an existing project (if already created on dashboard)
railway link

# Deploy the current directory
railway up

# Open the project in the Railway dashboard
railway open
```

---

## Deploy Python / FastAPI

### Step 1: Prepare Your FastAPI App

Create `main.py`:

```python
import os
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def read_root():
    return {"message": "Hello from Railway!"}

@app.get("/health")
def health_check():
    return {"status": "ok"}
```

Create `requirements.txt`:

```
fastapi==0.109.0
uvicorn[standard]==0.27.0
```

### Step 2: Configure the Start Command

Railway uses Nixpacks to detect Python projects. You can specify the start command in a `railway.toml` or `Procfile`.

Create `Procfile`:

```
web: uvicorn main:app --host 0.0.0.0 --port $PORT
```

Or set the start command in the Railway dashboard under your service's **Settings** > **Deploy** > **Custom Start Command**.

### Step 3: Push to GitHub and Deploy

```bash
git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/<username>/my-fastapi-app.git
git push -u origin main
```

Then import the repo from the Railway dashboard (same steps as Node.js above). Your API docs are automatically available at `https://<your-service>.up.railway.app/docs`.

---

## Railway CLI Reference

The Railway CLI provides full control over your projects from the terminal:

```bash
# Install
npm i -g @railway/cli

# Login to your Railway account
railway login

# Initialize a new project
railway init

# Link current directory to an existing project
railway link

# Deploy the current directory
railway up

# Stream deployment logs
railway logs

# Open the project dashboard
railway open

# Set an environment variable
railway variables set KEY=value

# List environment variables
railway variables

# Run a command with Railway env vars injected
railway run node server.js

# Connect to a database service locally
railway connect postgres
```

---

## railway.toml Configuration

Create `railway.toml` in your project root for build and deploy configuration:

```toml
[build]
builder = "nixpacks"
buildCommand = "npm run build"

[deploy]
startCommand = "npm start"
healthcheckPath = "/health"
healthcheckTimeout = 100
restartPolicyType = "on_failure"
restartPolicyMaxRetries = 5
```

### Python Example

```toml
[build]
builder = "nixpacks"
buildCommand = "pip install -r requirements.txt"

[deploy]
startCommand = "uvicorn main:app --host 0.0.0.0 --port $PORT"
healthcheckPath = "/health"
healthcheckTimeout = 100
```

### Nixpacks Configuration

Railway uses [Nixpacks](https://nixpacks.com/) to auto-detect and build your app. Nixpacks supports Node.js, Python, Go, Rust, Java, and many other languages. You can customize the build with a `nixpacks.toml`:

```toml
[phases.setup]
nixPkgs = ["...", "ffmpeg"]

[phases.install]
cmds = ["npm ci"]

[phases.build]
cmds = ["npm run build"]

[start]
cmd = "npm start"
```

---

## PostgreSQL and Redis Add-Ons

### Add PostgreSQL

1. Open your project on the Railway dashboard
2. Click **New** > **Database** > **Add PostgreSQL**
3. Railway provisions a PostgreSQL instance and sets `DATABASE_URL` automatically
4. Your app can access the database via `DATABASE_URL`

Via CLI:

```bash
# Add PostgreSQL to your project
railway add --plugin postgresql

# Connect to the database locally
railway connect postgres
```

### Add Redis

1. Open your project on the Railway dashboard
2. Click **New** > **Database** > **Add Redis**
3. Railway provisions a Redis instance and sets `REDIS_URL` automatically

Via CLI:

```bash
# Add Redis to your project
railway add --plugin redis

# Connect to Redis locally
railway connect redis
```

### Using Database Variables in Your App

```js
// Node.js -- PostgreSQL with pg
import pg from 'pg';

const pool = new pg.Pool({
  connectionString: process.env.DATABASE_URL,
});

const result = await pool.query('SELECT NOW()');
console.log(result.rows[0]);
```

```python
# Python -- PostgreSQL with asyncpg
import os
import asyncpg

DATABASE_URL = os.environ["DATABASE_URL"]

async def connect():
    conn = await asyncpg.connect(DATABASE_URL)
    row = await conn.fetchrow("SELECT NOW()")
    print(row)
    await conn.close()
```

---

## Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `PORT` | Server port (auto-set by Railway) | `3000` |
| `DATABASE_URL` | PostgreSQL connection string (auto-set if plugin added) | `postgresql://user:pass@host/db` |
| `REDIS_URL` | Redis connection string (auto-set if plugin added) | `redis://default:pass@host:6379` |
| `RAILWAY_ENVIRONMENT` | Current environment name | `production` |
| `RAILWAY_PUBLIC_DOMAIN` | Public domain of the service | `my-app-production.up.railway.app` |
| `NODE_ENV` | Node environment | `production` |
| `SECRET_KEY` | App secret key | `your-secret-key-here` |

### Set via Dashboard

1. Go to your project on [railway.app](https://railway.app/)
2. Click on your service
3. Navigate to the **Variables** tab
4. Click **New Variable** and enter key-value pairs
5. Changes are applied on the next deploy (Railway auto-redeploys)

### Set via CLI

```bash
# Set a single variable
railway variables set SECRET_KEY=my-super-secret

# Set multiple variables
railway variables set NODE_ENV=production API_KEY=abc123

# List all variables
railway variables

# Remove a variable
railway variables delete SECRET_KEY
```

### Shared Variables

Railway supports shared variables across services in a project. Set them at the project level in the dashboard under **Settings** > **Shared Variables**.

---

## Custom Domain

### Step 1: Generate a Railway Domain

1. Go to your service on Railway
2. Navigate to **Settings** > **Networking**
3. Click **Generate Domain** to get a `*.up.railway.app` URL

### Step 2: Add a Custom Domain

1. In **Settings** > **Networking**, click **Custom Domain**
2. Enter your domain (e.g., `api.yourdomain.com`)

### Step 3: Configure DNS

Add the DNS records shown by Railway at your domain registrar:

**For subdomain (`api.yourdomain.com`):**

| Type | Name | Value |
|------|------|-------|
| CNAME | api | `<your-service>.up.railway.app` |

**For apex domain (`yourdomain.com`):**

| Type | Name | Value |
|------|------|-------|
| CNAME | @ | `<your-service>.up.railway.app` |

> **Note:** Some DNS providers do not support CNAME records on the apex domain. In that case, use a CNAME-flattening provider (e.g., Cloudflare) or use a subdomain like `www`.

### Step 4: Verify and SSL

Railway automatically provisions an SSL certificate via Let's Encrypt once DNS propagates. No manual configuration is needed. Verification typically completes in a few minutes.

---

## Free Tier and Billing

Railway uses usage-based billing with a trial credit:

| Feature | Trial Plan | Hobby ($5/mo) | Pro ($20/mo) |
|---------|------------|----------------|--------------|
| **Credit** | $5 one-time | $5/month included | $20/month included |
| **Execution Hours** | 500 hours total | Unlimited | Unlimited |
| **Memory** | Up to 512 MB | Up to 8 GB | Up to 32 GB |
| **Shared CPU** | Yes | Up to 8 vCPU | Up to 32 vCPU |
| **Network** | 100 GB egress | 100 GB egress | Unlimited |
| **Services** | Limited | Unlimited | Unlimited |
| **Always-On** | Yes (while credits last) | Yes | Yes |

### Key things to know:

- **Trial gives $5 in free credit.** Once credits are exhausted, services stop until you add a payment method.
- **Railway does NOT spin down free services.** Unlike Render, services are always-on while credits last.
- **Usage is billed per minute** based on CPU and memory consumption. Idle services still use a small amount of resources.
- **The Hobby plan** ($5/month) is the lowest paid tier and includes $5 of usage credit. Most small apps cost well under $5/month.
- Monitor your credit usage in the dashboard under **Usage**.

---

## Troubleshooting

### Problem: Build fails with Nixpacks

**Cause:** Nixpacks cannot detect your project type or a dependency is missing.

**Fix:**

```bash
# Make sure your project has the correct files for detection:
# Node.js: package.json must exist
# Python: requirements.txt or Pipfile must exist

# Test the Nixpacks build locally
npx nixpacks build . --name test-build

# If Nixpacks detection fails, specify the builder explicitly in railway.toml:
```

```toml
# railway.toml
[build]
builder = "nixpacks"
nixpacksPlan = { providers = ["node"] }
```

Or switch to a Dockerfile-based build:

```toml
# railway.toml
[build]
builder = "dockerfile"
dockerfilePath = "./Dockerfile"
```

### Problem: App crashes with port binding errors

**Cause:** The app is not using the `PORT` environment variable provided by Railway, or it is binding to `127.0.0.1` instead of `0.0.0.0`.

**Fix:**

```js
// Node.js -- bind to 0.0.0.0 and use process.env.PORT
const PORT = process.env.PORT || 3000;
app.listen(PORT, '0.0.0.0', () => {
  console.log(`Listening on port ${PORT}`);
});
```

```python
# Python -- ensure uvicorn binds to 0.0.0.0 and uses $PORT
# Start command: uvicorn main:app --host 0.0.0.0 --port $PORT
```

Check deployment logs via the dashboard or CLI:

```bash
railway logs
```

### Problem: Database connection refused

**Cause:** The app is using a hardcoded connection string instead of the `DATABASE_URL` variable, or the database plugin has not been added.

**Fix:**

1. Verify the PostgreSQL plugin is added to your project (check the dashboard)
2. Ensure your app reads `DATABASE_URL` from the environment:

```js
// Node.js
const pool = new pg.Pool({
  connectionString: process.env.DATABASE_URL,
  ssl: { rejectUnauthorized: false },
});
```

```python
# Python
import os
DATABASE_URL = os.environ.get("DATABASE_URL")
```

3. Check that the variable is visible in the **Variables** tab of your service
4. Railway auto-injects database variables into linked services. If the variable is missing, go to the database service and copy the connection string manually

### Problem: Credits running out too fast

**Cause:** Services consume CPU and memory even when idle. Multiple services or databases compound usage.

**Fix:**

1. Check **Usage** in the Railway dashboard to see per-service costs
2. Remove unused services and databases
3. Reduce memory allocation in your app (e.g., limit Node.js heap size):

```bash
# Set in environment variables
NODE_OPTIONS=--max-old-space-size=256
```

4. Use Railway's sleep feature for development environments:
   - Go to service **Settings** > enable **Sleep** to pause the service when not in use
5. Upgrade to the Hobby plan ($5/month) which includes $5 of usage credit each month

### Problem: Service shows "No active deployment" or fails to start

**Cause:** The start command is missing or incorrect, or the build produced no output.

**Fix:**

1. Check that your `railway.toml` or dashboard settings have the correct start command
2. For Node.js, ensure `package.json` has a `start` script
3. For Python, ensure a `Procfile` or explicit start command is set
4. Check the build logs for errors:

```bash
railway logs --build
```

5. Try redeploying:

```bash
railway up
```

### Problem: Deployment succeeds but the app returns 502 Bad Gateway

**Cause:** The app starts but crashes shortly after, or the health check fails.

**Fix:**

1. Check the runtime logs for crash errors:

```bash
railway logs
```

2. If using a health check, make sure the endpoint returns a 200 response:

```js
app.get('/health', (req, res) => {
  res.status(200).json({ status: 'ok' });
});
```

3. Verify the `healthcheckPath` in `railway.toml` matches your actual endpoint
4. Increase `healthcheckTimeout` if the app takes time to start:

```toml
[deploy]
healthcheckPath = "/health"
healthcheckTimeout = 300
```

5. Check that all required environment variables are set -- a missing variable can cause the app to crash on startup
