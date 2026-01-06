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

# Celery Configuration (CRITICAL - prevents AMQP errors!)
# Pretix reads these via EnvOrParserConfig with PRETIX_ prefix
PRETIX_CELERY_BROKER=${{Redis.REDIS_URL}}/1
PRETIX_CELERY_BACKEND=${{Redis.REDIS_URL}}/2

# Performance Tuning (Important for Railway!)
# NUM_WORKERS controls Gunicorn worker processes
# Default: 2 × CPU cores (can be 8-16 workers on Railway!)
# Recommended by pretix: 2-4 × number of CPU cores
# Railway limitation: Memory is limited, so we need fewer workers
NUM_WORKERS=2

# Alternative: Use MAX_WORKERS (introduced in pretix 3.14.0)
# MAX_WORKERS=2
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

### Out of Memory Errors (Workers being killed with SIGKILL)

**Symptom**: Logs show `[ERROR] Worker (pid:XXXX) was sent SIGKILL! Perhaps out of memory?`

**Cause**: By default, pretix spawns `2 * CPU_CORES` Gunicorn workers. On Railway with 8 vCPUs, this means 16 workers, which can exceed memory during startup.

**Solution**: Set the `NUM_WORKERS` environment variable to limit workers based on your Railway plan:

**Railway Usage-Based Plans**:
- **Up to 8GB RAM / 8 vCPU**: `NUM_WORKERS=4` to `NUM_WORKERS=8` (recommended: start with 4)
- **Up to 4GB RAM / 4 vCPU**: `NUM_WORKERS=2` to `NUM_WORKERS=4`
- **Up to 2GB RAM / 2 vCPU**: `NUM_WORKERS=2` to `NUM_WORKERS=3`
- **Up to 1GB RAM / 1 vCPU**: `NUM_WORKERS=2`

**Pretix Recommendation**: 2-4 × number of CPU cores available

Add this to your Railway environment variables and redeploy.

### Celery AMQP Connection Error

**Symptom**: `consumer: Cannot connect to amqp://guest:**@127.0.0.1:5672//: [Errno 111] Connection refused`

**Cause**: Celery is trying to connect to RabbitMQ (AMQP) instead of Redis because the broker configuration is missing.

**Solution**: **CRITICAL** - Add these environment variables in Railway:
```env
PRETIX_CELERY_BROKER=${{Redis.REDIS_URL}}/1
PRETIX_CELERY_BACKEND=${{Redis.REDIS_URL}}/2
```

Without these, pretix will either:
1. Try to use RabbitMQ (AMQP) by default → Connection refused errors
2. Fall back to `CELERY_TASK_ALWAYS_EAGER=True` → No background tasks (bad for performance)

### Celery Hostname Warning

**Symptom**: `No hostname was supplied. Reverting to default 'localhost'`

**Status**: This is just a warning and won't affect functionality. Celery workers will use 'localhost' as hostname.

**Optional Fix**: Add `PRETIX_CELERY_WORKER_HOSTNAME` environment variable if you want custom hostname.

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

## Performance Optimization

### Recommended Settings for 8GB RAM / 8 vCPU Plan

Based on pretix documentation (2-4 × CPU cores) and your Railway plan:

```env
# Start conservative, then increase
NUM_WORKERS=4   # Good balance for 8 vCPU

# If stable and need more performance:
# NUM_WORKERS=6  # Higher performance
# NUM_WORKERS=8  # Maximum for 8 vCPU (monitor memory usage)
```

### Memory Usage Reference

According to pretix discussions:
- RAM usage per worker is **mostly constant** even under high load
- General purpose instances: 2-4 GB RAM per CPU core is typical
- Your 8GB/8vCPU = 1GB per core (tight but workable with tuning)

### Monitoring

Watch your Railway metrics dashboard:
- If memory usage < 70%: Can increase `NUM_WORKERS`
- If memory usage > 85%: Reduce `NUM_WORKERS`
- CPU usage should be < 80% under normal load

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

# Check worker status (from logs)
railway logs --filter "Booting worker"
```

## Support

For Pretix-specific issues: https://docs.pretix.eu
For Railway issues: https://docs.railway.com
