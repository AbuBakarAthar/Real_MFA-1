# Real_MFA - Multi-Factor Authentication System

A production-ready Django REST Framework application implementing comprehensive Multi-Factor Authentication (MFA) with OTP, TOTP, device verification, and email-based verification.

## 🚀 Features

- **Email-based Authentication** - Secure user registration and login
- **Email Verification** - Mandatory email verification before login
- **OTP (One-Time Password)** - SMS-based one-time passwords via Twilio
- **TOTP (Time-based OTP)** - Google Authenticator / Authy app support
- **Device Management** - Register and manage trusted devices
- **Session Management** - Track and manage user sessions
- **Audit Logging** - Complete audit trail of user activities
- **JWT Authentication** - Stateless authentication with refresh tokens
- **Rate Limiting** - Built-in protection against brute force attacks
- **PostgreSQL** - Production-grade database
- **Celery** - Asynchronous task processing
- **Redis** - Caching and message broker

## 📦 Tech Stack

- **Backend:** Django 4.2, Django REST Framework
- **Database:** PostgreSQL (required for production)
- **Cache/Queue:** Redis
- **Task Queue:** Celery + Celery Beat
- **Authentication:** JWT + Custom backends
- **Web Server:** Gunicorn + Nginx
- **OS:** Ubuntu 20.04+ / DigitalOcean Droplet

## 🏃 Quick Start (Development)

### 1. Clone Repository

```bash
git clone https://github.com/your-username/Real_MFA.git
cd Real_MFA
```

### 2. Create Virtual Environment

```bash
python3.11 -m venv venv
source venv/bin/activate
```

### 3. Install Dependencies

```bash
pip install -r requirements.txt
```

### 4. Setup Environment Variables

```bash
cp .env.example .env
# Edit .env with your settings
nano .env
```

**Required variables for development:**
```env
ENVIRONMENT=development
DEBUG=True
SECRET_KEY=django-insecure-your-development-key
ALLOWED_HOSTS=localhost,127.0.0.1

# For SQLite development (not for production!)
DB_ENGINE=django.db.backends.sqlite3
DB_NAME=db.sqlite3

# Email (console backend for development)
EMAIL_BACKEND=django.core.mail.backends.console.EmailBackend

# Redis (optional for development)
REDIS_HOST=localhost
REDIS_PORT=6379
CELERY_TASK_ALWAYS_EAGER=True
```

### 5. Run Migrations

```bash
python manage.py migrate
```

### 6. Create Superuser

```bash
python manage.py createsuperuser
```

### 7. Start Development Server

```bash
python manage.py runserver
```

Visit: http://localhost:8000/admin/

## 🐧 Production Deployment (Ubuntu/DigitalOcean)

### Prerequisites

- Ubuntu 20.04 LTS or newer
- Min 2GB RAM, 20GB storage
- Domain name (for SSL)
- Static IP address

### Quick Deployment

1. **Read the deployment guide first:**
   ```bash
   docs/DIGITALOCEAN_DROPLET_COMPLETE_SETUP.md
   ```

2. **Follow step-by-step setup:**
   - Phase 1: Initial server setup (30 mins)
   - Phase 2: Python & dependencies (15 mins)
   - Phase 3: PostgreSQL database (15 mins)
   - Phase 4: Clone & setup Real_MFA (30 mins)
   - Phase 5: Gunicorn & Celery (20 mins)
   - Phase 6: Nginx setup (15 mins)
   - Phase 7: SSL/HTTPS (10 mins)
   - Phase 8: Health checks (5 mins)

3. **Use deployment automation:**
   ```bash
   # Copy config files
   sudo cp config/real_mfa_gunicorn.service /etc/systemd/system/
   sudo cp config/real_mfa_celery.service /etc/systemd/system/
   sudo cp config/real_mfa_celery_beat.service /etc/systemd/system/
   
   # Run deployment
   bash config/deploy.sh
   
   # Verify
   bash config/monitor.sh
   ```

### Docker Deployment (Recommended for Droplets)

```bash
cp .env.docker.example .env.docker
# edit .env.docker with production values

docker compose -f docker-compose.ubuntu.yml up -d --build
docker compose -f docker-compose.ubuntu.yml exec web python manage.py createsuperuser
```

Detailed guide: `DOCKER_UBUNTU_DEPLOYMENT.md`

### Environment Variables for Production

See `.env.example` for all variables. Key production settings:

```bash
# Django
ENVIRONMENT=production
DEBUG=False
SECRET_KEY=<generate-with-get_random_secret_key()>
ALLOWED_HOSTS=your-domain.com,www.your-domain.com

# Database
DB_ENGINE=django.db.backends.postgresql
DB_NAME=real_mfa_db
DB_USER=real_mfa_user
DB_PASSWORD=<strong-password>
DB_HOST=localhost

# Redis
REDIS_HOST=localhost
REDIS_PORT=6379

# Email
EMAIL_HOST=smtp.gmail.com
EMAIL_HOST_USER=your-email@gmail.com
EMAIL_HOST_PASSWORD=<app-specific-password>

# Security
SECURE_SSL_REDIRECT=True
SESSION_COOKIE_SECURE=True
CSRF_COOKIE_SECURE=True
```

## 📚 Documentation

- [Deployment Guide](docs/DIGITALOCEAN_DROPLET_COMPLETE_SETUP.md) - Complete Ubuntu setup
- [Docker Ubuntu Deployment](DOCKER_UBUNTU_DEPLOYMENT.md) - Docker Compose deployment on droplet
- [Quick Reference](config/QUICK_REFERENCE.md) - Common commands
- [Pre-Deployment Checklist](config/PRE_DEPLOYMENT_CHECKLIST.md) - Final verification
- [Architecture Overview](docs/README_REFACTORED_ARCHITECTURE.md) - System design
- [Database Design](docs/README_DATABASE_DESIGN.md) - Schema documentation

## 🔧 Common Commands

### Development

```bash
# Run tests
python manage.py test

# Create migrations
python manage.py makemigrations

# Run migrations
python manage.py migrate

# Create superuser
python manage.py createsuperuser

# Shell access
python manage.py shell

# Clear cache
python manage.py shell
>>> from django.core.cache import cache
>>> cache.clear()
```

### Production

```bash
# Deploy updates
bash config/deploy.sh

# Health check
bash config/monitor.sh

# Database backup
bash config/backup.sh

# View logs
sudo tail -f /var/log/real_mfa_gunicorn_error.log
sudo tail -f /var/log/real_mfa_celery_worker.log

# Check services
sudo systemctl status real_mfa_gunicorn
sudo systemctl status real_mfa_celery
sudo systemctl status real_mfa_celery_beat
```

## 📊 API Endpoints

All endpoints require authentication (JWT token)

### Authentication
- `POST /api/register/` - User registration
- `POST /api/login/` - User login
- `POST /api/logout/` - User logout
- `POST /api/refresh/` - Refresh JWT token

### Email Verification
- `POST /api/send-verification/` - Send verification email
- `POST /api/verify-email/` - Verify email

### OTP (SMS)
- `POST /api/otp/send/` - Send OTP via SMS
- `POST /api/otp/verify/` - Verify OTP

### TOTP (Google Authenticator)
- `POST /api/totp/setup/` - Setup TOTP
- `POST /api/totp/verify/` - Verify TOTP
- `POST /api/totp/disable/` - Disable TOTP

### Devices
- `GET /api/devices/` - List devices
- `POST /api/devices/` - Register device
- `DELETE /api/devices/{id}/` - Remove device

### Profile
- `GET /api/profile/` - Get user profile
- `PUT /api/profile/` - Update profile
- `POST /api/password/change/` - Change password
- `POST /api/password/reset/` - Reset password

## 🔒 Security Features

- ✅ HTTPS/SSL enforced
- ✅ CSRF protection enabled
- ✅ CORS properly configured
- ✅ Rate limiting on API
- ✅ Input validation
- ✅ Secure password hashing
- ✅ JWT token blacklisting
- ✅ Session management
- ✅ Audit logging
- ✅ Secure headers (HSTS, X-Frame-Options, CSP)

## 🐛 Troubleshooting

### Gunicorn not starting
```bash
sudo systemctl status real_mfa_gunicorn
sudo journalctl -u real_mfa_gunicorn -n 50
```

### Database connection error
```bash
psql -U real_mfa_user -d real_mfa_db -h localhost
```

### Celery tasks not running
```bash
redis-cli ping  # Should return PONG
celery -A Real_MFA inspect active
```

### Static files not loading
```bash
python manage.py collectstatic --noinput --clear
sudo systemctl restart real_mfa_gunicorn
```

See [QUICK_REFERENCE.md](config/QUICK_REFERENCE.md) for more troubleshooting.

## 📝 Project Structure

```
Real_MFA/
├── accounts/              # User authentication & profile
├── devices/               # Device management
├── otp/                   # OTP & TOTP implementation
├── notification/          # Email notifications
├── audits_logs/           # Audit trail
├── Real_MFA/              # Django settings
│   ├── settings.py       # Production-aware settings
│   ├── celery.py         # Celery configuration
│   ├── wsgi.py           # WSGI application
│   └── urls.py           # URL routing
├── config/                # Deployment configurations
├── docs/                  # Documentation
├── requirements.txt       # Python dependencies
├── .env.example          # Environment template
├── manage.py             # Django CLI
└── README.md             # This file
```

## 🤝 Contributing

1. Create a feature branch
2. Make your changes
3. Write tests
4. Submit a pull request

## 📄 License

MIT License - See LICENSE file for details

## 👨‍💼 Support

For issues and questions:
1. Check [QUICK_REFERENCE.md](config/QUICK_REFERENCE.md)
2. Read [DIGITALOCEAN_DROPLET_COMPLETE_SETUP.md](docs/DIGITALOCEAN_DROPLET_COMPLETE_SETUP.md)
3. Review [PRODUCTION_READY_CHECKLIST.md](PRODUCTION_READY_CHECKLIST.md)

## 🚀 Deployment Links

- **Droplet IP:** 143.110.139.119 (Example - replace with yours)
- **Domain:** your-domain.com (Setup required)
- **Admin Panel:** https://your-domain.com/admin/

## 📋 Quick Links

- [Production Checklist](PRODUCTION_READY_CHECKLIST.md)
- [Deployment Guide](docs/DIGITALOCEAN_DROPLET_COMPLETE_SETUP.md)
- [Quick Commands](config/QUICK_REFERENCE.md)
- [Environment Template](.env.example)

---

**Status:** ✅ Production Ready

**Last Updated:** February 20, 2026

**Version:** 1.0
