# CLAUDE.md - Pretix Codebase Guide

This file provides guidance to Claude Code when working with the Pretix codebase.

## Project Overview

**Pretix** is a free and open-source ticket sales and event management platform written in Python and Django. It allows organizers to sell tickets for events, manage check-ins, and handle payment processing through multiple payment providers.

- **Website**: https://pretix.eu
- **Repository**: https://github.com/pretix/pretix
- **License**: AGPLv3 (with additional terms for Community vs Enterprise editions)
- **Current Version**: 2025.11.0.dev0
- **Python Support**: 3.9+
- **Django Version**: 4.2.x

## Technology Stack

### Backend
- **Framework**: Django 4.2.x
- **Language**: Python 3.9, 3.10, 3.11
- **Database**: PostgreSQL (primary), SQLite (development)
- **Cache/Sessions**: Redis (optional, via django-redis) or Memcached
- **Task Queue**: Celery 5.6.x (optional, for async tasks)
- **ORM**: Django ORM with support for PostgreSQL-specific features

### Frontend
- **Template Engine**: Django templates
- **JavaScript**: Vue.js 2.7.x with Rollup bundler
- **CSS**: Bootstrap 3 with custom SASS
- **Asset Pipeline**: django-compressor for CSS/JS optimization
- **Form Framework**: Django Forms with django-bootstrap3

### Key Dependencies
- **REST API**: Django REST Framework 3.16.x
- **Authentication**: django-oauth-toolkit 2.3.x, webauthn 2.7.x
- **Payments**: stripe 7.9.x, PayPal SDK, sepaxml 2.7.x
- **PDF Generation**: reportlab 4.4.x, pypdf 6.4.x
- **Internationalization**: django-i18nfield 1.11.x, Babel
- **Export**: openpyxl 3.1.x, vobject 0.9.x
- **Others**: bleach, lxml, requests, cryptography

## Project Architecture

```
pretix/
├── src/
│   ├── pretix/                         # Main application package
│   │   ├── __init__.py                 # Version info
│   │   ├── settings.py                 # Django settings (generated from config)
│   │   ├── _base_settings.py           # Base settings (shared with build system)
│   │   ├── _build_settings.py          # Build-time settings
│   │   ├── _build.py                   # Custom build process
│   │   ├── celery_app.py               # Celery configuration
│   │   ├── wsgi.py                     # WSGI entry point
│   │   ├── urls.py                     # Main URL router
│   │   ├── sentry.py                   # Error tracking integration
│   │   │
│   │   ├── base/                       # Core application logic
│   │   │   ├── models/                 # Core data models
│   │   │   │   ├── __init__.py         # Model exports
│   │   │   │   ├── auth.py             # User and team models
│   │   │   │   ├── event.py            # Event configuration (78KB!)
│   │   │   │   ├── organizer.py        # Organizer management
│   │   │   │   ├── orders.py           # Order model (155KB!)
│   │   │   │   ├── items.py            # Product/item models (102KB!)
│   │   │   │   ├── discount.py         # Discount/promotion logic
│   │   │   │   ├── vouchers.py         # Voucher management
│   │   │   │   ├── invoices.py         # Invoice models
│   │   │   │   ├── checkin.py          # Check-in tracking
│   │   │   │   ├── seating.py          # Seating arrangements
│   │   │   │   ├── customers.py        # Customer profiles
│   │   │   │   └── ... (more models)
│   │   │   │
│   │   │   ├── services/               # Business logic services
│   │   │   │   ├── auth.py             # Authentication service
│   │   │   │   ├── cart.py             # Shopping cart logic
│   │   │   │   ├── orders.py           # Order creation & management
│   │   │   │   ├── payments.py         # Payment processing
│   │   │   │   ├── mail.py             # Email sending
│   │   │   │   ├── invoices.py         # Invoice generation
│   │   │   │   ├── export.py           # Data export services
│   │   │   │   ├── checkin.py          # Check-in logic
│   │   │   │   ├── tickets.py          # Ticket generation
│   │   │   │   ├── vouchers.py         # Voucher services
│   │   │   │   └── ... (more services)
│   │   │   │
│   │   │   ├── payment.py              # Payment provider base class
│   │   │   ├── exporter.py             # Data exporter base class
│   │   │   ├── exporters/              # Built-in exporters
│   │   │   ├── invoicing/              # Invoice generation pipelines
│   │   │   ├── forms/                  # Reusable form classes
│   │   │   ├── auth.py                 # Custom auth backends
│   │   │   ├── middleware.py           # Request middleware
│   │   │   ├── apps.py                 # Django app config
│   │   │   └── migrations/             # Database migrations
│   │   │
│   │   ├── control/                    # Admin/management interface
│   │   │   ├── views/                  # Control panel views
│   │   │   │   ├── event.py            # Event management
│   │   │   │   ├── orders.py           # Order management
│   │   │   │   ├── items.py            # Product management
│   │   │   │   └── ... (25+ view modules)
│   │   │   ├── forms/                  # Admin forms
│   │   │   ├── navigation.py           # Admin menu generation
│   │   │   ├── permissions.py          # Permission checks
│   │   │   ├── middleware.py           # Admin-specific middleware
│   │   │   ├── logdisplay.py           # Activity log rendering
│   │   │   └── templates/              # Admin templates
│   │   │
│   │   ├── presale/                    # Customer-facing shop
│   │   │   ├── views/                  # Shop views
│   │   │   │   ├── checkout.py         # Checkout flow
│   │   │   │   ├── cart.py             # Cart management
│   │   │   │   ├── event.py            # Event page & listing
│   │   │   │   └── ... (17+ view modules)
│   │   │   ├── checkoutflow.py         # Checkout state machine (74KB!)
│   │   │   ├── forms/                  # Checkout forms
│   │   │   ├── middleware.py           # Shop-specific middleware
│   │   │   ├── utils.py                # Shop utilities
│   │   │   └── templates/              # Shop templates
│   │   │
│   │   ├── api/                        # REST API
│   │   │   ├── views/                  # API endpoints (viewsets)
│   │   │   │   ├── orders.py
│   │   │   │   ├── events.py
│   │   │   │   ├── items.py
│   │   │   │   └── ... (22+ endpoint modules)
│   │   │   ├── serializers/            # DRF serializers
│   │   │   ├── auth/                   # API authentication
│   │   │   │   ├── token.py            # Token auth
│   │   │   │   ├── device.py           # Device auth
│   │   │   │   └── permission.py       # Permission classes
│   │   │   ├── urls.py                 # API routing (v1)
│   │   │   ├── webhooks.py             # Webhook system
│   │   │   ├── middleware.py           # API middleware
│   │   │   └── migrations/             # API schema migrations
│   │   │
│   │   ├── plugins/                    # Plugin system
│   │   │   ├── __init__.py             # Entry point for plugins
│   │   │   ├── stripe/                 # Stripe payment provider
│   │   │   ├── paypal2/                # PayPal Checkout payment
│   │   │   ├── banktransfer/           # Bank transfer payment
│   │   │   ├── sendmail/               # Email notifications
│   │   │   ├── statistics/             # Event statistics
│   │   │   ├── ticketoutputpdf/        # PDF ticket output
│   │   │   ├── checkinlists/           # Check-in list generation
│   │   │   ├── badges/                 # Badge printing
│   │   │   ├── webcheckin/             # Web-based check-in
│   │   │   ├── pretixdroid/            # Pretix Droid integration
│   │   │   └── ... (14+ plugins total)
│   │   │
│   │   ├── multidomain/                # Multi-tenant support
│   │   │   ├── models.py               # KioskApplication model
│   │   │   ├── middlewares.py          # Domain routing
│   │   │   ├── urlreverse.py           # URL generation
│   │   │   ├── maindomain_urlconf.py   # Main domain routing
│   │   │   ├── event_domain_urlconf.py # Event domain routing
│   │   │   └── ... (organizer domain handlers)
│   │   │
│   │   ├── helpers/                    # Utility functions
│   │   │   ├── cache.py                # Cache utilities
│   │   │   ├── celery.py               # Celery utilities
│   │   │   ├── database.py             # Database helpers
│   │   │   ├── i18n.py                 # Translation utilities
│   │   │   ├── json.py                 # JSON handling
│   │   │   ├── jsonlogic.py            # JSON logic evaluation
│   │   │   ├── metrics.py              # Metrics/monitoring
│   │   │   ├── config.py               # Config parsing
│   │   │   └── ... (40+ helper modules)
│   │   │
│   │   ├── static/                     # Static assets
│   │   │   ├── npm_dir/                # Node.js dependencies
│   │   │   │   └── package.json        # Vue.js, Rollup config
│   │   │   ├── css/                    # SASS stylesheets
│   │   │   ├── js/                     # JavaScript files
│   │   │   └── dist/                   # Compiled assets
│   │   │
│   │   ├── locale/                     # Translations (59+ languages)
│   │   └── templates/                  # Base templates
│   │
│   ├── tests/                          # Test suite
│   │   ├── api/                        # API tests
│   │   ├── base/                       # Core functionality tests
│   │   ├── control/                    # Admin interface tests
│   │   ├── presale/                    # Shop tests
│   │   ├── plugins/                    # Plugin tests
│   │   ├── conftest.py                 # Pytest configuration
│   │   ├── settings.py                 # Test Django settings
│   │   └── concurrency_tests/          # Concurrency tests
│   │
│   ├── manage.py                       # Django CLI entry point
│   └── Makefile                        # Build commands
│
├── setup.py                            # Package configuration
├── pyproject.toml                      # Modern Python config
├── setup.cfg                           # Tool configurations
└── README.rst                          # Project README
```

## Core Concepts

### 1. Multi-Tenancy
Pretix supports hosting multiple independent organizers on a single instance:
- **Organization (Organizer)**: Top-level business entity managing events
- **Events**: Individual events created by an organizer
- **Subdomain Support**: Events/organizers can have custom domains
- **Domain Routing**: Middleware routes requests to correct event/organizer based on domain
- Location: `/src/pretix/multidomain/`

### 2. Payment Provider System
Extensible plugin architecture for payment methods:

```python
# Base class in src/pretix/base/payment.py
class BasePaymentProvider:
    """All payment providers inherit from this"""
    identifier = "unique_provider_id"
    
    def checkout_prepare(request, cart):
        """Process payment form during checkout"""
    
    def payment_is_valid_session(request):
        """Validate payment data before order creation"""
    
    def execute_payment(request, payment):
        """Actually process the payment"""
    
    def payment_refund_supported(payment):
        """Check if refund is possible"""
```

**Built-in Providers**:
- `free` - Free orders
- `manual` - Manual payment (bank transfer)
- `boxoffice` - Point of sale
- `giftcard` - Gift card payment
- `offsetting` - Internal offsetting

**Plugin Providers**:
- Stripe (credit cards, local methods)
- PayPal Checkout
- Bank transfer
- And more...

### 3. Plugin System
Dynamic plugin loading via setuptools entry points:
- Entry point group: `pretix.plugin`
- Plugin structure: Each plugin is a Django app with `apps.py` containing `PretixPluginMeta`
- Categories: `PAYMENT`, `EXPORT`, `REPORTS`, `FEATURE`
- Location: `/src/pretix/plugins/`

### 4. Order & Cart System
Complex order creation workflow:

```
Cart → Checkout Flow → Invoice Address → Payment → Order Creation
                                        ↓
                                  Payment Processing
                                        ↓
                                   Order Status
```

Key models: `CartPosition`, `Order`, `OrderPosition`, `OrderPayment`, `OrderRefund`
Services: `cart.py`, `orders.py`, `payments.py`

### 5. Multi-Domain/Multi-Tenant Architecture
- **Main domain**: Admin panel, organizer management
- **Event domains**: `https://events.example.com/eventslug/`
- **Custom organizer domain**: `https://organizer-custom.com/`
- Middleware strips domain and sets `request.event`/`request.organizer`

## Development Setup

### Installation
```bash
# Clone repository
git clone https://github.com/pretix/pretix.git
cd pretix

# Install in development mode
pip install -e .
pip install -e '.[dev]'

# Optional: Install memcached/redis support
pip install -e '.[memcached]'
```

### Configuration
Configuration via INI file (default locations):
- `/etc/pretix/pretix.cfg`
- `~/.pretix.cfg`
- `./pretix.cfg` (in current directory)

Or via environment variable: `PRETIX_CONFIG_FILE=/path/to/config.cfg`

Key config sections:
```ini
[pretix]
url = http://localhost:8000
registration = false
debug = true
plugins_default = pretix.plugins.sendmail,pretix.plugins.statistics

[database]
backend = postgresql
name = pretix
user = pretix
password = secret
host = localhost

[redis]
location = redis://localhost:6379/0

[celery]
broker = redis://localhost:6379/1
backend = redis://localhost:6379/2
```

### Running Development Server
```bash
# Using Django
cd src
python manage.py runserver

# Using Makefile
cd src
make localecompile staticfiles
python manage.py runserver

# Run background worker (if Celery configured)
python manage.py celery worker

# Run scheduler
python manage.py celery beat
```

### Building Assets
```bash
cd src

# Compile messages (translations)
make localegen          # Extract new translatable strings
make localecompile      # Compile .po files to .mo

# Static files
make staticfiles        # Collect and build assets
make compress           # Minify CSS/JS (production)

# Full build
make all                # localegen + staticfiles
make production         # localegen + staticfiles + compress
```

## Testing

### Running Tests
```bash
cd src

# Run all tests
python -m pytest tests

# Run specific test
python -m pytest tests/base/test_orders.py

# With coverage
coverage run -m pytest tests
coverage report
coverage html

# Parallel testing with xdist
python -m pytest tests -n 4
```

### Test Configuration
- Settings: `src/tests/settings.py`
- Fixtures: `src/tests/conftest.py`
- Database: SQLite by default, PostgreSQL via `ci_postgres.cfg`
- Fixtures use: pytest, pytest-django, fakeredis

### Test Structure
```
tests/
├── api/               # API endpoint tests
├── base/              # Core model/service tests
├── control/           # Admin interface tests
├── presale/           # Shop/checkout tests
├── plugins/           # Plugin-specific tests
└── conftest.py        # Pytest hooks & fixtures
```

## Key Development Patterns

### 1. Creating a Model
```python
# src/pretix/base/models/mymodel.py
from django.db import models
from .base import LoggingMixin

class MyModel(LoggingMixin, models.Model):
    event = models.ForeignKey('Event')
    name = models.CharField(max_length=255)
    created = models.DateTimeField(auto_now_add=True)
    
    class Meta:
        unique_together = [('event', 'name')]
```

Then add to migrations and register in `models/__init__.py`.

### 2. Creating a Service
```python
# src/pretix/base/services/myservice.py
import logging

logger = logging.getLogger(__name__)

def do_something(order):
    """
    Docstring with description.
    
    :param order: Order instance
    :return: Result
    """
    # Implementation
    return result
```

Import in `apps.py` `ready()` method to register.

### 3. Creating a View (Admin)
```python
# src/pretix/control/views/myview.py
from django.contrib.auth.mixins import LoginRequiredMixin
from django.views.generic import ListView
from pretix.control.permissions import EventPermissionRequired

class MyListView(LoginRequiredMixin, EventPermissionRequired, ListView):
    model = MyModel
    paginate_by = 25
    permission = 'can_change_event_settings'
    
    def get_queryset(self):
        return self.request.event.mymodels.all()
```

Register in `control/urls.py`.

### 4. Creating a Payment Provider Plugin
```python
# pretix_mypayment/pretix_mypayment/__init__.py
# or src/pretix/plugins/mypayment/__init__.py

from django.apps import AppConfig
from django.utils.translation import gettext_lazy as _
from pretix import __version__

class MyPaymentApp(AppConfig):
    name = 'pretix.plugins.mypayment'
    verbose_name = _("My Payment Method")
    
    class PretixPluginMeta:
        name = _("My Payment")
        author = _("Author Name")
        version = __version__
        category = 'PAYMENT'
        description = _("Description of payment method")
        featured = False
    
    def ready(self):
        from . import signals  # Register signal handlers
        from .payment import MyPaymentProvider
        from pretix.base.signals import register_payment_providers
        
        register_payment_providers.connect(lambda sender, **kwargs: [MyPaymentProvider])
```

### 5. Registering Signals
```python
# src/pretix/base/signals.py defines signal types
# In your module:

from django.dispatch import receiver
from pretix.base.signals import order_paid

@receiver(order_paid, dispatch_uid="my_handler")
def handle_order_paid(sender, order, **kwargs):
    # order is an Order instance
    # Sender is the model class
    pass
```

### 6. Creating an Exporter
```python
# Inherit from BaseExporter in src/pretix/base/exporter.py

class MyExporter(BaseExporter):
    identifier = 'myexporter'
    verbose_name = _('My Exporter')
    
    @property
    def export_form_fields(self):
        return OrderedDict([
            ('date_from', forms.DateField()),
        ])
    
    def render(self, form_data):
        # Return file content (bytes)
        return b'CSV data'
```

## Important Files & Patterns

### Configuration & Settings
- `src/pretix/settings.py` - Main Django settings (auto-generated from config file)
- `src/pretix/_base_settings.py` - Base settings shared with build system
- `src/pretix/celery_app.py` - Celery configuration
- `src/tests/settings.py` - Test-specific settings

### Database Models (Core Data)
- `src/pretix/base/models/auth.py` - User, Team, TeamMember
- `src/pretix/base/models/organizer.py` - Organizer, OrganizerMember
- `src/pretix/base/models/event.py` - Event, EventPermission
- `src/pretix/base/models/orders.py` - Order, OrderPosition, OrderPayment, OrderRefund
- `src/pretix/base/models/items.py` - Item, ItemVariation, ItemCategory
- `src/pretix/base/models/vouchers.py` - Voucher
- `src/pretix/base/models/checkin.py` - Checkin, CheckinList

### Business Logic (Services)
- `src/pretix/base/services/orders.py` - Order creation workflow
- `src/pretix/base/services/cart.py` - Shopping cart logic
- `src/pretix/base/services/payments.py` - Payment processing
- `src/pretix/base/services/mail.py` - Email sending
- `src/pretix/base/services/invoices.py` - Invoice generation
- `src/pretix/base/services/export.py` - Data export

### URL Routing
- `src/pretix/urls.py` - Main URL config entry point
- `src/pretix/multidomain/maindomain_urlconf.py` - Main domain routing
- `src/pretix/control/urls.py` - Admin panel URLs
- `src/pretix/presale/urls.py` - Shop URLs
- `src/pretix/api/urls.py` - API v1 endpoints

### Middleware
Ordered in `settings.py`:
1. `RequestIdMiddleware` - Request tracking
2. `IdempotencyMiddleware` - Idempotent API requests
3. `MultiDomainMiddleware` - Domain → event/organizer routing
4. `SessionMiddleware` - Session handling
5. `CsrfViewMiddleware` - CSRF protection
6. `AuthenticationMiddleware` - User authentication
7. `MessageMiddleware` - Django messages
8. `PermissionMiddleware` - Permission checking
9. `AuditLogMiddleware` - Activity logging
10. `EventMiddleware` - Event context extraction

### Signals (Django Signals)
Located in `src/pretix/base/signals.py`:
- `order_placed` - New order created
- `order_paid` - Order payment confirmed
- `order_pending` - Order became pending
- `register_payment_providers` - Load payment plugins
- And many more...

## PostgreSQL Compatibility

Pretix is optimized for PostgreSQL with custom compatibility for SQLite:
- **Advisory locks** for pessimistic locking: `database.advisory_lock()`
- **Database-specific features** in query paths
- Migrations handle PostgreSQL jsonb fields
- Set `USE_DATABASE_TLS` for SSL connections

## Common Commands

### Django Management Commands
```bash
cd src

# Database
python manage.py migrate
python manage.py makemigrations
python manage.py dbshell

# Users
python manage.py shell
>>> from pretix.base.models import User
>>> User.objects.create_superuser('admin', 'admin@localhost', 'password')

# Media
python manage.py clearmedia

# Cache
python manage.py clear_cache

# Development
python manage.py runserver 0.0.0.0:8000
python manage.py shell_plus  # With django-extensions
```

### Useful Utilities
```python
# In shell or scripts
from pretix.base.models import Event, Organizer, Order
from pretix.base.services.orders import create_order
from pretix.base.payment import PaymentException

# Query examples
org = Organizer.objects.get(slug='acme')
event = org.events.get(slug='conference2024')
orders = event.orders.filter(status='p')  # Paid orders

# Create order programmatically
from pretix.base.services.cart import get_or_create_cart_id
# See OrderCreationService in services/orders.py
```

## Debugging

### Logging
- Default log file: `data/logs/pretix.log`
- Log config in `settings.py` LOGGING dict
- Levels: DEBUG, INFO, WARNING, ERROR, CRITICAL
- Request tracking via `RequestIdHeader` (if configured)

### Performance
- Use Django Debug Toolbar (in DEBUG mode)
- Check slow queries: `django.db.backends` logger
- Profile code: `python manage.py runprofile`
- Monitor Celery tasks: `celery inspect active`

### Errors
- Sentry integration if `[sentry]` configured in config file
- Custom error templates in `control/templates/pretix/`
- CSP violations logged to `logs/csp.log`

## Security Considerations

- **Passwords**: Argon2 hashing by default (configurable)
- **Sessions**: HTTPS-only in production
- **CSRF**: Token-based protection on all forms
- **XSS**: Bleach HTML sanitization, template escaping
- **SQL Injection**: Django ORM usage prevents this
- **2FA**: TOTP and WebAuthn support built-in
- **Audit Log**: All admin actions logged to LogEntry

## Additional Resources

- **Documentation**: https://docs.pretix.eu
- **Blog**: https://pretix.eu/about/en/blog/
- **Community**: https://github.com/pretix/pretix/discussions
- **Issue Tracker**: https://github.com/pretix/pretix/issues
- **Contributing Guide**: https://docs.pretix.eu/dev/development/contribution/

## Code Style

- **Linter**: flake8 (config in `setup.cfg`)
- **Formatter**: isort for imports
- **Max line length**: 160 characters for flake8, 79 for isort
- **Docstrings**: Use triple quotes with descriptions
- **Type hints**: Gradually being added to codebase

## Architecture Highlights

1. **Service-Oriented**: Business logic in `services/` modules, not models
2. **Signals-Based**: Loose coupling via Django signals
3. **Plugin-First**: Core features designed as plugins
4. **Multi-Tenant**: Built from ground up for multiple organizers
5. **Payment Agnostic**: Pluggable payment providers
6. **Audit Trail**: Everything is logged and tracked
7. **Scalable**: Designed for Celery + Redis/RabbitMQ
8. **i18n Ready**: 59+ languages supported, all user-facing text translatable

---

Last Updated: 2025-01-04
Pretix Version: 2025.11.0.dev0
