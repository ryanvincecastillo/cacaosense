# CacaoSense Deployment Guide

## Overview

This guide covers deploying CacaoSense to production. The architecture uses:
- **Vercel** - Frontend (Landing Page + Dashboard)
- **Railway** - Backend API + Chatbot
- **Supabase** - Database + Auth + Storage

---

## Prerequisites

- Node.js 18+
- Python 3.10+
- Git
- Accounts on: Vercel, Railway, Supabase, Facebook Developer

---

## 1. Supabase Setup

### Create Project

1. Go to [supabase.com](https://supabase.com)
2. Create new project
3. Note down:
   - Project URL
   - Anon Key
   - Service Role Key
   - Database URL

### Run Migrations

1. Install Supabase CLI:
```bash
npm install -g supabase
```

2. Initialize and link:
```bash
supabase init
supabase link --project-ref your-project-ref
```

3. Run migrations:
```bash
supabase db push
```

### Enable Row Level Security

Ensure RLS is enabled on all tables (see database-schema.md).

### Configure Auth

1. Go to Authentication > Providers
2. Enable Email provider
3. Configure email templates
4. Set up redirect URLs:
   - `https://app.cacaosense.com/auth/callback`
   - `http://localhost:3000/auth/callback` (dev)

---

## 2. Backend API Deployment (Railway)

### Create Railway Project

1. Go to [railway.app](https://railway.app)
2. Create new project
3. Connect GitHub repository
4. Select `cacaosense-api` directory

### Configure Environment Variables

Add these in Railway dashboard:

```env
# Database
DATABASE_URL=postgresql://...
SUPABASE_URL=https://your-project.supabase.co
SUPABASE_KEY=your-service-role-key

# Auth
JWT_SECRET=your-jwt-secret
JWT_ALGORITHM=HS256

# OpenAI (for NLP)
OPENAI_API_KEY=sk-...

# Facebook Messenger
MESSENGER_VERIFY_TOKEN=your-verify-token
MESSENGER_PAGE_ACCESS_TOKEN=your-page-token

# Environment
ENVIRONMENT=production
ALLOWED_ORIGINS=https://app.cacaosense.com,https://cacaosense.com
```

### Railway Configuration

Create `railway.toml` in `cacaosense-api`:

```toml
[build]
builder = "DOCKERFILE"

[deploy]
healthcheckPath = "/health"
healthcheckTimeout = 100
restartPolicyType = "ON_FAILURE"
restartPolicyMaxRetries = 3
```

### Dockerfile

Create `Dockerfile` in `cacaosense-api`:

```dockerfile
FROM python:3.10-slim

WORKDIR /app

# Install dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application
COPY . .

# Run server
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### Deploy

```bash
# Railway will auto-deploy on git push
git push origin main
```

### Custom Domain

1. In Railway, go to Settings > Domains
2. Add custom domain: `api.cacaosense.com`
3. Update DNS with provided CNAME

---

## 3. Frontend Deployment (Vercel)

### Landing Page

1. Go to [vercel.com](https://vercel.com)
2. Import GitHub repository
3. Select `cacaosense-landing` directory
4. Framework: Other
5. Deploy

### Dashboard (Next.js App)

1. Import same repository
2. Select `cacaosense-app` directory
3. Framework: Next.js
4. Configure environment variables:

```env
NEXT_PUBLIC_SUPABASE_URL=https://your-project.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=your-anon-key
NEXT_PUBLIC_API_URL=https://api.cacaosense.com
```

### Custom Domains

1. Landing: `cacaosense.com` and `www.cacaosense.com`
2. Dashboard: `app.cacaosense.com`

### Vercel Configuration

Create `vercel.json` in `cacaosense-app`:

```json
{
    "framework": "nextjs",
    "regions": ["sin1"],
    "headers": [
        {
            "source": "/(.*)",
            "headers": [
                {
                    "key": "X-Content-Type-Options",
                    "value": "nosniff"
                },
                {
                    "key": "X-Frame-Options",
                    "value": "DENY"
                },
                {
                    "key": "X-XSS-Protection",
                    "value": "1; mode=block"
                }
            ]
        }
    ]
}
```

---

## 4. Messenger Bot Deployment

### Facebook Developer Setup

1. Go to [developers.facebook.com](https://developers.facebook.com)
2. Create new app (Business type)
3. Add Messenger product
4. Create Facebook Page for the bot

### Configure Webhook

1. In Messenger Settings > Webhooks
2. Callback URL: `https://api.cacaosense.com/webhook/messenger`
3. Verify Token: (same as MESSENGER_VERIFY_TOKEN)
4. Subscribe to: `messages`, `messaging_postbacks`

### Generate Page Access Token

1. In Messenger Settings > Access Tokens
2. Generate token for your page
3. Add to Railway environment variables

### Test Bot

1. Send message to your Facebook Page
2. Check Railway logs for incoming webhook
3. Verify response

---

## 5. Domain & SSL Configuration

### DNS Setup

Add these records to your domain:

| Type | Name | Value |
|------|------|-------|
| A | @ | Vercel IP |
| CNAME | www | cname.vercel-dns.com |
| CNAME | app | cname.vercel-dns.com |
| CNAME | api | your-railway-domain |

### SSL Certificates

- Vercel and Railway automatically provision SSL certificates
- Ensure all traffic uses HTTPS

---

## 6. Environment Variables Summary

### cacaosense-api (Railway)

```env
# Database
DATABASE_URL=postgresql://user:pass@host:5432/db
SUPABASE_URL=https://xxx.supabase.co
SUPABASE_KEY=eyJ...

# Auth
JWT_SECRET=your-secret-min-32-chars
JWT_ALGORITHM=HS256

# External APIs
OPENAI_API_KEY=sk-...
MESSENGER_VERIFY_TOKEN=cacaosense-verify-2024
MESSENGER_PAGE_ACCESS_TOKEN=EAA...

# App Config
ENVIRONMENT=production
ALLOWED_ORIGINS=https://app.cacaosense.com,https://cacaosense.com
LOG_LEVEL=INFO
```

### cacaosense-app (Vercel)

```env
NEXT_PUBLIC_SUPABASE_URL=https://xxx.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJ...
NEXT_PUBLIC_API_URL=https://api.cacaosense.com
NEXT_PUBLIC_APP_URL=https://app.cacaosense.com
```

---

## 7. CI/CD Pipeline

### GitHub Actions

Create `.github/workflows/deploy.yml`:

```yaml
name: Deploy

on:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.10'
      
      - name: Install dependencies
        run: |
          cd cacaosense-api
          pip install -r requirements.txt
          pip install pytest
      
      - name: Run tests
        run: |
          cd cacaosense-api
          pytest

  deploy-api:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Deploy to Railway
        uses: bervProject/railway-deploy@main
        with:
          railway_token: ${{ secrets.RAILWAY_TOKEN }}
          service: cacaosense-api

  deploy-frontend:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Deploy to Vercel
        uses: amondnet/vercel-action@v25
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-org-id: ${{ secrets.VERCEL_ORG_ID }}
          vercel-project-id: ${{ secrets.VERCEL_PROJECT_ID }}
          working-directory: ./cacaosense-app
```

---

## 8. Monitoring & Logging

### Railway Logs

View logs in Railway dashboard or CLI:

```bash
railway logs
```

### Vercel Analytics

Enable in Vercel dashboard for:
- Web Vitals
- Visitor analytics
- Error tracking

### Supabase Dashboard

Monitor:
- Database performance
- API requests
- Auth events

### External Monitoring (Recommended)

- **Uptime:** UptimeRobot or Better Uptime
- **Error Tracking:** Sentry
- **APM:** New Relic or DataDog

---

## 9. Backup & Recovery

### Database Backups

Supabase provides automatic daily backups on Pro plan.

Manual backup:
```bash
pg_dump $DATABASE_URL > backup.sql
```

### Restore

```bash
psql $DATABASE_URL < backup.sql
```

---

## 10. Scaling

### Vertical Scaling

- **Railway:** Increase memory/CPU in settings
- **Vercel:** Automatic based on traffic

### Horizontal Scaling

- **Railway:** Add more instances
- **Supabase:** Upgrade plan for more connections

### Database Optimization

- Add indexes for slow queries
- Use connection pooling (PgBouncer)
- Enable read replicas if needed

---

## 11. Security Checklist

- [ ] All secrets in environment variables (not in code)
- [ ] HTTPS enforced everywhere
- [ ] CORS configured properly
- [ ] Rate limiting enabled
- [ ] Input validation on all endpoints
- [ ] SQL injection prevention (use ORM)
- [ ] XSS prevention (sanitize outputs)
- [ ] CSRF protection
- [ ] Secure headers configured
- [ ] Regular dependency updates
- [ ] Database backups enabled
- [ ] Audit logging enabled

---

## 12. Troubleshooting

### API Not Responding

1. Check Railway logs
2. Verify environment variables
3. Check database connection
4. Verify health endpoint

### Database Connection Issues

1. Check DATABASE_URL format
2. Verify IP allowlist in Supabase
3. Check connection pool limits

### Messenger Bot Not Working

1. Verify webhook URL is correct
2. Check verify token matches
3. Verify page access token
4. Check Facebook app permissions

### Build Failures

1. Check build logs in Railway/Vercel
2. Verify all dependencies are listed
3. Check for TypeScript/linting errors

---

## 13. Rollback Procedure

### Railway

1. Go to Deployments
2. Select previous working deployment
3. Click "Redeploy"

### Vercel

1. Go to Deployments
2. Find previous deployment
3. Click "..." > "Promote to Production"

### Database

1. Restore from backup
2. Or run reverse migration

---

## 14. Post-Deployment Verification

After each deployment:

1. [ ] Health endpoint returns 200
2. [ ] Can log in to dashboard
3. [ ] API endpoints respond correctly
4. [ ] Messenger bot receives messages
5. [ ] Database queries work
6. [ ] No errors in logs

---

## Support

For deployment issues:
- Email: support@cacaosense.com
- GitHub Issues: github.com/ryanvincecastillo/cacaosense

---

## Deployment Commands Quick Reference

```bash
# Supabase
supabase db push              # Run migrations
supabase db reset             # Reset database

# Railway
railway up                    # Deploy
railway logs                  # View logs
railway run <cmd>             # Run command

# Vercel
vercel                        # Deploy preview
vercel --prod                 # Deploy production
vercel logs                   # View logs

# Local development
docker-compose up             # Start all services
```