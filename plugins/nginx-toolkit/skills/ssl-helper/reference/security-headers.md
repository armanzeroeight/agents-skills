# Security Headers

## Contents
- Strict-Transport-Security (HSTS)
- Content-Security-Policy (CSP)
- X-Frame-Options
- X-Content-Type-Options
- X-XSS-Protection
- Referrer-Policy
- Permissions-Policy
- Cross-Origin headers

## Strict-Transport-Security (HSTS)

### What It Does

Forces browsers to use HTTPS for all future requests to your domain.

### Basic Configuration

```nginx
add_header Strict-Transport-Security "max-age=31536000" always;
```

### Full Configuration

```nginx
add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload" always;
```

**Parameters:**
- `max-age`: Seconds to remember HTTPS-only (31536000 = 1 year)
- `includeSubDomains`: Apply to all subdomains
- `preload`: Eligible for browser preload list

### Deployment Strategy

**Step 1: Start with short max-age**
```nginx
add_header Strict-Transport-Security "max-age=300" always;
```

**Step 2: Increase gradually**
```nginx
add_header Strict-Transport-Security "max-age=86400" always;  # 1 day
add_header Strict-Transport-Security "max-age=604800" always;  # 1 week
add_header Strict-Transport-Security "max-age=2592000" always;  # 1 month
```

**Step 3: Add includeSubDomains**
```nginx
add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
```

**Step 4: Submit to preload list**
```nginx
add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload" always;
```

Submit at: https://hstspreload.org/

### Important Notes

- Only add on HTTPS server blocks
- Cannot be undone easily (must wait for max-age to expire)
- Test thoroughly before adding `includeSubDomains`
- Ensure all subdomains support HTTPS before `includeSubDomains`

### Example

```nginx
server {
    listen 443 ssl http2;
    server_name example.com www.example.com;
    
    ssl_certificate /path/to/cert.pem;
    ssl_certificate_key /path/to/key.pem;
    
    # HSTS
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload" always;
    
    location / {
        # ... your configuration
    }
}

# Redirect HTTP to HTTPS
server {
    listen 80;
    server_name example.com www.example.com;
    return 301 https://$server_name$request_uri;
}
```

## Content-Security-Policy (CSP)

### What It Does

Controls which resources the browser can load, preventing XSS and data injection attacks.

### Basic Configuration

```nginx
add_header Content-Security-Policy "default-src 'self'" always;
```

### Common Configurations

**Static site:**
```nginx
add_header Content-Security-Policy "default-src 'self'; img-src 'self' data: https:; style-src 'self' 'unsafe-inline'; script-src 'self'" always;
```

**Site with CDN:**
```nginx
add_header Content-Security-Policy "default-src 'self'; script-src 'self' https://cdn.example.com; style-src 'self' https://cdn.example.com; img-src 'self' https://cdn.example.com data:" always;
```

**Site with inline scripts (less secure):**
```nginx
add_header Content-Security-Policy "default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline'" always;
```

### CSP Directives

**Resource loading:**
- `default-src`: Fallback for other directives
- `script-src`: JavaScript sources
- `style-src`: CSS sources
- `img-src`: Image sources
- `font-src`: Font sources
- `connect-src`: AJAX, WebSocket, EventSource
- `media-src`: Audio and video sources
- `object-src`: Flash, Java, other plugins
- `frame-src`: Iframe sources
- `worker-src`: Web Workers, Service Workers
- `manifest-src`: Web app manifest

**Document directives:**
- `base-uri`: Restricts `<base>` tag URLs
- `form-action`: Restricts form submission URLs
- `frame-ancestors`: Restricts embedding (replaces X-Frame-Options)

**Other:**
- `upgrade-insecure-requests`: Upgrades HTTP to HTTPS
- `block-all-mixed-content`: Blocks mixed content

### CSP Values

- `'none'`: Block all
- `'self'`: Same origin only
- `'unsafe-inline'`: Allow inline scripts/styles (avoid if possible)
- `'unsafe-eval'`: Allow eval() (avoid if possible)
- `https:`: Any HTTPS source
- `data:`: Data URIs
- `https://example.com`: Specific domain
- `*.example.com`: Wildcard subdomain
- `'nonce-<random>'`: Specific inline script with nonce
- `'sha256-<hash>'`: Specific inline script with hash

### Deployment Strategy

**Step 1: Report-only mode**
```nginx
add_header Content-Security-Policy-Report-Only "default-src 'self'; report-uri /csp-report" always;
```

**Step 2: Review reports and adjust policy**

**Step 3: Enable enforcement**
```nginx
add_header Content-Security-Policy "default-src 'self'" always;
```

### CSP Reporting

```nginx
add_header Content-Security-Policy "default-src 'self'; report-uri /csp-report" always;

location /csp-report {
    # Log CSP violations
    access_log /var/log/nginx/csp-violations.log;
    
    # Return 204 No Content
    return 204;
}
```

### Example: Modern Web App

```nginx
add_header Content-Security-Policy "
    default-src 'self';
    script-src 'self' https://cdn.example.com;
    style-src 'self' https://cdn.example.com 'unsafe-inline';
    img-src 'self' https://cdn.example.com data: https:;
    font-src 'self' https://cdn.example.com;
    connect-src 'self' https://api.example.com;
    frame-ancestors 'none';
    base-uri 'self';
    form-action 'self';
    upgrade-insecure-requests;
" always;
```

## X-Frame-Options

### What It Does

Prevents clickjacking by controlling whether your site can be embedded in iframes.

### Configuration

```nginx
add_header X-Frame-Options "SAMEORIGIN" always;
```

**Values:**
- `DENY`: Never allow framing
- `SAMEORIGIN`: Allow framing from same origin
- `ALLOW-FROM https://example.com`: Allow from specific origin (deprecated)

### Modern Alternative

Use CSP `frame-ancestors` instead:

```nginx
add_header Content-Security-Policy "frame-ancestors 'self'" always;
```

**CSP frame-ancestors values:**
- `'none'`: Equivalent to X-Frame-Options: DENY
- `'self'`: Equivalent to X-Frame-Options: SAMEORIGIN
- `https://example.com`: Allow specific origin

### Example

```nginx
server {
    listen 443 ssl http2;
    server_name example.com;
    
    # Legacy browsers
    add_header X-Frame-Options "SAMEORIGIN" always;
    
    # Modern browsers
    add_header Content-Security-Policy "frame-ancestors 'self'" always;
}
```

## X-Content-Type-Options

### What It Does

Prevents MIME type sniffing, forcing browsers to respect declared content types.

### Configuration

```nginx
add_header X-Content-Type-Options "nosniff" always;
```

**Why it matters:**
- Prevents browsers from interpreting files as different types
- Stops execution of JavaScript disguised as images
- Prevents XSS attacks via MIME confusion

### Example

```nginx
server {
    listen 443 ssl http2;
    server_name example.com;
    
    add_header X-Content-Type-Options "nosniff" always;
    
    location / {
        # Ensure correct Content-Type headers
        types {
            text/html html htm;
            text/css css;
            application/javascript js;
            image/jpeg jpg jpeg;
            image/png png;
        }
    }
}
```

## X-XSS-Protection

### What It Does

Enables browser's built-in XSS filter (legacy feature).

### Configuration

```nginx
add_header X-XSS-Protection "1; mode=block" always;
```

**Values:**
- `0`: Disable XSS filter
- `1`: Enable XSS filter
- `1; mode=block`: Enable and block page if XSS detected
- `1; report=<url>`: Enable and report violations

### Modern Alternative

Use Content-Security-Policy instead:

```nginx
add_header Content-Security-Policy "default-src 'self'; script-src 'self'" always;
```

### Note

- Deprecated in modern browsers
- Can cause security issues in some cases
- CSP is better protection
- Keep for legacy browser support

### Example

```nginx
server {
    listen 443 ssl http2;
    server_name example.com;
    
    # Legacy browsers
    add_header X-XSS-Protection "1; mode=block" always;
    
    # Modern browsers (better protection)
    add_header Content-Security-Policy "default-src 'self'; script-src 'self'" always;
}
```

## Referrer-Policy

### What It Does

Controls how much referrer information is sent with requests.

### Configuration

```nginx
add_header Referrer-Policy "strict-origin-when-cross-origin" always;
```

**Values:**
- `no-referrer`: Never send referrer
- `no-referrer-when-downgrade`: Send referrer except HTTPS→HTTP (default)
- `origin`: Send only origin (no path)
- `origin-when-cross-origin`: Full URL for same-origin, origin for cross-origin
- `same-origin`: Send referrer for same-origin only
- `strict-origin`: Send origin except HTTPS→HTTP
- `strict-origin-when-cross-origin`: Full URL for same-origin, origin for cross-origin except HTTPS→HTTP (recommended)
- `unsafe-url`: Always send full URL (not recommended)

### Recommended Configuration

```nginx
add_header Referrer-Policy "strict-origin-when-cross-origin" always;
```

**Why:**
- Protects user privacy
- Sends useful referrer for same-origin
- Sends origin for cross-origin (analytics still work)
- Doesn't leak info on HTTPS→HTTP

### Example

```nginx
server {
    listen 443 ssl http2;
    server_name example.com;
    
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;
}
```

## Permissions-Policy

### What It Does

Controls which browser features and APIs can be used (formerly Feature-Policy).

### Configuration

```nginx
add_header Permissions-Policy "geolocation=(), microphone=(), camera=()" always;
```

### Common Directives

**Disable features:**
```nginx
add_header Permissions-Policy "geolocation=(), microphone=(), camera=(), payment=(), usb=()" always;
```

**Allow for same origin:**
```nginx
add_header Permissions-Policy "geolocation=(self), microphone=(self), camera=(self)" always;
```

**Allow for specific origins:**
```nginx
add_header Permissions-Policy "geolocation=(self 'https://maps.example.com'), camera=(self)" always;
```

### Available Directives

- `accelerometer`: Accelerometer sensor
- `ambient-light-sensor`: Ambient light sensor
- `autoplay`: Autoplay media
- `battery`: Battery status API
- `camera`: Camera access
- `display-capture`: Screen capture
- `document-domain`: document.domain
- `encrypted-media`: Encrypted media extensions
- `fullscreen`: Fullscreen API
- `geolocation`: Geolocation API
- `gyroscope`: Gyroscope sensor
- `magnetometer`: Magnetometer sensor
- `microphone`: Microphone access
- `midi`: MIDI access
- `payment`: Payment Request API
- `picture-in-picture`: Picture-in-picture
- `publickey-credentials-get`: Web Authentication API
- `screen-wake-lock`: Screen Wake Lock API
- `sync-xhr`: Synchronous XMLHttpRequest
- `usb`: WebUSB API
- `web-share`: Web Share API
- `xr-spatial-tracking`: WebXR Device API

### Example

```nginx
server {
    listen 443 ssl http2;
    server_name example.com;
    
    # Disable unnecessary features
    add_header Permissions-Policy "
        geolocation=(),
        microphone=(),
        camera=(),
        payment=(),
        usb=(),
        magnetometer=(),
        gyroscope=(),
        accelerometer=()
    " always;
}
```

## Cross-Origin Headers

### Cross-Origin-Opener-Policy (COOP)

Controls whether a document can be opened in the same browsing context group.

```nginx
add_header Cross-Origin-Opener-Policy "same-origin" always;
```

**Values:**
- `unsafe-none`: Default, allows cross-origin
- `same-origin-allow-popups`: Isolates except popups
- `same-origin`: Full isolation

### Cross-Origin-Embedder-Policy (COEP)

Controls whether a document can load cross-origin resources.

```nginx
add_header Cross-Origin-Embedder-Policy "require-corp" always;
```

**Values:**
- `unsafe-none`: Default, allows cross-origin
- `require-corp`: Requires CORP header on cross-origin resources

### Cross-Origin-Resource-Policy (CORP)

Controls whether a resource can be loaded cross-origin.

```nginx
add_header Cross-Origin-Resource-Policy "same-origin" always;
```

**Values:**
- `same-site`: Same site only
- `same-origin`: Same origin only
- `cross-origin`: Allow cross-origin

### Example: Enable SharedArrayBuffer

```nginx
server {
    listen 443 ssl http2;
    server_name example.com;
    
    # Required for SharedArrayBuffer
    add_header Cross-Origin-Opener-Policy "same-origin" always;
    add_header Cross-Origin-Embedder-Policy "require-corp" always;
}
```

## Complete Security Headers Configuration

### Recommended Configuration

```nginx
server {
    listen 443 ssl http2;
    server_name example.com;
    
    ssl_certificate /path/to/cert.pem;
    ssl_certificate_key /path/to/key.pem;
    
    # HSTS
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload" always;
    
    # CSP
    add_header Content-Security-Policy "default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline'; img-src 'self' data: https:; font-src 'self'; connect-src 'self'; frame-ancestors 'self'; base-uri 'self'; form-action 'self'" always;
    
    # Clickjacking protection
    add_header X-Frame-Options "SAMEORIGIN" always;
    
    # MIME type sniffing protection
    add_header X-Content-Type-Options "nosniff" always;
    
    # XSS protection (legacy)
    add_header X-XSS-Protection "1; mode=block" always;
    
    # Referrer policy
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;
    
    # Permissions policy
    add_header Permissions-Policy "geolocation=(), microphone=(), camera=()" always;
    
    location / {
        # Your application configuration
        proxy_pass http://backend;
    }
}
```

### Using a Separate Configuration File

Create `/etc/nginx/conf.d/security-headers.conf`:

```nginx
# Security headers
add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload" always;
add_header Content-Security-Policy "default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline'; img-src 'self' data: https:; font-src 'self'; connect-src 'self'; frame-ancestors 'self'; base-uri 'self'; form-action 'self'" always;
add_header X-Frame-Options "SAMEORIGIN" always;
add_header X-Content-Type-Options "nosniff" always;
add_header X-XSS-Protection "1; mode=block" always;
add_header Referrer-Policy "strict-origin-when-cross-origin" always;
add_header Permissions-Policy "geolocation=(), microphone=(), camera=()" always;
```

Include in server blocks:

```nginx
server {
    listen 443 ssl http2;
    server_name example.com;
    
    include /etc/nginx/conf.d/security-headers.conf;
    
    location / {
        # Your configuration
    }
}
```

## Testing Security Headers

### Online Tools

- **Security Headers**: https://securityheaders.com/
- **Mozilla Observatory**: https://observatory.mozilla.org/
- **SSL Labs**: https://www.ssllabs.com/ssltest/

### Command Line

```bash
# Check all headers
curl -I https://example.com

# Check specific header
curl -I https://example.com | grep -i strict-transport-security

# Check with verbose output
curl -v https://example.com 2>&1 | grep -i "< "
```

### Browser DevTools

1. Open DevTools (F12)
2. Go to Network tab
3. Reload page
4. Click on main document
5. View Response Headers

## Common Issues

### Headers Not Appearing

**Issue:** Headers not showing in response

**Solutions:**
1. Check `always` parameter is present
2. Verify nginx configuration syntax: `nginx -t`
3. Reload nginx: `nginx -s reload`
4. Check if headers are being removed by backend
5. Verify location block inheritance

### CSP Blocking Resources

**Issue:** CSP blocking legitimate resources

**Solutions:**
1. Use report-only mode first
2. Check browser console for violations
3. Add necessary sources to CSP
4. Use nonces or hashes for inline scripts

### HSTS Causing Issues

**Issue:** Can't access site after enabling HSTS

**Solutions:**
1. Clear HSTS cache in browser
2. Wait for max-age to expire
3. Use shorter max-age during testing
4. Test with incognito/private browsing

### Mixed Content Warnings

**Issue:** HTTPS page loading HTTP resources

**Solutions:**
1. Update all resources to HTTPS
2. Use protocol-relative URLs: `//example.com/script.js`
3. Add `upgrade-insecure-requests` to CSP
4. Check for hardcoded HTTP URLs

## Best Practices

1. **Always use `always` parameter** - Ensures headers are sent even on error responses
2. **Test in report-only mode first** - Especially for CSP
3. **Start with strict policies** - Easier to relax than tighten
4. **Monitor for violations** - Set up CSP reporting
5. **Keep policies updated** - Review as site changes
6. **Use HTTPS everywhere** - Required for many security features
7. **Test across browsers** - Different browsers have different support
8. **Document your policies** - Explain why each header is configured
9. **Regular security audits** - Use online tools monthly
10. **Stay informed** - Security headers evolve over time
