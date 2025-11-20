# Deployment Options Comparison

This document compares different deployment strategies for Open Notebook, especially when you want to use the API from other applications.

## Quick Comparison

| Feature | Railway (Full-Stack) | Vercel + Railway | Vercel + Other Backend |
|---------|---------------------|------------------|------------------------|
| **Frontend** | ✅ Railway | ✅ Vercel | ✅ Vercel |
| **Backend** | ✅ Railway | ✅ Railway | ✅ Other Platform |
| **Database** | ✅ Railway | ✅ Railway | ✅ Other Platform |
| **Single Platform** | ✅ Yes | ❌ No | ❌ No |
| **Private Networking** | ✅ Yes | ❌ No | ❌ No |
| **API Integration** | ✅ Excellent | ⚠️ Good | ⚠️ Good |
| **Setup Complexity** | ⭐ Easy | ⭐⭐ Medium | ⭐⭐⭐ Complex |
| **Cost** | ~$10-20/mo | ~$15-25/mo | Varies |
| **Best For** | API usage, full-stack | Next.js focus | Existing setups |

## Option 1: Railway (Full-Stack) ⭐ Recommended

**Deploy everything on Railway in a single service.**

### Architecture
```
Railway Service
├── Next.js Frontend (Port 8502)
├── FastAPI Backend (Port 5055)
└── SurrealDB (Internal)
```

### Pros
- ✅ **Simplest setup** - one platform, one service
- ✅ **Private networking** - services communicate privately (faster, more secure)
- ✅ **Perfect for API usage** - your other app can use Railway's private networking
- ✅ **No CORS issues** - private network doesn't need CORS
- ✅ **Lower latency** - services on same platform
- ✅ **Easier management** - everything in one place
- ✅ **Service discovery** - Railway provides environment variables for service URLs

### Cons
- ⚠️ Railway-specific (less portable)
- ⚠️ Slightly higher cost than Vercel free tier

### Best For
- ✅ **You want to use Open Notebook API from other apps** ← Perfect for you!
- ✅ Full-stack applications
- ✅ Production deployments
- ✅ When you want everything in one place

### Setup
See [RAILWAY_DEPLOYMENT.md](RAILWAY_DEPLOYMENT.md) or [docs/deployment/railway.md](docs/deployment/railway.md)

### Using API from Other App on Railway
```python
# In your other app's environment variables:
OPEN_NOTEBOOK_API_URL=${{OpenNotebook.RAILWAY_PRIVATE_DOMAIN}}:5055

# In your code:
import os
api_url = os.getenv('OPEN_NOTEBOOK_API_URL')
# Private network - fast and secure!
```

## Option 2: Vercel (Frontend) + Railway (Backend)

**Deploy frontend on Vercel, backend on Railway.**

### Architecture
```
Vercel                    Railway
├── Next.js Frontend ────▶ FastAPI Backend
                          └── SurrealDB
```

### Pros
- ✅ Vercel's excellent Next.js support
- ✅ Vercel free tier available
- ✅ Global CDN for frontend
- ✅ Serverless functions support

### Cons
- ⚠️ Two platforms to manage
- ⚠️ Public API calls (HTTPS only)
- ⚠️ CORS configuration needed
- ⚠️ Slightly higher latency
- ⚠️ More complex setup

### Best For
- Next.js-focused development
- Want Vercel's Next.js optimizations
- Don't mind managing two platforms

### Setup
See [VERCEL_DEPLOYMENT.md](VERCEL_DEPLOYMENT.md) or [docs/deployment/vercel.md](docs/deployment/vercel.md)

### Using API from Other App
```python
# Public API URL
api_url = 'https://your-open-notebook.railway.app:5055'
# Public HTTPS call - requires CORS configuration
```

## Option 3: Both Apps on Railway (Recommended for API Usage)

**Deploy both Open Notebook and your other app on Railway.**

### Architecture
```
Railway Project
├── Open Notebook Service
│   ├── Frontend
│   ├── Backend API
│   └── Database
└── Your App Service
    └── Your code (calls Open Notebook API)
        └── Private Network ──▶ Open Notebook API
```

### Pros
- ✅ **Best for API integration** - private networking
- ✅ **Lowest latency** - same platform
- ✅ **No CORS issues** - private network
- ✅ **Easier management** - both apps in one project
- ✅ **Shared environment variables**
- ✅ **Service discovery** - Railway provides service URLs

### Cons
- ⚠️ Both apps must be on Railway

### Best For
- ✅ **You want to use Open Notebook API from another app** ← Perfect!
- ✅ Production applications
- ✅ Frequent API calls
- ✅ Low latency requirements

### Setup
1. Deploy Open Notebook to Railway (see [RAILWAY_DEPLOYMENT.md](RAILWAY_DEPLOYMENT.md))
2. Deploy your app to the same Railway project
3. In your app's environment variables:
   ```
   OPEN_NOTEBOOK_API_URL=${{OpenNotebook.RAILWAY_PRIVATE_DOMAIN}}:5055
   ```

### Example Integration
```python
# In your other app on Railway
import os
import requests

# Railway provides private domain via environment variable
api_url = os.getenv('OPEN_NOTEBOOK_API_URL')

# Call Open Notebook API via private network
response = requests.get(f'{api_url}/api/notebooks')
notebooks = response.json()
```

## Recommendation for Your Use Case

**Since you want to use Open Notebook API from another app:**

### 🏆 Best Choice: **Railway for Both Apps**

1. **Deploy Open Notebook to Railway** (single service with frontend + backend + database)
2. **Deploy your other app to the same Railway project**
3. **Use Railway's private networking** for API calls

**Why?**
- Private networking = faster, more secure, no CORS
- Same platform = easier management
- Lower latency
- Service discovery built-in

### Alternative: Railway (Open Notebook) + Your App Elsewhere

If your other app must be on a different platform:
- Deploy Open Notebook to Railway
- Call the API via public HTTPS URL
- Configure CORS on Open Notebook backend
- See [docs/deployment/api-integration.md](docs/deployment/api-integration.md)

## Cost Comparison

### Railway (Full-Stack)
- **Hobby Plan**: $5/month + usage
- **Typical Cost**: ~$10-20/month
- **Includes**: Frontend + Backend + Database

### Vercel + Railway
- **Vercel**: Free tier (100GB bandwidth)
- **Railway**: $5/month + usage
- **Typical Cost**: ~$10-15/month
- **Includes**: Frontend (Vercel) + Backend + Database (Railway)

### Railway (Both Apps)
- **Hobby Plan**: $5/month + usage
- **Typical Cost**: ~$15-25/month (depending on usage)
- **Includes**: Open Notebook + Your App

## Next Steps

1. **For Railway deployment**: See [RAILWAY_DEPLOYMENT.md](RAILWAY_DEPLOYMENT.md)
2. **For Vercel deployment**: See [VERCEL_DEPLOYMENT.md](VERCEL_DEPLOYMENT.md)
3. **For API integration**: See [docs/deployment/api-integration.md](docs/deployment/api-integration.md)

## Need Help?

- Join [Discord](https://discord.gg/37XJPXfz2w)
- Check [GitHub Issues](https://github.com/lfnovo/open-notebook/issues)
- Review deployment guides in `docs/deployment/`

