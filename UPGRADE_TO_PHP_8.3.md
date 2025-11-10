# PHP 8.3 Upgrade Guide

This document outlines the changes made to upgrade the Laravel POS application from PHP 7.3 to PHP 8.3.

## Summary of Changes

### 1. Composer Dependencies Updated

The following major changes were made to `composer.json`:

- **PHP Version**: Updated from `^7.3.0` to `^8.3`
- **Laravel Framework**: Updated from `^8.0` to `^10.0`
- **Laravel Passport**: Updated from `^10.1.3` to `^11.10`
- **Laravel UI**: Updated from `^3.0` to `^4.2`
- **Laravel Tinker**: Updated from `^2.0` to `^2.8`

#### Removed Packages:
- `fideloper/proxy` - Replaced with built-in Laravel trust proxies
- `nexmo/laravel` - Replaced with `vonage/client ^4.0`
- `facade/ignition` - Replaced with `spatie/laravel-ignition ^2.0`
- `fzaninotto/faker` - Replaced with `fakerphp/faker ^1.23`
- `oscarafdev/migrations-generator` - Removed (needs manual reinstall if required)

#### Updated Packages:
- `barryvdh/laravel-dompdf`: `^0.9.0` → `^2.0`
- `doctrine/dbal`: `2.*` → `^3.6`
- `gumlet/php-image-resize`: `1.9.*` → `^2.0`
- `guzzlehttp/guzzle`: `^7.0.1` → `^7.8`
- `intervention/image`: `^2.5` → `^2.7`
- `lcobucci/jwt`: `^3.4` → `^4.3`
- `maatwebsite/excel`: `^3.0.1` → `^3.1`
- `nesbot/carbon`: `^2.38` → `^2.71`
- `nwidart/laravel-modules`: `^8.3` → `^10.0`
- `stripe/stripe-php`: `^7.76` → `^13.0`
- `twilio/sdk`: `^6.22` → `^7.0`
- `mockery/mockery`: `^1.3.1` → `^1.6`
- `nunomaduro/collision`: `^5.0` → `^7.0`
- `phpunit/phpunit`: `^9.0` → `^10.5`

### 2. Middleware Updates

#### `app/Http/Middleware/TrustProxies.php`
- Changed from `Fideloper\Proxy\TrustProxies` to `Illuminate\Http\Middleware\TrustProxies`
- Updated headers to use explicit flags instead of `HEADER_X_FORWARDED_ALL`

#### `app/Http/Kernel.php`
- Replaced `CheckForMaintenanceMode` with `PreventRequestsDuringMaintenance`

### 3. Database Configuration

#### `config/database.php`
- Added `'collation' => 'utf8mb4_unicode_ci'` to MySQL connection
- Added `'prefix' => ''` to MySQL connection
- Added `'prefix_indexes' => true` to MySQL connection

### 4. Required Manual Actions

#### Step 1: Install Composer Dependencies

```bash
composer install
```

If you encounter conflicts, you may need to:

```bash
composer update --with-all-dependencies
```

#### Step 2: Update Nexmo/Vonage Integration

The Nexmo package has been replaced with Vonage. You need to:

1. **Remove Nexmo service provider and facade from `config/app.php`:**
   - Remove: `Nexmo\Laravel\NexmoServiceProvider::class`
   - Remove: `'Nexmo' => Nexmo\Laravel\Facade\Nexmo::class`

2. **Update all Nexmo references in controllers to use Vonage:**

Files that need updating:
- `app/Http/Controllers/Sms_SettingsController.php`
- `app/Http/Controllers/SalesReturnController.php`
- `app/Http/Controllers/SalesController.php`
- `app/Http/Controllers/QuotationsController.php`
- `app/Http/Controllers/PurchasesReturnController.php`
- `app/Http/Controllers/PurchasesController.php`
- `app/Http/Controllers/PaymentSalesController.php`
- `app/Http/Controllers/PaymentSaleReturnsController.php`
- `app/Http/Controllers/PaymentPurchasesController.php`
- `app/Http/Controllers/PaymentPurchaseReturnsController.php`

Replace:
```php
$basic  = new \Nexmo\Client\Credentials\Basic(env("NEXMO_KEY"), env("NEXMO_SECRET"));
$client = new \Nexmo\Client($basic);
```

With:
```php
$credentials = new \Vonage\Client\Credentials\Basic(env("NEXMO_KEY"), env("NEXMO_SECRET"));
$client = new \Vonage\Client($credentials);
```

3. **Rename config file (optional):**
   - Rename `config/nexmo.php` to `config/vonage.php` (or keep it for backward compatibility)

#### Step 3: Clear Caches

```bash
php artisan cache:clear
php artisan config:clear
php artisan route:clear
php artisan view:clear
```

#### Step 4: Run Migrations (if needed)

```bash
php artisan migrate
```

#### Step 5: Test the Application

Test all major features, especially:
- SMS functionality (Nexmo/Vonage integration)
- PDF generation
- Excel exports/imports
- Payment processing (Stripe, Twilio)
- Authentication and Passport OAuth

## PHP 8.3 Breaking Changes to Watch

### 1. Deprecated $dates Property in Models (Laravel 10)
The `$dates` property is deprecated in favor of `$casts`. Update all models:

```php
// OLD (Deprecated in Laravel 10)
protected $dates = ['deleted_at'];

// NEW (Use $casts instead)
protected $casts = [
    'deleted_at' => 'datetime',
];
```

**Note**: This affects many models in the application. While it will still work, you'll see deprecation warnings. The models are automatically handling this for now, but should be updated eventually.

Affected models (20+ files):
- User, Product, Sale, Purchase, Client, Provider, etc.
- All models in `app/Models/` directory

### 2. Deprecated Dynamic Properties
PHP 8.2+ deprecated dynamic properties. Ensure all classes declare properties explicitly:

```php
// Bad
$this->newProperty = 'value'; // Without declaration

// Good
class MyClass {
    public $newProperty;
}
```

### 2. Null Parameter Deprecations
Some functions no longer accept null values. Check for:
- `strlen(null)` → Use `strlen($var ?? '')`
- `trim(null)` → Use `trim($var ?? '')`

### 3. Return Type Declarations
Consider adding return type declarations for better type safety:

```php
public function getName(): string
{
    return $this->name;
}
```

## Testing Checklist

- [ ] Application starts without errors
- [ ] User authentication works
- [ ] Sales/Purchase operations function correctly
- [ ] PDF generation works
- [ ] Excel import/export works
- [ ] SMS notifications send properly (Vonage)
- [ ] Payment integrations work (Stripe, Twilio)
- [ ] All API endpoints respond correctly
- [ ] Database operations complete successfully

## Rollback Plan

If issues arise:

1. Restore the original `composer.json`
2. Run `composer install`
3. Restore original middleware files
4. Clear caches
5. Switch back to PHP 7.3/7.4

## Additional Resources

- [Laravel 10 Upgrade Guide](https://laravel.com/docs/10.x/upgrade)
- [PHP 8.3 Migration Guide](https://www.php.net/manual/en/migration83.php)
- [Vonage PHP SDK Documentation](https://github.com/Vonage/vonage-php-sdk-core)

## Notes

- This upgrade moves the application from Laravel 8 to Laravel 10, which is the LTS version compatible with PHP 8.3
- Laravel 11 is also compatible with PHP 8.3, but Laravel 10 provides better backward compatibility
- Some third-party packages may need additional updates or replacements
- Test thoroughly in a development environment before deploying to production
