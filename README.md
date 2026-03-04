# Deploy Guide

> Step-by-step deployment guides for every platform and framework. Stop reading docs for hours — deploy in minutes.

Whether you're deploying a React app, a FastAPI backend, or a full-stack MERN project, this repo has a battle-tested guide for it.

## Platforms

| Platform | Guide | Free Tier | Best For |
|----------|-------|-----------|----------|
| [GitHub Pages](guides/github-pages.md) | Static sites, SPAs | Unlimited | React, Vue, static HTML |
| [Vercel](guides/vercel.md) | Frontend + serverless | Hobby (free) | Next.js, React, Svelte |
| [Render](guides/render.md) | Full-stack apps | 750 hrs/mo | Node.js, Python, Docker |
| [Railway](guides/railway.md) | Full-stack + databases | $5 credit | Any stack |
| [Fly.io](guides/fly-io.md) | Edge computing | 3 shared VMs | Docker, Go, Elixir |
| [Netlify](guides/netlify.md) | JAMstack | 100 GB/mo | Static, serverless |
| [AWS (ECS)](guides/aws-ecs.md) | Production containers | 12-month free tier | Enterprise, Docker |
| [DigitalOcean](guides/digitalocean.md) | VPS + App Platform | $200 credit | Self-hosted |

## Frameworks

| Framework | Guide | Recommended Platform |
|-----------|-------|---------------------|
| [React (Vite)](frameworks/react-vite.md) | SPA deployment | GitHub Pages / Vercel |
| [Next.js](frameworks/nextjs.md) | SSR/SSG/ISR | Vercel |
| [Express.js](frameworks/express.md) | Node.js backend | Render / Railway |
| [FastAPI](frameworks/fastapi.md) | Python backend | Render / Railway |
| [Django](frameworks/django.md) | Python full-stack | Render / Railway |
| [MERN Stack](frameworks/mern.md) | Full-stack JS | Render + GitHub Pages |
| [FARM Stack](frameworks/farm.md) | FastAPI + React + MongoDB | Render + GitHub Pages |

## Databases

| Database | Guide | Free Option |
|----------|-------|-------------|
| [MongoDB Atlas](guides/mongodb-atlas.md) | Cloud MongoDB | 512 MB free |
| [Neon](guides/neon.md) | Serverless PostgreSQL | 0.5 GB free |
| [Supabase](guides/supabase.md) | Postgres + Auth + Storage | 500 MB free |
| [Redis (Upstash)](guides/upstash.md) | Serverless Redis | 10K commands/day |

## Quick Decision Tree

```
What are you deploying?
|
+-- Static site / SPA (React, Vue, HTML)
|   +-- Need custom domain? --> GitHub Pages (free)
|   +-- Need serverless functions? --> Vercel / Netlify
|   +-- Just want it live fast? --> Vercel (zero config)
|
+-- Backend API (Express, FastAPI, Django)
|   +-- Free is priority? --> Render (sleeps after 15 min)
|   +-- Need always-on? --> Railway ($5/mo) or Fly.io
|   +-- Production scale? --> AWS ECS or DigitalOcean
|
+-- Full-stack (frontend + backend + database)
|   +-- MERN/FARM? --> Render (backend) + GitHub Pages (frontend)
|   +-- Next.js? --> Vercel (all-in-one)
|   +-- Docker? --> Fly.io or Railway
|
+-- Database only
    +-- PostgreSQL --> Neon (serverless, free)
    +-- MongoDB --> MongoDB Atlas (free 512 MB)
    +-- MySQL --> PlanetScale or Railway
    +-- Redis --> Upstash (free tier)
```

## Cost Comparison

| Platform | Free Tier | Cheapest Paid | Always-On | Custom Domain |
|----------|-----------|---------------|-----------|---------------|
| GitHub Pages | Unlimited | N/A | Yes | Yes (free SSL) |
| Vercel | Hobby (free) | Pro ($20/mo) | Yes | Yes |
| Render | 750 hrs/mo | Starter ($7/mo) | Paid only | Yes |
| Railway | $5 credit | Usage-based | Yes | Yes |
| Fly.io | 3 shared VMs | ~$2-5/mo | Yes | Yes |
| Netlify | 100 GB/mo | Pro ($19/mo) | Yes | Yes |

## Contributing

Have a deployment guide for a platform or framework not listed here? Contributions are welcome!

1. Fork this repo
2. Create a guide in `guides/` (platform) or `frameworks/` (framework)
3. Follow the [guide template](CONTRIBUTING.md#guide-template)
4. Submit a PR

See [CONTRIBUTING.md](CONTRIBUTING.md) for full guidelines.

## License

[MIT](LICENSE)
