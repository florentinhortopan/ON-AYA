# Integrating Open Notebook API with Other Applications

This guide explains how to use the Open Notebook API from other applications, including best practices for deployment and architecture.

## Architecture Options

### Option 1: Both Apps on Railway (Recommended)

**Best for**: Production applications, frequent API calls, low latency requirements

```
┌─────────────────────────────────────────────────────────┐
│  Railway Project                                         │
│  ┌──────────────────────┐  ┌──────────────────────┐     │
│  │  Open Notebook       │  │  Your App           │     │
│  │  (Service 1)         │  │  (Service 2)        │     │
│  │  - Frontend          │  │  - Your code       │     │
│  │  - Backend API       │  │  - Calls ON API    │     │
│  │  - Database           │  │                     │     │
│  └──────────────────────┘  └──────────────────────┘     │
│         │                            │                   │
│         └──────────┬─────────────────┘                   │
│                    │                                     │
│              Private Network                            │
│         (Fast, Secure, No CORS)                         │
└─────────────────────────────────────────────────────────┘
```

**Benefits:**
- ✅ Private networking (faster, more secure)
- ✅ No CORS issues
- ✅ Lower latency
- ✅ Easier management (same platform)
- ✅ Shared environment variables
- ✅ Service discovery via Railway variables

**Setup:**
1. Deploy Open Notebook to Railway (see [railway.md](railway.md))
2. Deploy your app to the same Railway project
3. In your app's environment variables:
   ```
   OPEN_NOTEBOOK_API_URL=${{OpenNotebook.RAILWAY_PRIVATE_DOMAIN}}:5055
   ```

### Option 2: Open Notebook on Railway, Your App on Vercel

**Best for**: Next.js frontend apps, serverless functions

```
┌──────────────────────┐         ┌──────────────────────┐
│  Vercel              │         │  Railway             │
│  Your App            │────────▶│  Open Notebook       │
│  (Next.js)           │  HTTPS  │  - Backend API       │
│                      │         │  - Database          │
└──────────────────────┘         └──────────────────────┘
```

**Benefits:**
- ✅ Vercel's excellent Next.js support
- ✅ Serverless functions
- ✅ Global CDN
- ✅ Free tier available

**Considerations:**
- ⚠️ Public API calls (HTTPS)
- ⚠️ CORS configuration needed
- ⚠️ Slightly higher latency
- ⚠️ Need to handle API authentication

**Setup:**
1. Deploy Open Notebook to Railway
2. Deploy your app to Vercel
3. In Vercel environment variables:
   ```
   NEXT_PUBLIC_OPEN_NOTEBOOK_API_URL=https://your-open-notebook.railway.app:5055
   ```
4. Configure CORS on Open Notebook backend (see below)

### Option 3: Both Apps on Different Platforms

**Best for**: Existing deployments, specific platform requirements

**Considerations:**
- Public API calls only
- CORS must be configured
- Higher latency
- More complex setup

## CORS Configuration

If calling the API from a browser or different domain, configure CORS in `api/main.py`:

```python
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=[
        "https://your-app.vercel.app",
        "https://your-custom-domain.com",
        # Add all allowed origins
    ],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

**For development/testing**, you can temporarily allow all origins:
```python
allow_origins=["*"]  # ⚠️ Only for testing - restrict in production
```

## API Client Examples

### Python Client

```python
import os
import requests
from typing import List, Dict, Optional
from requests.adapters import HTTPAdapter
from urllib3.util.retry import Retry

class OpenNotebookClient:
    """Client for interacting with Open Notebook API"""
    
    def __init__(self, api_url: str, timeout: int = 300):
        self.api_url = api_url.rstrip('/')
        self.session = requests.Session()
        
        # Configure retry strategy
        retry_strategy = Retry(
            total=3,
            backoff_factor=1,
            status_forcelist=[429, 500, 502, 503, 504],
        )
        adapter = HTTPAdapter(
            max_retries=retry_strategy,
            pool_connections=10,
            pool_maxsize=20
        )
        self.session.mount("http://", adapter)
        self.session.mount("https://", adapter)
        self.timeout = timeout
    
    def get_notebooks(self) -> List[Dict]:
        """Get all notebooks"""
        response = self.session.get(
            f'{self.api_url}/api/notebooks',
            timeout=self.timeout
        )
        response.raise_for_status()
        return response.json()
    
    def get_notebook(self, notebook_id: str) -> Dict:
        """Get a specific notebook"""
        response = self.session.get(
            f'{self.api_url}/api/notebooks/{notebook_id}',
            timeout=self.timeout
        )
        response.raise_for_status()
        return response.json()
    
    def create_notebook(self, name: str, description: str = "") -> Dict:
        """Create a new notebook"""
        response = self.session.post(
            f'{self.api_url}/api/notebooks',
            json={'name': name, 'description': description},
            timeout=self.timeout
        )
        response.raise_for_status()
        return response.json()
    
    def get_sources(self, notebook_id: str) -> List[Dict]:
        """Get sources for a notebook"""
        response = self.session.get(
            f'{self.api_url}/api/notebooks/{notebook_id}/sources',
            timeout=self.timeout
        )
        response.raise_for_status()
        return response.json()
    
    def upload_source(self, notebook_id: str, file_path: str, source_type: str = "file") -> Dict:
        """Upload a source file"""
        with open(file_path, 'rb') as f:
            files = {'file': f}
            data = {'notebook_id': notebook_id, 'type': source_type}
            response = self.session.post(
                f'{self.api_url}/api/sources',
                files=files,
                data=data,
                timeout=self.timeout
            )
        response.raise_for_status()
        return response.json()
    
    def chat(self, notebook_id: str, message: str, model: str = "gpt-4") -> Dict:
        """Send a chat message to a notebook"""
        response = self.session.post(
            f'{self.api_url}/api/notebooks/{notebook_id}/chat',
            json={'message': message, 'model': model},
            timeout=self.timeout
        )
        response.raise_for_status()
        return response.json()
    
    def search(self, notebook_id: str, query: str, limit: int = 10) -> List[Dict]:
        """Search within a notebook"""
        response = self.session.get(
            f'{self.api_url}/api/notebooks/{notebook_id}/search',
            params={'q': query, 'limit': limit},
            timeout=self.timeout
        )
        response.raise_for_status()
        return response.json()

# Usage example
if __name__ == '__main__':
    # Get API URL from environment
    api_url = os.getenv('OPEN_NOTEBOOK_API_URL', 'https://your-app.railway.app:5055')
    
    client = OpenNotebookClient(api_url)
    
    # Get all notebooks
    notebooks = client.get_notebooks()
    print(f"Found {len(notebooks)} notebooks")
    
    # Create a new notebook
    notebook = client.create_notebook("Research Notes", "My research project")
    print(f"Created notebook: {notebook['id']}")
    
    # Search in notebook
    results = client.search(notebook['id'], "machine learning")
    print(f"Found {len(results)} results")
    
    # Chat with notebook
    response = client.chat(notebook['id'], "Summarize the main topics")
    print(f"Response: {response.get('message', 'No response')}")
```

### JavaScript/TypeScript Client

```typescript
class OpenNotebookClient {
  private apiUrl: string;
  private timeout: number;

  constructor(apiUrl: string, timeout: number = 300000) {
    this.apiUrl = apiUrl.replace(/\/$/, '');
    this.timeout = timeout;
  }

  async getNotebooks(): Promise<any[]> {
    const response = await fetch(`${this.apiUrl}/api/notebooks`, {
      signal: AbortSignal.timeout(this.timeout),
    });
    if (!response.ok) throw new Error(`HTTP error! status: ${response.status}`);
    return await response.json();
  }

  async createNotebook(name: string, description: string = ''): Promise<any> {
    const response = await fetch(`${this.apiUrl}/api/notebooks`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ name, description }),
      signal: AbortSignal.timeout(this.timeout),
    });
    if (!response.ok) throw new Error(`HTTP error! status: ${response.status}`);
    return await response.json();
  }

  async chat(notebookId: string, message: string, model: string = 'gpt-4'): Promise<any> {
    const response = await fetch(`${this.apiUrl}/api/notebooks/${notebookId}/chat`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ message, model }),
      signal: AbortSignal.timeout(this.timeout),
    });
    if (!response.ok) throw new Error(`HTTP error! status: ${response.status}`);
    return await response.json();
  }

  async search(notebookId: string, query: string, limit: number = 10): Promise<any[]> {
    const url = new URL(`${this.apiUrl}/api/notebooks/${notebookId}/search`);
    url.searchParams.set('q', query);
    url.searchParams.set('limit', limit.toString());
    
    const response = await fetch(url.toString(), {
      signal: AbortSignal.timeout(this.timeout),
    });
    if (!response.ok) throw new Error(`HTTP error! status: ${response.status}`);
    return await response.json();
  }
}

// Usage
const client = new OpenNotebookClient(process.env.OPEN_NOTEBOOK_API_URL!);
const notebooks = await client.getNotebooks();
```

### Next.js API Route Example

```typescript
// app/api/notebooks/route.ts
import { NextRequest, NextResponse } from 'next/server';

const OPEN_NOTEBOOK_API = process.env.OPEN_NOTEBOOK_API_URL!;

export async function GET(request: NextRequest) {
  try {
    const response = await fetch(`${OPEN_NOTEBOOK_API}/api/notebooks`, {
      headers: {
        'Content-Type': 'application/json',
      },
    });
    
    if (!response.ok) {
      return NextResponse.json(
        { error: 'Failed to fetch notebooks' },
        { status: response.status }
      );
    }
    
    const notebooks = await response.json();
    return NextResponse.json(notebooks);
  } catch (error) {
    return NextResponse.json(
      { error: 'Internal server error' },
      { status: 500 }
    );
  }
}
```

## Best Practices

### 1. Environment Variables

Always use environment variables for API URLs:

```python
# Good
api_url = os.getenv('OPEN_NOTEBOOK_API_URL')

# Bad
api_url = 'https://hardcoded-url.com'
```

### 2. Error Handling

Implement proper error handling:

```python
try:
    response = client.get_notebooks()
except requests.exceptions.Timeout:
    # Handle timeout
    pass
except requests.exceptions.ConnectionError:
    # Handle connection error
    pass
except requests.exceptions.HTTPError as e:
    # Handle HTTP error
    if e.response.status_code == 404:
        # Not found
        pass
    elif e.response.status_code == 429:
        # Rate limited
        pass
```

### 3. Timeout Configuration

Set appropriate timeouts for different operations:

```python
# Quick operations (search, list)
client.session.get(url, timeout=10)

# Long operations (chat, transformations)
client.session.post(url, timeout=300)  # 5 minutes
```

### 4. Rate Limiting

Respect rate limits and implement exponential backoff:

```python
import time

def call_with_backoff(func, max_retries=5):
    for attempt in range(max_retries):
        try:
            return func()
        except requests.exceptions.HTTPError as e:
            if e.response.status_code == 429:
                wait_time = 2 ** attempt
                time.sleep(wait_time)
                continue
            raise
```

### 5. Connection Pooling

Use connection pooling for frequent requests:

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

## Authentication

If password protection is enabled on Open Notebook:

```python
# Option 1: Session-based (if supported)
session = requests.Session()
session.post(f'{api_url}/api/auth/login', json={
    'password': 'your_password'
})

# Option 2: Token-based (if implemented)
headers = {'Authorization': f'Bearer {token}'}
response = session.get(f'{api_url}/api/notebooks', headers=headers)
```

## API Documentation

Access the interactive API documentation:
- **Swagger UI**: `https://your-app.railway.app:5055/docs`
- **ReDoc**: `https://your-app.railway.app:5055/redoc`
- **OpenAPI JSON**: `https://your-app.railway.app:5055/openapi.json`

## Testing

### Unit Tests

```python
import unittest
from unittest.mock import patch, Mock
from your_app import OpenNotebookClient

class TestOpenNotebookClient(unittest.TestCase):
    @patch('requests.Session.get')
    def test_get_notebooks(self, mock_get):
        mock_get.return_value.json.return_value = [{'id': '1', 'name': 'Test'}]
        mock_get.return_value.raise_for_status = Mock()
        
        client = OpenNotebookClient('https://test.api')
        notebooks = client.get_notebooks()
        
        self.assertEqual(len(notebooks), 1)
        self.assertEqual(notebooks[0]['name'], 'Test')
```

### Integration Tests

```python
import pytest

@pytest.mark.integration
def test_api_integration():
    api_url = os.getenv('OPEN_NOTEBOOK_API_URL')
    if not api_url:
        pytest.skip("OPEN_NOTEBOOK_API_URL not set")
    
    client = OpenNotebookClient(api_url)
    notebooks = client.get_notebooks()
    assert isinstance(notebooks, list)
```

## Monitoring

Monitor API usage and performance:

```python
import time
import logging

logger = logging.getLogger(__name__)

def monitored_request(func):
    def wrapper(*args, **kwargs):
        start_time = time.time()
        try:
            result = func(*args, **kwargs)
            duration = time.time() - start_time
            logger.info(f"API call completed in {duration:.2f}s")
            return result
        except Exception as e:
            duration = time.time() - start_time
            logger.error(f"API call failed after {duration:.2f}s: {e}")
            raise
    return wrapper
```

## Need Help?

- Check [API documentation](http://localhost:5055/docs) (replace with your URL)
- See [Railway deployment guide](railway.md)
- Join [Discord](https://discord.gg/37XJPXfz2w)
- Open an issue on [GitHub](https://github.com/lfnovo/open-notebook/issues)

