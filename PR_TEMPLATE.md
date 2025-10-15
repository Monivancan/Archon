# Add Coolify Production Deployment Support

## Summary

This PR adds production deployment support for Coolify with proper subdomain configuration, environment-based service discovery, and comprehensive deployment documentation.

## Changes Made

### Core Infrastructure

1. **Production Docker Compose** (`docker-compose.prod.yml`)
   - Production-ready configuration without dev-only features
   - Environment-based service URLs for subdomain communication
   - Optimized builds and health checks
   - Multi-stage frontend build with nginx

2. **Service Discovery** (`python/src/server/config/service_discovery.py`)
   - Added `PRODUCTION` environment mode
   - Support for full HTTPS URLs in production
   - Environment variable-based service URLs
   - Backwards compatible with existing modes

3. **CORS Security** (`python/src/server/main.py`)
   - Configurable via `CORS_ORIGINS` environment variable
   - Restricted origins in production (no wildcard)
   - Support for multiple origins

4. **Frontend Production Build** (`archon-ui-main/Dockerfile`, `archon-ui-main/nginx.conf`)
   - Multi-stage Dockerfile with production target
   - Nginx-based static file serving
   - Security headers and asset optimization
   - Production health checks

5. **Vite Configuration** (`archon-ui-main/vite.config.ts`)
   - Production mode detection
   - Conditional dev-only features
   - Proper environment variable handling

### Documentation

1. **Deployment Guide** (`DEPLOYMENT.md`)
   - Comprehensive Coolify deployment instructions
   - DNS and SSL configuration
   - Database setup guide
   - Troubleshooting section
   - Security best practices

2. **Quick Start** (`COOLIFY_QUICK_START.md`)
   - 15-minute deployment guide
   - Condensed step-by-step instructions
   - Quick verification checklist

3. **Environment Variables** (`COOLIFY_ENV_VARS.txt`, `env.example`, `env.production.example`)
   - Ready-to-paste environment variables
   - Development and production templates
   - Comprehensive variable documentation

4. **Summary** (`DEPLOYMENT_SUMMARY.md`)
   - Implementation overview
   - Architecture details
   - Testing checklist
   - Update procedures

## Production Architecture

### Subdomain Mapping
- `archon.craftedbymonish.space` → Frontend (nginx on port 80)
- `api.craftedbymonish.space` → API Server (port 8181)
- `mcp.craftedbymonish.space` → MCP Server (port 8051)
- `agents.craftedbymonish.space` → Agents Service (port 8052)

### Environment Configuration
- Production mode via `SERVICE_DISCOVERY_MODE=production`
- Services communicate via full HTTPS URLs
- CORS restricted to frontend domain
- All secrets managed via Coolify environment variables

## Testing

Tested locally with:
- ✅ Local Docker Compose (development mode)
- ✅ Production Docker Compose build
- ✅ Service discovery in all modes
- ✅ CORS configuration
- ✅ Frontend production build
- ✅ Health checks

Ready for deployment on Coolify with:
- Domain: `craftedbymonish.space`
- 4 subdomains configured
- SSL/TLS enabled
- Supabase database ready

## Backwards Compatibility

All changes are backwards compatible:
- Local development unchanged (`docker-compose.yml` intact)
- Existing environment variables work as before
- Service discovery auto-detects mode
- Development tools still available in dev mode

## Files Added

- `docker-compose.prod.yml` - Production configuration
- `archon-ui-main/nginx.conf` - Nginx configuration
- `env.example` - Development template
- `env.production.example` - Production template
- `DEPLOYMENT.md` - Comprehensive guide
- `COOLIFY_QUICK_START.md` - Quick start
- `COOLIFY_ENV_VARS.txt` - Environment variables
- `DEPLOYMENT_SUMMARY.md` - Implementation summary
- `PR_TEMPLATE.md` - This template

## Files Modified

- `python/src/server/config/service_discovery.py` - Production mode support
- `python/src/server/main.py` - CORS configuration
- `archon-ui-main/Dockerfile` - Multi-stage build
- `archon-ui-main/vite.config.ts` - Production mode handling

## Deployment Instructions

See `COOLIFY_QUICK_START.md` for 15-minute deployment guide or `DEPLOYMENT.md` for comprehensive instructions.

Quick steps:
1. Run database migration in Supabase
2. Configure DNS records
3. Create Docker Compose service in Coolify
4. Add environment variables from `COOLIFY_ENV_VARS.txt`
5. Configure 4 subdomains with SSL
6. Deploy!

## Breaking Changes

None. This PR only adds production deployment capability without affecting existing functionality.

## Related Issues

- Addresses need for production deployment support
- Enables Coolify hosting with proper subdomain configuration
- Improves security with environment-based CORS

## Next Steps After Merge

1. Update fork from upstream periodically
2. Redeploy in Coolify to get updates
3. Monitor logs for any production issues
4. Consider contributing improvements back to upstream

---

**Ready for production deployment on Coolify! 🚀**

