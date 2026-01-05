# 🚂 Deploying Pretix to Railway

## Prerequisites

- Railway account
- PostgreSQL service
- Redis service

## Setup Instructions

### 1. Create Railway Project

1. Go to [Railway](https://railway.app)
2. Click "New Project"
3. Select "Deploy from GitHub repo"
4. Choose this repository

### 2. Add Services

Add these services to your project:

#### PostgreSQL
- Click "New" → "Database" → "Add PostgreSQL"

#### Redis
- Click "New" → "Database" → "Add Redis"

### 3. Configure Volumes (Important!)

Railway requires explicit volume configuration:

1. Go to your service settings
2. Click "Volumes"
3. Add two volumes:
   - Mount path: `/data` (for media files, database backups)
   - Mount path: `/etc/pretix` (for configuration)

### 4. Environment Variables

Add these variables to your Pretix service:

```env
# Database (auto-filled by Railway PostgreSQL)
DATABASE_URL=${{Postgres.DATABASE_URL}}

# Redis (auto-filled by Railway Redis)
REDIS_URL=${{Redis.REDIS_URL}}

# Pretix Configuration
PRETIX_INSTANCE_NAME=your-event-name
PRETIX_URL=https://your-app.up.railway.app
PRETIX_CURRENCY=COP
PRETIX_REGISTRATION=False

# Locale
PRETIX_LOCALE_DEFAULT=es-419
PRETIX_TIMEZONE=America/Bogota

# Django
DJANGO_DEBUG=False
DJANGO_SECRET_KEY=<generate-a-random-secret-key>

# Email (optional - configure SMTP)
PRETIX_MAIL_FROM=noreply@yourdomain.com
PRETIX_MAIL_HOST=smtp.sendgrid.net
PRETIX_MAIL_PORT=587
PRETIX_MAIL_USER=apikey
PRETIX_MAIL_PASSWORD=<your-sendgrid-api-key>
PRETIX_MAIL_TLS=True

# Celery
CELERY_BROKER_URL=${{Redis.REDIS_URL}}/1
CELERY_RESULT_BACKEND=${{Redis.REDIS_URL}}/2
```

### 5. Generate Secret Key

Generate a secure Django secret key:

```bash
python -c "from django.core.management.utils import get_random_secret_key; print(get_random_secret_key())"
```

### 6. Port Configuration

Railway automatically detects port 80 from the Dockerfile.

### 7. Deploy

1. Click "Deploy"
2. Wait for build to complete
3. Run migrations (first time only):
   - Go to service → "Settings" → "Deploy"
   - Or use Railway CLI

### 8. Create Admin User

After first deployment, create superuser:

```bash
railway run python -m pretix shell
```

Then in the shell:
```python
from pretix.base.models import User
User.objects.create_superuser('admin@localhost', 'your-password')
```

## Volume Mounting

The app expects these directories to persist:

- `/data` - Media files, uploads, cache
- `/etc/pretix` - Configuration files

Make sure to mount Railway volumes at these paths.

## Important Notes

⚠️ **Volume Declaration Removed**: The original `VOLUME` declaration in Dockerfile has been removed for Railway compatibility. Railway requires explicit volume configuration through the dashboard.

⚠️ **Database**: Use Railway's PostgreSQL (MySQL/MariaDB not supported by Pretix anymore)

⚠️ **Static Files**: Built during Docker build process

## Troubleshooting

### Build fails
- Check that all services (PostgreSQL, Redis) are running
- Verify environment variables are set correctly

### App crashes on startup
- Check logs: `railway logs`
- Verify database connection
- Ensure migrations ran successfully

### Static files not loading
- Rebuild the app to regenerate static files
- Check nginx configuration in `/etc/nginx/nginx.conf`

## Useful Commands

```bash
# View logs
railway logs

# Run migrations
railway run python -m pretix migrate

# Create superuser
railway run python -m pretix shell

# Rebuild static files
railway run python -m pretix rebuild
```

## Support

For Pretix-specific issues: https://docs.pretix.eu
For Railway issues: https://docs.railway.com
