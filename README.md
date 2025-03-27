# Simplified Laravel Deployment Guide (Ubuntu EC2)

This guide outlines the essential steps to deploy Laravel 8.3+ on Ubuntu, using either Nginx or Apache.

---

## Prerequisites

Before starting, ensure you have the following:

- An Ubuntu EC2 instance running.
- SSH access to your EC2 instance.
- Laravel project code (e.g., in a Git repository).
- AWS RDS database details (endpoint, username, password, database name).

---

## Option 1: Apache Setup

### Step 1: Update Your Server

```bash
sudo apt update
sudo apt upgrade -y
```

**Explanation:** Updates package lists and upgrades installed software.

---

### Step 2: Install Apache, PHP, and Tools

```bash
# Install Apache web server
sudo apt install apache2 -y

# Add repository for latest PHP versions
sudo apt install software-properties-common -y
sudo add-apt-repository ppa:ondrej/php -y
sudo apt update

# Install PHP 8.3 and necessary extensions
sudo apt install php8.3 libapache2-mod-php8.3 php8.3-mysql php8.3-mbstring php8.3-xml php8.3-curl php8.3-zip php8.3-bcmath php8.3-intl php8.3-gd -y

# Install Composer (PHP package manager)
curl -sS https://getcomposer.org/installer | sudo php -- --install-dir=/usr/local/bin --filename=composer

# Install Git
sudo apt install git -y
```

**Explanation:** Installs Apache, PHP 8.3, Laravel-required extensions, Composer, and Git.

---

### Step 3: Enable Apache Rewrite Module

```bash
sudo a2enmod rewrite
sudo systemctl restart apache2
```

**Explanation:** Enables Apache's `mod_rewrite` for Laravel's routing.

---

### Step 4: Configure Apache for Laravel

```bash
sudo nano /etc/apache2/sites-available/laravel.conf
```

Paste the following configuration, replacing `your_domain_or_public_ip` with your EC2's public IP or domain:

```apache
<VirtualHost *:80>
    ServerName your_domain_or_public_ip
    DocumentRoot /var/www/laravel/public

    <Directory /var/www/laravel/public>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/laravel-error.log
    CustomLog ${APACHE_LOG_DIR}/laravel-access.log combined
</VirtualHost>
```

---

### Step 5: Enable Apache Config and Restart

```bash
sudo a2ensite laravel.conf
sudo a2dissite 000-default.conf
sudo apache2ctl configtest
sudo systemctl restart apache2
```

**Explanation:** Activates the Laravel site configuration and restarts Apache.

---

### Step 6: Get Your Laravel Code

```bash
sudo mkdir -p /var/www/laravel
cd /var/www/laravel
sudo git clone https://github.com/yourusername/your-repo.git .
```

**Explanation:** Creates the project directory and downloads your Laravel code.

---

### Step 7: Set Permissions

```bash
sudo chown -R www-data:www-data /var/www/laravel
sudo chmod -R 755 /var/www/laravel
sudo chmod -R 775 /var/www/laravel/storage
sudo chmod -R 775 /var/www/laravel/bootstrap/cache
```

**Explanation:** Ensures proper ownership and permissions for Laravel.

---

### Step 8: Configure Laravel

```bash
cd /var/www/laravel
sudo cp .env.example .env
sudo nano .env
```

Update the `.env` file with your RDS database details:

```dotenv
APP_ENV=production
APP_DEBUG=false
APP_URL=http://your_domain_or_public_ip

DB_CONNECTION=mysql
DB_HOST=your-rds-endpoint.rds.amazonaws.com
DB_PORT=3306
DB_DATABASE=your_database_name
DB_USERNAME=your_db_username
DB_PASSWORD=your_db_password
```

Run the following commands:

```bash
sudo composer install --no-dev --optimize-autoloader
sudo php artisan key:generate
sudo php artisan config:cache
sudo php artisan route:cache
sudo php artisan view:cache
```

---

### Step 9: Verify Deployment

Visit `http://your_domain_or_public_ip` in your browser to see your Laravel application.

---

## Troubleshooting

1. **Permissions Issue:** Re-run the `chown` and `chmod` commands.
2. **Check Logs:**
   - Apache: `sudo tail -f /var/log/apache2/laravel-error.log`
   - Laravel: `sudo tail -f /var/www/laravel/storage/logs/laravel.log`
3. **Database Connection:** Verify `.env` settings and ensure Security Groups allow MySQL traffic.
4. **Restart Services:**
   ```bash
   sudo systemctl restart apache2
   ```

Good luck with your Laravel deployment!
