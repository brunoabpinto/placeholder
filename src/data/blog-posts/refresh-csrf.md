---
title: Solving Livewire's 419 After Session Expiration
slug: refresh-csrf
publishDate: 22 Jan 2026
description: If you've worked with Livewire in Laravel applications, you've probably encountered this frustrating scenario a user is actively working in your application, filling out a long form or interacting with components...
---

<!-- @format -->

## The Problem

If you've worked with Livewire in Laravel applications, you've probably encountered this frustrating scenario: a user is actively working in your application, filling out a long form or interacting with components, when suddenly—**BAM!** A 419 error appears, and all their work is lost. The dreaded "CSRF token mismatch" strikes again.

This happens because Laravel's session expires (typically after 120 minutes by default), invalidating the CSRF token. When the user tries to interact with a Livewire component after this expiration, Laravel rejects the request for security reasons. The user is forced to refresh the page manually and start over.

Not a great user experience.

## Why This Happens

Laravel's CSRF protection is essential for security—it prevents cross-site request forgery attacks by ensuring requests come from your application. Each session gets a unique CSRF token that's validated on every state-changing request.

When a session expires:

1. The CSRF token in the user's browser becomes stale
2. The server generates a new token for the new session
3. Any Livewire action sends the old, invalid token
4. Laravel returns a 419 status code
5. User frustration ensues

While Livewire does handle this with a `page-expired` event, it typically just shows an error or reloads the page—still losing user data.

## The Solution: Automatic CSRF Token Refresh

I built a simple Laravel package that silently refreshes the CSRF token in the background before the session expires, preventing the 419 error entirely. The user never knows it's happening—they just keep working without interruption.

The package:

- Automatically calculates the refresh interval based on your session lifetime
- Fetches a fresh CSRF token via AJAX
- Updates the meta tag in the DOM
- Works seamlessly with Livewire and all Laravel applications
- Requires zero configuration

## Installation

Install via Composer:

```bash
composer require brunoabpinto/csrf-refresh
```

The package automatically:

- Publishes the JavaScript file to `public/vendor/csrf-refresh/`
- Registers the refresh route
- Registers the `@csrfRefresh` Blade directive

## Usage

Just add one line to your layout file:

```blade
<!DOCTYPE html>
<html>
<head>
    <meta name="csrf-token" content="{{ csrf_token() }}">
    <title>My App</title>
</head>
<body>
    {{ $slot }}

    @csrfRefresh
</body>
</html>
```

That's it! Your CSRF tokens will now refresh automatically before the session expires.

## How It Works

The package does four things:

### 1. **Calculates the Refresh Interval**

Based on your `config/session.php` lifetime setting, the package calculates when to refresh the token—50 seconds before expiration by default:

```php
$interval = (config('session.lifetime') * 60 - 50) * 1000; // in milliseconds
```

### 2. **Registers a Refresh Endpoint**

A simple route that returns a fresh CSRF token:

```php
Route::get('/csrf-token/refresh', function () {
    return response()->json(['token' => csrf_token()]);
})->middleware('web')->name('csrf.refresh');
```

### 3. **Injects Minified JavaScript**

The `@csrfRefresh` directive injects:

- The calculated interval as a global variable
- A minified script that runs in the background

### 4. **Silently Updates the Token**

The JavaScript periodically fetches a new token and updates the meta tag:

```javascript
async function refreshCsrfToken() {
  try {
    const response = await fetch("/csrf-token/refresh", {
      method: "GET",
      headers: {
        "X-Requested-With": "XMLHttpRequest",
        Accept: "application/json",
      },
    });

    if (response.ok) {
      const data = await response.json();
      const metaTag = document.querySelector('meta[name="csrf-token"]');

      if (metaTag && data.token) {
        metaTag.setAttribute("content", data.token);
      }
    }
  } catch (error) {
    console.warn("Failed to refresh CSRF token:", error);
  }
}

setInterval(refreshCsrfToken, refreshInterval);
```

## Why I Built This as a Package

Initially, I was copying and pasting this solution across multiple projects. Every time I started a new Laravel/Livewire application, I'd recreate the same JavaScript, the same route, the same logic.

By packaging it, I can now:

- **Install once**: `composer require brunoabpinto/csrf-refresh`
- **Add one directive**: `@csrfRefresh`
- **Forget about it**: It just works

It's saved me countless hours and eliminated a frustrating user experience issue across all my projects.

## Benefits

**No more 419 errors** after session expiration
**Zero configuration** needed
**Automatic refresh** before expiration
**Works with Livewire, Inertia, and traditional Laravel apps**
**Minimal overhead** (one AJAX request per hour)
**Reusable across projects**

## Conclusion

If you're building Livewire applications (or any long-running Laravel sessions), silent CSRF token refresh is a must-have for good UX. Instead of forcing users to refresh and lose their work, let this package handle it automatically in the background.

Check it out on [GitHub](#) or install it now:

```bash
composer require brunoabpinto/csrf-refresh
```
