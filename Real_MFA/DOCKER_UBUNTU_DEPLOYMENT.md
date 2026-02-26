# Real_MFA Docker Deployment on Ubuntu Droplet

This guide deploys Real_MFA with Docker Compose using:
- Django + Gunicorn
- PostgreSQL
- Redis
- Celery worker + Celery beat
- Nginx (port 80)

## 1) Install Docker on Ubuntu

```bash
sudo apt update
sudo apt install -y ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo $VERSION_CODENAME) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo usermod -aG docker $USER
```

Log out and log in again so Docker group permissions apply.

## 2) Clone project and prepare environment

```bash
git clone https://github.com/your-username/Real_MFA.git ~/real_mfa
cd ~/real_mfa/Real_MFA
cp .env.docker.example .env.docker
nano .env.docker
```

Minimum required edits in `.env.docker`:
- `SECRET_KEY`
- `DB_PASSWORD`
- `ALLOWED_HOSTS`
- `CORS_ALLOWED_ORIGINS`
- `CSRF_TRUSTED_ORIGINS`
- `EMAIL_*`

## 3) Build and run stack

```bash
docker compose -f docker-compose.ubuntu.yml up -d --build
```

Check status:

```bash
docker compose -f docker-compose.ubuntu.yml ps
docker compose -f docker-compose.ubuntu.yml logs -f web
```

## 4) Create Django superuser

```bash
docker compose -f docker-compose.ubuntu.yml exec web python manage.py createsuperuser
```

## 5) Open firewall ports (if UFW enabled)

```bash
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw status
```

## 6) HTTPS options

Current stack serves HTTP on port 80. For production HTTPS you can:
- put Cloudflare in front with Full (strict) TLS, or
- terminate TLS on host Nginx/Caddy and proxy to `127.0.0.1:80`, or
- extend compose with Certbot.

## 7) Common commands

```bash
# Restart services
docker compose -f docker-compose.ubuntu.yml restart

# Stop services
docker compose -f docker-compose.ubuntu.yml down

# Update deployment
git pull origin main
docker compose -f docker-compose.ubuntu.yml up -d --build

# View logs
docker compose -f docker-compose.ubuntu.yml logs -f web
docker compose -f docker-compose.ubuntu.yml logs -f celery
docker compose -f docker-compose.ubuntu.yml logs -f celery_beat
docker compose -f docker-compose.ubuntu.yml logs -f nginx
```
