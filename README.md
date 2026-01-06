عالی 👌
این دقیقاً همون چیزیه که یه README حرفه‌ای لازم داره.
پایین‌تر یه **داکیومنت انگلیسی تمیز، رسمی و قابل کپی** برات نوشتم که مستقیم می‌تونی بذاری توی `README.md` گیت‌هابت.

---

# Laravel Sanctum – Custom Admin Token via Header (Laravel 11 / 12)

This project uses **Laravel Sanctum** with a **custom HTTP header (`X-Admin-Token`)** for authenticating admin users in an API-only backend.

The setup follows **Laravel 11+ architecture** and avoids legacy configuration patterns.

---

## Why This Approach?

Starting from **Laravel 11**, the framework introduced major structural changes:

* ❌ `config/app.php` no longer contains a `providers` array
* ❌ Manual provider registration in `config/app.php` is obsolete
* ✅ Service providers are now registered via `bootstrap/providers.php`

This results in:

* Faster application boot
* Cleaner configuration
* More explicit control over service loading

---

## Where Are Service Providers Registered?

In Laravel 11+, **all custom service providers** must be registered in:

```
bootstrap/providers.php
```

Example default content:

```php
<?php

return [
    App\Providers\AppServiceProvider::class,
];
```

---

## Goal

Authenticate admin API requests using:

```
X-Admin-Token: {SANCTUM_TOKEN}
```

instead of the default:

```
Authorization: Bearer {token}
```

---

## Step-by-Step Setup

### 1. Create a Custom Service Provider

```bash
php artisan make:provider SanctumServiceProvider
```

This will create:

```
app/Providers/SanctumServiceProvider.php
```

---

### 2. Customize Sanctum Token Extraction

Edit `app/Providers/SanctumServiceProvider.php`:

```php
<?php

namespace App\Providers;

use Illuminate\Support\ServiceProvider;
use Laravel\Sanctum\Sanctum;

class SanctumServiceProvider extends ServiceProvider
{
    public function boot(): void
    {
        Sanctum::getAccessTokenFromRequestUsing(function ($request) {
            return $request->header('X-Admin-Token');
        });
    }
}
```

✅ Sanctum will now automatically read tokens from `X-Admin-Token`.

---

### 3. Register the Provider (Important)

Open `bootstrap/providers.php` and register the provider:

```php
<?php

return [
    App\Providers\AppServiceProvider::class,
    App\Providers\SanctumServiceProvider::class,
];
```

⚠️ Do **NOT** register this in `config/app.php` — it no longer applies.

---

## Protecting Admin Routes

Example admin route protected by Sanctum:

```php
Route::prefix('admin')
    ->middleware(['auth:sanctum'])
    ->get('/auth/check', function () {
        return response()->json([
            'valid' => true,
            'admin' => auth()->user(),
        ]);
    });
```

---

## Testing the Setup

Send a request with the custom header:

```
GET /api/admin/auth/check
```

Headers:

```
X-Admin-Token: YOUR_ADMIN_SANCTUM_TOKEN
```

### Expected Response

```json
{
  "valid": true,
  "admin": {
    "id": 1,
    "name": "Admin User",
    ...
  }
}
```

If `auth()->user()` is populated, authentication is successful 🎯

---

## Why This Is the Recommended Solution

* ✅ Uses official Sanctum extension points
* ✅ No middleware hacks
* ✅ No custom guards required
* ✅ Clean separation of concerns
* ✅ Fully compatible with Laravel 11 & 12

This is the **cleanest and most maintainable** way to support custom authentication headers with Sanctum.

---

## Optional Next Steps

You can extend this setup with:

* Separate `admin` guard
* Token abilities / scopes (`admin:*`)
* Global token revocation
* IP-restricted admin tokens

---

## Summary

* Laravel 11+ moved providers to `bootstrap/providers.php`
* Sanctum supports custom token extraction via `getAccessTokenFromRequestUsing`
* `X-Admin-Token` is ideal for admin-only APIs
* This architecture is production-ready and future-proof

---
