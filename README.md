# Krishaka Mitram

**Karnataka Farmer-to-Buyer Agricultural Marketplace with Real-Time Chat & Moderation**

[![Django](https://img.shields.io/badge/Django-5.0%2B-green)](https://www.djangoproject.com/)
[![Python](https://img.shields.io/badge/Python-3.12-blue)](https://www.python.org/)
[![License](https://img.shields.io/badge/License-MIT-yellow)](LICENSE)

---

## Project Overview

Krishaka Mitram (ಕೃಷಕ ಮಿತ್ರಂ — "Farmer's Friend") is a Django-based agricultural marketplace connecting marginal farmers in Karnataka (Dakshina Kannada, Udupi, and surrounding regions) directly with buyers. The platform eliminates middlemen, enables fair price discovery, and provides transparent produce listings with real-time communication and content moderation.

## Implemented Features

### Core Marketplace

- **Farmer Listings** — Create, update, delete produce listings with crop type, price, quantity, quality grade, farm size, location
- **Crop Catalog** — 10 major Karnataka crops (Arecanut, Coconut, Paddy, Cashew, Maize, Jowar, Ragi, Sugarcane, Toor, Cotton) with local images
- **Buyer Inquiries** — Express interest in listings with quantity, budget, and message
- **My Listings & Inquiries** — Dashboards for farmers and buyers to manage their posts and applications

### Real-Time Chat System

- **WebSocket Conversations** — Django Channels with Redis backend for production, InMemory for development
- **Farmer-Buyer Chat** — Initiated from listing inquiries or farmer detail pages
- **Typing Indicators** — Real-time typing status
- **Read Receipts** — Message read tracking
- **HTTP Fallback** — AJAX message sending when WebSocket unavailable
- **Authorization Gates** — Only conversation participants can join/chat

### Content Moderation & Safety

- **Multi-Layer Filtering Engine** — Detects phone numbers, emails, UPI IDs, bank details (IFSC, account, card), URLs, addresses, prohibited keywords
- **Kannada/English Support** — Phonetic number detection in both languages
- **Filter Actions** — Block (prevent send), Warn (flag & allow), Log (record only)
- **Incident Lifecycle** — NEW → VIEWED → PENDING_ACTION → RESOLVED with atomic transitions
- **Sensitive Data Purge** — Automatic purging after configurable retention (default 30 days)
- **Moderator Dashboard** — Real-time violation alerts, review workflow, bulk actions
- **User Restrictions** — Temporary/permanent chat restrictions with violation tracking

### Notifications & Audit

- **Real-Time Notifications** — WebSocket + database for chat messages, violations, restrictions
- **Audit Logging** — Append-only log of all security-sensitive actions (login, moderation, restrictions, rule changes)
- **Privacy-First** — Sensitive content never stored in audit logs; only metadata recorded

### Internationalization

- **Bilingual UI** — English/Kannada (ಕನ್ನಡ) with client-side language switching
- **Localizable Templates** — `data-i18n` attributes throughout
- **Kannada Phonetic Detection** — Filters recognize numbers spelled in Kannada (ondhu, eradu, mooru...)

### Security & Production Hardening

- **Role-Based Access** — Farmer, Buyer, Moderator, Owner roles with granular permissions
- **Production Security** — HSTS, secure cookies, CSP headers, X-Frame-Options, referrer policy
- **CSRF Protection** — All forms and AJAX endpoints protected
- **Environment-Driven Config** — SECRET_KEY, DEBUG, DATABASE_URL, REDIS_URL via environment
- **WhiteNoise** — Compressed static file serving with manifest

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                        Django Project (kisan)                   │
├─────────────────────────────────────────────────────────────────┤
│  ┌─────────┐  ┌─────────┐  ┌──────┐  ┌───────────┐  ┌────────┐ │
│  │accounts │  │ farmer  │  │ chat │  │moderation │  │audit   │ │
│  └────┬────┘  └────┬────┘  └──┬───┘  └─────┬─────┘  └────┬────┘ │
│       │            │          │            │             │      │
│       └────────────┴──────────┴────────────┴─────────────┘      │
│                            │                                      │
│              ┌─────────────┴─────────────┐                       │
│              │      Django ORM           │                       │
│              │   (SQLite dev / PG prod)  │                       │
│              └─────────────┬─────────────┘                       │
│                            │                                      │
│              ┌─────────────┴─────────────┐                       │
│              │     Django Channels       │                       │
│              │  (Redis prod / InMemory)  │                       │
│              └───────────────────────────┘                       │
└─────────────────────────────────────────────────────────────────┘
```

### Applications

| App             | Purpose                                         | Key Models                                                                   |
| --------------- | ----------------------------------------------- | ---------------------------------------------------------------------------- |
| `accounts`      | Custom user model, authentication               | `CustomUser` (roles: farmer, buyer, moderator, owner)                        |
| `farmer`        | Produce listings, crop catalog, buyer inquiries | `Crop`, `Farmer`, `Buyer` (inquiry)                                          |
| `chat`          | Real-time messaging, conversations              | `Conversation`, `Message`                                                    |
| `moderation`    | Content filtering, violations, restrictions     | `FilterRule`, `ModerationIncident`, `ModerationViolation`, `UserRestriction` |
| `notifications` | User notifications                              | `Notification` (generic FK)                                                  |
| `audit`         | Security audit trail                            | `AuditLog`                                                                   |

---

## Technology Stack

| Layer           | Technology                           | Version         |
| --------------- | ------------------------------------ | --------------- |
| Web Framework   | Django                               | 5.0 – 7.0 (LTS) |
| ASGI Server     | Daphne                               | 4.2+            |
| Real-Time       | Django Channels                      | 4.3+            |
| Channel Layer   | channels-redis (Redis)               | 4.3+            |
| In-Memory (Dev) | channels.layers.InMemoryChannelLayer | Built-in        |
| Database        | SQLite (dev) / PostgreSQL (prod)     | Django ORM      |
| Static Files    | WhiteNoise                           | 6.6+            |
| Phone Parsing   | phonenumbers                         | 9.0+            |
| Crypto          | cryptography                         | 42.0+           |
| Config          | dj-database-url                      | 2.0+            |
| Python          | CPython                              | 3.12+           |

---

## Repository Structure

```
Krishaka Mitram/
├── accounts/                 # Custom user model & auth
│   ├── models.py             # CustomUser with roles & restriction fields
│   ├── views.py              # signup, login_view, logout_view
│   ├── urls.py               # /accounts/signup/, /login/, /logout/
│   ├── forms.py              # CustomUserCreationForm
│   ├── admin.py              # CustomUserAdmin
│   └── migrations/
├── farmer/                   # Marketplace core
│   ├── models.py             # Crop, Farmer (listing), Buyer (inquiry)
│   ├── views.py              # ListView, DetailView, CRUD, inquiries
│   ├── urls.py               # /, /farmers/, /farmer/<pk>/, /my-listings/, /my-inquiries/
│   ├── forms.py              # Farmerform
│   ├── crop_images/          # Local crop images (served as static)
│   ├── management/commands/  # load_agri_data
│   └── migrations/
├── chat/                     # Real-time chat
│   ├── models.py             # Conversation, Message
│   ├── consumers.py          # ChatConsumer, NotificationConsumer, ModeratorConsumer
│   ├── routing.py            # WebSocket URL patterns
│   ├── views.py              # chat_list, chat_detail, start_chat, send_message
│   ├── urls.py               # /chat/ namespace
│   ├── templates/chat/
│   └── migrations/
├── moderation/               # Content filtering & moderation
│   ├── models.py             # FilterRule, ModerationIncident, ModerationViolation, UserRestriction
│   ├── filters.py            # MessageFilteringEngine (comprehensive detection)
│   ├── signals.py            # Real-time moderator notifications
│   ├── views.py              # Dashboard, violation detail, filter rules CRUD
│   ├── urls.py               # /moderation/
│   ├── management/commands/  # purge_expired_moderation_data
│   ├── templates/moderation/
│   └── migrations/
├── notifications/            # User notifications
│   ├── models.py             # Notification (generic FK)
│   ├── consumers.py          # NotificationConsumer
│   ├── views.py              # List, mark read, API count
│   ├── urls.py               # /notifications/
│   ├── templates/notifications/
│   └── migrations/
├── audit/                    # Audit logging
│   ├── models.py             # AuditLog
│   ├── admin.py
│   └── migrations/
├── kisan/                    # Project settings
│   ├── settings.py           # Main settings (env-driven)
│   ├── test_settings.py      # Test-only settings (in-memory, no Redis)
│   ├── urls.py               # Root URLconf
│   ├── asgi.py               # ASGI application (Channels)
│   ├── wsgi.py               # WSGI application
│   └── __init__.py
├── templates/                # Base templates
│   └── base.html             # Main template with i18n, cookie consent, footer
├── staticfiles/              # Collected static (generated)
├── farmer/crop_images/       # Source crop images
├── requirements.txt          # Production dependencies
├── render.yaml               # Render.com deployment config
├── build.sh                  # Build script for Render
├── manage.py                 # Django management script
├── db.sqlite3                # Development database (gitignored in prod)
├── karnataka_agri_data.json  # Seed data for load_agri_data
├── .gitignore
├── LICENSE
├── README.md                 # This file
├── docs/
│   ├── ARCHITECTURE.md
│   ├── DEPLOYMENT.md
│   ├── DEVELOPMENT.md
│   └── SECURITY_AUDIT.md
└── .claude/                  # Claude Code config (local)
```

---

## Prerequisites

- **Python** 3.12 or higher
- **Redis** server (for production WebSockets) — Docker: `docker run -d -p 6380:6379 --name krishaka-redis redis:7-alpine`
- **PostgreSQL** 14+ (for production) — or use SQLite for development
- **Git** for version control

---

## Local Development Setup

### Windows PowerShell

```powershell
# 1. Clone and navigate
git clone <repository-url>
cd "Krishaka Mitram"

# 2. Create virtual environment
python -m venv .venv
.\.venv\Scripts\Activate.ps1

# 3. Upgrade pip and install dependencies
python -m pip install --upgrade pip
pip install -r requirements.txt

# 4. Start Redis (required for WebSocket chat)
# Option A: Docker (recommended, uses port 6380 to avoid conflicts)
docker run -d -p 6380:6379 --name krishaka-redis redis:7-alpine

# Option B: Native Redis on default port 6379
# redis-server

# 5. Set environment variables (PowerShell)
$env:DEBUG = "true"
$env:SECRET_KEY = "django-insecure-dev-only-key-change-in-production"
$env:REDIS_URL = "redis://127.0.0.1:6380/0"   # Matches Docker mapping
# $env:DATABASE_URL = "postgres://user:pass@localhost:5432/dbname"  # Optional: for PostgreSQL

# 6. Run migrations
python manage.py migrate

# 7. (Optional) Load Karnataka agricultural seed data
python manage.py load_agri_data

# 8. Create superuser (optional)
python manage.py createsuperuser

# 9. Run development server
python manage.py runserver

# 10. In a SEPARATE terminal, run Daphne for WebSocket support
# (activate venv first: .\.venv\Scripts\Activate.ps1)
daphne -b 127.0.0.1 -p 8001 kisan.asgi:application
```

**Access:**

- HTTP: `http://127.0.0.1:8000/`
- WebSocket: `ws://127.0.0.1:8001/ws/chat/<conversation_id>/`

> **Note:** The Django dev server (`runserver`) handles HTTP. Daphne handles WebSockets on a separate port. For a unified dev experience, you can run Daphne on port 8000 for both HTTP and WebSocket: `daphne -b 127.0.0.1 -p 8000 kisan.asgi:application`

### Linux / macOS

```bash
# 1. Clone and navigate
git clone <repository-url>
cd "Krishaka Mitram"

# 2. Create virtual environment
python3 -m venv .venv
source .venv/bin/activate

# 3. Upgrade pip and install dependencies
python -m pip install --upgrade pip
pip install -r requirements.txt

# 4. Start Redis (required for WebSocket chat)
# Option A: Docker (recommended, uses port 6380)
docker run -d -p 6380:6379 --name krishaka-redis redis:7-alpine

# Option B: Native Redis
# redis-server

# 5. Set environment variables
export DEBUG=true
export SECRET_KEY="django-insecure-dev-only-key-change-in-production"
export REDIS_URL="redis://127.0.0.1:6380/0"
# export DATABASE_URL="postgres://user:pass@localhost:5432/dbname"  # Optional

# 6. Run migrations
python manage.py migrate

# 7. (Optional) Load Karnataka agricultural seed data
python manage.py load_agri_data

# 8. Create superuser (optional)
python manage.py createsuperuser

# 9. Run development server
python manage.py runserver

# 10. In a SEPARATE terminal, run Daphne for WebSocket support
# (activate venv first: source .venv/bin/activate)
daphne -b 127.0.0.1 -p 8001 kisan.asgi:application
```

---

## Environment Variable Configuration

| Variable                    | Required       | Default                    | Description                                                     |
| --------------------------- | -------------- | -------------------------- | --------------------------------------------------------------- |
| `DEBUG`                     | No             | `False`                    | Set `true` for development                                      |
| `SECRET_KEY`                | **Yes (prod)** | —                          | Django secret key (50+ random chars)                            |
| `ALLOWED_HOSTS`             | No             | `*`                        | Comma-separated hosts (auto-appends `RENDER_EXTERNAL_HOSTNAME`) |
| `DATABASE_URL`              | No             | SQLite                     | PostgreSQL connection string (prod)                             |
| `REDIS_URL`                 | **Yes (prod)** | `redis://localhost:6379/0` | Redis for Channels (prod uses 6379, dev uses 6380)              |
| `RENDER_EXTERNAL_HOSTNAME`  | Auto           | —                          | Set by Render; adds to `ALLOWED_HOSTS` & `CSRF_TRUSTED_ORIGINS` |
| `VIOLATION_AUTO_PURGE_DAYS` | No             | `30`                       | Days before purging sensitive moderation data                   |

### `.env.example` (create for local use)

```bash
# Copy to .env and customize for local development
DEBUG=true
SECRET_KEY=django-insecure-dev-only-key-change-in-production
REDIS_URL=redis://127.0.0.1:6380/0
# DATABASE_URL=postgres://user:pass@localhost:5432/krishaka_mitram
VIOLATION_AUTO_PURGE_DAYS=30
```

---

## Database & Migrations

```bash
# Create migrations (after model changes)
python manage.py makemigrations

# Apply migrations
python manage.py migrate

# Show migration status
python manage.py showmigrations

# Check for pending migrations (CI)
python manage.py makemigrations --check --dry-run
```

**Important:** Never delete or squash existing migrations. The project uses standard Django migrations:

- `accounts`: 2 migrations (CustomUser + restriction fields)
- `farmer`: 3 migrations (Crop, Farmer, Buyer inquiries)
- `chat`: 2 migrations (Conversation, Message; removed `original_content`)
- `moderation`: 3 migrations (FilterRule, Incident, Violation, Restriction)
- `notifications`: 1 migration
- `audit`: 2 migrations (added `created_at` rename from `timestamp`)

---

## Redis & Django Channels Setup

### Local Development (Port 6380)

The project uses **host port 6380 → container port 6379** to avoid conflict with another project using port 6379.

```bash
# Start Redis container
docker run -d -p 6380:6379 --name krishaka-redis redis:7-alpine

# Verify
docker exec krishaka-redis redis-cli ping
# Should return PONG
```

### Channel Layer Configuration

**Development (`DEBUG=True`):**

```python
CHANNEL_LAYERS = {
    'default': {
        'BACKEND': 'channels.layers.InMemoryChannelLayer',
    }
}
```

**Production (`DEBUG=False`):**

```python
CHANNEL_LAYERS = {
    'default': {
        'BACKEND': 'channels_redis.core.RedisChannelLayer',
        'CONFIG': {
            'hosts': [os.environ.get('REDIS_URL', 'redis://localhost:6379/0')],
            'capacity': 1500,
            'expiry': 10,
            'group_expiry': 86400,
            'symmetric_encryption_keys': [SECRET_KEY],
        },
    }
}
```

> **Note:** The `symmetric_encryption_keys` setting uses `SECRET_KEY` for message encryption. The Redis connection uses `socket_timeout=None` (default) which works with `redis-py>=5.0` and `channels-redis>=4.3`.

### WebSocket Testing

```bash
# With Daphne running on port 8001
# Connect via browser dev tools or CLI:
wscat -c ws://127.0.0.1:8001/ws/chat/1/
```

---

## Running Tests

```bash
# Full test suite (87 tests)
python manage.py test --settings=kisan.test_settings --verbosity=2

# Individual app tests
python manage.py test accounts --settings=kisan.test_settings -v2
python manage.py test farmer --settings=kisan.test_settings -v2
python manage.py test chat --settings=kisan.test_settings -v2
python manage.py test moderation --settings=kisan.test_settings -v2
python manage.py test notifications --settings=kisan.test_settings -v2
python manage.py test audit --settings=kisan.test_settings -v2

# System checks
python manage.py check --settings=kisan.test_settings
python manage.py check --deploy --settings=kisan.test_settings
```

**Test Settings (`kisan/test_settings.py`):**

- In-memory SQLite database (`:memory:`)
- In-memory channel layer (no Redis required)
- Fast MD5 password hasher
- Disabled HTTPS redirects and secure cookies
- Local memory email backend

---

## Production Deployment

### Render.com (Current Configuration)

The project includes `render.yaml` and `build.sh` for Render deployment:

```yaml
# render.yaml
services:
  - type: web
    name: krishaka-mitram
    runtime: python
    plan: free
    buildCommand: "./build.sh"
    startCommand: "daphne -b 0.0.0.0 -p $PORT kisan.asgi:application"
    envVars:
      - key: PYTHON_VERSION
        value: 3.12.0
      - key: SECRET_KEY
        generateValue: true
      - key: DEBUG
        value: "False"
      - key: ALLOWED_HOSTS
        value: "*"
      - key: REDIS_URL
        sync: false # Add Redis add-on and link here
```

```bash
# build.sh
#!/usr/bin/env bash
set -o errexit
pip install --upgrade pip
pip install -r requirements.txt
python manage.py collectstatic --no-input
python manage.py migrate
python manage.py load_agri_data
```

**Required Render Setup:**

1. Create a **Redis add-on** (or external Redis)
2. Link `REDIS_URL` environment variable to the Redis add-on
3. Set `SECRET_KEY` (auto-generated) and `DEBUG=False`
4. Custom domain → updates `ALLOWED_HOSTS` automatically

### Manual Linux Production Deployment

```bash
# 1. Server preparation
sudo apt update && sudo apt install -y python3.12 python3.12-venv postgresql redis nginx

# 2. Create database
sudo -u postgres createuser --pwprompt krishaka
sudo -u postgres createdb -O krishaka krishaka_mitram

# 3. Clone and setup
git clone <repo> /opt/krishaka-mitram
cd /opt/krishaka-mitram
python3.12 -m venv venv
source venv/bin/activate
pip install --upgrade pip
pip install -r requirements.txt

# 4. Environment file (/opt/krishaka-mitram/.env)
cat > .env << 'EOF'
DEBUG=False
SECRET_KEY=<generate-with: python -c "import secrets; print(secrets.token_urlsafe(50))">
DATABASE_URL=postgres://krishaka:password@localhost:5432/krishaka_mitram
REDIS_URL=redis://localhost:6379/0
ALLOWED_HOSTS=yourdomain.com,www.yourdomain.com
VIOLATION_AUTO_PURGE_DAYS=30
EOF

# 5. Django setup
python manage.py migrate
python manage.py collectstatic --no-input
python manage.py load_agri_data
python manage.py createsuperuser

# 6. Systemd service for Daphne (ASGI)
cat > /etc/systemd/system/krishaka-mitram.service << 'EOF'
[Unit]
Description=Krishaka Mitram Daphne ASGI
After=network.target redis.service postgresql.service

[Service]
Type=simple
User=www-data
Group=www-data
WorkingDirectory=/opt/krishaka-mitram
EnvironmentFile=/opt/krishaka-mitram/.env
ExecStart=/opt/krishaka-mitram/venv/bin/daphne -b 127.0.0.1 -p 8000 kisan.asgi:application
Restart=on-failure

[Install]
WantedBy=multi-user.target
EOF

# 7. Nginx reverse proxy (with WebSocket support)
cat > /etc/nginx/sites-available/krishaka-mitram << 'EOF'
server {
    listen 80;
    server_name yourdomain.com www.yourdomain.com;

    location /static/ {
        alias /opt/krishaka-mitram/staticfiles/;
        expires 1y;
        add_header Cache-Control "public, immutable";
    }

    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_read_timeout 86400;
    }
}
EOF

# 8. Enable and start
sudo ln -s /etc/nginx/sites-available/krishaka-mitram /etc/nginx/sites-enabled/
sudo nginx -t && sudo systemctl reload nginx
sudo systemctl enable --now krishaka-mitram
sudo systemctl enable --now redis
sudo systemctl enable --now postgresql
```

---

## Static Files & Media

| Setting               | Value                                                     |
| --------------------- | --------------------------------------------------------- |
| `STATIC_URL`          | `/static/`                                                |
| `STATICFILES_DIRS`    | `BASE_DIR / 'farmer' / 'crop_images'`                     |
| `STATIC_ROOT`         | `BASE_DIR / 'staticfiles'`                                |
| `STATICFILES_STORAGE` | `whitenoise.storage.CompressedManifestStaticFilesStorage` |
| `MEDIA_URL`           | `/media/` (not configured — user uploads not implemented) |
| `MEDIA_ROOT`          | `BASE_DIR / 'media'` (not configured)                     |

**Collect static files:**

```bash
python manage.py collectstatic --no-input
```

---

## Security Considerations

### Production Hardening (Auto-enabled when `DEBUG=False`)

- `SECURE_SSL_REDIRECT = True`
- `SESSION_COOKIE_SECURE = True`
- `CSRF_COOKIE_SECURE = True`
- `SESSION_COOKIE_HTTPONLY = True`
- `CSRF_COOKIE_HTTPONLY = True`
- `SECURE_HSTS_SECONDS = 31536000` (1 year)
- `SECURE_HSTS_INCLUDE_SUBDOMAINS = True`
- `SECURE_HSTS_PRELOAD = True`
- `SECURE_CONTENT_TYPE_NOSNIFF = True`
- `SECURE_BROWSER_XSS_FILTER = True`
- `X_FRAME_OPTIONS = 'DENY'`
- `SECURE_REFERRER_POLICY = 'strict-origin-when-cross-origin'`

### Authentication & Authorization

- Custom user model with 4 roles (farmer, buyer, moderator, owner)
- `LoginRequiredMixin` + `UserPassesTestMixin` on all protected views
- WebSocket consumers verify authentication AND conversation participation
- Moderator-only WebSocket group (`moderators`) with role check
- Account restrictions automatically enforced (`is_currently_restricted()`)

### Content Security

- **No raw sensitive content in Message model** — only filtered content stored
- **ModerationIncident.blocked_content** — temporary, purged on first view
- **ModerationViolation.detected_content** — purged after retention period
- **AuditLog** — append-only, stores only metadata (never sensitive content)
- **Filter engine** — skips empty patterns, deduplicates violations

### CSRF & XSS Protection

- All forms use `{% csrf_token %}`
- AJAX endpoints receive CSRF token via `X-CSRFToken` header
- Template auto-escaping enabled (Django default)
- User content rendered as plain text (no `|safe` filters on messages)

### Secrets Management

- **Never commit secrets** — `.env`, `*.key`, `*.pem`, `secrets.yml` in `.gitignore`
- `SECRET_KEY` from environment only (dev fallback for management commands only)
- Render generates `SECRET_KEY` automatically
- Database credentials via `DATABASE_URL` environment variable

---

## Logging, Backups & Maintenance

### Logging

Production logging configured in `kisan/settings.py` (extend as needed):

```python
LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'handlers': {
        'console': {'class': 'logging.StreamHandler'},
        'file': {'class': 'logging.FileHandler', 'filename': 'logs/django.log'},
    },
    'root': {'handlers': ['console', 'file'], 'level': 'INFO'},
    'loggers': {
        'django': {'handlers': ['console', 'file'], 'level': 'INFO'},
        'chat': {'handlers': ['console', 'file'], 'level': 'INFO'},
        'moderation': {'handlers': ['console', 'file'], 'level': 'INFO'},
    },
}
```

### Database Backups

```bash
# PostgreSQL backup
pg_dump -U krishaka -h localhost krishaka_mitram > backup_$(date +%F).sql

# Restore
psql -U krishaka -h localhost krishaka_mitram < backup_2026-10-19.sql

# SQLite backup (dev only)
cp db.sqlite3 db_backup_$(date +%F).sqlite3
```

### Moderation Data Retention

```bash
# Dry run (see what would be purged)
python manage.py purge_expired_moderation_data --dry-run

# Force purge (ignore schedule)
python manage.py purge_expired_moderation_data --force

# Scheduled purge (respects VIOLATION_AUTO_PURGE_DAYS)
python manage.py purge_expired_moderation_data
```

**Schedule via cron (daily at 2 AM):**

```cron
0 2 * * * /opt/krishaka-mitram/venv/bin/python /opt/krishaka-mitram/manage.py purge_expired_moderation_data >> /var/log/krishaka_purge.log 2>&1
```

---

## Troubleshooting

| Issue                             | Cause                           | Solution                                                        |
| --------------------------------- | ------------------------------- | --------------------------------------------------------------- |
| `SECRET_KEY` error on `manage.py` | Missing env var                 | Set `DEBUG=true` or `SECRET_KEY` env var                        |
| WebSocket connection fails        | Redis not running / wrong port  | Start Redis on 6380 (dev) or 6379 (prod); check `REDIS_URL`     |
| `ALLOWED_HOSTS` error             | Host not in list                | Add domain to `ALLOWED_HOSTS` or set `RENDER_EXTERNAL_HOSTNAME` |
| 301 redirects in tests            | `SECURE_SSL_REDIRECT=True`      | Use `kisan.test_settings` (disables HTTPS)                      |
| CSRF failure on AJAX              | Missing token                   | Include `X-CSRFToken` header from `csrfmiddlewaretoken`         |
| Migrations fail                   | Model changes without migration | Run `makemigrations` then `migrate`                             |
| Static files 404                  | Not collected                   | Run `collectstatic`; check `STATIC_ROOT` in nginx               |
| "Another project uses port 6379"  | Port conflict                   | Use port 6380 for this project's Redis container                |

---

## Contributing

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/your-feature`
3. Make changes with tests
4. Run tests: `python manage.py test --settings=kisan.test_settings`
5. Run checks: `python manage.py check --settings=kisan.test_settings`
6. Submit a Pull Request

### Code Style

- Follow PEP 8 (use `black` and `flake8` if desired)
- Type hints encouraged for new code
- Docstrings for public methods
- Keep templates DRY — use `base.html` blocks

### Commit Messages

```
<type>(<scope>): <description>

[optional body]

[optional footer]
```

Types: `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`

---

## License

This project is licensed under the **MIT License** — see [LICENSE](LICENSE) for details.

Copyright (c) 2026 Pannaga G Bhat

---

## Support & Resources

- **Kisan Helpline:** 1800-180-1551
- **Raita Mitra Karnataka:** https://raitamitra.karnataka.gov.in
- **PM-Kisan Samman Nidhi:** https://pmkisan.gov.in
- **Karnataka APMC Mandi Rates:** https://krishimaratavahini.karnataka.gov.in
- **KVK Brahmavar / Mangaluru:** https://kvk.icar.gov.in
- **Soil Health Card Scheme:** https://soilhealth.dac.gov.in
#   k r i s h a k a  
 