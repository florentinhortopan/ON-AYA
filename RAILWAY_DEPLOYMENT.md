# Quick Start: Deploy to Railway

Railway is perfect for deploying Open Notebook because it supports the full stack (frontend + backend + database) in a single service. This is especially great if you want to use the Open Notebook API from other applications.

## Why Railway for Full-Stack?

✅ **Single Platform**: Everything in one place  
✅ **Private Networking**: Services can communicate privately (faster, more secure)  
✅ **Easy API Access**: Perfect for calling Open Notebook API from other apps  
✅ **Simple Setup**: Automatic Docker detection  

## Quick Deployment

### 1. Deploy to Railway

**Option A: GitHub Integration (Easiest)**

1. Go to [railway.app](https://railway.app)
2. Click "New Project" → "Deploy from GitHub repo"
3. Select your `open-notebook` repository
4. Railway will auto-detect `Dockerfile.single` and start building

**Option B: Railway CLI**

```bash
npm i -g @railway/cli
railway login
cd /path/to/open-notebook
railway init
railway up
```

### 2. Configure Environment Variables

In Railway service settings → Variables, add:

```
SURREAL_URL=ws://localhost:8000/rpc
SURREAL_USER=root
SURREAL_PASSWORD=your_secure_password
SURREAL_NAMESPACE=open_notebook
SURREAL_DATABASE=production
OPENAI_API_KEY=your_key_here
```

### 3. Configure Ports

In Railway service settings → Networking:
- Expose **Port 8502** (Frontend)
- Expose **Port 5055** (API)

### 4. Access Your App

Railway provides a public URL:
- Frontend: `https://your-app.railway.app`
- API: `https://your-app.railway.app:5055` (or separate Railway URL)
- API Docs: `https://your-app.railway.app:5055/docs`

## Using Open Notebook API from Other Apps

### Best Practice: Deploy Both on Railway

If you're building another app that uses Open Notebook API, deploy both on Railway:

**Benefits:**
- Private networking (faster, more secure)
- Same platform = easier management
- Lower latency
- Shared environment variables

**Setup:**
1. Deploy Open Notebook as Service 1
2. Deploy your app as Service 2
3. In your app's environment variables:
   ```
   OPEN_NOTEBOOK_API_URL=${{OpenNotebook.RAILWAY_PRIVATE_DOMAIN}}:5055
   ```

**Example API Client:**
```python
import os
import requests

# Get API URL from environment
api_url = os.getenv('OPEN_NOTEBOOK_API_URL')

# Call the API
response = requests.get(f'{api_url}/api/notebooks')
notebooks = response.json()
```

### Alternative: Use Public API URL

If your other app is elsewhere:

1. Get Open Notebook API URL from Railway (check Networking tab)
2. Call the API:
   ```python
   import requests
   
   API_URL = 'https://your-open-notebook.railway.app:5055'
   response = requests.get(f'{API_URL}/api/notebooks')
   ```

## Data Persistence

**Important**: Configure volumes to persist data:

1. Go to Service Settings → Volumes
2. Create volumes:
   - `/app/data` - Application data
   - `/mydata` - SurrealDB data

Without volumes, data is lost on restart!

## Custom Domain

1. Service Settings → Networking
2. Click "Add Custom Domain"
3. Follow DNS setup instructions

## Cost

- **Hobby**: $5/month + usage (~$10-15/month typical)
- **Pro**: $20/month + usage (~$20-30/month typical)

## Troubleshooting

**Build fails?**
- Check `Dockerfile.single` exists
- Verify all dependencies are in place

**App won't start?**
- Check environment variables
- Verify ports are exposed
- Check logs in Railway dashboard

**API not accessible?**
- Verify port 5055 is exposed
- Check CORS settings if calling from browser
- Use private networking for service-to-service

## Full Documentation

See [docs/deployment/railway.md](docs/deployment/railway.md) for complete guide.

## Need Help?

- Join [Discord](https://discord.gg/37XJPXfz2w)
- Check [GitHub Issues](https://github.com/lfnovo/open-notebook/issues)
- Review [API Docs](https://your-app.railway.app:5055/docs)

