# Proxy Caching

## Contents
- Proxy cache basics
- Cache configuration
- Cache key customization
- Cache purging and invalidation
- Cache performance tuning
- Cache monitoring

## Proxy Cache Basics

nginx can cache responses from upstream servers to reduce backend load and improve response times.

### Basic Cache Configuration

```nginx
# Define cache path and settings
proxy_cache_path /var/cache/nginx/proxy
                 levels=1:2
                 keys_zone=my_cache:10m
                 max_size=1g
                 inactive=60m
                 use_temp_path=off;

upstream backend {
    server backend1.example.com:8080;
    server backend2.example.com:8080;
}

server {
    listen 80;
    server_name example.com;
    
    location / {
        proxy_pass http://backend;
        
        # Enable caching
        proxy_cache my_cache;
        
        # Cache successful responses for 10 minutes
        proxy_cache_valid 200 302 10m;
        proxy_cache_valid 404 1m;
        
        # Add cache status header
        add_header X-Cache-Status $upstream_cache_status;
    }
}
```

**Cache status values:**
- `MISS`: Response not in cache, fetched from backend
- `HIT`: Response served from cache
- `EXPIRED`: Cached response expired, fetched from backend
- `STALE`: Serving stale response (backend unavailable)
- `UPDATING`: Cache is being updated (serving stale)
- `REVALIDATED`: Cached response still valid (304 from backend)
- `BYPASS`: Cache bypassed by configuration

### Cache Path Parameters

```nginx
proxy_cache_path /var/cache/nginx/proxy
                 levels=1:2
                 keys_zone=my_cache:10m
                 max_size=10g
                 inactive=24h
                 use_temp_path=off
                 loader_threshold=300ms
                 loader_files=200;
```

**Parameters:**
- `levels`: Directory hierarchy (1:2 = 2 levels, 1 char then 2 chars)
- `keys_zone`: Shared memory zone name and size (1MB ≈ 8000 keys)
- `max_size`: Maximum cache size on disk
- `inactive`: Remove cached items not accessed in this time
- `use_temp_path`: Write directly to cache path (faster)
- `loader_threshold`: Time limit for cache loader iteration
- `loader_files`: Files to load per iteration

## Cache Configuration

### Cache Validity

```nginx
location / {
    proxy_pass http://backend;
    proxy_cache my_cache;
    
    # Cache by response code
    proxy_cache_valid 200 302 10m;
    proxy_cache_valid 301 1h;
    proxy_cache_valid 404 1m;
    proxy_cache_valid any 5m;
    
    # Or use upstream Cache-Control headers
    proxy_cache_valid 200 302 10m;
    proxy_cache_revalidate on;
}
```

### Respect Upstream Headers

```nginx
location / {
    proxy_pass http://backend;
    proxy_cache my_cache;
    
    # Respect Cache-Control and Expires headers
    proxy_cache_revalidate on;
    
    # Don't cache if Set-Cookie present
    proxy_no_cache $http_set_cookie;
    
    # Don't cache if Cache-Control: no-cache
    proxy_cache_bypass $http_cache_control;
}
```

### Conditional Caching

```nginx
# Don't cache authenticated requests
map $http_authorization $no_cache {
    default 1;
    "" 0;
}

# Don't cache POST requests
map $request_method $no_cache_method {
    default 0;
    POST 1;
}

server {
    location / {
        proxy_pass http://backend;
        proxy_cache my_cache;
        
        # Skip cache for authenticated or POST requests
        proxy_no_cache $no_cache $no_cache_method;
        proxy_cache_bypass $no_cache $no_cache_method;
    }
}
```

### Cache by URL Pattern

```nginx
server {
    # Cache static assets aggressively
    location ~* \.(jpg|jpeg|png|gif|ico|css|js)$ {
        proxy_pass http://backend;
        proxy_cache my_cache;
        proxy_cache_valid 200 1d;
        expires 1d;
    }
    
    # Cache API responses briefly
    location /api/ {
        proxy_pass http://backend;
        proxy_cache my_cache;
        proxy_cache_valid 200 1m;
    }
    
    # Don't cache dynamic pages
    location / {
        proxy_pass http://backend;
        proxy_cache off;
    }
}
```

## Cache Key Customization

The cache key determines what makes responses unique.

### Default Cache Key

```nginx
# Default: $scheme$proxy_host$request_uri
proxy_cache_key "$scheme$proxy_host$request_uri";
```

### Custom Cache Keys

**Include query parameters:**
```nginx
proxy_cache_key "$scheme$proxy_host$request_uri$is_args$args";
```

**Include specific headers:**
```nginx
proxy_cache_key "$scheme$proxy_host$request_uri$http_accept_language";
```

**Include cookies:**
```nginx
proxy_cache_key "$scheme$proxy_host$request_uri$cookie_user_id";
```

**Normalize query parameters:**
```nginx
# Sort query parameters for consistent caching
map $args $sorted_args {
    default $args;
    # Use Lua or external script to sort
}

proxy_cache_key "$scheme$proxy_host$request_uri$sorted_args";
```

**Vary by device type:**
```nginx
map $http_user_agent $device {
    default "desktop";
    ~*mobile "mobile";
    ~*tablet "tablet";
}

proxy_cache_key "$scheme$proxy_host$request_uri$device";
```

### Cache Slicing

Split large files into slices for better caching:

```nginx
location / {
    proxy_pass http://backend;
    proxy_cache my_cache;
    
    # Enable slicing
    slice 1m;
    proxy_cache_key "$uri$is_args$args$slice_range";
    proxy_set_header Range $slice_range;
    
    # Cache slices
    proxy_cache_valid 200 206 1h;
}
```

**Benefits:**
- Faster cache population for large files
- Better cache hit ratio
- Reduced memory usage

## Cache Purging and Invalidation

### Purge by Request (nginx Plus)

```nginx
map $request_method $purge_method {
    PURGE 1;
    default 0;
}

server {
    location / {
        proxy_pass http://backend;
        proxy_cache my_cache;
        
        # Enable purging
        proxy_cache_purge $purge_method;
    }
}
```

**Purge cache:**
```bash
curl -X PURGE http://example.com/path/to/resource
```

### Purge by Pattern (nginx Plus)

```nginx
location ~ /purge(/.*) {
    allow 127.0.0.1;
    deny all;
    
    proxy_cache_purge my_cache "$scheme$proxy_host$1";
}
```

**Purge cache:**
```bash
curl http://example.com/purge/path/to/resource
```

### Wildcard Purge (nginx Plus)

```nginx
map $request_uri $purge_cache {
    ~^/purge/(.*)$ $1;
}

location / {
    proxy_pass http://backend;
    proxy_cache my_cache;
    
    proxy_cache_purge my_cache "$scheme$proxy_host$purge_cache*";
}
```

**Purge all matching:**
```bash
curl http://example.com/purge/api/*
```

### Manual Cache Clearing (Open Source)

```bash
# Clear entire cache
rm -rf /var/cache/nginx/proxy/*

# Clear specific files
find /var/cache/nginx/proxy -name "*example.com*" -delete

# Reload nginx to rebuild cache metadata
nginx -s reload
```

### Cache Invalidation Strategies

**Time-based:**
```nginx
proxy_cache_valid 200 5m;
```

**Event-based (application triggers):**
```python
# Application sends purge request after update
import requests
requests.request('PURGE', 'http://nginx/api/resource/123')
```

**Stale-while-revalidate:**
```nginx
location / {
    proxy_pass http://backend;
    proxy_cache my_cache;
    proxy_cache_valid 200 10m;
    
    # Serve stale content while updating
    proxy_cache_use_stale updating;
    proxy_cache_background_update on;
}
```

## Cache Performance Tuning

### Optimize Cache Hit Ratio

**Use consistent cache keys:**
```nginx
# Normalize URLs
location / {
    # Remove trailing slashes
    rewrite ^/(.*)/$ /$1 permanent;
    
    proxy_pass http://backend;
    proxy_cache my_cache;
}
```

**Cache more aggressively:**
```nginx
location / {
    proxy_pass http://backend;
    proxy_cache my_cache;
    
    # Ignore backend cache headers
    proxy_ignore_headers Cache-Control Expires;
    proxy_cache_valid 200 10m;
}
```

**Serve stale content:**
```nginx
location / {
    proxy_pass http://backend;
    proxy_cache my_cache;
    
    # Serve stale if backend is down or slow
    proxy_cache_use_stale error timeout updating http_500 http_502 http_503 http_504;
    
    # Update cache in background
    proxy_cache_background_update on;
    
    # Lock to prevent cache stampede
    proxy_cache_lock on;
    proxy_cache_lock_timeout 5s;
}
```

### Prevent Cache Stampede

When cache expires, multiple requests may hit backend simultaneously.

```nginx
location / {
    proxy_pass http://backend;
    proxy_cache my_cache;
    
    # Only one request updates cache
    proxy_cache_lock on;
    proxy_cache_lock_timeout 5s;
    proxy_cache_lock_age 10s;
    
    # Others get stale content
    proxy_cache_use_stale updating;
}
```

**How it works:**
1. First request acquires lock and fetches from backend
2. Subsequent requests wait for lock (up to `lock_timeout`)
3. If lock not released, serve stale content
4. After `lock_age`, another request can acquire lock

### Optimize Cache Storage

**Use SSD for cache:**
```nginx
proxy_cache_path /mnt/ssd/nginx/cache
                 levels=1:2
                 keys_zone=my_cache:100m
                 max_size=50g;
```

**Tune directory levels:**
```nginx
# For many small files
proxy_cache_path /var/cache/nginx levels=2:2 ...;

# For fewer large files
proxy_cache_path /var/cache/nginx levels=1:1 ...;
```

**Adjust shared memory:**
```nginx
# 1MB ≈ 8000 keys
# For 1 million cached items, use ~125MB
proxy_cache_path /var/cache/nginx
                 keys_zone=my_cache:125m
                 max_size=100g;
```

### Optimize Cache Loading

```nginx
proxy_cache_path /var/cache/nginx
                 keys_zone=my_cache:10m
                 loader_threshold=300ms
                 loader_files=200
                 loader_sleeps=50ms;
```

**Parameters:**
- `loader_threshold`: Max time per iteration (default: 200ms)
- `loader_files`: Max files per iteration (default: 100)
- `loader_sleeps`: Sleep between iterations (default: 50ms)

## Cache Monitoring

### Cache Statistics (nginx Plus)

```nginx
server {
    listen 8080;
    location /api {
        api;
    }
}
```

**Query cache stats:**
```bash
curl http://localhost:8080/api/6/http/caches/my_cache
```

**Response:**
```json
{
  "size": 1073741824,
  "max_size": 10737418240,
  "cold": false,
  "hit": {
    "responses": 12345,
    "bytes": 123456789
  },
  "stale": {
    "responses": 10,
    "bytes": 10240
  },
  "updating": {
    "responses": 5,
    "bytes": 5120
  },
  "miss": {
    "responses": 1234,
    "bytes": 12345678
  }
}
```

### Log Cache Status (Open Source)

```nginx
log_format cache '$remote_addr - $remote_user [$time_local] '
                 '"$request" $status $body_bytes_sent '
                 '"$http_referer" "$http_user_agent" '
                 'cache: $upstream_cache_status '
                 'upstream: $upstream_response_time';

server {
    access_log /var/log/nginx/cache.log cache;
    
    location / {
        proxy_pass http://backend;
        proxy_cache my_cache;
        add_header X-Cache-Status $upstream_cache_status;
    }
}
```

**Analyze cache performance:**
```bash
# Count cache hits vs misses
awk '{print $NF}' /var/log/nginx/cache.log | sort | uniq -c

# Calculate hit ratio
grep -c "cache: HIT" /var/log/nginx/cache.log
grep -c "cache: MISS" /var/log/nginx/cache.log
```

### Monitor Cache Size

```bash
# Check cache disk usage
du -sh /var/cache/nginx/proxy

# Count cached files
find /var/cache/nginx/proxy -type f | wc -l

# Monitor in real-time
watch -n 5 'du -sh /var/cache/nginx/proxy'
```

### Cache Metrics to Track

**Hit ratio:**
```
hit_ratio = hits / (hits + misses)
```
Target: > 80% for static content, > 50% for dynamic content

**Cache size:**
Monitor disk usage vs `max_size`

**Eviction rate:**
How often items are removed due to `inactive` or `max_size`

**Stale responses:**
How often serving stale content (indicates backend issues)

**Cache lock timeouts:**
Indicates cache stampede or slow backend

## Advanced Caching Patterns

### Microcaching

Cache dynamic content for very short periods:

```nginx
location / {
    proxy_pass http://backend;
    proxy_cache my_cache;
    
    # Cache for 1 second
    proxy_cache_valid 200 1s;
    
    # Serve stale while updating
    proxy_cache_use_stale updating;
    proxy_cache_background_update on;
    proxy_cache_lock on;
}
```

**Benefits:**
- Protects backend from traffic spikes
- Minimal staleness
- Huge performance improvement

### Conditional Caching by User

```nginx
map $cookie_user_type $cache_zone {
    "premium" premium_cache;
    default standard_cache;
}

proxy_cache_path /var/cache/nginx/standard
                 keys_zone=standard_cache:10m;

proxy_cache_path /var/cache/nginx/premium
                 keys_zone=premium_cache:10m;

server {
    location / {
        proxy_pass http://backend;
        proxy_cache $cache_zone;
    }
}
```

### Cache Warming

Pre-populate cache before traffic arrives:

```bash
#!/bin/bash
# warm-cache.sh

URLS=(
    "http://example.com/"
    "http://example.com/popular-page"
    "http://example.com/api/data"
)

for url in "${URLS[@]}"; do
    curl -s "$url" > /dev/null
    echo "Warmed: $url"
done
```

Run after deployment or cache clear.

### Hierarchical Caching

```nginx
# Edge cache (short TTL)
server {
    listen 80;
    
    location / {
        proxy_pass http://origin;
        proxy_cache edge_cache;
        proxy_cache_valid 200 1m;
    }
}

# Origin cache (long TTL)
upstream origin {
    server origin.example.com:8080;
}

server {
    listen 8080;
    
    location / {
        proxy_pass http://backend;
        proxy_cache origin_cache;
        proxy_cache_valid 200 1h;
    }
}
```
