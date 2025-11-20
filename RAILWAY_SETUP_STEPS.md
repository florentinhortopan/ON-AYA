# Railway Setup: Step-by-Step Configuration

## Required Environment Variables

Go to your Railway service → **Variables** tab → Add these:

### 1. Database Configuration (REQUIRED)
```
SURREAL_URL=ws://localhost:8000/rpc
SURREAL_USER=root
SURREAL_PASSWORD=root
SURREAL_NAMESPACE=open_notebook
SURREAL_DATABASE=production
```

### 2. AI Provider (REQUIRED - at least one)
```
OPENAI_API_KEY=sk-your-actual-openai-key-here
```

### 3. Optional: API URL (for frontend)
```
API_URL=https://your-app.railway.app:5055
```
*(Railway will auto-detect this, but you can set it explicitly)*

## Step 2: Configure Ports

1. Go to **Settings** → **Networking**
2. Verify these ports are exposed:
   - **8502** (Frontend)
   - **5055** (API)

If they're not there, Railway should auto-detect from Dockerfile, but you can add them manually.

## Step 3: Configure Volumes (IMPORTANT!)

Without volumes, data is lost on restart:

1. Go to **Settings** → **Volumes**
2. Click **"New Volume"**
3. Add two volumes:
   - **Mount Path**: `/app/data` (Application data)
   - **Mount Path**: `/mydata` (SurrealDB database)

## Step 4: Redeploy

After setting environment variables:
1. Go to **Deployments** tab
2. Click **"Redeploy"** or push a new commit
3. Watch the logs for:
   - ✅ "SurrealDB started"
   - ✅ "Database migrations completed successfully"
   - ✅ "API initialization completed successfully"

## Step 5: Verify It's Working

### Check API Health
Visit: `https://your-app.railway.app:5055/health`
- Should return: `{"status":"healthy"}`

### Check API Endpoints
Visit: `https://your-app.railway.app:5055/api/notebooks`
- Should return: `[]` (empty array) or a list of notebooks
- If error, database isn't connected

### Check Frontend Config
Visit: `https://your-app.railway.app/config`
- Should return JSON with `apiUrl` field

### Check Frontend
Visit: `https://your-app.railway.app`
- Should show the Open Notebook interface
- If blank, check browser console (F12) for errors

## Common Issues

### Blank Screen
**Check:**
1. Are all environment variables set? (Especially SURREAL_URL, SURREAL_USER, SURREAL_PASSWORD)
2. Are ports 8502 and 5055 exposed?
3. Check Railway logs for errors
4. Check browser console (F12) for connection errors

### "Database migration failed"
**Check:**
1. SURREAL_URL is exactly: `ws://localhost:8000/rpc` (not `http://`)
2. SURREAL_USER=root
3. SURREAL_PASSWORD=root
4. Check logs for specific error message

### API not accessible
**Check:**
1. Port 5055 is exposed in Railway
2. Try accessing: `https://your-app.railway.app:5055/docs`
3. Check Railway logs for API startup errors

## Quick Checklist

Before redeploying, verify:
- [ ] SURREAL_URL=ws://localhost:8000/rpc
- [ ] SURREAL_USER=root
- [ ] SURREAL_PASSWORD=root
- [ ] SURREAL_NAMESPACE=open_notebook
- [ ] SURREAL_DATABASE=production
- [ ] OPENAI_API_KEY=your-key-here
- [ ] Ports 8502 and 5055 exposed
- [ ] Volumes configured for /app/data and /mydata

## Expected Logs (Success)

When working correctly, you should see in Railway logs:

```
[program:surrealdb] SurrealDB started
[program:api] Starting API initialization...
[program:api] Current database version: 0
[program:api] Database migrations are pending. Running migrations...
[program:api] Migrations completed successfully. Database is now at version 9
[program:api] API initialization completed successfully
[program:frontend] Next.js server started
```

## Need More Help?

See `RAILWAY_TROUBLESHOOTING.md` for detailed troubleshooting steps.

