# Fix: API Connection Issue on Railway

## Problem
The frontend is trying to call `/api/config` but can't reach the FastAPI backend because Railway routes traffic by port, not by path.

## Solution: Set API_URL Environment Variable

Railway exposes each port separately. You need to tell the frontend where to find the API.

### Step 1: Get Your Railway API URL

Railway provides a URL for each exposed port. You need the URL for port **5055** (the API).

1. Go to your Railway service → **Settings** → **Networking**
2. Find the public URL for port **5055**
3. It should look like: `https://on-aya-production.up.railway.app` (but Railway might give you a different URL for port 5055)

**OR** Railway might use the same domain with different routing. Check your Railway dashboard for:
- Port 8502 URL (Frontend)
- Port 5055 URL (API)

### Step 2: Set API_URL Environment Variable

In Railway → **Variables** tab, add:

```
API_URL=https://your-api-url-here:5055
```

**Important**: Replace `your-api-url-here` with your actual Railway URL for port 5055.

If Railway uses the same domain for both ports, try:
```
API_URL=https://on-aya-production.up.railway.app:5055
```

**OR** if Railway provides a separate URL for the API port, use that:
```
API_URL=https://your-api-railway-url.railway.app
```

### Step 3: Alternative - Use Railway's Internal Networking

If both services are in the same Railway project, you can use Railway's private networking:

```
API_URL=${{YourService.RAILWAY_PRIVATE_DOMAIN}}:5055
```

But since you're using a single container, this might not apply.

### Step 4: Verify Port 5055 is Exposed

1. Go to **Settings** → **Networking**
2. Make sure port **5055** is listed and has a public URL
3. Test the API directly: Visit `https://your-api-url:5055/health`
   - Should return: `{"status":"healthy"}`

### Step 5: Redeploy

After setting `API_URL`:
1. Go to **Deployments** tab
2. Click **Redeploy**
3. Wait for deployment to complete

### Step 6: Test

1. Visit your frontend: `https://on-aya-production.up.railway.app`
2. Open browser console (F12)
3. Check for connection errors
4. The frontend should now be able to reach the API

## Quick Fix Summary

**Add this to Railway Variables:**
```
API_URL=https://on-aya-production.up.railway.app:5055
```

**OR** if Railway gave you a separate URL for port 5055:
```
API_URL=https://your-separate-api-url.railway.app
```

Then redeploy!

## How to Find Your API URL on Railway

1. **Method 1**: Check Railway dashboard → Your service → Networking tab
   - Look for port 5055's public URL

2. **Method 2**: Railway might use the same domain
   - Try: `https://on-aya-production.up.railway.app:5055`
   - Test in browser: Should show API docs or health endpoint

3. **Method 3**: Check Railway logs
   - Look for "Listening on" messages
   - Should show which URL/port the API is using

## Still Not Working?

If port 5055 isn't accessible:

1. **Check Railway Networking Settings**:
   - Make sure port 5055 is exposed
   - Railway should auto-detect from Dockerfile, but verify

2. **Check Railway Logs**:
   - Look for API startup messages
   - Should see: "Uvicorn running on 0.0.0.0:5055"

3. **Test API Directly**:
   - Try: `https://on-aya-production.up.railway.app:5055/health`
   - Try: `https://on-aya-production.up.railway.app:5055/docs`
   - If these don't work, port 5055 isn't exposed properly

4. **Railway Port Configuration**:
   - Some Railway setups require explicit port configuration
   - Check if Railway has a "Ports" or "Networking" section where you need to enable port 5055

