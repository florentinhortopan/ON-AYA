# Railway Troubleshooting: Blank Screen / No Notebooks

If you're seeing a blank screen with no notebooks, the database likely isn't connected or migrations haven't run. Follow these steps:

## Step 1: Check Railway Logs

1. Go to your Railway service dashboard
2. Click on the **"Deployments"** tab
3. Click on the latest deployment
4. Check the logs for:
   - ✅ "Database migrations completed successfully"
   - ❌ "CRITICAL: Database migration failed"
   - ❌ Connection errors

## Step 2: Configure Environment Variables

In Railway, go to your service → **Variables** tab and add these **required** variables:

### Database Configuration (Required)
```
SURREAL_URL=ws://localhost:8000/rpc
SURREAL_USER=root
SURREAL_PASSWORD=root
SURREAL_NAMESPACE=open_notebook
SURREAL_DATABASE=production
```

### AI Provider (At least one required)
```
OPENAI_API_KEY=your_openai_key_here
```

### Optional: Additional Configuration
```
API_URL=https://your-app.railway.app
# Only needed if you want to override auto-detection
```

## Step 3: Configure Ports

1. Go to your Railway service → **Settings** → **Networking**
2. Make sure these ports are exposed:
   - **Port 8502** (Frontend)
   - **Port 5055** (API)

Railway should auto-detect these from the Dockerfile, but verify they're there.

## Step 4: Configure Volumes (Data Persistence)

**Important**: Without volumes, your data will be lost on restart!

1. Go to **Settings** → **Volumes**
2. Create two volumes:
   - **Path**: `/app/data` → For application data
   - **Path**: `/mydata` → For SurrealDB database files

## Step 5: Verify Service Startup Order

The single container uses supervisord to start services in order:
1. SurrealDB (starts first)
2. API (waits for DB, runs migrations)
3. Worker
4. Frontend

Check logs to ensure this order is followed.

## Step 6: Check API Health

1. Visit: `https://your-app.railway.app:5055/health`
   - Should return: `{"status":"healthy"}`

2. Visit: `https://your-app.railway.app:5055/api/notebooks`
   - Should return: `[]` (empty array if no notebooks) or a list of notebooks
   - If you get an error, the API isn't connecting to the database

## Step 7: Check Frontend Configuration

1. Visit: `https://your-app.railway.app/config`
   - Should return JSON with `apiUrl`
   - Check if it's pointing to the correct API URL

2. Open browser console (F12) and check for errors:
   - Connection errors
   - CORS errors
   - API URL misconfiguration

## Common Issues & Solutions

### Issue: "CRITICAL: Database migration failed"

**Solution:**
- Check that `SURREAL_URL=ws://localhost:8000/rpc` is set
- Verify `SURREAL_USER=root` and `SURREAL_PASSWORD=root` are set
- Check logs for specific error messages
- Ensure SurrealDB started before the API (check startup order in logs)

### Issue: Blank screen, no errors in console

**Solution:**
- Check that port 5055 is exposed and accessible
- Verify `API_URL` environment variable (or let it auto-detect)
- Check Railway logs for API startup errors
- Try accessing API directly: `https://your-app.railway.app:5055/docs`

### Issue: "Unable to connect to server"

**Solution:**
- Set `API_URL=https://your-app.railway.app:5055` in Railway variables
- Or use Railway's auto-generated URL for port 5055
- Check that both ports (8502 and 5055) are exposed

### Issue: Database connection timeout

**Solution:**
- Verify `SURREAL_URL=ws://localhost:8000/rpc` (not `http://`)
- Check that SurrealDB service started successfully
- Look for SurrealDB startup messages in logs

### Issue: Migrations not running

**Solution:**
- Migrations run automatically on API startup
- Check logs for "Starting API initialization..."
- If migrations fail, the API won't start (fail-fast)
- Check for specific migration errors in logs

## Quick Verification Checklist

- [ ] All environment variables set (SURREAL_URL, SURREAL_USER, SURREAL_PASSWORD, etc.)
- [ ] At least one AI provider key set (OPENAI_API_KEY)
- [ ] Ports 8502 and 5055 exposed
- [ ] Volumes configured for `/app/data` and `/mydata`
- [ ] API health endpoint returns `{"status":"healthy"}`
- [ ] API `/api/notebooks` endpoint accessible
- [ ] Frontend `/config` endpoint returns correct API URL
- [ ] No errors in Railway deployment logs
- [ ] No errors in browser console

## Still Having Issues?

1. **Check Railway Logs**: Look for specific error messages
2. **Test API Directly**: Visit `https://your-app.railway.app:5055/docs`
3. **Check Browser Console**: Look for connection errors
4. **Verify Environment Variables**: Double-check all variables are set correctly
5. **Redeploy**: Sometimes a fresh deployment helps

## Expected Log Output

When everything works, you should see in Railway logs:

```
[program:surrealdb] SurrealDB started successfully
[program:api] Starting API initialization...
[program:api] Current database version: X
[program:api] Migrations completed successfully. Database is now at version Y
[program:api] API initialization completed successfully
[program:frontend] Next.js server started on port 8502
```

If you see errors instead, that's where to focus your troubleshooting.

