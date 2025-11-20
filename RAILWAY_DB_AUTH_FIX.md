# Fix: Database Authentication Error on Railway

## The Problem

The API is failing to connect to SurrealDB because of authentication/configuration issues.

## Root Causes

1. **SURREAL_URL format is wrong** - Should be `ws://localhost:8000/rpc` not `ws://localhost/rpc:8000`
2. **Credentials mismatch** - SurrealDB starts with `root/root` but API might be using different credentials
3. **Missing environment variables** - Required DB variables not set in Railway

## Solution: Set Correct Environment Variables

In Railway → **Variables** tab, add/update these **exact** values:

### Required Database Variables

```
SURREAL_URL=ws://localhost:8000/rpc
SURREAL_USER=root
SURREAL_PASSWORD=root
SURREAL_NAMESPACE=open_notebook
SURREAL_DATABASE=production
```

**Critical Notes:**
- `SURREAL_URL` must be exactly: `ws://localhost:8000/rpc` (not `/rpc:8000`)
- `SURREAL_USER` must be: `root` (matches SurrealDB startup)
- `SURREAL_PASSWORD` must be: `root` (matches SurrealDB startup)

### Why These Values?

Looking at `supervisord.single.conf`:
```bash
surreal start --log trace --user root --pass root rocksdb:/mydata/mydatabase.db
```

SurrealDB is started with `--user root --pass root`, so the API must use the same credentials.

## Step-by-Step Fix

### Step 1: Check Current Variables

In Railway → Variables tab, check if you have:
- `SURREAL_URL` - should be `ws://localhost:8000/rpc`
- `SURREAL_USER` - should be `root`
- `SURREAL_PASSWORD` - should be `root`
- `SURREAL_NAMESPACE` - should be `open_notebook`
- `SURREAL_DATABASE` - should be `production`

### Step 2: Fix/Add Variables

If any are missing or wrong, update them:

1. Click on the variable to edit, or
2. Add new variable if missing
3. Set the exact values shown above

### Step 3: Verify Format

**Common mistakes:**
- ❌ `SURREAL_URL=ws://localhost/rpc:8000` (wrong format)
- ✅ `SURREAL_URL=ws://localhost:8000/rpc` (correct format)

- ❌ `SURREAL_USER=admin` (doesn't match SurrealDB startup)
- ✅ `SURREAL_USER=root` (matches SurrealDB startup)

- ❌ `SURREAL_PASSWORD=password` (doesn't match)
- ✅ `SURREAL_PASSWORD=root` (matches SurrealDB startup)

### Step 4: Redeploy

After fixing variables:
1. Go to **Deployments** tab
2. Click **Redeploy**
3. Watch logs for:
   - ✅ "SurrealDB started"
   - ✅ "Database migrations completed successfully"
   - ✅ "API initialization completed successfully"

## Expected Log Output (Success)

When working correctly, you should see:

```
[program:surrealdb] SurrealDB started successfully
[program:api] Starting API initialization...
[program:api] Current database version: 0
[program:api] Database migrations are pending. Running migrations...
[program:api] Migrations completed successfully. Database is now at version 9
[program:api] API initialization completed successfully
[program:api] Uvicorn running on 0.0.0.0:5055
```

## Error Messages to Look For

If you see these in logs, it's an authentication issue:

- ❌ "Authentication failed"
- ❌ "Invalid credentials"
- ❌ "Connection refused"
- ❌ "CRITICAL: Database migration failed"
- ❌ "Failed to run database migrations"

## Verification Checklist

Before redeploying, verify:

- [ ] `SURREAL_URL=ws://localhost:8000/rpc` (exact format)
- [ ] `SURREAL_USER=root`
- [ ] `SURREAL_PASSWORD=root`
- [ ] `SURREAL_NAMESPACE=open_notebook`
- [ ] `SURREAL_DATABASE=production`
- [ ] No typos or extra spaces
- [ ] All variables are set (none are empty)

## Still Not Working?

### Check Railway Logs For:

1. **SurrealDB Startup**:
   ```
   [program:surrealdb] SurrealDB started
   ```
   If you don't see this, SurrealDB isn't starting.

2. **API Connection Attempt**:
   ```
   [program:api] Starting API initialization...
   ```
   If this fails, check the error message.

3. **Database Connection**:
   Look for connection errors or authentication failures.

### Common Issues:

**Issue**: "Connection refused"
- **Fix**: Check `SURREAL_URL` format - must be `ws://localhost:8000/rpc`
- **Fix**: Make sure SurrealDB started before API (check startup order in logs)

**Issue**: "Authentication failed"
- **Fix**: Verify `SURREAL_USER=root` and `SURREAL_PASSWORD=root`
- **Fix**: These must match what SurrealDB was started with

**Issue**: "Database not found"
- **Fix**: Check `SURREAL_NAMESPACE=open_notebook` and `SURREAL_DATABASE=production`
- **Fix**: These will be created automatically on first connection if they don't exist

## Summary

**The Fix:**
Set these exact values in Railway Variables:
```
SURREAL_URL=ws://localhost:8000/rpc
SURREAL_USER=root
SURREAL_PASSWORD=root
SURREAL_NAMESPACE=open_notebook
SURREAL_DATABASE=production
```

Then redeploy!

