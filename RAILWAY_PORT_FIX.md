# Railway Port 5055 Not Accessible - Solution

## Problem
Railway typically only exposes **one port per service** publicly. Since your frontend (port 8502) is the primary service, Railway is routing all traffic to port 8502. Port 5055 (API) is not publicly accessible.

## Solution: Use Next.js Internal Proxying

Since both frontend and API are in the **same container**, we can use Next.js rewrites to proxy API requests internally. This doesn't require port 5055 to be publicly exposed.

### Step 1: Verify API is Running Internally

Check Railway logs to confirm:
- ✅ "Uvicorn running on 0.0.0.0:5055"
- ✅ "API initialization completed successfully"

If you don't see these, the API isn't starting. Check for errors.

### Step 2: Configure Internal API URL

In Railway → **Variables** tab, add:

```
INTERNAL_API_URL=http://localhost:5055
```

This tells Next.js where to proxy `/api/*` requests internally.

### Step 3: Remove or Don't Set API_URL

**Don't set `API_URL`** - this would make the browser try to reach port 5055 directly, which won't work.

The frontend should use Next.js rewrites to proxy to the internal API.

### Step 4: Verify Next.js Rewrites Are Working

The `next.config.ts` should have rewrites configured. Check that:
- Rewrites are NOT disabled (they should work since we're not on Vercel)
- `INTERNAL_API_URL` is set to `http://localhost:5055`

### Step 5: Test the Setup

1. Visit: `https://on-aya-production.up.railway.app/config`
   - Should return JSON with `apiUrl` field
   - The `apiUrl` should point to where the browser should make requests

2. The frontend should now proxy `/api/*` requests to `localhost:5055` internally

## Alternative: If Rewrites Don't Work

If Next.js rewrites aren't working on Railway, we need to make the frontend call the API through the same port. Update the config to use the same domain:

In Railway → **Variables**:
```
API_URL=https://on-aya-production.up.railway.app
```

Then we'd need to configure Railway to route `/api/*` to port 5055, but Railway doesn't support path-based routing easily.

## Best Solution: Fix Next.js Rewrites

The rewrites should work. Let's verify:

1. **Check Railway Logs** for Next.js startup:
   - Look for: `[Next.js Rewrites] Proxying /api/* to http://localhost:5055/api/*`

2. **Set INTERNAL_API_URL**:
   ```
   INTERNAL_API_URL=http://localhost:5055
   ```

3. **Don't set API_URL** (or remove it if set)

4. **Redeploy** and test

## Debugging Steps

1. **Check if API is running**:
   - Railway logs should show API startup messages
   - Look for "Uvicorn running on 0.0.0.0:5055"

2. **Check if Next.js rewrites are active**:
   - Railway logs should show rewrite configuration
   - Look for "[Next.js Rewrites] Proxying /api/*"

3. **Test internal connectivity**:
   - The API should be reachable from within the container at `http://localhost:5055`
   - Next.js should be able to proxy to it

4. **Check browser console**:
   - Look for the actual URL being called
   - Should be `/api/config` (relative) not an absolute URL with port 5055

## Expected Behavior

With proper configuration:
- Browser requests: `https://on-aya-production.up.railway.app/api/config`
- Next.js rewrites to: `http://localhost:5055/api/config` (internal)
- API responds
- Next.js returns response to browser

This all happens internally in the container - no need for port 5055 to be publicly exposed.

