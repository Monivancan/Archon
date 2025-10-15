# Quick Start: Deploy Archon to Coolify

This is a condensed guide to get Archon running on Coolify in ~15 minutes.

## Prerequisites Checklist

- [ ] Coolify instance running
- [ ] Domain configured: `craftedbymonish.space`
- [ ] Supabase project created
- [ ] GitHub fork of Archon repository

## Step 1: Database Setup (5 minutes)

1. Log into your Supabase project at [supabase.com](https://supabase.com)
2. Go to **SQL Editor**
3. Open `migration/complete_setup.sql` from the Archon repository
4. Copy the entire file contents and paste into SQL Editor
5. Click **Run**
6. Verify tables were created in **Table Editor**

## Step 2: DNS Configuration (2 minutes)

Add these DNS records (A records or CNAMEs) pointing to your Coolify server IP:

```
archon.craftedbymonish.space   → [Coolify Server IP]
api.craftedbymonish.space      → [Coolify Server IP]
mcp.craftedbymonish.space      → [Coolify Server IP]
agents.craftedbymonish.space   → [Coolify Server IP]
```

## Step 3: Deploy to Coolify (8 minutes)

### 3.1 Create New Service

1. In Coolify dashboard, click **+ Add Resource**
2. Select **New Service** → **Docker Compose**

### 3.2 Configure Repository

- **Repository**: `https://github.com/YOUR_USERNAME/Archon`
- **Branch**: `main`
- **Docker Compose Path**: `docker-compose.prod.yml`

### 3.3 Add Environment Variables

Copy all variables from `COOLIFY_ENV_VARS.txt` and paste into Coolify's Environment section.

The file is in your repository root and contains all required variables already filled in for your domain.

### 3.4 Configure Domains

Set up these domain mappings in Coolify:

| Service | Domain | Port | SSL |
|---------|--------|------|-----|
| archon-frontend | archon.craftedbymonish.space | 80 | ✓ |
| archon-server | api.craftedbymonish.space | 8181 | ✓ |
| archon-mcp | mcp.craftedbymonish.space | 8051 | ✓ |
| archon-agents | agents.craftedbymonish.space | 8052 | ✓ |

### 3.5 Deploy

Click **Deploy** and wait 2-5 minutes for all services to start.

## Step 4: Verify Deployment (2 minutes)

Check each URL in your browser:

- ✓ `https://archon.craftedbymonish.space` - Should show Archon UI
- ✓ `https://api.craftedbymonish.space/health` - Should return `{"status":"healthy"}`
- ✓ `https://mcp.craftedbymonish.space/health` - Should return `{"status":"ok"}`
- ✓ `https://agents.craftedbymonish.space/health` - Should return `{"status":"healthy"}`

## Step 5: Initial Setup (2 minutes)

1. Visit `https://archon.craftedbymonish.space`
2. Complete the onboarding flow
3. Add your OpenAI/Anthropic/Google API key
4. Start using Archon!

## Quick Test

Try these to confirm everything works:

1. **Knowledge Base**: Crawl a website (e.g., `https://ai.pydantic.dev/llms-full.txt`)
2. **MCP**: Copy connection config from MCP Dashboard
3. **Projects**: Create a test project

## Troubleshooting

### Services won't start
- Check Coolify logs for each container
- Verify all environment variables are set
- Ensure `SERVICE_DISCOVERY_MODE=production`

### CORS errors in browser
- Verify `CORS_ORIGINS=https://archon.craftedbymonish.space`
- No trailing slash

### Can't connect to database
- Double-check Supabase credentials
- Use the **legacy** service role key from Supabase
- Verify migration ran successfully

### Frontend shows blank page
- Check `VITE_API_URL=https://api.craftedbymonish.space`
- Rebuild if you changed environment variables after first deploy

## Need More Help?

See the full [DEPLOYMENT.md](DEPLOYMENT.md) guide for detailed instructions and advanced configuration.

---

**You're all set! 🎉**

Your Archon instance should now be running at `https://archon.craftedbymonish.space`

