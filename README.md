# Deploy Pages to GitHub Pages

## Overview
You have the frontend pages repository (`EDT-Pages.github.io-main`). This guide will help you deploy it to GitHub Pages and configure it to work with your secure Worker.

---

## Step 1: Fix Login Page for CSRF Token Support

The login page needs to be updated to work with the secure Worker. Here's what needs to change:

### Current Issue
- Login page posts to `/login` but should use `/api/login`
- Doesn't store CSRF token from response
- Doesn't send CSRF token in POST requests

### Fix Required

Update `login/index.html` around line 653:

**Change from:**
```javascript
const response = await fetch('/login', {
    method: 'POST',
    headers: {
        'Content-Type': 'application/x-www-form-urlencoded',
    },
    body: 'password=' + encodeURIComponent(password)
});
```

**Change to:**
```javascript
const response = await fetch('/api/login', {
    method: 'POST',
    headers: {
        'Content-Type': 'application/json',
    },
    credentials: 'include',  // Important: Include cookies
    body: JSON.stringify({ password: password })
});
```

**And update the response handling (around line 664):**
```javascript
if (response.ok) {
    const contentType = response.headers.get('content-type');
    
    if (contentType && contentType.includes('application/json')) {
        const data = await response.json();
        if (data.success && data.csrfToken) {
            // Store CSRF token for future requests
            localStorage.setItem('csrfToken', data.csrfToken);
            // Login successful, redirect to admin
            window.location.href = '/admin';
        } else {
            errorMsg.textContent = data.error || '密码错误，请重试';
            errorMsg.classList.remove('login-error-hidden');
            passwordInput.focus();
            passwordInput.select();
        }
    } else {
        errorMsg.textContent = '登录失败，请重试';
        errorMsg.classList.remove('login-error-hidden');
    }
} else {
    const errorData = await response.json().catch(() => ({}));
    errorMsg.textContent = errorData.error || '登录失败，请重试';
    errorMsg.classList.remove('login-error-hidden');
}
```

---

## Step 2: Deploy to GitHub Pages

### Option A: Deploy from Private Repository (Recommended)

#### Using Cloudflare Pages (Free, Supports Private Repos)

1. **Go to Cloudflare Dashboard** → **Pages**
2. **Create a project** → **Connect to Git**
3. **Authorize GitHub** and select your repository
4. **Configure build**:
   - **Project name**: `pages-billow` (or your choice)
   - **Production branch**: `main` (or `master`)
   - **Build command**: (leave empty)
   - **Build output directory**: `/`
5. **Save and Deploy**

Your Pages URL will be: `https://pages-billow.pages.dev` (or your custom domain)

#### Update Worker Configuration

```bash
# Set PAGES_URL environment variable
wrangler secret put PAGES_URL
# Enter: https://pages-billow.pages.dev
```

Or in Cloudflare Dashboard:
- Worker → Settings → Variables → Add `PAGES_URL` = `https://pages-billow.pages.dev`

#### Update ALLOWED_API_DOMAINS

Edit `_worker.secure.js` line ~21:
```javascript
const ALLOWED_API_DOMAINS = [
    // ... existing domains ...
    'pages-billow.pages.dev',  // Add your Pages domain
];
```

### Option B: Deploy to GitHub Pages (Public Repo Required)

1. **Create GitHub Repository**:
   ```bash
   cd pages-billow.github.io-main
   git init
   git add .
   git commit -m "Initial commit: Frontend pages"
   ```

2. **Create GitHub Repository**:
   - Go to github.com → New repository
   - Name: `pages-billow` (or `your-username.github.io`)
   - Make it **public** (required for free GitHub Pages)
   - Don't initialize with README

3. **Push to GitHub**:
   ```bash
   git remote add origin https://github.com/YOUR_USERNAME/pages-billow.git
   git branch -M main
   git push -u origin main
   ```

4. **Enable GitHub Pages**:
   - Go to repository → **Settings** → **Pages**
   - Source: `main` branch, `/ (root)` folder
   - Save

5. **Get Your Pages URL**:
   - `https://YOUR_USERNAME.github.io/pages-billow`
   - Or if repo name matches username: `https://YOUR_USERNAME.github.io`

6. **Update Worker**:
   ```bash
   wrangler secret put PAGES_URL
   # Enter: https://YOUR_USERNAME.github.io/pages-billow
   ```

---

## Step 3: Verify Deployment

1. **Visit your Pages URL directly**:
   - Should show the login page
   - Should load without errors

2. **Visit your Worker URL**:
   - `https://YOUR_WORKER.workers.dev/login`
   - Should show the login page (fetched from Pages)

3. **Test Login**:
   - Enter password
   - Should redirect to `/admin`
   - Check DevTools → Application → Local Storage
   - Should see `csrfToken` stored

4. **Test Save Functionality**:
   - Go to admin page
   - Make a change
   - Click "保存"
   - Should work without errors (CSRF token automatically added by Worker injection)

---

## Step 4: Quick Fix Script

I can create a fixed version of the login page. Would you like me to:

1. **Update the login page** with CSRF token support?
2. **Create a deployment script**?
3. **Create a GitHub Actions workflow** for automatic deployment?

---

## Current File Structure

```
pages-billow.github.io-main/
├── README.md
├── index.html              # Landing page (nginx welcome)
├── login/
│   └── index.html         # Login page (needs CSRF fix)
├── admin/
│   └── index.html         # Admin dashboard (CSRF handled by Worker injection)
├── noADMIN/
│   └── index.html         # Error: No admin password
├── noKV/
│   └── index.html         # Error: No KV namespace
├── locations               # Cloudflare locations data
└── sub                     # Subscription endpoint
```

---

## Summary

**To deploy your pages:**

1. ✅ **Fix login page** - Update to use `/api/login` and handle CSRF token
2. ✅ **Choose deployment platform**:
   - **Cloudflare Pages** (recommended - free, private repos)
   - **GitHub Pages** (free, public repos only)
3. ✅ **Configure Worker** - Set `PAGES_URL` environment variable
4. ✅ **Update ALLOWED_API_DOMAINS** - Add your Pages domain
5. ✅ **Test** - Verify login and save functionality

**Note**: The Worker already injects CSRF token handling into the admin page, so the admin page should work without changes. Only the login page needs updating.

---
