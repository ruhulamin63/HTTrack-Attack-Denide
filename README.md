# HTTrack-Attack-Denide
Solutions to Prevent HTTrack and Website Cloning.

### ✅ Using IP Deny Manager in cPanel
🔹 Steps:
- Log in to cPanel
- Go to ```Security > IP Blocker``` or IP Deny Manager
- In the IP Address or Domain field, add:

```bash
192.168.1.1
```

### ✅ Protect Against Scrapers with Middleware (Block HTTrack)
- Create a custom middleware to detect bad bots by User-Agent:

```bash
php artisan make:middleware BlockScrapers
```
Inside ```app/Http/Middleware/BlockScrapers.php```:

```bash
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;

class BlockScrapers
{
    public function handle(Request $request, Closure $next)
    {
        $userAgent = $request->header('User-Agent');

        if (preg_match('/HTTrack|wget|curl|python|libwww|Scrapy/i', $userAgent)) {
            abort(403, 'Access denied – Suspicious bot');
        }
        
        $blockedIps = ['192.168.1.1', '203.0.113.0']; // your IP list
        if (in_array($request->ip(), $blockedIps)) {
            abort(403, 'Blocked IP');
        }

        return $next($request);
    }
}
```

### ✅ Using ModSecurity in cPanel (If Available)
- In cPanel, go to ModSecurity (under Security)
- Turn on ModSecurity for your domain
- If your host allows custom rules, ask support to add a rule like:

```bash
SecRule REQUEST_HEADERS:User-Agent "HTTrack|Wget|curl" "id:123456,phase:1,deny,status:403,msg:'Blocked scraper bot'"
```

Register it in ```app/Http/Kernel.php```:

```bash
protected $middleware = [
    // other middleware
    \App\Http\Middleware\BlockScrapers::class,
];
```

Register it in ```bootstrap/app.php```:

```bash
$middleware->alias([
    'block.scrapers' => \App\Http\Middleware\BlockScrapers::class,
]);
```

### ✅ Rate Limiting with Cloudflare (Optional but Powerful)
If you use Cloudflare, you can block or rate-limit requests from suspicious IPs or bots:

- Add a WAF rule in Cloudflare:

```bash
If User-Agent contains "HTTrack"

Then Block
```

### ✅ Block HTTrack User-Agent in .htaccess (Apache)
- HTTrack uses a specific User-Agent header. You can block it by editing your ```.htaccess ``` file (in your root directory):

```bash
RewriteEngine On
RewriteCond %{HTTP_USER_AGENT} ^HTTrack [NC,OR]
RewriteCond %{HTTP_USER_AGENT} ^Wget [NC,OR]
RewriteCond %{HTTP_USER_AGENT} ^WebCopier [NC,OR]
RewriteCond %{HTTP_USER_AGENT} ^Offline\ Explorer [NC]
RewriteRule ^.*$ - [F,L]
```
- This will return 403 Forbidden to these bots.

### ✅ Deny HTTrack via PHP
Add this to the top of your main PHP file (like ```index.php```):

```bash
<?php
$user_agent = $_SERVER['HTTP_USER_AGENT'];
if (preg_match('/HTTrack|Wget|WebCopier|Offline Explorer/i', $user_agent)) {
    header('HTTP/1.0 403 Forbidden');
    exit("Access Denied");
}
?>
```

### ✅ Rate Limit or Block IPs Using Firewall (Advanced)
- Use server tools like Cloudflare, Fail2Ban, or your VPS/hosting firewall to:
- Limit requests per second
- Block IPs that download hundreds of pages in seconds

### ✅ Use robots.txt (for ethical crawlers only)
- Add this file in your root directory:
```bash
User-agent: HTTrack
Disallow: /

User-agent: *
Disallow: /admin/
```

### ✅ Disable Directory Listing (if not already)
- In ```.htaccess```:

```bash
Options -Indexes
```
- This prevents someone from accessing folders directly (like ```/images/```).

### 🚫 What HTTrack Can’t Copy
- HTTrack only copies static HTML content. It cannot copy:

- Backend PHP logic

- Databases

- Server-side authentication

- Files behind login forms (unless poorly protected)

- So make sure:

- Critical files are not directly linked or publicly accessible

- Use authentication and session checks in all PHP pages


