# Quick CyberPanel Setup (Manual)

## Step 1: SSH into Your Server

```bash
ssh root@mrj.jetlougetravels-ph.com
# or
ssh root@YOUR_SERVER_IP
```

## Step 2: Navigate to Website Directory

```bash
cd /home/mrj.jetlougetravels-ph.com
```

## Step 3: Remove Empty Directory and Clone Repository

```bash
# Remove the empty programmers-space.com directory
rm -rf programmers-space.com

# Clone your repository
git clone https://github.com/Dino-16/programmers-space.git programmers-space.com

# Enter the directory
cd programmers-space.com
```

## Step 4: Install Dependencies

```bash
composer install --no-dev --optimize-autoloader
```

## Step 5: Set Up Environment

```bash
# Copy environment file
cp .env.example .env

# Generate application key
php artisan key:generate

# Edit .env file
nano .env
```

Update these values in `.env`:
```env
APP_NAME="Programmers Space"
APP_ENV=production
APP_DEBUG=false
APP_URL=https://programmers-space.com

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=programmers_space_db
DB_USERNAME=programmers_space_user
DB_PASSWORD=your_secure_password
```

Press `CTRL+X`, then `Y`, then `ENTER` to save.

## Step 6: Create Database in CyberPanel

1. Open CyberPanel: `https://YOUR_SERVER_IP:8090`
2. Go to **Databases → Create Database**
3. Select website: `programmers-space.com`
4. Database name: `programmers_space_db`
5. Username: `programmers_space_user`
6. Password: (generate a secure password)
7. Click **Create Database**

## Step 7: Run Migrations

```bash
php artisan migrate --force
```

## Step 8: Set Permissions

```bash
chmod -R 755 storage bootstrap/cache
chown -R nobody:nobody storage bootstrap/cache
php artisan storage:link
```

## Step 9: Cache Configuration

```bash
php artisan config:cache
php artisan route:cache
php artisan view:cache
```

## Step 10: Verify Document Root

1. In CyberPanel, go to **Websites → List Websites**
2. Click **Manage** on `programmers-space.com`
3. Click **vHost Conf**
4. Find `documentRoot` and ensure it points to:
   ```
   documentRoot /home/mrj.jetlougetravels-ph.com/programmers-space.com/public
   ```
5. Click **Save vHost Configuration**
6. Restart OpenLiteSpeed:
   ```bash
   systemctl restart lsws
   ```

## Done!

Visit your website: `https://programmers-space.com`

## Future Deployments

After this initial setup, you can use GitHub Actions to deploy automatically:
- Go to GitHub → Actions → Laravel CI/CD → Run workflow
