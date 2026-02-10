---
title: "Four Spaces Before <?php"
slug: four-spaces-before-php
publishDate: 10 Feb 2026
description: A debugging session that ended with deleting four spaces.
---

I was working on a Livewire component when every interaction started throwing 419 errors. CSRF token mismatch, on every click. The kind of thing that makes you question your setup immediately.

At first I assumed it was a Livewire issue. But then I noticed something stranger — every page refresh generated a new session ID. Sessions worked fine with `php artisan serve`, but the moment I switched to Herd, no cookies stuck. Each request was a clean slate.

The 419 errors were just a symptom. Laravel couldn't maintain sessions because the browser never received a cookie.

## Chasing the usual suspects

I started with the `.env` file:

```env
SESSION_DRIVER=redis
SESSION_LIFETIME=120
SESSION_DOMAIN=null
SESSION_SECURE_COOKIE=false
```

Redis was running, PHP could connect to it. I created a test route to confirm sessions worked at the PHP level — they did. Data was stored, but the browser never got the `Set-Cookie` header.

I checked DevTools. No `laravel_session` cookie. No `Set-Cookie` header at all.

I went through the usual list: cookie domain, HTTPS settings, session path, Redis connection. All fine. Switched to the file driver. Same result. Tried setting a cookie manually:

```php
Route::get('/cookie-test', function () {
    return response('Test')
        ->cookie('test_cookie', 'test_value', 60);
});
```

Still no `Set-Cookie` header. At this point it clearly wasn't a Laravel config problem. Something was preventing PHP from sending cookies entirely.

## The actual problem

The breakthrough came from a raw `header()` call:

```php
Route::get('/header-test', function () {
    header('X-Custom-Test: working');
    // ...
});
```

PHP responded with: _"Cannot modify header information — headers already sent"_, and pointed to `routes/web.php:1`.

Line 1 should just be `<?php`. I opened the file and there it was — the tag was indented. Four spaces sitting right there, plain as day. I just hadn't noticed.

## Why it only showed up in Herd

The bug existed all along. `php artisan serve` has output buffering enabled by default, so those spaces never reached the browser before the headers. The problem was silently masked.

Herd uses nginx + PHP-FPM with `output_buffering = 0`. Output goes out immediately. Those four spaces hit the browser before Laravel could send any headers — which meant no cookies, which meant no sessions.

## The fix

Delete four spaces. That's it.

I made sure `routes/web.php` started with `<?php` on the very first character of the very first line. Saved. Everything worked.

Livewire relies on the session to verify CSRF tokens on every request. Without a `Set-Cookie` header, the browser never stores a session — and without a session, every Livewire interaction fails with a 419.

Four spaces before `<?php` were enough to start the response body, and once the body starts, PHP can't send headers anymore.