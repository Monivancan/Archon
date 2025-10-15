# Archon Production Deployment Guide for Coolify

This guide will help you deploy Archon to Coolify with proper subdomain configuration and production-ready settings.

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Architecture Overview](#architecture-overview)
3. [Subdomain Setup](#subdomain-setup)
4. [Environment Variables](#environment-variables)
5. [Database Setup](#database-setup)
6. [Coolify Deployment Steps](#coolify-deployment-steps)
7. [Post-Deployment Configuration](#post-deployment-configuration)
8. [Troubleshooting](#troubleshooting)
9. [Updating Your Deployment](#updating-your-deployment)

## Prerequisites

Before deploying to Coolify, ensure you have:

1. **Coolify Instance**: A running Coolify installation with Docker Compose support
2. **Domain Access**: A domain with DNS management capabilities (e.g., `craftedbymonish.space`)
3. **Supabase Account**: A Supabase project (free tier works fine)
   - Get your `SUPABASE_URL` from Project Settings → API
   - Get your `SUPABASE_SERVICE_KEY` (use the **legacy** service role key format)
4. **SSL Certificates**: Coolify should be configured to issue SSL certificates automatically
5. **Git Repository**: Your forked Archon repository on GitHub

## Architecture Overview

Archon consists of 4 microservices that run as separate containers:

| Service | Subdomain | Port | Purpose |
|---------|-----------|------|---------|
| Frontend | `archon.craftedbymonish.space` | 80 (nginx) | React UI served via nginx |
| API Server | `api.craftedbymonish.space` | 8181 | FastAPI backend, crawling, Socket.IO |
| MCP Server | `mcp.craftedbymonish.space` | 8051 | Model Context Protocol server for AI tools |
| Agents | `agents.craftedbymonish.space` | 8052 | AI agents service (optional but recommended) |

## Subdomain Setup

### DNS Configuration

In your DNS provider (e.g., Cloudflare, Namecheap), create A records or CNAME records pointing to your Coolify server:

```
archon.craftedbymonish.space  →  [Your Coolify Server IP]
api.craftedbymonish.space     →  [Your Coolify Server IP]
mcp.craftedbymonish.space     →  [Your Coolify Server IP]
agents.craftedbymonish.space  →  [Your Coolify Server IP]
```

**Note**: If using Cloudflare, make sure the proxy (orange cloud) is enabled for SSL/TLS encryption.

## Environment Variables

### Copy the Production Template

The repository includes `env.production.example` with all required variables. You'll configure these in Coolify.

### Required Environment Variables

Configure these in Coolify's environment section for your application:

```env
# Supabase Configuration (REQUIRED)
SUPABASE_URL=https://mcwffikvukcdsrshyeyj.supabase.co
SUPABASE_SERVICE_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6Im1jd2ZmaWt2dWtjZHNyc2h5ZXlqIiwicm9sZSI6InNlcnZpY2Vfcm9sZSIsImlhdCI6MTc2MDMzNjUzNiwiZXhwIjoyMDc1OTEyNTM2fQ.UNptu0oNm4M_yAheyNPKqtH4R4ql0reXxdoiH1vfV6g

# Service Ports (Keep as defaults)
ARCHON_SERVER_PORT=8181
ARCHON_MCP_PORT=8051
ARCHON_AGENTS_PORT=8052
ARCHON_UI_PORT=3737

# Production Mode
SERVICE_DISCOVERY_MODE=production
LOG_LEVEL=INFO
PROD=true

# Production Service URLs (Update with your domain)
ARCHON_SERVER_URL=https://api.craftedbymonish.space
ARCHON_MCP_URL=https://mcp.craftedbymonish.space
ARCHON_AGENTS_URL=https://agents.craftedbymonish.space
ARCHON_FRONTEND_URL=https://archon.craftedbymonish.space

# Frontend API URL
VITE_API_URL=https://api.craftedbymonish.space

# CORS Security (your frontend domain)
CORS_ORIGINS=https://archon.craftedbymonish.space

# Features
AGENTS_ENABLED=true
```

### Optional Environment Variables

Add these for full functionality:

```env
# AI Provider API Keys (at least one recommended)
OPENAI_API_KEY=sk-your-openai-key
ANTHROPIC_API_KEY=sk-ant-your-anthropic-key
GOOGLE_API_KEY=your-google-key

# Observability (optional)
LOGFIRE_TOKEN=your-logfire-token
```

## Database Setup

### 1. Create Supabase Project

If you don't have one already:
1. Go to [supabase.com](https://supabase.com)
2. Create a new project
3. Wait for it to initialize (takes ~2 minutes)

### 2. Run Database Migration

1. In your Supabase project, go to **SQL Editor**
2. Open the file `migration/complete_setup.sql` from your repository
3. Copy and paste the entire contents into the SQL Editor
4. Click **Run** to execute the migration
5. Verify tables were created: Go to **Table Editor** and confirm you see:
   - `sources`
   - `documents`
   - `projects`
   - `tasks`
   - `code_examples`
   - And other tables

## Coolify Deployment Steps

### Option 1: Docker Compose Deployment (Recommended)

This is the simplest method as Coolify supports Docker Compose deployments directly.

#### Step 1: Create New Service in Coolify

1. Log into your Coolify dashboard
2. Go to your project
3. Click **+ Add Resource** → **New Service** → **Docker Compose**

#### Step 2: Configure Repository

1. **Repository**: Enter your forked GitHub repository URL
   ```
   https://github.com/YOUR_USERNAME/Archon
   ```
2. **Branch**: `main` (or your custom branch)
3. **Docker Compose File Path**: `docker-compose.prod.yml`

#### Step 3: Configure Environment Variables

1. In the Coolify service settings, go to **Environment** tab
2. Add all the environment variables from the [Environment Variables](#environment-variables) section
3. Update the domain names to match your actual domain

#### Step 4: Configure Domains

For each service, configure the subdomain in Coolify:

1. **archon-frontend**:
   - Domain: `archon.craftedbymonish.space`
   - Port: `80`
   - SSL: Enable (Coolify will auto-provision)

2. **archon-server**:
   - Domain: `api.craftedbymonish.space`
   - Port: `8181`
   - SSL: Enable

3. **archon-mcp**:
   - Domain: `mcp.craftedbymonish.space`
   - Port: `8051`
   - SSL: Enable

4. **archon-agents**:
   - Domain: `agents.craftedbymonish.space`
   - Port: `8052`
   - SSL: Enable

#### Step 5: Deploy

1. Click **Deploy** button
2. Monitor the build logs
3. Wait for all services to become healthy (this may take 2-5 minutes)

### Option 2: Individual Service Deployment

If your Coolify setup requires individual services:

1. Create 4 separate services in Coolify
2. For each service, point to the same repository but use different Dockerfiles:
   - Frontend: `archon-ui-main/Dockerfile` (target: production)
   - Server: `python/Dockerfile.server`
   - MCP: `python/Dockerfile.mcp`
   - Agents: `python/Dockerfile.agents`
3. Configure environment variables for each service
4. Set up the appropriate subdomain for each

## Post-Deployment Configuration

### 1. Verify Services are Running

Check each subdomain in your browser:

- `https://archon.craftedbymonish.space` → Should show the Archon UI
- `https://api.craftedbymonish.space/health` → Should return `{"status":"healthy"}`
- `https://mcp.craftedbymonish.space/health` → Should return `{"status":"ok"}`
- `https://agents.craftedbymonish.space/health` → Should return `{"status":"healthy"}`

### 2. Complete Onboarding

1. Visit `https://archon.craftedbymonish.space`
2. You'll be guided through an onboarding flow
3. Configure your AI provider API key (OpenAI, Anthropic, or Google)
4. The key will be stored in Supabase

### 3. Test Core Features

1. **Knowledge Base**:
   - Go to Knowledge Base → Crawl Website
   - Try crawling a documentation site (e.g., `https://ai.pydantic.dev/llms-full.txt`)
   - Verify documents appear in the knowledge base

2. **MCP Connection**:
   - Go to MCP Dashboard
   - Copy the connection configuration
   - Test connecting from Claude Desktop or Cursor

3. **Projects** (optional):
   - Go to Projects
   - Create a test project
   - Add some tasks

## Troubleshooting

### Issue: Services Can't Communicate

**Symptom**: Frontend shows "Backend connection error" or MCP can't reach API

**Solution**:
1. Verify all environment variables are set correctly
2. Check that `SERVICE_DISCOVERY_MODE=production`
3. Ensure `ARCHON_SERVER_URL`, `ARCHON_MCP_URL`, etc. are set to the full HTTPS URLs
4. Check Coolify logs for each service

### Issue: CORS Errors

**Symptom**: Browser console shows CORS policy errors

**Solution**:
1. Verify `CORS_ORIGINS` includes your frontend domain
2. Must be the full URL: `https://archon.craftedbymonish.space`
3. No trailing slash
4. Can be comma-separated for multiple origins

### Issue: Database Connection Fails

**Symptom**: Services fail to start with Supabase errors

**Solution**:
1. Verify `SUPABASE_URL` and `SUPABASE_SERVICE_KEY` are correct
2. Use the **legacy** service role key (the longer one) from Supabase
3. Check that the database migration was run successfully
4. Verify your Supabase project is not paused (free tier pauses after inactivity)

### Issue: Frontend Shows Blank Page

**Symptom**: White screen or loading forever

**Solution**:
1. Check browser console for errors
2. Verify `VITE_API_URL` is set correctly during build
3. Check that nginx is serving the files: `https://archon.craftedbymonish.space/health` should return "OK"
4. Rebuild the frontend with correct environment variables

### Issue: Build Fails

**Symptom**: Docker build errors in Coolify logs

**Solution**:
1. Check that `docker-compose.prod.yml` exists in your repository
2. Verify all Dockerfiles are present and valid
3. Check for syntax errors in docker-compose.prod.yml
4. Ensure your Coolify server has enough disk space and memory

### Issue: SSL Certificate Not Provisioning

**Symptom**: HTTPS not working, browser shows "Not Secure"

**Solution**:
1. Verify DNS records are pointing to Coolify server
2. Wait 5-10 minutes for DNS propagation
3. Check Coolify SSL settings for each service
4. Ensure ports 80 and 443 are open on your server
5. Try manually triggering SSL certificate generation in Coolify

## Updating Your Deployment

### Pulling Updates from Upstream

Since you're using a fork, you'll want to pull updates from the main Archon repository:

```bash
# Add upstream remote (one time)
git remote add upstream https://github.com/coleam00/archon.git

# Fetch and merge updates
git fetch upstream
git checkout main
git merge upstream/main

# Push to your fork
git push origin main
```

### Deploying Updates

After pushing updates to your fork:

1. Go to Coolify dashboard
2. Click **Redeploy** on your Archon service
3. Coolify will pull the latest code and rebuild
4. Monitor the deployment logs

### Database Migrations

If the update includes database changes:

1. Check `migration/0.1.0/` for new migration files
2. Run new migrations in Supabase SQL Editor
3. Follow instructions in `migration/0.1.0/DB_UPGRADE_INSTRUCTIONS.md`

## Environment-Specific Configuration

### Development vs Production

The repository includes two Docker Compose files:

- `docker-compose.yml` - For local development (hot-reload, volume mounts)
- `docker-compose.prod.yml` - For production (optimized, no dev tools)

Always use `docker-compose.prod.yml` for Coolify deployments.

### Multiple Environments

To deploy staging and production:

1. Create separate Coolify services
2. Use different branches (e.g., `staging` and `main`)
3. Configure different domains (e.g., `staging.archon.yourdomain.com`)
4. Use separate Supabase projects or different environment variables

## Security Best Practices

1. **Never commit secrets**: Use Coolify's environment variable management
2. **Use HTTPS**: Always enable SSL/TLS for all subdomains
3. **Restrict CORS**: Set `CORS_ORIGINS` to only your frontend domain
4. **Service Role Key**: Keep your Supabase service role key secure
5. **API Keys**: Rotate API keys periodically
6. **Firewall**: Configure firewall rules to only allow necessary ports
7. **Updates**: Regularly pull updates from upstream for security patches

## Resource Requirements

### Minimum Requirements

- **CPU**: 2 vCPU
- **RAM**: 4 GB
- **Disk**: 20 GB SSD
- **Network**: 100 Mbps

### Recommended for Production

- **CPU**: 4 vCPU
- **RAM**: 8 GB
- **Disk**: 50 GB SSD
- **Network**: 1 Gbps

### Service Resource Usage

| Service | CPU | Memory | Notes |
|---------|-----|--------|-------|
| Frontend | Low | ~100MB | Static files served by nginx |
| API Server | Medium | ~500MB | Handles crawling, embeddings |
| MCP Server | Low | ~200MB | Lightweight protocol server |
| Agents | Low-Medium | ~300MB | AI agents, optional |

## Monitoring and Logs

### Viewing Logs in Coolify

1. Go to your service in Coolify
2. Click **Logs** tab
3. Select the specific container to view logs
4. Use filters to find specific errors

### Optional: Logfire Integration

For advanced monitoring:

1. Sign up at [logfire.dev](https://logfire.dev)
2. Get your Logfire token
3. Add to environment variables: `LOGFIRE_TOKEN=your-token`
4. Redeploy services
5. View metrics and traces in Logfire dashboard

## Support and Community

- **GitHub Issues**: [github.com/coleam00/Archon/issues](https://github.com/coleam00/Archon/issues)
- **Discussions**: [github.com/coleam00/Archon/discussions](https://github.com/coleam00/Archon/discussions)
- **Documentation**: See `docs/` folder in repository
- **Contributing**: See `CONTRIBUTING.md`

## Next Steps

After successful deployment:

1. **Configure AI Providers**: Add API keys for OpenAI, Anthropic, or Google
2. **Build Knowledge Base**: Start crawling documentation and uploading files
3. **Connect AI Tools**: Set up MCP in Claude Desktop, Cursor, or Windsurf
4. **Create Projects**: Organize your work with projects and tasks
5. **Explore Features**: Check out RAG search, code extraction, and agent chat

---

**Happy Deploying! 🚀**

If you encounter any issues not covered in this guide, please open an issue on GitHub or ask in the discussions forum.

