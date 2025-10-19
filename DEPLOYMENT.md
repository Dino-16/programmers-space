# CyberPanel Deployment Guide

## Prerequisites

1. CyberPanel server with SSH access
2. Domain/subdomain created in CyberPanel
3. Git installed on server
4. Composer installed on server

## Step 1: Prepare Your CyberPanel Server

### 1.1 Install Required Software

SSH into your CyberPanel server:

```bash
ssh root@your-server-ip
```

Install Git and Composer if not already installed:

```bash
# Install Git
yum install git -y  # For CentOS/AlmaLinux
# or
apt install git -y  # For Ubuntu

# Install Composer
curl -sS https://getcomposer.org/installer | php
mv composer.phar /usr/local/bin/composer
chmod +x /usr/local/bin/composer
```

### 1.2 Create Website in CyberPanel

1. Login to CyberPanel (https://your-server-ip:8090)
2. Go to **Websites → Create Website**
3. Enter your domain (e.g., `programmers-space.com`)
4. Select PHP version **8.2** or higher
5. Click **Create Website**

### 1.3 Set Up Git Repository

The website files are typically located at:
```
/home/yourdomain.com/public_html
```

Navigate to your website directory:

```bash
cd /home/yourdomain.com
```

Clone your repository:

```bash
# Remove default public_html
rm -rf public_html

# Clone your repo
git clone https://github.com/Dino-16/programmers-space.git public_html

# Enter the directory
cd public_html
```

### 1.4 Install Dependencies

```bash
composer install --no-dev --optimize-autoloader
```

### 1.5 Configure Environment

```bash
# Copy environment file
cp .env.example .env

# Generate application key
php artisan key:generate

# Edit .env file with your database credentials
nano .env
```

Update these values in `.env`:
```env
APP_NAME="Programmers Space"
APP_ENV=production
APP_DEBUG=false
APP_URL=https://yourdomain.com

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=your_database_name
DB_USERNAME=your_database_user
DB_PASSWORD=your_database_password
```

### 1.6 Set Up Database

Create database in CyberPanel:
1. Go to **Databases → Create Database**
2. Select your website
3. Enter database name, username, and password
4. Click **Create Database**

Run migrations:
```bash
php artisan migrate --force
```

### 1.7 Set Permissions

```bash
chmod -R 755 storage bootstrap/cache
chown -R nobody:nobody storage bootstrap/cache
php artisan storage:link
```

### 1.8 Configure Document Root

In CyberPanel, you need to point the document root to the `public` directory:

1. Go to **Websites → List Websites**
2. Click **Manage** on your website
3. Click **vHost Conf**
4. Find the line with `documentRoot` and change it to:
   ```
   documentRoot /home/yourdomain.com/public_html/public
   ```
5. Click **Save vHost Configuration**
6. Restart OpenLiteSpeed:
   ```bash
   systemctl restart lsws
   ```

## Step 2: Set Up GitHub Secrets

Go to your GitHub repository → **Settings → Secrets and variables → Actions** → **New repository secret**

Add these secrets:

1. **SSH_HOST**: Your server IP or domain (e.g., `123.45.67.89`)
2. **SSH_USER**: SSH username (usually `root` for CyberPanel)
3. **SSH_KEY**: Your private SSH key
4. **DEPLOY_PATH**: Full path to your Laravel app (e.g., `/home/yourdomain.com/public_html`)

### Generate SSH Key (if needed)

On your local machine:

```bash
ssh-keygen -t rsa -b 4096 -C "github-actions"
```

Copy the **private key** to GitHub Secret `SSH_KEY`:
```bash
cat ~/.ssh/id_rsa
```

Copy the **public key** to your server:
```bash
ssh-copy-id root@your-server-ip
```

Or manually add it to `/root/.ssh/authorized_keys` on the server.

## Step 3: Deploy

### Manual Deployment

1. Go to GitHub → **Actions** tab
2. Select **Laravel CI/CD** workflow
3. Click **Run workflow**
4. Select `main` branch
5. Click **Run workflow**

### Automatic Deployment

The workflow is currently set to only deploy on manual trigger. To enable automatic deployment on every push:

Edit `.github/workflows/deploy.yml` and change:
```yaml
if: github.event_name == 'workflow_dispatch'
```
to:
```yaml
if: github.event_name == 'push' || github.event_name == 'workflow_dispatch'
```

## Troubleshooting

### Permission Denied Errors

```bash
chmod -R 755 storage bootstrap/cache
chown -R nobody:nobody storage bootstrap/cache
```

### 500 Internal Server Error

Check Laravel logs:
```bash
tail -f storage/logs/laravel.log
```

Check OpenLiteSpeed error logs:
```bash
tail -f /usr/local/lsws/logs/error.log
```

### Composer Memory Issues

```bash
php -d memory_limit=-1 /usr/local/bin/composer install --no-dev
```

### Database Connection Issues

Verify database credentials in `.env` and ensure the database exists in CyberPanel.

## Post-Deployment Checklist

- [ ] Website loads correctly
- [ ] Database migrations ran successfully
- [ ] Storage directory is writable
- [ ] SSL certificate is installed (use CyberPanel's free SSL)
- [ ] `.env` file has correct production settings
- [ ] `APP_DEBUG=false` in production
- [ ] Cron jobs set up (if needed)

## Maintenance

### Clear Cache

```bash
php artisan cache:clear
php artisan config:clear
php artisan route:clear
php artisan view:clear
```

### Update Application

```bash
cd /home/yourdomain.com/public_html
git pull origin main
composer install --no-dev --optimize-autoloader
php artisan migrate --force
php artisan config:cache
php artisan route:cache
php artisan view:cache
```
