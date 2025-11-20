# Quick Start: Deploy to Vercel

This is a quick reference guide. For detailed instructions, see [docs/deployment/vercel.md](docs/deployment/vercel.md).

## ⚠️ Important: Architecture Limitation

**Vercel can only host the Next.js frontend.** The Python FastAPI backend and SurrealDB database must be hosted separately on platforms like Railway, Render, or Fly.io.

## 💡 Alternative: Railway for Full-Stack

**Want to deploy everything together?** Consider using [Railway](https://railway.app) instead, which can host the entire application (frontend + backend + database) in a single service. This is especially great if you want to use the Open Notebook API from other applications. See [RAILWAY_DEPLOYMENT.md](RAILWAY_DEPLOYMENT.md) for details.

## Quick Deployment Steps

### 1. Deploy Backend First

Choose one platform:

**Railway (Easiest)**:
1. Go to [railway.app](https://railway.app)
2. New Project → Deploy from GitHub
3. Select your repository
4. Add environment variables:
   ```
   SURREAL_URL=ws://localhost:8000/rpc
   SURREAL_USER=root
   SURREAL_PASSWORD=your_secure_password
   SURREAL_NAMESPACE=open_notebook
   SURREAL_DATABASE=production
   OPENAI_API_KEY=your_key_here
   ```
5. Note your backend URL (e.g., `https://your-app.railway.app`)

**Render**:
1. Go to [render.com](https://render.com)
2. New Web Service → Connect GitHub
3. Use Docker deployment
4. Add same environment variables as above

### 2. Update Backend CORS

Edit `api/main.py` to allow your Vercel domain:

```python
app.add_middleware(
    CORSMiddleware,
    allow_origins=[
        "https://your-app.vercel.app",  # Your Vercel domain
        "https://your-custom-domain.com",  # If using custom domain
        "*"  # ⚠️ Temporary for testing - restrict in production
    ],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

### 3. Deploy Frontend to Vercel

**Option A: Using Vercel CLI**

```bash
# Install Vercel CLI
npm i -g vercel

# Navigate to project root
cd /path/to/open-notebook

# Login
vercel login

# Deploy
vercel

# When prompted:
# - Set up and deploy? Yes
# - Directory? ./frontend
# - Override settings? No

# Set environment variable
vercel env add NEXT_PUBLIC_API_URL production
# Enter your backend URL when prompted

# Deploy to production
vercel --prod
```

**Option B: Using Vercel Dashboard**

1. Go to [vercel.com/new](https://vercel.com/new)
2. Import your GitHub repository
3. Configure:
   - **Root Directory**: `frontend`
   - **Framework**: Next.js (auto-detected)
4. Go to Settings → Environment Variables
5. Add `NEXT_PUBLIC_API_URL` = your backend URL
6. Deploy

### 4. Verify

1. Visit your Vercel URL
2. Check browser console for errors
3. Test the application

## Environment Variables Summary

### Vercel (Frontend)
- `NEXT_PUBLIC_API_URL`: Your backend URL (e.g., `https://your-backend.railway.app`)

### Backend Platform
- `SURREAL_URL`: `ws://localhost:8000/rpc`
- `SURREAL_USER`: `root`
- `SURREAL_PASSWORD`: Your secure password
- `SURREAL_NAMESPACE`: `open_notebook`
- `SURREAL_DATABASE`: `production`
- `OPENAI_API_KEY`: Your OpenAI API key
- (Add other API keys as needed)

## Troubleshooting

**Frontend can't connect to backend?**
- Verify `NEXT_PUBLIC_API_URL` is set correctly
- Check backend CORS allows your Vercel domain
- Ensure backend is accessible (try URL in browser)

**CORS errors?**
- Update backend CORS in `api/main.py`
- Redeploy backend after CORS changes

**Still having issues?**
- See [full deployment guide](docs/deployment/vercel.md)
- Join [Discord](https://discord.gg/37XJPXfz2w)
- Check [GitHub Issues](https://github.com/lfnovo/open-notebook/issues)

## Cost Estimate

- **Vercel**: Free tier (100GB bandwidth/month)
- **Backend hosting**: ~$5-20/month (Railway/Render/Fly.io)
- **Total**: ~$5-20/month minimum

## Next Steps

- Set up custom domain (optional)
- Configure password protection
- Set up monitoring
- Review [production checklist](docs/deployment/vercel.md#production-checklist)

