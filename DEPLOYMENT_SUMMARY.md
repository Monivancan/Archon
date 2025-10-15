# Archon Coolify Deployment - Implementation Summary

## What Was Changed

This fork has been prepared for production deployment on Coolify with the following modifications:

### 1. Production Docker Compose Configuration
**File**: `docker-compose.prod.yml`

- Created production-ready compose file without development features
- Removed source code volume mounts (no hot-reload needed in production)
- Configured environment-based service URLs for subdomain communication
- Optimized health checks and restart policies
- Multi-stage frontend build with nginx

### 2. Service Discovery Updates
**File**: `python/src/server/config/service_discovery.py`

- Added `PRODUCTION` environment mode
- Services now read URLs from environment variables in production
- Supports full HTTPS URLs for inter-service communication
- Automatic detection based on `SERVICE_DISCOVERY_MODE` environment variable
- Backwards compatible with existing local/docker-compose modes

### 3. CORS Security Configuration
**File**: `python/src/server/main.py`

- CORS origins now configurable via `CORS_ORIGINS` environment variable
- Production mode uses specific allowed origins (not wildcard)
- Supports comma-separated list for multiple origins
- Secure by default

### 4. Frontend Production Build
**Files**: `archon-ui-main/Dockerfile`, `archon-ui-main/nginx.conf`

- Multi-stage Dockerfile with production target
- Production builds served via nginx (not Vite dev server)
- Static asset optimization and caching
- Health check endpoint
- Proper security headers
- Build-time API URL configuration

### 5. Vite Configuration Updates
**File**: `archon-ui-main/vite.config.ts`

- Production mode detection
- Test runner endpoints only in development
- Proxy configuration disabled in production builds
- Proper environment variable handling

### 6. Environment Configuration Templates
**Files**: `env.example`, `env.production.example`

- Development environment template with defaults
- Production environment template with Coolify-specific values
- Comprehensive documentation of all variables
- Clear separation of required vs optional variables

### 7. Deployment Documentation
**Files**: 
- `DEPLOYMENT.md` - Comprehensive deployment guide
- `COOLIFY_QUICK_START.md` - 15-minute quick start
- `COOLIFY_ENV_VARS.txt` - Ready-to-paste environment variables

Complete deployment documentation including:
- Step-by-step Coolify setup instructions
- DNS configuration guide
- Database migration steps
- Subdomain mapping
- Troubleshooting guide
- Security best practices
- Update procedures

## Files Created

1. `docker-compose.prod.yml` - Production Docker Compose configuration
2. `archon-ui-main/nginx.conf` - Nginx configuration for frontend
3. `env.example` - Development environment template
4. `env.production.example` - Production environment template
5. `DEPLOYMENT.md` - Comprehensive deployment guide
6. `COOLIFY_QUICK_START.md` - Quick start guide
7. `COOLIFY_ENV_VARS.txt` - Environment variables for copy-paste
8. `DEPLOYMENT_SUMMARY.md` - This file

## Files Modified

1. `python/src/server/config/service_discovery.py` - Added production mode
2. `python/src/server/main.py` - Environment-based CORS configuration
3. `archon-ui-main/Dockerfile` - Multi-stage build with production target
4. `archon-ui-main/vite.config.ts` - Production mode handling

## Architecture

### Subdomain Mapping

| Service | Subdomain | Container Port | Public Port | Purpose |
|---------|-----------|----------------|-------------|---------|
| Frontend | archon.craftedbymonish.space | 3737 (internal) | 80 (nginx) | React UI |
| API Server | api.craftedbymonish.space | 8181 | 8181 | FastAPI backend |
| MCP Server | mcp.craftedbymonish.space | 8051 | 8051 | MCP protocol |
| Agents | agents.craftedbymonish.space | 8052 | 8052 | AI agents |

### Service Communication

In production mode (`SERVICE_DISCOVERY_MODE=production`):
- Services communicate via full HTTPS URLs (not container names)
- URLs read from environment variables
- Frontend calls API via `VITE_API_URL`
- Backend services use `ARCHON_SERVER_URL`, `ARCHON_MCP_URL`, etc.

### Security Features

1. **CORS**: Restricted to frontend domain only
2. **HTTPS**: All services behind SSL/TLS
3. **Environment Variables**: Secrets managed by Coolify
4. **Service Role Key**: Supabase service key (not anon key)
5. **Security Headers**: X-Frame-Options, X-Content-Type-Options, etc.

## Environment Variables Reference

### Required Variables

```bash
# Supabase
SUPABASE_URL=https://mcwffikvukcdsrshyeyj.supabase.co
SUPABASE_SERVICE_KEY=<your-service-key>

# Production Mode
SERVICE_DISCOVERY_MODE=production
PROD=true

# Service URLs
ARCHON_SERVER_URL=https://api.craftedbymonish.space
ARCHON_MCP_URL=https://mcp.craftedbymonish.space
ARCHON_AGENTS_URL=https://agents.craftedbymonish.space
ARCHON_FRONTEND_URL=https://archon.craftedbymonish.space
VITE_API_URL=https://api.craftedbymonish.space

# Security
CORS_ORIGINS=https://archon.craftedbymonish.space
```

### Optional Variables

```bash
# AI Providers
OPENAI_API_KEY=<optional>
ANTHROPIC_API_KEY=<optional>
GOOGLE_API_KEY=<optional>

# Monitoring
LOGFIRE_TOKEN=<optional>
```

## Deployment Steps (Summary)

1. **Database Setup**: Run `migration/complete_setup.sql` in Supabase SQL Editor
2. **DNS Configuration**: Point subdomains to Coolify server
3. **Coolify Setup**: 
   - Create Docker Compose service
   - Repository: Your fork
   - Docker Compose file: `docker-compose.prod.yml`
4. **Environment Variables**: Paste from `COOLIFY_ENV_VARS.txt`
5. **Domain Mapping**: Configure 4 subdomains in Coolify
6. **Deploy**: Click deploy and wait ~5 minutes
7. **Verify**: Check all health endpoints
8. **Configure**: Complete onboarding at `https://archon.craftedbymonish.space`

## Testing Checklist

After deployment, verify:

- [ ] Frontend loads: `https://archon.craftedbymonish.space`
- [ ] API health: `https://api.craftedbymonish.space/health`
- [ ] MCP health: `https://mcp.craftedbymonish.space/health`
- [ ] Agents health: `https://agents.craftedbymonish.space/health`
- [ ] No CORS errors in browser console
- [ ] Can complete onboarding flow
- [ ] Can crawl a website
- [ ] Can upload a document
- [ ] Can create a project
- [ ] Can connect via MCP

## Differences from Upstream

This fork maintains all original functionality while adding:

1. Production deployment support
2. Environment-based configuration
3. Multi-stage Docker builds
4. Comprehensive deployment documentation
5. Coolify-specific optimizations

All changes are backwards compatible with the original local development setup.

## Updating from Upstream

To pull updates from the main Archon repository:

```bash
git remote add upstream https://github.com/coleam00/archon.git
git fetch upstream
git merge upstream/main
git push origin main
```

Then redeploy in Coolify.

## Support

For deployment issues specific to this fork:
- Check `DEPLOYMENT.md` for detailed troubleshooting
- Review `COOLIFY_QUICK_START.md` for common setup issues

For Archon functionality issues:
- Refer to upstream repository: https://github.com/coleam00/archon
- Check original documentation in `docs/` folder

## License

Same as upstream Archon repository.

---

**Implementation completed successfully! Ready for Coolify deployment. 🚀**

