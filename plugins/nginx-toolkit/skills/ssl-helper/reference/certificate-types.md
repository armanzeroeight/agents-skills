# Certificate Types

## Contents
- Domain Validation (DV) certificates
- Organization Validation (OV) certificates
- Extended Validation (EV) certificates
- Wildcard certificates
- Multi-domain (SAN) certificates
- Self-signed certificates
- Certificate authorities comparison

## Domain Validation (DV) Certificates

**What they are:**
- Verify domain ownership only
- Automated validation process
- Issued within minutes
- Most common type

**When to use:**
- Personal websites
- Blogs
- Small business sites
- Development/staging environments
- Any site needing basic HTTPS

**Validation methods:**
- HTTP validation: Place file on web server
- DNS validation: Add TXT record to DNS
- Email validation: Receive email at admin@domain.com

**Examples:**
- Let's Encrypt (free)
- Sectigo DV
- DigiCert DV

**Let's Encrypt setup:**
```bash
# HTTP validation
certbot --nginx -d example.com

# DNS validation (manual)
certbot certonly --manual --preferred-challenges dns -d example.com

# DNS validation (automatic with plugin)
certbot certonly --dns-cloudflare \
  --dns-cloudflare-credentials ~/.secrets/cloudflare.ini \
  -d example.com
```

**Pros:**
- Free (Let's Encrypt)
- Fast issuance
- Automated renewal
- Good for most use cases

**Cons:**
- No organization verification
- No visual trust indicators
- 90-day validity (Let's Encrypt)

## Organization Validation (OV) Certificates

**What they are:**
- Verify domain ownership AND organization identity
- Manual validation by CA
- Issued within 1-3 days
- Shows organization name in certificate

**When to use:**
- Business websites
- E-commerce sites
- Corporate applications
- When organization identity matters

**Validation requirements:**
- Domain ownership verification
- Business registration documents
- Phone verification
- Physical address verification

**Examples:**
- Sectigo OV
- DigiCert OV
- GlobalSign OV

**Certificate details:**
```
Subject: CN=example.com, O=Example Inc, L=San Francisco, ST=California, C=US
```

**Pros:**
- Organization identity verified
- 1-2 year validity
- Better trust for business sites

**Cons:**
- Costs $50-200/year
- Manual validation process
- Longer issuance time

## Extended Validation (EV) Certificates

**What they are:**
- Highest level of validation
- Extensive organization verification
- Shows organization name in browser (older browsers)
- Issued within 1-2 weeks

**When to use:**
- Financial institutions
- E-commerce with high transaction values
- Sites handling sensitive data
- When maximum trust is needed

**Validation requirements:**
- All OV requirements
- Legal existence verification
- Physical location verification
- Operational existence (3+ years)
- Exclusive domain control
- Final verification call

**Examples:**
- DigiCert EV
- Sectigo EV
- GlobalSign EV

**Browser display (legacy):**
```
[Green bar] Example Inc [US]
```

**Modern browsers:**
- No special visual indicator
- Organization name in certificate details only

**Pros:**
- Highest validation level
- Maximum trust
- 1-2 year validity

**Cons:**
- Expensive ($200-1000/year)
- Lengthy validation process
- No visual indicator in modern browsers
- Diminishing value

**Note:** EV certificates have lost much of their value since browsers removed the green address bar. Consider OV certificates instead for most use cases.

## Wildcard Certificates

**What they are:**
- Secure domain and all first-level subdomains
- Single certificate for *.example.com
- Available as DV or OV

**When to use:**
- Multiple subdomains
- Dynamic subdomain creation
- Microservices architecture
- Multi-tenant applications

**Coverage:**
```
*.example.com covers:
✓ www.example.com
✓ api.example.com
✓ blog.example.com
✓ app.example.com

Does NOT cover:
✗ example.com (apex domain)
✗ api.v2.example.com (second-level subdomain)
```

**Let's Encrypt wildcard:**
```bash
# Requires DNS validation
certbot certonly --manual --preferred-challenges dns \
  -d example.com -d *.example.com

# With DNS plugin (automatic)
certbot certonly --dns-cloudflare \
  --dns-cloudflare-credentials ~/.secrets/cloudflare.ini \
  -d example.com -d *.example.com
```

**nginx configuration:**
```nginx
server {
    listen 443 ssl http2;
    server_name *.example.com;
    
    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;
    
    # Route based on subdomain
    location / {
        proxy_pass http://$host:8080;
    }
}
```

**Pros:**
- Single certificate for many subdomains
- Easier management
- Cost-effective for multiple subdomains

**Cons:**
- Requires DNS validation
- Doesn't cover apex domain
- Doesn't cover nested subdomains
- Higher security risk if key compromised

## Multi-Domain (SAN) Certificates

**What they are:**
- Subject Alternative Name (SAN) certificates
- Secure multiple different domains
- Available as DV, OV, or EV

**When to use:**
- Multiple unrelated domains
- Domain and www subdomain
- Multiple brand domains
- Consolidate certificate management

**Coverage example:**
```
Single certificate for:
- example.com
- www.example.com
- example.net
- example.org
- api.example.com
```

**Let's Encrypt multi-domain:**
```bash
certbot --nginx \
  -d example.com \
  -d www.example.com \
  -d api.example.com \
  -d example.net
```

**nginx configuration:**
```nginx
server {
    listen 443 ssl http2;
    server_name example.com www.example.com example.net;
    
    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;
    
    # ... configuration
}
```

**Pros:**
- Single certificate for multiple domains
- Easier renewal management
- Cost-effective

**Cons:**
- All domains visible in certificate
- Must renew all domains together
- Limited number of SANs (typically 100)

## Self-Signed Certificates

**What they are:**
- Certificates signed by yourself, not a CA
- No third-party validation
- Browser warnings

**When to use:**
- Development environments
- Internal applications
- Testing
- Private networks
- Never for public production sites

**Generate self-signed certificate:**
```bash
# Simple self-signed cert
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout selfsigned.key \
  -out selfsigned.crt \
  -subj "/C=US/ST=State/L=City/O=Org/CN=localhost"

# With SAN for multiple domains
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout selfsigned.key \
  -out selfsigned.crt \
  -config <(cat <<EOF
[req]
default_bits = 2048
prompt = no
default_md = sha256
distinguished_name = dn
x509_extensions = v3_req

[dn]
C=US
ST=State
L=City
O=Organization
CN=localhost

[v3_req]
subjectAltName = @alt_names

[alt_names]
DNS.1 = localhost
DNS.2 = *.localhost
IP.1 = 127.0.0.1
EOF
)
```

**nginx configuration:**
```nginx
server {
    listen 443 ssl http2;
    server_name localhost;
    
    ssl_certificate /etc/nginx/ssl/selfsigned.crt;
    ssl_certificate_key /etc/nginx/ssl/selfsigned.key;
    
    # ... configuration
}
```

**Trust self-signed certificate (development):**

**macOS:**
```bash
sudo security add-trusted-cert -d -r trustRoot \
  -k /Library/Keychains/System.keychain selfsigned.crt
```

**Linux:**
```bash
sudo cp selfsigned.crt /usr/local/share/ca-certificates/
sudo update-ca-certificates
```

**Windows:**
```powershell
Import-Certificate -FilePath selfsigned.crt -CertStoreLocation Cert:\LocalMachine\Root
```

**Pros:**
- Free
- Instant issuance
- Full control
- Good for development

**Cons:**
- Browser warnings
- No trust chain
- Not suitable for production
- Manual trust required

## Certificate Authorities Comparison

### Let's Encrypt

**Type:** DV only
**Cost:** Free
**Validity:** 90 days
**Automation:** Excellent (ACME protocol)
**Support:** Community

**Best for:**
- Personal sites
- Small businesses
- Automated deployments
- Cost-sensitive projects

**Setup:**
```bash
certbot --nginx -d example.com
```

### Sectigo (formerly Comodo)

**Types:** DV, OV, EV, Wildcard
**Cost:** $8-300/year
**Validity:** 1-2 years
**Automation:** Good (ACME support)
**Support:** Commercial

**Best for:**
- Business sites
- Multi-year certificates
- Organization validation

### DigiCert

**Types:** DV, OV, EV, Wildcard
**Cost:** $200-1000/year
**Validity:** 1-2 years
**Automation:** Good
**Support:** Premium

**Best for:**
- Enterprise
- High-value sites
- Premium support needs

### GlobalSign

**Types:** DV, OV, EV, Wildcard
**Cost:** $250-800/year
**Validity:** 1-2 years
**Automation:** Good
**Support:** Commercial

**Best for:**
- International businesses
- IoT devices
- Code signing

### AWS Certificate Manager (ACM)

**Type:** DV only
**Cost:** Free (for AWS services)
**Validity:** 13 months (auto-renewed)
**Automation:** Automatic
**Support:** AWS support

**Best for:**
- AWS infrastructure
- CloudFront
- Load Balancers
- API Gateway

**Limitations:**
- Only works with AWS services
- Cannot export private key
- Cannot use outside AWS

### Cloudflare SSL

**Types:** Universal SSL (DV), Dedicated (custom)
**Cost:** Free (Universal), $5-10/month (Dedicated)
**Validity:** Automatic renewal
**Automation:** Automatic
**Support:** Varies by plan

**Best for:**
- Sites using Cloudflare
- Easy SSL setup
- DDoS protection

**Modes:**
- Flexible: Cloudflare to client (HTTPS), Cloudflare to origin (HTTP)
- Full: HTTPS end-to-end, accepts self-signed on origin
- Full (Strict): HTTPS end-to-end, requires valid cert on origin

## Certificate Selection Guide

**Choose DV (Let's Encrypt) if:**
- Personal website or blog
- Small business site
- Budget is limited
- Automated renewal is important
- Organization identity not critical

**Choose OV if:**
- Business or corporate site
- E-commerce platform
- Organization identity matters
- Multi-year validity preferred
- Budget allows $50-200/year

**Choose EV if:**
- Financial institution
- High-value transactions
- Maximum trust required
- Budget allows $200-1000/year
- Note: Consider if worth the cost vs OV

**Choose Wildcard if:**
- Multiple subdomains
- Dynamic subdomain creation
- Microservices architecture
- Easier management preferred

**Choose Multi-domain (SAN) if:**
- Multiple unrelated domains
- Consolidate certificate management
- Cost-effective for 3+ domains

**Choose Self-signed if:**
- Development environment
- Internal applications
- Testing purposes
- Private network only
