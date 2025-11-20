# Deploying Open Notebook to Railway

Railway is an excellent choice for deploying Open Notebook because it supports full-stack deployments (frontend + backend + database) in a single service. This guide covers deploying the complete application to Railway.

## Why Railway?

✅ **Single Platform**: Deploy frontend, backend, and database together  
✅ **Easy Setup**: Automatic Docker detection  
✅ **Private Networking**: Services on Railway can communicate privately  
✅ **Simple Scaling**: Easy to scale up as needed  
✅ **Good for API Usage**: Perfect if you want to call Open Notebook API from other apps  

## Architecture on Railway

```
┌─────────────────────────────────────────────────────────┐
│  Railway Service (Single Container)                     │
│  ┌───────────────────────────────────────────────────┐  │
│  │  Next.js Frontend (Port 8502)                     │  │
│  │  FastAPI Backend (Port 5055)                      │  │
│  │  SurrealDB (Internal, Port 8000)                  │  │
│  └───────────────────────────────────────────────────┘  │
│                                                          │
│  Public URLs:                                            │
│  - Frontend: https://your-app.railway.app               │
│  - API: https://your-app.railway.app:5055               │
│    (or use Railway's port mapping)                       │
└─────────────────────────────────────────────────────────┘
```

## Prerequisites

1. **Railway Account**: Sign up at [railway.app](https://railway.app)
2. **GitHub Account**: Railway deploys from GitHub repositories
3. **API Keys**: Your AI provider API keys (OpenAI, Anthropic, etc.)

## Step 1: Deploy to Railway

### Option A: Deploy from GitHub (Recommended)

1. **Go to Railway Dashboard**
   - Visit [railway.app](https://railway.app)
   - Click "New Project"

2. **Deploy from GitHub**
   - Select "Deploy from GitHub repo"
   - Authorize Railway to access your repositories
   - Select your `open-notebook` repository

3. **Railway Auto-Detection**
   - Railway will automatically detect the `Dockerfile.single`
   - It will create a new service

4. **Configure Service**
   - Railway will start building automatically
   - Wait for the build to complete

### Option B: Deploy with Railway CLI

1. **Install Railway CLI**:
   ```bash
   npm i -g @railway/cli
   ```

2. **Login**:
   ```bash
   railway login
   ```

3. **Initialize Project**:
   ```bash
   cd /path/to/open-notebook
   railway init
   ```

4. **Deploy**:
   ```bash
   railway up
   ```

## Step 2: Configure Environment Variables

In your Railway service settings, add these environment variables:

### Required Variables

```
SURREAL_URL=ws://localhost:8000/rpc
SURREAL_USER=root
SURREAL_PASSWORD=your_secure_password_here
SURREAL_NAMESPACE=open_notebook
SURREAL_DATABASE=production
```

### AI Provider Keys (Add as needed)

```
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_anthropic_key_here
# Add other provider keys as needed
```

### Optional Configuration

```
API_URL=https://your-app.railway.app
# Only needed if you want to override auto-detection
```

**How to Add Variables:**
1. Go to your Railway service
2. Click on "Variables" tab
3. Click "New Variable"
4. Add each variable one by one

## Step 3: Configure Ports

Railway needs to know which ports to expose:

1. **Go to Service Settings**
2. **Navigate to "Networking"**
3. **Add Ports**:
   - **Port 8502**: For Next.js frontend
   - **Port 5055**: For FastAPI backend API

Railway will automatically generate public URLs for these ports.

## Step 4: Deploy and Verify

1. **Trigger Deployment**
   - If using GitHub, push a commit or click "Redeploy"
   - If using CLI, run `railway up`

2. **Check Logs**
   - Go to "Deployments" tab
   - Click on the latest deployment
   - Check logs for any errors

3. **Access Your Application**
   - Railway provides a public URL (e.g., `https://your-app.railway.app`)
   - The frontend should be accessible at this URL
   - The API will be at the same domain with port 5055, or Railway may provide a separate URL

## Step 5: Using Open Notebook API from Other Apps

If you want to call the Open Notebook API from another application, here are the best practices:

### Option 1: Deploy Both Apps on Railway (Recommended)

**Benefits:**
- Private networking between services (faster, more secure)
- Same platform = easier management
- Lower latency
- Shared environment variables

**Setup:**
1. Deploy Open Notebook as Service 1
2. Deploy your other app as Service 2
3. Both services can communicate via Railway's private network
4. Use Railway's service discovery or environment variables

**Example API Call from Your App:**
```python
# In your other app on Railway
import os
import requests

# Railway provides service URLs via environment variables
# Or use the public URL
open_notebook_api = os.getenv('OPEN_NOTEBOOK_API_URL', 'https://your-open-notebook.railway.app:5055')

response = requests.get(f'{open_notebook_api}/api/notebooks')
notebooks = response.json()
```

### Option 2: Use Public API URL

If your other app is deployed elsewhere:

1. **Get Open Notebook API URL from Railway**
   - Go to your Open Notebook service
   - Check the "Networking" tab
   - Copy the public URL for port 5055

2. **Call the API from Your App**:
   ```python
   import requests

   API_URL = 'https://your-open-notebook.railway.app:5055'
   
   # Example: Get all notebooks
   response = requests.get(f'{API_URL}/api/notebooks')
   notebooks = response.json()
   
   # Example: Create a notebook
   response = requests.post(
       f'{API_URL}/api/notebooks',
       json={'name': 'My Notebook', 'description': 'Test notebook'}
   )
   ```

3. **Handle Authentication** (if password protection is enabled):
   ```python
   # If password protection is enabled, you'll need to authenticate
   # Check the API docs at https://your-open-notebook.railway.app:5055/docs
   ```

### Option 3: Railway Service References

Railway supports service references for private networking:

1. **In Your Other App's Environment Variables**:
   ```
   OPEN_NOTEBOOK_API_URL=${{OpenNotebook.RAILWAY_PRIVATE_DOMAIN}}:5055
   ```

2. **This allows private communication** between services without exposing the API publicly.

## API Documentation

Once deployed, access the interactive API documentation:

- **Swagger UI**: `https://your-app.railway.app:5055/docs`
- **ReDoc**: `https://your-app.railway.app:5055/redoc`
- **OpenAPI JSON**: `https://your-app.railway.app:5055/openapi.json`

## Custom Domain Setup

1. **Go to Service Settings**
2. **Navigate to "Networking"**
3. **Click "Generate Domain"** or **"Add Custom Domain"**
4. **Follow Railway's instructions** to configure DNS

## Data Persistence

Railway provides persistent volumes for your data:

1. **Go to Service Settings**
2. **Navigate to "Volumes"**
3. **Create Volumes**:
   - `/app/data` - For application data
   - `/mydata` - For SurrealDB data

**Important**: Without volumes, data will be lost when the service restarts.

## Scaling

Railway makes it easy to scale:

1. **Go to Service Settings**
2. **Navigate to "Scaling"**
3. **Adjust Resources**:
   - CPU: Increase for better performance
   - Memory: Increase for larger datasets
   - Instances: Scale horizontally (if needed)

## Monitoring

Railway provides built-in monitoring:

- **Metrics**: CPU, Memory, Network usage
- **Logs**: Real-time application logs
- **Deployments**: Deployment history and status

## Troubleshooting

### Build Fails

**Symptoms**: Build errors in Railway logs

**Solutions**:
1. Check Dockerfile.single exists
2. Verify all dependencies are in pyproject.toml and package.json
3. Check build logs for specific errors
4. Ensure Node.js and Python versions are compatible

### Application Won't Start

**Symptoms**: Service keeps restarting

**Solutions**:
1. Check environment variables are set correctly
2. Verify SurrealDB connection settings
3. Check application logs in Railway dashboard
4. Ensure ports 8502 and 5055 are exposed

### API Not Accessible

**Symptoms**: Can't reach API from other services

**Solutions**:
1. Verify port 5055 is exposed in Railway
2. Check CORS settings if calling from browser
3. Use Railway's private networking for service-to-service calls
4. Verify API_URL environment variable is set correctly

### Database Connection Issues

**Symptoms**: Database errors in logs

**Solutions**:
1. Verify SURREAL_URL is `ws://localhost:8000/rpc` (for single container)
2. Check SURREAL_USER and SURREAL_PASSWORD are set
3. Ensure volumes are mounted for data persistence
4. Check SurrealDB logs in application logs

## Cost Considerations

Railway pricing:
- **Hobby Plan**: $5/month + usage
- **Pro Plan**: $20/month + usage
- **Team Plan**: Custom pricing

**Estimated Monthly Cost**:
- Small deployment: ~$10-15/month
- Medium usage: ~$20-30/month
- High usage: ~$50+/month

## Production Checklist

- [ ] Environment variables configured
- [ ] Ports 8502 and 5055 exposed
- [ ] Data volumes configured for persistence
- [ ] Custom domain set up (optional)
- [ ] API documentation accessible
- [ ] Monitoring enabled
- [ ] Backup strategy in place
- [ ] Password protection enabled (if needed)
- [ ] CORS configured for API access (if needed)
- [ ] Tested API calls from other apps

## Best Practices for API Usage

### 1. Use Environment Variables

Store API URLs in environment variables, not hardcoded:

```python
# Good
api_url = os.getenv('OPEN_NOTEBOOK_API_URL')

# Bad
api_url = 'https://hardcoded-url.com'
```

### 2. Implement Retry Logic

Network calls can fail, implement retries:

```python
import requests
from requests.adapters import HTTPAdapter
from urllib3.util.retry import Retry

session = requests.Session()
retry_strategy = Retry(
    total=3,
    backoff_factor=1,
    status_forcelist=[429, 500, 502, 503, 504],
)
adapter = HTTPAdapter(max_retries=retry_strategy)
session.mount("http://", adapter)
session.mount("https://", adapter)
```

### 3. Handle Rate Limiting

Respect rate limits and implement exponential backoff:

```python
import time

def call_api_with_backoff(url, max_retries=5):
    for attempt in range(max_retries):
        try:
            response = requests.get(url)
            if response.status_code == 429:
                wait_time = 2 ** attempt
                time.sleep(wait_time)
                continue
            return response
        except Exception as e:
            if attempt == max_retries - 1:
                raise
            time.sleep(2 ** attempt)
```

### 4. Use Connection Pooling

For frequent API calls, use connection pooling:

```python
from requests.adapters import HTTPAdapter
from urllib3.util.retry import Retry

session = requests.Session()
adapter = HTTPAdapter(
    pool_connections=10,
    pool_maxsize=20,
    max_retries=Retry(total=3)
)
session.mount('https://', adapter)
```

## Example: Integrating Open Notebook API

Here's a complete example of calling the Open Notebook API from another Python app:

```python
import os
import requests
from typing import List, Dict, Optional

class OpenNotebookClient:
    def __init__(self, api_url: str):
        self.api_url = api_url.rstrip('/')
        self.session = requests.Session()
        
    def get_notebooks(self) -> List[Dict]:
        """Get all notebooks"""
        response = self.session.get(f'{self.api_url}/api/notebooks')
        response.raise_for_status()
        return response.json()
    
    def create_notebook(self, name: str, description: str = "") -> Dict:
        """Create a new notebook"""
        response = self.session.post(
            f'{self.api_url}/api/notebooks',
            json={'name': name, 'description': description}
        )
        response.raise_for_status()
        return response.json()
    
    def get_sources(self, notebook_id: str) -> List[Dict]:
        """Get sources for a notebook"""
        response = self.session.get(
            f'{self.api_url}/api/notebooks/{notebook_id}/sources'
        )
        response.raise_for_status()
        return response.json()
    
    def chat(self, notebook_id: str, message: str, model: str = "gpt-4") -> Dict:
        """Send a chat message"""
        response = self.session.post(
            f'{self.api_url}/api/notebooks/{notebook_id}/chat',
            json={'message': message, 'model': model}
        )
        response.raise_for_status()
        return response.json()

# Usage
if __name__ == '__main__':
    # Get API URL from environment (set in Railway)
    api_url = os.getenv('OPEN_NOTEBOOK_API_URL', 'https://your-app.railway.app:5055')
    
    client = OpenNotebookClient(api_url)
    
    # Get all notebooks
    notebooks = client.get_notebooks()
    print(f"Found {len(notebooks)} notebooks")
    
    # Create a new notebook
    notebook = client.create_notebook("My Research", "Research notes")
    print(f"Created notebook: {notebook['id']}")
    
    # Send a chat message
    response = client.chat(notebook['id'], "What are the main topics?")
    print(f"Chat response: {response['message']}")
```

## Need Help?

- Check the [main deployment guide](../deployment/index.md)
- Review [API documentation](http://localhost:5055/docs) (replace with your URL)
- Join our [Discord server](https://discord.gg/37XJPXfz2w)
- Open an issue on [GitHub](https://github.com/lfnovo/open-notebook/issues)

