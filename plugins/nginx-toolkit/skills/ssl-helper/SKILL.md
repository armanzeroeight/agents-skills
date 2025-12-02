---
name: ssl-helper
description: Configures SSL/TLS certificates, implements secure protocols and ciphers, and sets up security headers. Use when setting up HTTPS, SSL certificates, TLS configuration, or web security hardening.
---

# SSL/TLS Configuration Helper

## Quick Start

Configure nginx with SSL/TLS certificates, modern security protocols, and recommended security headers.

## Instructions

### Step 1: Obtain SSL certificate

**Option A: Let's Encrypt (recommended for production)**
```bash
# Install certbot
apt-get install certbot python3-certbot-nginx

# Obtain certificate
certbot --nginx -d example.com -d www.example.com

# Auto-renewal is configured automatically
```

**Option B: Self-signed certificate (development only)**
```bash
# Generate self-signed certificate
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout /etc/nginx/ssl/selfsigned.key \
  -out /etc/nginx/ssl/selfsigned.crt \
  -subj "/C=US/ST=State/L=City/O=Organization/CN=example.com"

# Generate DH parameters
openssl dhparam -out /etc/nginx/ssl/dhparam.pem 2048
```

**Option C: Commercial certificate**
```bash
# Generate CSR
openssl req -new -newkey rsa:2048 -nodes \
  -keyout /etc/nginx/ssl/example.com.key \
  -out /etc/nginx/ssl/example.com.csr

# Submit CSR to certificate authority
# Download certificate and intermediate certificates
# Place in /etc/nginx/ssl/
```

### Step 2: Configure SSL in nginx

**Basic SSL configuration:**
```nginx
server {
    listen 443 ssl http2;
    server_name example.com www.example.com;
    
    # SSL certificate files
    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;
    
    # SSL protocols and ciphers
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers 'ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384';
    ssl_prefer_server_ciphers off;
    
    # SSL session cache
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 10m;
    
    # OCSP stapling
    ssl_stapling on;
    ssl_stapling_verify on;
    ssl_trusted_certificate /etc/letsencrypt/live/example.com/chain.pem;
    
    # Security headers
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;
    
    location / {
        # Your application configuration
        proxy_pass http://backend;
    }
}

# Redirect HTTP to HTTPS
server {
    listen 80;
    server_name example.com www.example.com;
    return 301 https://$server_name$request_uri;
}
```

### Step 3: Test SSL configuration

```bash
# Test nginx configuration
nginx -t

# Reload nginx
nginx -s reload

# Test SSL with curl
curl -I https://example.com

# Check SSL certificate
openssl s_client -connect example.com:443 -servername example.com
```

### Step 4: Verify security

**Online tools:**
- SSL Labs: https://www.ssllabs.com/ssltest/
- Security Headers: https://securityheaders.com/

**Command line:**
```bash
# Check certificate expiration
echo | openssl s_client -connect example.com:443 2>/dev/null | \
  openssl x509 -noout -dates

# Test TLS versions
openssl s_client -connect example.com:443 -tls1_2
openssl s_client -connect example.com:443 -tls1_3
```

## Modern SSL Configuration

**Mozilla Modern profile (recommended for new sites):**
```nginx
ssl_protocols TLSv1.3;
ssl_prefer_server_ciphers off;
ssl_session_cache shared:SSL:10m;
ssl_session_timeout 1d;
ssl_session_tickets off;

# OCSP stapling
ssl_stapling on;
ssl_stapling_verify on;
resolver 8.8.8.8 8.8.4.4 valid=300s;
resolver_timeout 5s;
```

**Mozilla Intermediate profile (broader compatibility):**
```nginx
ssl_protocols TLSv1.2 TLSv1.3;
ssl_ciphers 'ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384';
ssl_prefer_server_ciphers off;
ssl_session_cache shared:SSL:10m;
ssl_session_timeout 1d;
ssl_session_tickets off;

# DH parameters
ssl_dhparam /etc/nginx/ssl/dhparam.pem;

# OCSP stapling
ssl_stapling on;
ssl_stapling_verify on;
```

## Security Headers

**Essential security headers:**
```nginx
# HSTS (HTTP Strict Transport Security)
add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload" always;

# Prevent clickjacking
add_header X-Frame-Options "SAMEORIGIN" always;

# Prevent MIME type sniffing
add_header X-Content-Type-Options "nosniff" always;

# XSS protection (legacy browsers)
add_header X-XSS-Protection "1; mode=block" always;

# Referrer policy
add_header Referrer-Policy "strict-origin-when-cross-origin" always;

# Content Security Policy (customize for your site)
add_header Content-Security-Policy "default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline';" always;

# Permissions policy
add_header Permissions-Policy "geolocation=(), microphone=(), camera=()" always;
```

## Common Patterns

### Multiple domains with separate certificates

```nginx
server {
    listen 443 ssl http2;
    server_name example.com www.example.com;
    
    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;
    
    # ... rest of configuration
}

server {
    listen 443 ssl http2;
    server_name api.example.com;
    
    ssl_certificate /etc/letsencrypt/live/api.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/api.example.com/privkey.pem;
    
    # ... rest of configuration
}
```

### Wildcard certificate

```nginx
server {
    listen 443 ssl http2;
    server_name *.example.com;
    
    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;
    
    # ... rest of configuration
}
```

### Client certificate authentication

```nginx
server {
    listen 443 ssl http2;
    server_name example.com;
    
    ssl_certificate /etc/nginx/ssl/server.crt;
    ssl_certificate_key /etc/nginx/ssl/server.key;
    
    # Client certificate verification
    ssl_client_certificate /etc/nginx/ssl/ca.crt;
    ssl_verify_client on;
    ssl_verify_depth 2;
    
    location / {
        # Pass client certificate info to backend
        proxy_set_header X-SSL-Client-Cert $ssl_client_cert;
        proxy_set_header X-SSL-Client-DN $ssl_client_s_dn;
        proxy_pass http://backend;
    }
}
```

### SSL termination for load balancing

```nginx
upstream backend {
    server backend1.example.com:8080;
    server backend2.example.com:8080;
}

server {
    listen 443 ssl http2;
    server_name example.com;
    
    ssl_certificate /etc/nginx/ssl/example.com.crt;
    ssl_certificate_key /etc/nginx/ssl/example.com.key;
    
    location / {
        # Terminate SSL at nginx, use HTTP to backends
        proxy_pass http://backend;
        
        # Tell backend about original protocol
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

## Certificate Management

### Let's Encrypt auto-renewal

```bash
# Test renewal
certbot renew --dry-run

# Renewal is automatic via systemd timer
systemctl status certbot.timer

# Manual renewal
certbot renew

# Reload nginx after renewal
certbot renew --deploy-hook "nginx -s reload"
```

### Certificate monitoring

```bash
# Check expiration dates
for cert in /etc/letsencrypt/live/*/cert.pem; do
    echo "Certificate: $cert"
    openssl x509 -in "$cert" -noout -enddate
done

# Alert if certificate expires soon
#!/bin/bash
CERT="/etc/letsencrypt/live/example.com/cert.pem"
DAYS_UNTIL_EXPIRY=$(( ($(date -d "$(openssl x509 -in $CERT -noout -enddate | cut -d= -f2)" +%s) - $(date +%s)) / 86400 ))

if [ $DAYS_UNTIL_EXPIRY -lt 30 ]; then
    echo "Certificate expires in $DAYS_UNTIL_EXPIRY days!"
fi
```

## Advanced

For detailed information, see:
- [Certificate Types](reference/certificate-types.md) - Different certificate types and when to use them
- [TLS Protocols](reference/tls-protocols.md) - TLS version comparison and configuration
- [Security Headers](reference/security-headers.md) - Comprehensive security header guide
