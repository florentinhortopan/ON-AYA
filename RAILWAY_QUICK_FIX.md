# Railway Quick Fix: API Not Accessible

## The Problem
- Port 5055 isn't publicly accessible on Railway (Railway only exposes one port per service)
- The frontend can't reach the API
- Health endpoint doesn't work

## The Solution

Since both services are in the same container, use Next.js rewrites to proxy API requests internally.

### Step 1: Set INTERNAL_API_URL (Required)

In Railway → **Variables** tab, add:

```
INTERNAL_API_URL=http://localhost:5055
```

This tells Next.js where to proxy `/api/*` requests.

### Step 2: Remove API_URL (If Set)

**Remove or don't set `API_URL`** - we want to use internal proxying, not direct API calls.

### Step 3: Verify API is Running

Check Railway logs for:
- ✅ "Uvicorn running on 0.0.0.0:5055"
- ✅ "API initialization completed successfully"

If you don't see these, check for errors in the logs.

### Step 4: Verify Required Environment Variables

Make sure these are set in Railway:

```
SURREAL_URL=ws://localhost:8000/rpc
SURREAL_USER=root
SURREAL_PASSWORD=root
SURREAL_NAMESPACE=open_notebook
SURREAL_DATABASE=production
OPENAI_API_KEY=your_key_here
INTERNAL_API_URL=http://localhost:5055
```

### Step 5: Redeploy

After setting `INTERNAL_API_URL`:
1. Go to **Deployments** tab
2. Click **Redeploy**
3. Watch logs for:
   - API startup
   - Next.js rewrite configuration message

### Step 6: Test

1. Visit: `https://on-aya-production.up.railway.app/config`
   - Should return JSON (might show localhost:5055, that's OK)

2. Visit: `https://on-aya-production.up.railway.app`
   - Should now work! The frontend will proxy `/api/*` to the internal API

## How It Works

```
Browser Request: https://on-aya-production.up.railway.app/api/config
         ↓
Next.js Rewrite: http://localhost:5055/api/config (internal)
         ↓
FastAPI Backend: Responds
         ↓
Next.js: Returns response to browser
```

All happens inside the container - no need for port 5055 to be public!

## Still Not Working?

### Check Railway Logs For:

1. **API Startup**:
   ```
   [program:api] Uvicorn running on 0.0.0.0:5055
   ```

2. **Next.js Rewrites**:
   ```
   [Next.js Rewrites] Proxying /api/* to http://localhost:5055/api/*
   ```

3. **Database Connection**:
   ```
   [program:api] Database migrations completed successfully
   ```

### Common Issues:

**Issue**: API not starting
- **Fix**: Check database environment variables (SURREAL_URL, SURREAL_USER, SURREAL_PASSWORD)
- **Fix**: Check for errors in Railway logs

**Issue**: Rewrites not working
- **Fix**: Make sure `INTERNAL_API_URL=http://localhost:5055` is set
- **Fix**: Make sure `VERCEL` environment variable is NOT set

**Issue**: Frontend can't connect
- **Fix**: Check that API is running (look for "Uvicorn running" in logs)
- **Fix**: Check browser console for specific error messages

## Summary

**Required Railway Variables:**
```
INTERNAL_API_URL=http://localhost:5055
SURREAL_URL=ws://localhost:8000/rpc
SURREAL_USER=root
SURREAL_PASSWORD=root
SURREAL_NAMESPACE=open_notebook
SURREAL_DATABASE=production
OPENAI_API_KEY=your_key_here
```

**Don't set:**
- `API_URL` (we're using internal proxying)

Then redeploy and it should work!

