# Deploy to Fly.io

Deploy MCP Anywhere to Fly.io for a production-ready, cloud-hosted instance with automatic SSL certificates and global availability.

## Why Fly.io?

- Global edge deployment
- Automatic SSL certificates  
- Built-in persistent storage
- Simple deployment process
- Free tier available

## Prerequisites

- Fly.io account ([sign up free](https://fly.io))
- MCP Anywhere repository cloned locally
- Docker running locally

## Installation

### 1. Install Fly CLI

=== "macOS/Linux"
    ```bash
    curl -L https://fly.io/install.sh | sh
    ```

=== "Windows"
    ```powershell
    iwr https://fly.io/install.ps1 -useb | iex
    ```

### 2. Authenticate

```bash
fly auth login
```

## Configuration

MCP Anywhere supports two LLM providers for repository analysis:

### Anthropic (Default)
```bash
fly secrets set ANTHROPIC_API_KEY="your-anthropic-api-key"
fly secrets set ANTHROPIC_MODEL_NAME="claude-sonnet-4-20250514"  # optional
```

### OpenAI-Compatible (OpenRouter, etc.)
```bash
fly secrets set ANALYZER_PROVIDER="openai"
fly secrets set OPENAI_API_KEY="your-openrouter-api-key"
fly secrets set OPENAI_BASE_URL="https://openrouter.ai/api/v1"
fly secrets set OPENAI_MODEL_NAME="anthropic/claude-3.5-sonnet"  # or any OpenRouter model
```

### OIDC Authentication (Authentik, Keycloak, etc.)

To enable OIDC authentication (e.g., Authentik):

```bash
fly secrets set OIDC_CLIENT_ID="your-oidc-client-id"
fly secrets set OIDC_CLIENT_SECRET="your-oidc-client-secret"
fly secrets set OIDC_ISSUER_URL="https://auth.marinfam.cloud/application/o/your-app/"
fly secrets set OIDC_AUTH_URL="https://auth.marinfam.cloud/application/o/authorize/"
fly secrets set OIDC_TOKEN_URL="https://auth.marinfam.cloud/application/o/token/"
fly secrets set OIDC_USERINFO_URL="https://auth.marinfam.cloud/application/o/userinfo/"
fly secrets set OIDC_SCOPE="openid email profile"
fly secrets set OIDC_REDIRECT_URI="/auth/oidc/callback"
```

The OIDC provider will add a "Sign in with SSO" button to the login page.

## Deployment

### 1. Initialize Your App

```bash
cd mcp-anywhere
fly launch
```

When prompted:

- Choose an app name (e.g., `my-mcp-anywhere`)
- Select a region close to you
- Don't create PostgreSQL or Redis databases
- Yes to create the app

### 2. Configure Environment Variables

Set required secrets:

```bash
# Generate secure keys
SECRET_KEY=$(python -c "import secrets; print(secrets.token_urlsafe(32))")

# Set secrets
fly secrets set SECRET_KEY="$SECRET_KEY"
fly secrets set ANTHROPIC_API_KEY="your-anthropic-api-key-here"
```

### 3. Create Persistent Storage

MCP Anywhere needs persistent storage for data:

```bash
# Create a 10GB volume 
fly volumes create mcp_data --size 10
```

### 4. Deploy

```bash
fly deploy
```

The deployment will:

1. Build the Docker image
2. Push to Fly's registry
3. Deploy to your selected region
4. Start the application

### 5. Verify Deployment

```bash
# Check status
fly status

# View logs
fly logs

# Open in browser
fly open
```

Your MCP Anywhere instance is now live at `https://your-app-name.fly.dev`

## Monitoring

### View Logs

```bash
# Live logs
fly logs -f

# Recent logs
fly logs -n 100
```

### Metrics

```bash
# Open metrics dashboard
fly dashboard metrics
```

## Updating

To deploy updates:

```bash
# Pull latest changes
git pull origin main

# Deploy updates
fly deploy
```



## Troubleshooting

### App Won't Start

```bash
# Check logs for errors
fly logs -n 200

# Common issues:
# - Missing environment variables
# - Docker build failures
# - Port binding issues
```

### Reset Data

If you need to reset your instance:

```bash
# SSH into instance
fly ssh console

# Run reset command
/app/venv/bin/python -m mcp_anywhere reset --confirm
```

---

