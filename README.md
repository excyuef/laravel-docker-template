# 🐳 Laravel 13 Docker Environment

Production-ready, development-optimized Docker setup untuk Laravel 13.  
Stack: **PHP 8.4-FPM** + **Nginx 1.27** + **MySQL 8.4** + **Redis 7.4**

---

## 📁 Struktur Direktori

```
project-root/
│
├── docker/
│   ├── mysql/
│   │   └── my.cnf                  # MySQL tuning config
│   │
│   ├── nginx/
│   │   ├── nginx.conf              # Nginx main config (gzip, workers, dll)
│   │   └── default.conf            # Virtual host Laravel
│   │
│   └── php/
│       ├── Dockerfile              # Multi-stage: base → development → production
│       ├── php.ini                 # PHP settings (memory, upload, timezone)
│       ├── php-fpm.conf            # PHP-FPM pool (workers, connections)
│       └── opcache.ini             # OPcache (JIT enabled)
│
├── app/                            # ← Laravel source code di sini
├── public/
├── ...
│
├── docker-compose.yml              # Orchestrasi semua service
├── .env.example                    # Template environment variables
├── .env                            # Actual env (jangan di-commit!)
└── README.md
```

---

## 🚀 Setup dari Awal

### 1. Clone / Buat Project Laravel

```bash
# Jika belum ada project Laravel, buat dulu:
composer create-project laravel/laravel . --prefer-dist

# Atau clone project yang sudah ada:
git clone https://github.com/your-org/your-app.git .
```

### 2. Copy File Docker ke Root Project

Salin semua file dari repo ini ke root directory Laravel Anda:

```
docker/
docker-compose.yml
.env.example
```

### 3. Setup Environment

```bash
# Copy template env
cp .env.example .env

# Cari UID/GID user Anda (penting untuk permission!)
id -u    # biasanya 1000
id -g    # biasanya 1000

# Edit .env sesuai kebutuhan
nano .env
```

**Nilai penting yang harus diisi di `.env`:**

| Key | Keterangan |
|-----|-----------|
| `APP_KEY` | Dikosongkan dulu, generate setelah container jalan |
| `APP_TIMEZONE` | Timezone app, contoh: `Asia/Jakarta` |
| `DB_PASSWORD` | Password MySQL untuk user `laravel` |
| `DB_ROOT_PASSWORD` | Password root MySQL |
| `DOCKER_UID` / `DOCKER_GID` | Output dari `id -u` / `id -g` |

### 4. Build dan Jalankan Container

```bash
# Build image dan start semua service (detached mode)
docker compose up -d --build

# Pantau proses startup
docker compose logs -f
```

### 5. Generate APP_KEY

```bash
docker exec laravel-php php artisan key:generate
```

### 6. Install Composer Dependencies

```bash
# Jika vendor/ belum ada
docker exec laravel-php composer install
```

### 7. Jalankan Migration

```bash
docker exec laravel-php php artisan migrate
```

### 8. Akses Aplikasi

Buka browser: **http://localhost:8080**

---

## 🔑 Masuk ke Container

```bash
# Masuk ke PHP-FPM container (untuk artisan, composer, dll)
docker exec -it laravel-php bash

# Masuk ke MySQL
docker exec -it laravel-mysql mysql -u laravel -psecret laravel

# Masuk ke Redis CLI
docker exec -it laravel-redis redis-cli

# Masuk ke Nginx (jarang perlu, tapi bisa)
docker exec -it laravel-nginx sh
```

---

## ⚡ Artisan Commands yang Sering Dipakai

### Aplikasi

```bash
# Generate APP_KEY
docker exec laravel-php php artisan key:generate

# Clear semua cache sekaligus
docker exec laravel-php php artisan optimize:clear

# Cache config + route + view (untuk production)
docker exec laravel-php php artisan optimize

# Lihat semua route
docker exec laravel-php php artisan route:list

# Buat Controller
docker exec laravel-php php artisan make:controller UserController --resource

# Buat Model + Migration + Factory + Seeder sekaligus
docker exec laravel-php php artisan make:model Post -mfs

# Buat Request (Form Validation)
docker exec laravel-php php artisan make:request StorePostRequest

# Buat Job (Queue)
docker exec laravel-php php artisan make:job SendWelcomeEmail

# Tinker (REPL interaktif)
docker exec -it laravel-php php artisan tinker
```

### Database

```bash
# Jalankan migration
docker exec laravel-php php artisan migrate

# Migration dengan seed
docker exec laravel-php php artisan migrate --seed

# Rollback satu batch terakhir
docker exec laravel-php php artisan migrate:rollback

# Reset semua migration lalu jalankan ulang + seed (HATI-HATI di prod!)
docker exec laravel-php php artisan migrate:fresh --seed

# Lihat status migration
docker exec laravel-php php artisan migrate:status

# Buat migration baru
docker exec laravel-php php artisan make:migration create_posts_table

# Buat seeder
docker exec laravel-php php artisan make:seeder PostSeeder

# Jalankan seeder tertentu
docker exec laravel-php php artisan db:seed --class=PostSeeder
```

### Queue

```bash
# Proses queue sekali (untuk testing)
docker exec laravel-php php artisan queue:work --once

# Lihat failed jobs
docker exec laravel-php php artisan queue:failed

# Retry semua failed job
docker exec laravel-php php artisan queue:retry all

# Flush semua failed jobs
docker exec laravel-php php artisan queue:flush

# Lihat queue monitor di Horizon (jika terinstall)
# docker exec laravel-php php artisan horizon
```

### Cache & Config

```bash
# Cache config (production)
docker exec laravel-php php artisan config:cache

# Clear config cache
docker exec laravel-php php artisan config:clear

# Cache routes
docker exec laravel-php php artisan route:cache

# Cache views
docker exec laravel-php php artisan view:cache

# Clear semua cache
docker exec laravel-php php artisan cache:clear

# Clear semua cache sekaligus (config + route + view + app)
docker exec laravel-php php artisan optimize:clear
```

---

## 📦 Composer Commands

```bash
# Install semua dependency
docker exec laravel-php composer install

# Install tanpa dev dependencies (untuk production)
docker exec laravel-php composer install --no-dev --optimize-autoloader

# Tambah package baru
docker exec laravel-php composer require vendor/package

# Update semua package
docker exec laravel-php composer update

# Dump autoload
docker exec laravel-php composer dump-autoload -o
```

---

## 🛠 Docker Management Commands

```bash
# Lihat status semua container
docker compose ps

# Lihat logs semua service
docker compose logs -f

# Lihat log service tertentu
docker compose logs -f laravel-php
docker compose logs -f laravel-nginx
docker compose logs -f laravel-mysql
docker compose logs -f laravel-queue

# Restart service tertentu
docker compose restart laravel-php

# Stop semua container
docker compose stop

# Stop dan hapus container (volume tetap ada)
docker compose down

# Stop dan hapus container + semua volume (HAPUS DATABASE!)
docker compose down -v

# Rebuild image setelah ubah Dockerfile
docker compose build laravel-php
docker compose up -d --build

# Lihat resource usage container
docker stats
```

---

## 🏗 Penjelasan Setiap Service

| Container | Image | Port | Peran |
|-----------|-------|------|-------|
| `laravel-nginx` | nginx:1.27-alpine | 8080 | Web server: melayani request HTTP, serve static files langsung tanpa PHP, proxy ke PHP-FPM untuk `.php` files |
| `laravel-php` | Custom (PHP 8.4-FPM Alpine) | — | Application server: eksekusi PHP, handle request dari Nginx via FastCGI (port 9000 internal) |
| `laravel-mysql` | mysql:8.4 | 3306 | Database utama: menyimpan data aplikasi |
| `laravel-redis` | redis:7.4-alpine | 6379 | In-memory store: cache, session, queue |
| `laravel-queue` | Same as laravel-php | — | Background job processor: menjalankan `queue:work` secara terus-menerus |
| `laravel-scheduler` | Same as laravel-php | — | Task scheduler: menjalankan `schedule:run` setiap 60 detik |

**Mengapa Nginx bukan Apache?**  
Nginx menggunakan event-driven, non-blocking architecture sehingga lebih efisien untuk melayani banyak koneksi secara bersamaan dengan RAM lebih rendah. Apache dengan mod_php spawn satu thread per request, kurang ideal untuk traffic tinggi. Nginx + PHP-FPM adalah kombinasi standard untuk Laravel production.

---

## 🔐 Best Practice Permission

File ownership diatur agar PHP-FPM bisa baca/tulis direktori yang diperlukan:

```bash
# Dari dalam container, set permission Laravel
docker exec laravel-php chmod -R 755 storage bootstrap/cache
docker exec laravel-php chown -R laravel:laravel storage bootstrap/cache

# Jika ada masalah permission dari host
sudo chown -R $USER:$USER storage bootstrap/cache
chmod -R 775 storage bootstrap/cache
```

Directory yang perlu writable oleh PHP:
- `storage/` — logs, cache, uploaded files, compiled views
- `bootstrap/cache/` — config cache, route cache

---

## ⚙️ Optimasi RAM (untuk host 8GB)

Alokasi perkiraan per service saat development:

| Service | RAM |
|---------|-----|
| laravel-nginx | ~30 MB |
| laravel-php | ~150–300 MB |
| laravel-mysql | ~300–512 MB |
| laravel-redis | ~30–50 MB |
| laravel-queue | ~80–150 MB |
| laravel-scheduler | ~80 MB |
| **Total** | **~700 MB – 1.1 GB** |

Masih menyisakan 6–7 GB untuk OS, IDE, browser, dan proses lain.

Jika ingin lebih hemat RAM, matikan service yang tidak dipakai:

```bash
# Matikan queue dan scheduler jika belum perlu
docker compose stop laravel-queue laravel-scheduler
```

---

## 🌐 Production Deployment Notes

Untuk deploy ke production, ganti `target: development` di `docker-compose.yml` ke `target: production`, dan:

```bash
# 1. Set environment production
APP_ENV=production
APP_DEBUG=false
PHP_DISPLAY_ERRORS=0
OPCACHE_VALIDATE_TIMESTAMPS=0

# 2. Install tanpa dev deps
docker exec laravel-php composer install --no-dev --optimize-autoloader

# 3. Cache semua config
docker exec laravel-php php artisan optimize

# 4. Jalankan migration
docker exec laravel-php php artisan migrate --force
```
