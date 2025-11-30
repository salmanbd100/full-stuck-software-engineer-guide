# Next.js Deployment

## Table of Contents
- [Vercel Deployment](#vercel-deployment)
- [Self-Hosting](#self-hosting)
- [Docker Deployment](#docker-deployment)
- [Environment Variables](#environment-variables)
- [Build Optimization](#build-optimization)
- [Interview Questions](#interview-questions)

---

## Vercel Deployment

Vercel is the easiest way to deploy Next.js (made by the same team).

### Deploy with Vercel CLI

```bash
# Install Vercel CLI
npm i -g vercel

# Deploy
vercel

# Deploy to production
vercel --prod
```

### Deploy with Git Integration

1. Push code to GitHub/GitLab/Bitbucket
2. Import project on [vercel.com](https://vercel.com)
3. Automatic deployments on git push

**Features:**
- Automatic HTTPS
- Global CDN
- Instant rollbacks
- Preview deployments for PRs
- Analytics

---

## Self-Hosting

### Node.js Server

```bash
# Build for production
npm run build

# Start production server
npm start
```

**package.json:**
```json
{
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start -p 3000"
  }
}
```

### PM2 Process Manager

```bash
# Install PM2
npm install -g pm2

# Start with PM2
pm2 start npm --name "my-app" -- start

# Save process list
pm2 save

# Auto-restart on reboot
pm2 startup

# Monitor
pm2 monit

# Logs
pm2 logs

# Restart
pm2 restart my-app

# Stop
pm2 stop my-app
```

### Nginx Reverse Proxy

```nginx
# /etc/nginx/sites-available/my-app
server {
    listen 80;
    server_name example.com;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

```bash
# Enable site
sudo ln -s /etc/nginx/sites-available/my-app /etc/nginx/sites-enabled/

# Test config
sudo nginx -t

# Restart Nginx
sudo systemctl restart nginx
```

---

## Docker Deployment

### Basic Dockerfile

```dockerfile
# Dockerfile
FROM node:18-alpine AS base

# Dependencies
FROM base AS deps
RUN apk add --no-cache libc6-compat
WORKDIR /app

COPY package*.json ./
RUN npm ci

# Builder
FROM base AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .

ENV NEXT_TELEMETRY_DISABLED 1
RUN npm run build

# Runner
FROM base AS runner
WORKDIR /app

ENV NODE_ENV production
ENV NEXT_TELEMETRY_DISABLED 1

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

COPY --from=builder /app/public ./public
COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static

USER nextjs

EXPOSE 3000

ENV PORT 3000
ENV HOSTNAME "0.0.0.0"

CMD ["node", "server.js"]
```

### next.config.js for Docker

```js
// next.config.js
module.exports = {
  output: 'standalone',
};
```

### Build and Run

```bash
# Build image
docker build -t my-next-app .

# Run container
docker run -p 3000:3000 my-next-app

# With environment variables
docker run -p 3000:3000 \
  -e DATABASE_URL=postgres://... \
  -e API_KEY=... \
  my-next-app
```

### Docker Compose

```yaml
# docker-compose.yml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      - DATABASE_URL=${DATABASE_URL}
      - API_KEY=${API_KEY}
    restart: unless-stopped
```

```bash
# Start
docker-compose up -d

# Stop
docker-compose down
```

---

## Environment Variables

### Local Development

```bash
# .env.local (not committed)
DATABASE_URL=postgres://localhost/mydb
API_KEY=secret-key-123
NEXT_PUBLIC_API_URL=http://localhost:3000/api
```

### Production

```bash
# .env.production
DATABASE_URL=postgres://prod-server/mydb
API_KEY=prod-secret-key
NEXT_PUBLIC_API_URL=https://api.example.com
```

### Usage

```js
// Server-side only
const dbUrl = process.env.DATABASE_URL;

// Client and server (must start with NEXT_PUBLIC_)
const apiUrl = process.env.NEXT_PUBLIC_API_URL;
```

### Vercel Environment Variables

1. Go to Project Settings → Environment Variables
2. Add variables for Production/Preview/Development
3. Redeploy to apply changes

---

## Build Optimization

### Analyze Bundle Size

```bash
# Install analyzer
npm install @next/bundle-analyzer

# Configure
// next.config.js
const withBundleAnalyzer = require('@next/bundle-analyzer')({
  enabled: process.env.ANALYZE === 'true',
});

module.exports = withBundleAnalyzer({
  // config
});

# Run analysis
ANALYZE=true npm run build
```

### Output File Tracing

```js
// next.config.js
module.exports = {
  output: 'standalone',
};
```

This creates `.next/standalone` with minimal dependencies.

### Compression

```js
// next.config.js
module.exports = {
  compress: true, // Enable gzip compression
};
```

### Image Optimization

```js
// next.config.js
module.exports = {
  images: {
    formats: ['image/avif', 'image/webp'],
    minimumCacheTTL: 60,
  },
};
```

---

## Interview Questions

### Q1: What are the deployment options for Next.js?

**Answer:**

1. **Vercel** - Easiest, automatic deployments
2. **Self-hosting** - Node.js server (PM2, Nginx)
3. **Docker** - Containerized deployment
4. **Static export** - CDN hosting (limited features)

```bash
# Vercel
vercel

# Node.js
npm run build && npm start

# Docker
docker build -t app . && docker run -p 3000:3000 app

# Static export
npm run build && next export
```

---

### Q2: How do you deploy Next.js with Docker?

**Answer:**

1. Create Dockerfile with standalone output:

```dockerfile
FROM node:18-alpine AS base
# ... (see full example above)

FROM base AS runner
WORKDIR /app
COPY --from=builder /app/.next/standalone ./
CMD ["node", "server.js"]
```

2. Configure next.config.js:

```js
module.exports = {
  output: 'standalone',
};
```

3. Build and run:

```bash
docker build -t my-app .
docker run -p 3000:3000 my-app
```

---

### Q3: How do environment variables work in Next.js?

**Answer:**

**Server-only (secret keys):**
```js
// Accessible only on server
const apiKey = process.env.API_KEY;
```

**Client and server (public):**
```js
// Must start with NEXT_PUBLIC_
const apiUrl = process.env.NEXT_PUBLIC_API_URL;
```

**Files:**
- `.env.local` - Local development (gitignored)
- `.env.production` - Production build
- `.env.development` - Development mode

---

### Q4: How do you optimize the bundle size?

**Answer:**

1. **Analyze bundle:**
```bash
ANALYZE=true npm run build
```

2. **Dynamic imports:**
```jsx
const Component = dynamic(() => import('./Component'));
```

3. **Tree shaking:**
```js
// Import only what you need
import { specific } from 'library';
```

4. **Remove unused dependencies**

5. **Use standalone output:**
```js
module.exports = {
  output: 'standalone',
};
```

---

### Q5: What's the difference between Vercel and self-hosting?

**Answer:**

**Vercel:**
- ✅ Easiest setup
- ✅ Automatic deployments
- ✅ Global CDN
- ✅ Edge functions
- ❌ Vendor lock-in
- ❌ Cost at scale

**Self-hosting:**
- ✅ Full control
- ✅ No vendor lock-in
- ✅ Cheaper at scale
- ❌ Manual setup
- ❌ Manage infrastructure
- ❌ No edge functions

---

## Key Takeaways

1. **Vercel** - Easiest deployment, automatic CI/CD
2. **Self-hosting** - Use Node.js + PM2 + Nginx
3. **Docker** - Containerized with standalone output
4. **Environment variables** - Use NEXT_PUBLIC_ for client-side
5. **Build optimization** - Analyze bundle, dynamic imports
6. **Compression** - Enable gzip/brotli
7. **Image optimization** - WebP/AVIF formats

---

[← Back to Next.js](./README.md)
