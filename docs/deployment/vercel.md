# Deploying Open Notebook to Vercel

This guide will help you deploy the Open Notebook frontend to Vercel. **Important**: Vercel can only host the Next.js frontend. The Python FastAPI backend and SurrealDB database must be hosted separately.

## Architecture Overview

```
┌─────────────────────────────────────────────────────────┐
│  Vercel (Frontend)                                      │
│  - Next.js Application                                  │
│  - Static assets and serverless functions               │
└────────────────┬────────────────────────────────────────┘
                 │
                 │ API Requests (HTTPS)
                 ▼
┌─────────────────────────────────────────────────────────┐
│  External Backend Hosting                               │
│  - FastAPI Backend (Python)                             │
│  - SurrealDB Database                                   │
│  Options: Railway, Render, Fly.io, DigitalOcean, etc.    │
└─────────────────────────────────────────────────────────┘
```

## Prerequisites

1. **Vercel Account**: Sign up at [vercel.com](https://vercel.com)
2. **Backend Hosting**: Choose a platform for your backend:
   - **Railway** (Recommended): Easy setup, good for Python apps
   - **Render**: Free tier available, supports Docker
   - **Fly.io**: Good performance, global distribution
   - **DigitalOcean App Platform**: Simple deployment
   - **Your own VPS**: Full control, requires more setup

3. **Database Hosting**: 
   - SurrealDB can run alongside your backend on the same platform
   - Or use a managed SurrealDB service if available

## Step 1: Deploy the Backend

First, you need to deploy your FastAPI backend and SurrealDB. Here are platform-specific guides:

### Option A: Railway (Recommended)

1. Go to [railway.app](https://railway.app) and create an account
2. Create a new project
3. Add a new service from GitHub (connect your repository)
4. Configure environment variables:
   ```
   SURREAL_URL=ws://localhost:8000/rpc
   SURREAL_USER=root
   SURREAL_PASSWORD=your_secure_password
   SURREAL_NAMESPACE=open_notebook
   SURREAL_DATABASE=production
   OPENAI_API_KEY=your_key_here
   ```
5. Railway will automatically detect your Dockerfile and deploy
6. Note your backend URL (e.g., `https://your-app.railway.app`)

### Option B: Render

1. Go to [render.com](https://render.com) and create an account
2. Create a new "Web Service"
3. Connect your GitHub repository
4. Configure:
   - **Build Command**: (leave empty, uses Dockerfile)
   - **Start Command**: (auto-detected from Dockerfile)
   - **Environment**: Docker
5. Add environment variables (same as Railway above)
6. Deploy and note your backend URL

### Option C: Fly.io

1. Install Fly CLI: `curl -L https://fly.io/install.sh | sh`
2. Authenticate: `fly auth login`
3. Create app: `fly launch`
4. Set secrets:
   ```bash
   fly secrets set SURREAL_URL=ws://localhost:8000/rpc
   fly secrets set SURREAL_USER=root
   fly secrets set SURREAL_PASSWORD=your_secure_password
   fly secrets set SURREAL_NAMESPACE=open_notebook
   fly secrets set SURREAL_DATABASE=production
   fly secrets set OPENAI_API_KEY=your_key_here
   ```
5. Deploy: `fly deploy`
6. Note your backend URL

## Step 2: Configure CORS on Backend

Your backend needs to allow requests from your Vercel domain. Update your backend's CORS settings:

In `api/main.py`, ensure CORS allows your Vercel domain:

```python
app.add_middleware(
    CORSMiddleware,
    allow_origins=[
        "https://your-app.vercel.app",
        "https://your-custom-domain.com",
        # Add other allowed origins
    ],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

Or for development/testing, you can temporarily allow all origins:

```python
allow_origins=["*"]  # ⚠️ Only for testing - restrict in production
```

## Step 3: Deploy Frontend to Vercel

### Method 1: Vercel CLI (Recommended)

1. **Install Vercel CLI**:
   ```bash
   npm i -g vercel
   ```

2. **Navigate to project root**:
   ```bash
   cd /path/to/open-notebook
   ```

3. **Login to Vercel**:
   ```bash
   vercel login
   ```

4. **Deploy**:
   ```bash
   vercel
   ```
   
   When prompted:
   - Set up and deploy? **Yes**
   - Which scope? Choose your account
   - Link to existing project? **No** (first time) or **Yes** (updates)
   - Project name? `open-notebook` (or your choice)
   - Directory? `./frontend`
   - Override settings? **No**

5. **Set Environment Variables**:
   ```bash
   vercel env add NEXT_PUBLIC_API_URL
   ```
   Enter your backend URL when prompted (e.g., `https://your-backend.railway.app`)

   Or set it via the Vercel dashboard:
   - Go to your project settings
   - Navigate to "Environment Variables"
   - Add `NEXT_PUBLIC_API_URL` with your backend URL

6. **Redeploy** to apply environment variables:
   ```bash
   vercel --prod
   ```

### Method 2: Vercel Dashboard

1. Go to [vercel.com/new](https://vercel.com/new)
2. Import your GitHub repository
3. Configure project:
   - **Framework Preset**: Next.js
   - **Root Directory**: `frontend`
   - **Build Command**: `npm run build` (or leave default)
   - **Output Directory**: `.next` (or leave default)
   - **Install Command**: `npm install` (or leave default)

4. **Add Environment Variable**:
   - Go to Project Settings → Environment Variables
   - Add `NEXT_PUBLIC_API_URL` with your backend URL
   - Example: `https://your-backend.railway.app`

5. **Deploy**: Click "Deploy"

## Step 4: Verify Deployment

1. Visit your Vercel deployment URL (e.g., `https://open-notebook.vercel.app`)
2. Check the browser console for any connection errors
3. Test the application:
   - Try logging in (if password protection is enabled)
   - Create a notebook
   - Upload a source
   - Test the chat functionality

## Troubleshooting

### Frontend can't connect to backend

**Symptoms**: Blank page, connection errors in console

**Solutions**:
1. Verify `NEXT_PUBLIC_API_URL` is set correctly in Vercel
2. Check backend CORS settings allow your Vercel domain
3. Ensure backend is accessible (try accessing backend URL directly)
4. Check browser console for specific error messages

### CORS Errors

**Symptoms**: `Access-Control-Allow-Origin` errors in console

**Solutions**:
1. Update backend CORS to include your Vercel domain
2. Ensure `allow_credentials=True` in CORS middleware
3. Check that backend is returning proper CORS headers

### Environment Variables Not Working

**Symptoms**: Still connecting to localhost or wrong URL

**Solutions**:
1. Redeploy after setting environment variables
2. Use `NEXT_PUBLIC_API_URL` (not `API_URL`) for client-side access
3. Check Vercel dashboard → Settings → Environment Variables
4. Clear browser cache and hard refresh

### Backend Timeout Issues

**Symptoms**: Requests timing out, especially for long operations

**Solutions**:
1. Increase timeout in your backend hosting platform
2. For Vercel: Consider using Vercel Functions for API routes (requires refactoring)
3. For long operations: Implement async job processing

## Custom Domain Setup

1. In Vercel dashboard, go to your project → Settings → Domains
2. Add your custom domain
3. Update backend CORS to include your custom domain
4. Update `NEXT_PUBLIC_API_URL` if needed (usually not necessary)

## Production Checklist

- [ ] Backend deployed and accessible
- [ ] Database configured and running
- [ ] CORS configured on backend for Vercel domain
- [ ] Environment variables set in Vercel
- [ ] `NEXT_PUBLIC_API_URL` points to your backend
- [ ] Password protection enabled (if needed)
- [ ] Custom domain configured (optional)
- [ ] SSL/HTTPS enabled (automatic on Vercel)
- [ ] Tested all major features
- [ ] Monitoring/logging set up

## Cost Considerations

### Vercel (Frontend)
- **Free Tier**: 
  - 100GB bandwidth/month
  - Unlimited requests
  - Perfect for most use cases
- **Pro Tier**: $20/month for more bandwidth and features

### Backend Hosting
- **Railway**: ~$5-20/month depending on usage
- **Render**: Free tier available, then ~$7/month
- **Fly.io**: Pay-as-you-go, typically $5-15/month
- **DigitalOcean**: ~$5-12/month

### Total Estimated Cost
- **Minimum**: ~$5-10/month (free Vercel + cheapest backend)
- **Recommended**: ~$15-30/month for reliable production setup

## Alternative: Full-Stack Deployment

If you prefer a simpler setup without splitting frontend and backend:

Consider deploying the entire application (frontend + backend + database) to:
- **Railway**: Supports full Docker Compose deployments
- **Render**: Can deploy multiple services
- **Fly.io**: Multi-service deployments
- **DigitalOcean App Platform**: Full-stack support

This keeps everything together but may have different scaling characteristics.

## Need Help?

- Check the [main deployment guide](../deployment/index.md)
- Join our [Discord server](https://discord.gg/37XJPXfz2w)
- Open an issue on [GitHub](https://github.com/lfnovo/open-notebook/issues)

