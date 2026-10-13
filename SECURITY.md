# Security Checklist untuk Production Deployment

## ✅ Yang Sudah Diterapkan

### Database Security
- **3NF Normalization**: Tabel `categories` terpisah, relasi foreign key
- **SQL Injection Protection**: Eloquent ORM + prepared statements
- **Password Hashing**: bcrypt via `Hash::make()`

### Application Security
- **CSRF Protection**: `@csrf` token di semua form
- **XSS Protection**: Blade auto-escape `{{ }}`, manual `{!! !!}` hanya trusted data
- **Rate Limiting**: Login throttle 5 attempts per menit
- **Authentication**: `auth()` middleware di routes sensitif
- **Authorization**: `isAdmin()` check di controller actions

### Input Validation
- **Form Validation**: `$request->validate()` di semua POST/PATCH
- **Type Casting**: Model casts untuk dates, JSON
- **Max Length**: Database column limits + validation rules

## 🔧 Setup untuk Production Hosting

### 1. Environment Configuration
```bash
# .env production
APP_ENV=production
APP_DEBUG=false
APP_KEY=base64:... # Generate: php artisan key:generate
APP_URL=https://perpus.akti.ac.id

DB_CONNECTION=mysql  # Ganti dari sqlite
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=perpustakaan_akti
DB_USERNAME=perpus_user
DB_PASSWORD=<strong-password>

SESSION_DRIVER=database
SESSION_SECURE_COOKIE=true
SESSION_DOMAIN=.akti.ac.id
```

### 2. HTTPS & SSL
```bash
# Force HTTPS di .htaccess atau nginx
RewriteCond %{HTTPS} off
RewriteRule ^(.*)$ https://%{HTTP_HOST}/$1 [R=301,L]
```

### 3. File Permissions
```bash
chmod -R 755 storage bootstrap/cache
chmod -R 775 storage/logs
chown -R www-data:www-data storage bootstrap/cache
```

### 4. Hide Sensitive Files
```
# .htaccess di root
<FilesMatch "^\.env">
    Require all denied
</FilesMatch>
```

### 5. Database Migration
```bash
# Export dari SQLite
php artisan db:seed --force  # Seed production data

# Atau migrate fresh di server
php artisan migrate:fresh --force
```

### 6. Optimize untuk Production
```bash
php artisan config:cache
php artisan route:cache
php artisan view:cache
php artisan optimize
```

### 7. Monitoring & Logging
- Log errors ke `storage/logs/laravel.log`
- Setup log rotation (logrotate)
- Monitor `activity_logs` table untuk audit trail

### 8. Backup Strategy
```bash
# Backup database daily
mysqldump perpustakaan_akti > backup-$(date +%F).sql

# Backup uploaded files
tar -czf uploads-backup.tar.gz storage/app/public
```

## 🔐 Additional Security (Optional)

### API Rate Limiting (jika ada public API)
```php
Route::middleware('throttle:60,1')->group(function () {
    // API routes
});
```

### 2FA untuk Admin (future enhancement)
- Install `pragmarx/google2fa-laravel`
- Add `google2fa_secret` column ke users table

### Security Headers
```php
// Add to public/index.php or middleware
header('X-Content-Type-Options: nosniff');
header('X-Frame-Options: SAMEORIGIN');
header('X-XSS-Protection: 1; mode=block');
header('Referrer-Policy: strict-origin-when-cross-origin');
```

## 📊 Database 3NF Explanation

**Sebelum (Tidak Normal):**
```
books: id, code, title, author, stock (tema embedded di code)
```

**Sesudah (3NF):**
```
categories: id, code, name
books: id, code, category_id (FK), title, author, stock
```

**Keuntungan:**
- No data redundancy (nama kategori tidak duplikat)
- Update anomaly eliminated (ubah nama kategori sekali)
- Query efficiency (JOIN vs string parsing)
- Referential integrity (foreign key constraint)

## 🚀 Deployment Commands

```bash
# 1. Clone/upload ke server
git clone <repo> atau upload via FTP

# 2. Install dependencies
composer install --optimize-autoloader --no-dev

# 3. Setup environment
cp .env.example .env
php artisan key:generate

# 4. Database
php artisan migrate --force
php artisan db:seed --force

# 5. Permissions
chmod -R 755 storage bootstrap/cache

# 6. Optimize
php artisan optimize

# 7. Restart services
sudo systemctl restart php8.3-fpm nginx
```

## 🔍 Security Audit Checklist

- [ ] `APP_DEBUG=false` di production
- [ ] `.env` tidak ter-commit ke Git
- [ ] HTTPS aktif dengan valid SSL certificate
- [ ] Database credentials strong (min 16 chars, mixed)
- [ ] File permissions correct (755/775, not 777)
- [ ] Login rate limiting aktif
- [ ] Error messages tidak expose stack trace
- [ ] Backup otomatis running
- [ ] Log monitoring setup
- [ ] Update Laravel ke latest security patch
