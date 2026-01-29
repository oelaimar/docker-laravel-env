# Laravel Docker Environment Setup

A complete Docker environment for running Laravel applications with Nginx, MySQL, PHP-FPM, and phpMyAdmin.

## 📋 Prerequisites

Before you start, ensure you have the following installed on your system:

- **Docker** (version 20.10+) - [Install Docker](https://docs.docker.com/get-docker/)
- **Docker Compose** (version 1.29+) - [Install Docker Compose](https://docs.docker.com/compose/install/)
- **Git** (optional, for cloning repositories)

## 🚀 Quick Start

### 1. Clone or Download the Project

```bash
git clone <your-repo-url> oelaimar-docker-laravel-env
cd oelaimar-docker-laravel-env
```

### 2. Create Laravel Application

If you don't have a Laravel application yet, create one in the `src` directory:

```bash
# Option 1: Using Laravel Installer
composer create-project laravel/laravel src

# Option 2: Using Docker (recommended)
docker-compose exec app composer create-project laravel/laravel .
```

### 3. Build and Start Containers

```bash
# Build and start all containers in the background
docker-compose up -d

# Or, to see logs in real-time
docker-compose up
```

### 4. Install Dependencies

```bash
# Install PHP dependencies
docker-compose exec app composer install

# Install Node.js dependencies
docker-compose exec app npm install

# Build frontend assets
docker-compose exec app npm run build
```

### 5. Configure Environment

```bash
# Copy the example environment file
docker-compose exec app cp .env.example .env

# Generate application key
docker-compose exec app php artisan key:generate
```

### 6. Run Database Migrations

```bash
# Run migrations
docker-compose exec app php artisan migrate

# (Optional) Seed the database
docker-compose exec app php artisan db:seed
```

## 🌐 Access Your Application

Once everything is running, you can access:

- **Laravel Application**: http://localhost:8000
- **phpMyAdmin**: http://localhost:8080

## 🗂️ Project Structure

```
oelaimar-docker-laravel-env/
├── docker-compose.yml          # Docker Compose configuration
├── docker/
│   ├── nginx/
│   │   └── default.conf        # Nginx configuration
│   └── php/
│       └── Dockerfile          # PHP-FPM Dockerfile
└── src/                        # Your Laravel application (create this)
```

## 🐳 Services Overview

### App (PHP-FPM)
- **Image**: PHP 8.3-FPM
- **Container**: `project_name_app`
- **Working Directory**: `/var/www`
- **Includes**:
  - PHP Extensions: PDO, MySQL, mbstring, zip, exif, pcntl
  - Composer (latest version 2)
  - Node.js 20 + npm
  - Git, curl, zip utilities

### Web (Nginx)
- **Image**: Nginx Alpine
- **Container**: `project_name_nginx`
- **Port**: `8000:80`
- **Configuration**: `/etc/nginx/conf.d/default.conf`
- **Root**: `/var/www/public`

### Database (MySQL)
- **Image**: MySQL 8.0
- **Container**: `project_name_db`
- **Port**: `3306:3306`
- **Database**: `laravel`
- **Root User**: `root` / `root`
- **Application User**: `laravel` / `secret`
- **Persistent Volume**: `db_data`

### phpMyAdmin
- **Image**: phpMyAdmin
- **Container**: `project_name_pma`
- **Port**: `8080:80`
- **Default User**: `laravel` / `secret`

## 📝 Common Commands

### Docker Compose Commands

```bash
# Start containers
docker-compose up -d

# Stop containers
docker-compose down

# View logs
docker-compose logs -f

# View logs for a specific service
docker-compose logs -f app

# Rebuild containers
docker-compose up -d --build
```

### Laravel Commands

```bash
# Execute Artisan commands
docker-compose exec app php artisan <command>

# Examples:
docker-compose exec app php artisan migrate
docker-compose exec app php artisan tinker
docker-compose exec app php artisan queue:work
docker-compose exec app php artisan make:model Post -mcr
```

### Composer Commands

```bash
docker-compose exec app composer require <package>
docker-compose exec app composer update
docker-compose exec app composer dump-autoload
```

### NPM Commands

```bash
docker-compose exec app npm install
docker-compose exec app npm run dev
docker-compose exec app npm run build
docker-compose exec app npm run watch
```

### Database Commands

```bash
# Access MySQL command line
docker-compose exec db mysql -u laravel -p laravel

# Create a database backup
docker-compose exec db mysqldump -u laravel -p laravel > backup.sql

# Restore from backup
docker-compose exec -T db mysql -u laravel -p laravel < backup.sql
```

## 🔧 Configuration

### Environment Variables

Update your `.env` file with the following database credentials:

```env
DB_CONNECTION=mysql
DB_HOST=db
DB_PORT=3306
DB_DATABASE=laravel
DB_USERNAME=laravel
DB_PASSWORD=secret
```

### Nginx Configuration

The Nginx configuration is located at `docker/nginx/default.conf`. It's pre-configured to:
- Serve Laravel applications
- Handle PHP files through FPM
- Block access to `.ht` files
- Support pretty URLs with `try_files`

### PHP Configuration

The PHP Dockerfile includes all necessary extensions for Laravel. To add more PHP extensions, edit `docker/php/Dockerfile`:

```dockerfile
RUN docker-php-ext-install <extension-name>
```

Then rebuild:

```bash
docker-compose up -d --build
```

## 🐛 Troubleshooting

### Port Already in Use

If ports 8000, 3306, or 8080 are already in use, modify the port mappings in `docker-compose.yml`:

```yaml
ports:
  - "8001:80"  # Changed from 8000:80
```

### Permission Issues

If you encounter permission issues with Laravel storage:

```bash
docker-compose exec app chmod -R 775 storage bootstrap/cache
```

### Database Connection Failed

Ensure the database service is running and healthy:

```bash
docker-compose ps
docker-compose logs db
```

### Composer/npm Installation Fails

Clear cache and reinstall:

```bash
docker-compose exec app composer clear-cache
docker-compose exec app composer install
```

### Containers Won't Start

Check for errors:

```bash
docker-compose logs
docker-compose logs <service-name>
```

## 🔐 Security Considerations

For production environments:

1. **Change default passwords** in `docker-compose.yml`:
   - `MYSQL_ROOT_PASSWORD`
   - `MYSQL_PASSWORD`

2. **Use environment variables** instead of hardcoding credentials

3. **Disable phpMyAdmin** in production by removing the service from `docker-compose.yml`

4. **Set strong APP_KEY** in `.env`

5. **Use HTTPS** (configure Nginx with SSL certificates)

## 📦 Adding Services

To add additional services (Redis, Memcached, etc.), add them to `docker-compose.yml`:

```yaml
services:
  redis:
    image: redis:7-alpine
    container_name: project_name_redis
    ports:
      - "6379:6379"
    depends_on:
      - app
```

## 📚 Useful Resources

- [Laravel Documentation](https://laravel.com/docs)
- [Docker Documentation](https://docs.docker.com/)
- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [Nginx Documentation](https://nginx.org/en/docs/)
- [MySQL Documentation](https://dev.mysql.com/doc/)

## 💡 Tips

- Always run `composer install` after pulling new code
- Use `docker-compose exec app php artisan migrate` to run migrations
- Keep Docker images updated regularly
- Use named volumes for persistent data
- Backup your database regularly: `docker-compose exec db mysqldump -u laravel -p laravel > backup.sql`

## 📄 License

This project is open source and available under the MIT License.

## 🤝 Support

If you encounter any issues or have questions, please check the troubleshooting section or refer to the official Docker and Laravel documentation.

---

**Happy coding!** 🎉
