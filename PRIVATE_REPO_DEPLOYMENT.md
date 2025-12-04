# Deploying from Private Repository

## Overview
Yes, you can deploy GitHub Pages from a private repository, but there are different options depending on your GitHub plan and requirements.

---

## Option 1: GitHub Pages with Private Repo (Paid Plans Only)

### Requirements
- **GitHub Pro** ($4/month) - Individual
- **GitHub Team** ($4/user/month) - Organization
- **GitHub Enterprise** - Enterprise

### How It Works
1. Create a **private repository**
2. Enable GitHub Pages in Settings → Pages
3. Select branch/folder as source
4. Your site will be **publicly accessible** even though the repo is private

### Limitations
- ✅ Source code stays private
- ✅ Site is publicly accessible
- ❌ Requires paid GitHub plan
- ❌ Free accounts cannot use this

---

## Option 2: GitHub Actions → Public Repo (Free Alternative)

### How It Works
1. Keep your **source code in a private repository**
2. Use GitHub Actions to automatically deploy to a **public repository**
3. The public repo serves as the GitHub Pages source
4. Your source code remains private

### Setup Steps

#### Step 1: Create Two Repositories
- **Private repo**: `my-site-source` (your source code)
- **Public repo**: `my-site-pages` (deployed files)

#### Step 2: Create GitHub Actions Workflow

In your **private repository**, create `.github/workflows/deploy.yml`:

```yaml
name: Deploy to GitHub Pages

on:
  push:
    branches:
      - main
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout source
        uses: actions/checkout@v4
        with:
          path: source
      
      - name: Checkout pages repo
        uses: actions/checkout@v4
        with:
          repository: YOUR_USERNAME/my-site-pages
          token: ${{ secrets.PAGES_REPO_TOKEN }}
          path: pages
      
      - name: Copy files
        run: |
          cp -r source/login pages/
          cp -r source/admin pages/
          cp -r source/noADMIN pages/
          cp -r source/noKV pages/
      
      - name: Commit and push
        run: |
          cd pages
          git config user.name "GitHub Actions"
          git config user.email "actions@github.com"
          git add .
          git commit -m "Deploy from private repo" || exit 0
          git push
```

#### Step 3: Create Personal Access Token

1. Go to GitHub → Settings → Developer settings → Personal access tokens → Tokens (classic)
2. Generate new token with `repo` scope
3. Add to private repo → Settings → Secrets → Actions
4. Name: `PAGES_REPO_TOKEN`

#### Step 4: Enable Pages on Public Repo

1. Go to public repository (`my-site-pages`)
2. Settings → Pages
3. Source: `main` branch, `/ (root)`

---

## Option 3: Alternative Hosting Platforms (Recommended for Private)

These platforms allow **private deployments from private repositories** on free plans:

### A. Cloudflare Pages (Recommended)

**Why**: Free, fast, and integrates well with Cloudflare Workers

**Setup**:
1. Go to Cloudflare Dashboard → Pages
2. Connect GitHub account
3. Select your **private repository**
4. Build settings:
   - Framework preset: None
   - Build command: (leave empty)
   - Build output directory: `/`
5. Deploy

**Your Pages URL**: `https://YOUR_SITE.pages.dev`

**Update Worker**:
```javascript
// Set environment variable
PAGES_URL = https://YOUR_SITE.pages.dev
```

### B. Netlify

**Setup**:
1. Go to netlify.com
2. Connect GitHub account
3. Select your **private repository**
4. Deploy settings:
   - Build command: (empty)
   - Publish directory: `/`
5. Deploy

**Your Pages URL**: `https://YOUR_SITE.netlify.app`

### C. Vercel

**Setup**:
1. Go to vercel.com
2. Connect GitHub account
3. Import your **private repository**
4. Deploy

**Your Pages URL**: `https://YOUR_SITE.vercel.app`

---

## Option 4: Keep Repo Public (Simplest)

If you don't need to keep the frontend code private:

1. Create a **public repository**
2. Deploy directly to GitHub Pages
3. **Free** and **simple**

**Note**: Frontend code will be public, but this is usually fine since:
- Frontend code is client-side anyway
- No sensitive data should be in frontend
- Your Worker handles authentication

---

## Comparison Table

| Option | Cost | Privacy | Complexity | Best For |
|--------|------|---------|-----------|----------|
| **GitHub Pages (Private Repo)** | Paid ($4+/mo) | Source private, site public | Easy | Paid GitHub users |
| **GitHub Actions → Public Repo** | Free | Source private, site public | Medium | Free users who want privacy |
| **Cloudflare Pages** | Free | Source private, site public | Easy | Cloudflare users |
| **Netlify** | Free | Source private, site public | Easy | General use |
| **Vercel** | Free | Source private, site public | Easy | Next.js/React apps |
| **Public Repo** | Free | Both public | Easy | No privacy needed |

---

## Recommended Solution

### For Your Use Case (Cloudflare Worker):

**Use Cloudflare Pages** because:
- ✅ **Free** private repo deployment
- ✅ **Fast** CDN (same as Workers)
- ✅ **Easy** setup
- ✅ **Integrates** well with Cloudflare Workers
- ✅ **Same ecosystem** (Cloudflare)

### Setup Steps:

1. **Create Cloudflare Pages site**:
   ```bash
   # Install Wrangler Pages
   npm install -g wrangler
   
   # Login
   wrangler login
   
   # Create pages project
   wrangler pages project create my-frontend
   ```

2. **Or use Cloudflare Dashboard**:
   - Go to Cloudflare Dashboard → Pages
   - Connect GitHub
   - Select your private repository
   - Deploy

3. **Get your Pages URL**:
   - Format: `https://my-frontend.pages.dev`
   - Or custom domain: `https://pages.yourdomain.com`

4. **Update Worker**:
   ```bash
   # Set environment variable
   wrangler secret put PAGES_URL
   # Enter: https://my-frontend.pages.dev
   ```

5. **Update ALLOWED_API_DOMAINS** in `_worker.secure.js`:
   ```javascript
   const ALLOWED_API_DOMAINS = [
       // ... existing domains ...
       'my-frontend.pages.dev',  // Add your Pages domain
   ];
   ```

---

## Security Considerations

### What Should Be Private?
- ✅ **Source code** (if proprietary)
- ✅ **Build scripts** (if sensitive)
- ✅ **Environment variables** (never commit these)

### What Can Be Public?
- ✅ **Frontend HTML/CSS/JS** (runs in browser anyway)
- ✅ **Static assets** (images, fonts)
- ✅ **Deployed site** (needs to be accessible)

### Best Practice:
- Keep **source repository** private if it contains sensitive build logic
- Deploy to **public hosting** (site needs to be accessible)
- Store **secrets** in environment variables (never in code)

---

## Quick Start: Cloudflare Pages

### Method 1: Via Dashboard (Easiest)

1. Go to [Cloudflare Dashboard](https://dash.cloudflare.com)
2. Select your account → **Pages**
3. Click **"Create a project"** → **"Connect to Git"**
4. Authorize GitHub and select your **private repository**
5. Configure:
   - **Project name**: `my-frontend`
   - **Production branch**: `main`
   - **Build command**: (leave empty)
   - **Build output directory**: `/`
6. Click **"Save and Deploy"**

### Method 2: Via Wrangler CLI

```bash
# Install Wrangler
npm install -g wrangler

# Login
wrangler login

# Create project
wrangler pages project create my-frontend

# Deploy
cd your-frontend-folder
wrangler pages deploy . --project-name=my-frontend
```

---

## Troubleshooting

### Issue: Pages not accessible
- **Check**: Is the repository private? (Use Cloudflare Pages or paid GitHub)
- **Check**: Are files in correct directory structure?
- **Check**: Wait a few minutes for deployment

### Issue: CORS errors
- **Check**: Is domain in `ALLOWED_API_DOMAINS`?
- **Check**: Are CORS headers set correctly?

### Issue: Worker can't fetch pages
- **Check**: Is `PAGES_URL` environment variable set correctly?
- **Check**: Is domain in `ALLOWED_API_DOMAINS`?
- **Check**: Can you access the Pages URL directly in browser?

---

## Summary

**For private repository deployment:**

1. **Paid GitHub**: Use GitHub Pages directly (easiest)
2. **Free GitHub**: Use GitHub Actions → Public repo (more complex)
3. **Cloudflare Pages**: **Recommended** - Free, easy, integrates with Workers
4. **Netlify/Vercel**: Good alternatives, also free

**Recommendation**: Use **Cloudflare Pages** for the best integration with your Cloudflare Worker setup.

---

## Next Steps

1. ✅ Choose your deployment platform
2. ✅ Set up Pages site (Cloudflare Pages recommended)
3. ✅ Get your Pages URL
4. ✅ Update Worker `PAGES_URL` environment variable
5. ✅ Update `ALLOWED_API_DOMAINS` in Worker code
6. ✅ Test deployment

Your frontend can now be deployed from a private repository! 🔒

