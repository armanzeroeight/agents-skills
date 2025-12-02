# TLS Protocols

## Contents
- TLS version comparison
- Protocol configuration
- Cipher suites
- Perfect Forward Secrecy
- TLS 1.3 features
- Compatibility considerations

## TLS Version Comparison

### SSL 2.0 (1995) - DEPRECATED

**Status:** Completely insecure, removed from all browsers

**Issues:**
- Weak cipher suites
- No protection against downgrade attacks
- Vulnerable to DROWN attack

**nginx:** Not supported

### SSL 3.0 (1996) - DEPRECATED

**Status:** Insecure, disabled everywhere

**Issues:**
- Vulnerable to POODLE attack
- Weak cipher suites
- No modern security features

**nginx:** Not supported (removed in OpenSSL 1.1.0)

### TLS 1.0 (1999) - DEPRECATED

**Status:** Deprecated, disabled by major browsers

**Issues:**
- Vulnerable to BEAST attack
- Weak cipher suites (RC4, MD5)
- No modern security features

**Browser support:** Removed from Chrome 84+, Firefox 78+, Safari 14+

**nginx configuration (DO NOT USE):**
```nginx
ssl_protocols TLSv1;  # Don't do this!
```

### TLS 1.1 (2006) - DEPRECATED

**Status:** Deprecated, disabled by major browsers

**Improvements over TLS 1.0:**
- Protection against BEAST attack
- Better cipher suites

**Issues:**
- Still lacks modern security features
- No support for AEAD ciphers

**Browser support:** Removed from Chrome 84+, Firefox 78+, Safari 14+

**nginx configuration (DO NOT USE):**
```nginx
ssl_protocols TLSv1.1;  # Don't do this!
```

### TLS 1.2 (2008) - CURRENT STANDARD

**Status:** Widely supported, current standard

**Features:**
- AEAD cipher suites (GCM, CCM)
- SHA-256 support
- Authenticated encryption
- Perfect Forward Secrecy support

**Browser support:** Universal (IE 11+, all modern browsers)

**nginx configuration:**
```nginx
ssl_protocols TLSv1.2 TLSv1.3;
ssl_ciphers 'ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384';
ssl_prefer_server_ciphers off;
```

**When to use:**
- Need compatibility with older clients
- Support for IE 11, Android 4.4, Java 7
- Default choice for most sites

### TLS 1.3 (2018) - MODERN STANDARD

**Status:** Modern standard, growing adoption

**Major improvements:**
- Faster handshake (1-RTT, 0-RTT)
- Removed weak cipher suites
- Always uses Perfect Forward Secrecy
- Encrypted handshake
- Simplified cipher suite selection

**Browser support:** Chrome 70+, Firefox 63+, Safari 12.1+, Edge 79+

**nginx configuration:**
```nginx
ssl_protocols TLSv1.3;
ssl_prefer_server_ciphers off;

# Optional: Enable 0-RTT (with caution)
ssl_early_data on;
```

**When to use:**
- Modern sites with modern clients
- Maximum security
- Best performance
- Can combine with TLS 1.2 for compatibility

## Protocol Configuration

### Modern Configuration (TLS 1.3 only)

**Best security and performance, limited compatibility:**

```nginx
server {
    listen 443 ssl http2;
    server_name example.com;
    
    ssl_certificate /path/to/cert.pem;
    ssl_certificate_key /path/to/key.pem;
    
    # TLS 1.3 only
    ssl_protocols TLSv1.3;
    
    # Let client choose cipher (TLS 1.3 ciphers are all secure)
    ssl_prefer_server_ciphers off;
    
    # Session resumption
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 1d;
    ssl_session_tickets off;
    
    # OCSP stapling
    ssl_stapling on;
    ssl_stapling_verify on;
    resolver 8.8.8.8 8.8.4.4 valid=300s;
    resolver_timeout 5s;
}
```

**Supported clients:**
- Chrome 70+
- Firefox 63+
- Safari 12.1+
- Edge 79+
- Android 10+
- iOS 12.2+

### Intermediate Configuration (TLS 1.2 + 1.3)

**Good security and performance, broad compatibility:**

```nginx
server {
    listen 443 ssl http2;
    server_name example.com;
    
    ssl_certificate /path/to/cert.pem;
    ssl_certificate_key /path/to/key.pem;
    
    # TLS 1.2 and 1.3
    ssl_protocols TLSv1.2 TLSv1.3;
    
    # Strong cipher suites for TLS 1.2
    ssl_ciphers 'ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384';
    
    # Let client choose (TLS 1.3) or server choose (TLS 1.2)
    ssl_prefer_server_ciphers off;
    
    # DH parameters for DHE ciphers
    ssl_dhparam /etc/nginx/ssl/dhparam.pem;
    
    # Session resumption
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 1d;
    ssl_session_tickets off;
    
    # OCSP stapling
    ssl_stapling on;
    ssl_stapling_verify on;
    ssl_trusted_certificate /path/to/chain.pem;
    resolver 8.8.8.8 8.8.4.4 valid=300s;
    resolver_timeout 5s;
}
```

**Supported clients:**
- All modern browsers
- IE 11
- Android 4.4.2+
- Java 8+
- OpenSSL 1.0.1+

**Generate DH parameters:**
```bash
openssl dhparam -out /etc/nginx/ssl/dhparam.pem 2048
```

### Old Configuration (TLS 1.0+)

**DO NOT USE unless absolutely required for legacy clients:**

```nginx
server {
    listen 443 ssl http2;
    server_name example.com;
    
    ssl_certificate /path/to/cert.pem;
    ssl_certificate_key /path/to/key.pem;
    
    # Include old protocols (NOT RECOMMENDED)
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3;
    
    # Weak ciphers for old clients
    ssl_ciphers 'ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES128-SHA256:DHE-RSA-AES256-SHA256:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256';
    
    ssl_prefer_server_ciphers on;
    ssl_dhparam /etc/nginx/ssl/dhparam.pem;
}
```

**Only use if you must support:**
- IE 8-10 on Windows 7
- Android 2.3-4.3
- Java 6-7
- Very old systems

## Cipher Suites

### Understanding Cipher Suite Names

Format: `KEY_EXCHANGE-AUTHENTICATION-ENCRYPTION-MAC`

Example: `ECDHE-RSA-AES128-GCM-SHA256`
- `ECDHE`: Elliptic Curve Diffie-Hellman Ephemeral (key exchange)
- `RSA`: RSA authentication
- `AES128-GCM`: AES 128-bit in GCM mode (encryption)
- `SHA256`: SHA-256 (MAC/PRF)

### Recommended Cipher Suites (TLS 1.2)

**Priority order:**

1. **ECDHE-ECDSA-AES128-GCM-SHA256**
   - ECDSA certificate
   - Fast and secure
   - Hardware acceleration

2. **ECDHE-RSA-AES128-GCM-SHA256**
   - RSA certificate
   - Fast and secure
   - Hardware acceleration

3. **ECDHE-ECDSA-AES256-GCM-SHA384**
   - ECDSA certificate
   - 256-bit encryption
   - Slightly slower

4. **ECDHE-RSA-AES256-GCM-SHA384**
   - RSA certificate
   - 256-bit encryption
   - Slightly slower

5. **ECDHE-ECDSA-CHACHA20-POLY1305**
   - ECDSA certificate
   - Fast on mobile
   - No hardware acceleration needed

6. **ECDHE-RSA-CHACHA20-POLY1305**
   - RSA certificate
   - Fast on mobile
   - No hardware acceleration needed

**nginx configuration:**
```nginx
ssl_ciphers 'ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305';
```

### TLS 1.3 Cipher Suites

TLS 1.3 has only 5 cipher suites, all secure:

1. **TLS_AES_128_GCM_SHA256** (default, fastest)
2. **TLS_AES_256_GCM_SHA384**
3. **TLS_CHACHA20_POLY1305_SHA256** (mobile-friendly)
4. **TLS_AES_128_CCM_SHA256**
5. **TLS_AES_128_CCM_8_SHA256**

**nginx configuration:**
```nginx
# TLS 1.3 ciphers (optional, defaults are good)
ssl_conf_command Ciphersuites TLS_AES_128_GCM_SHA256:TLS_AES_256_GCM_SHA384:TLS_CHACHA20_POLY1305_SHA256;
```

Usually not needed - defaults are secure.

### Cipher Suite Selection

**Prefer ECDHE over DHE:**
- Faster
- Smaller key sizes for same security
- Better mobile performance

**Prefer GCM over CBC:**
- Authenticated encryption
- No padding oracle attacks
- Hardware acceleration

**Prefer AES-128 over AES-256:**
- Faster
- Sufficient security
- Better performance

**Include ChaCha20-Poly1305:**
- Fast on mobile devices
- No hardware acceleration needed
- Good for ARM processors

## Perfect Forward Secrecy (PFS)

### What is PFS?

If server's private key is compromised, past communications remain secure.

**How it works:**
- Generate ephemeral (temporary) keys for each session
- Keys are never stored
- Past sessions cannot be decrypted

### Enabling PFS

**Use ECDHE or DHE key exchange:**

```nginx
# ECDHE ciphers (recommended)
ssl_ciphers 'ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256';

# DHE ciphers (slower, needs DH params)
ssl_ciphers 'DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384';
ssl_dhparam /etc/nginx/ssl/dhparam.pem;
```

**TLS 1.3 always uses PFS** - no configuration needed.

### Verify PFS

```bash
# Test with OpenSSL
openssl s_client -connect example.com:443 -cipher 'ECDHE'

# Check for "Server Temp Key" in output
```

## TLS 1.3 Features

### 1-RTT Handshake

**TLS 1.2 handshake:** 2 round trips
```
Client -> ClientHello
       <- ServerHello, Certificate, ServerKeyExchange, ServerHelloDone
Client -> ClientKeyExchange, ChangeCipherSpec, Finished
       <- ChangeCipherSpec, Finished
Client -> Application Data
```

**TLS 1.3 handshake:** 1 round trip
```
Client -> ClientHello, KeyShare
       <- ServerHello, KeyShare, Certificate, Finished
Client -> Finished, Application Data
```

**Result:** Faster connection establishment

### 0-RTT Resumption

Send data in first packet (even faster):

```nginx
ssl_early_data on;

location / {
    proxy_pass http://backend;
    
    # Prevent replay attacks
    proxy_set_header Early-Data $ssl_early_data;
}
```

**Backend should check Early-Data header:**
```python
if request.headers.get('Early-Data') == '1':
    # This is 0-RTT data, might be replayed
    # Only allow idempotent operations (GET, HEAD)
    if request.method not in ['GET', 'HEAD']:
        return 'Forbidden', 403
```

**Security consideration:**
- 0-RTT data can be replayed
- Only use for idempotent requests
- Disable for sensitive operations

### Encrypted Handshake

TLS 1.3 encrypts more of the handshake:

**TLS 1.2:** Certificate sent in plaintext
**TLS 1.3:** Certificate encrypted

**Benefit:** Better privacy, harder to fingerprint

### Simplified Cipher Suites

TLS 1.3 removed:
- All CBC mode ciphers
- RC4
- 3DES
- MD5
- SHA-1
- Static RSA key exchange
- Static DH key exchange

**Result:** Only secure ciphers remain

## Compatibility Considerations

### Browser Compatibility Matrix

| Browser | TLS 1.0 | TLS 1.1 | TLS 1.2 | TLS 1.3 |
|---------|---------|---------|---------|---------|
| Chrome 84+ | ✗ | ✗ | ✓ | ✓ |
| Firefox 78+ | ✗ | ✗ | ✓ | ✓ |
| Safari 14+ | ✗ | ✗ | ✓ | ✓ |
| Edge 79+ | ✗ | ✗ | ✓ | ✓ |
| IE 11 | ✓ | ✓ | ✓ | ✗ |
| Android 5+ | ✗ | ✗ | ✓ | ✗ |
| Android 10+ | ✗ | ✗ | ✓ | ✓ |
| iOS 12.2+ | ✗ | ✗ | ✓ | ✓ |

### Testing Compatibility

**Test TLS versions:**
```bash
# Test TLS 1.2
openssl s_client -connect example.com:443 -tls1_2

# Test TLS 1.3
openssl s_client -connect example.com:443 -tls1_3

# Test specific cipher
openssl s_client -connect example.com:443 -cipher 'ECDHE-RSA-AES128-GCM-SHA256'
```

**Online tools:**
- SSL Labs: https://www.ssllabs.com/ssltest/
- ImmuniWeb: https://www.immuniweb.com/ssl/

### Handling Legacy Clients

**Option 1: Separate endpoint**
```nginx
# Modern endpoint
server {
    listen 443 ssl http2;
    server_name example.com;
    ssl_protocols TLSv1.3;
}

# Legacy endpoint
server {
    listen 8443 ssl http2;
    server_name legacy.example.com;
    ssl_protocols TLSv1.2;
}
```

**Option 2: Conditional configuration**
```nginx
map $ssl_protocol $legacy_client {
    "TLSv1" 1;
    "TLSv1.1" 1;
    default 0;
}

server {
    listen 443 ssl http2;
    server_name example.com;
    
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3;
    
    if ($legacy_client) {
        # Log or redirect legacy clients
        access_log /var/log/nginx/legacy.log;
    }
}
```

**Option 3: Upgrade notice**
```nginx
server {
    listen 443 ssl http2;
    server_name example.com;
    
    ssl_protocols TLSv1.2 TLSv1.3;
    
    # Return error page for old clients
    error_page 495 496 497 /ssl-error.html;
    
    location = /ssl-error.html {
        root /var/www/html;
        internal;
    }
}
```

## Performance Optimization

### Session Resumption

**Session cache:**
```nginx
ssl_session_cache shared:SSL:10m;
ssl_session_timeout 10m;
```

**Session tickets (TLS 1.2):**
```nginx
ssl_session_tickets on;
ssl_session_ticket_key /etc/nginx/ssl/ticket.key;
```

**Disable session tickets (TLS 1.3):**
```nginx
ssl_session_tickets off;
```

TLS 1.3 has better resumption built-in.

### OCSP Stapling

Reduce client-side OCSP lookups:

```nginx
ssl_stapling on;
ssl_stapling_verify on;
ssl_trusted_certificate /path/to/chain.pem;
resolver 8.8.8.8 8.8.4.4 valid=300s;
resolver_timeout 5s;
```

### HTTP/2

Enable HTTP/2 for better performance:

```nginx
listen 443 ssl http2;
```

### Hardware Acceleration

**Check for AES-NI support:**
```bash
grep -o aes /proc/cpuinfo | wc -l
```

**Use AES-GCM ciphers** for hardware acceleration.

## Security Best Practices

1. **Use TLS 1.2 minimum** (TLS 1.3 preferred)
2. **Disable TLS 1.0 and 1.1** (deprecated)
3. **Use strong cipher suites** (ECDHE, GCM)
4. **Enable Perfect Forward Secrecy** (ECDHE/DHE)
5. **Disable weak ciphers** (RC4, 3DES, MD5)
6. **Use 2048-bit RSA keys minimum** (4096-bit better)
7. **Enable OCSP stapling**
8. **Disable session tickets** (or rotate keys)
9. **Add security headers** (HSTS, etc.)
10. **Monitor certificate expiration**

## Troubleshooting

### Handshake Failures

```bash
# Check supported protocols
openssl s_client -connect example.com:443 -showcerts

# Test specific protocol
openssl s_client -connect example.com:443 -tls1_2

# Test specific cipher
openssl s_client -connect example.com:443 -cipher 'ECDHE-RSA-AES128-GCM-SHA256'
```

### Certificate Issues

```bash
# Verify certificate chain
openssl s_client -connect example.com:443 -showcerts

# Check certificate details
echo | openssl s_client -connect example.com:443 2>/dev/null | openssl x509 -noout -text

# Verify certificate matches key
openssl x509 -noout -modulus -in cert.pem | openssl md5
openssl rsa -noout -modulus -in key.pem | openssl md5
```

### Performance Issues

```bash
# Measure handshake time
time openssl s_client -connect example.com:443 </dev/null

# Check session resumption
openssl s_client -connect example.com:443 -reconnect
```
