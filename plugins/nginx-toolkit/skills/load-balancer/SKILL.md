---
name: load-balancer
description: Configures nginx load balancing with upstream servers, health checks, and failover strategies. Use when setting up load balancing, distributing traffic across multiple servers, or configuring upstream backends.
---

# Load Balancer Configuration

## Quick Start

Configure nginx to distribute traffic across multiple backend servers with health checks and automatic failover.

## Instructions

### Step 1: Define upstream block

Create an upstream block with your backend servers:

```nginx
upstream backend {
    # Load balancing method (optional, defaults to round-robin)
    least_conn;  # or ip_hash, or omit for round-robin
    
    # Backend servers
    server backend1.example.com:8080 weight=3;
    server backend2.example.com:8080 weight=2;
    server backend3.example.com:8080;
    
    # Backup server (used when all primary servers are down)
    server backup.example.com:8080 backup;
    
    # Health check parameters
    keepalive 32;
}
```

### Step 2: Configure proxy in server block

Add proxy configuration to route traffic to the upstream:

```nginx
server {
    listen 80;
    server_name example.com;
    
    location / {
        proxy_pass http://backend;
        
        # Essential proxy headers
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # Timeouts
        proxy_connect_timeout 60s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;
        
        # Buffering
        proxy_buffering on;
        proxy_buffer_size 4k;
        proxy_buffers 8 4k;
    }
}
```

### Step 3: Add health checks

Configure passive health checks (active checks require nginx Plus):

```nginx
upstream backend {
    server backend1.example.com:8080 max_fails=3 fail_timeout=30s;
    server backend2.example.com:8080 max_fails=3 fail_timeout=30s;
    server backend3.example.com:8080 max_fails=3 fail_timeout=30s;
}
```

Parameters:
- `max_fails`: Number of failed attempts before marking server as unavailable
- `fail_timeout`: Time to wait before retrying a failed server

### Step 4: Test and reload

```bash
# Test configuration
nginx -t

# Reload nginx
nginx -s reload
```

## Load Balancing Methods

**Round-robin (default)**: Distributes requests evenly across servers
```nginx
upstream backend {
    server backend1.example.com:8080;
    server backend2.example.com:8080;
}
```

**Least connections**: Routes to server with fewest active connections
```nginx
upstream backend {
    least_conn;
    server backend1.example.com:8080;
    server backend2.example.com:8080;
}
```

**IP hash**: Routes same client IP to same server (session persistence)
```nginx
upstream backend {
    ip_hash;
    server backend1.example.com:8080;
    server backend2.example.com:8080;
}
```

**Weighted**: Distributes based on server capacity
```nginx
upstream backend {
    server backend1.example.com:8080 weight=3;  # Gets 3x traffic
    server backend2.example.com:8080 weight=1;
}
```

## Common Patterns

### WebSocket proxying

```nginx
location /ws {
    proxy_pass http://backend;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
}
```

### Sticky sessions with cookie

```nginx
upstream backend {
    server backend1.example.com:8080;
    server backend2.example.com:8080;
    
    # Requires nginx Plus or third-party module
    sticky cookie srv_id expires=1h domain=.example.com path=/;
}
```

### Slow start (nginx Plus)

```nginx
upstream backend {
    server backend1.example.com:8080 slow_start=30s;
    server backend2.example.com:8080 slow_start=30s;
}
```

## Advanced

For detailed information, see:
- [Upstream Patterns](reference/upstream-patterns.md) - Advanced load balancing algorithms and patterns
- [Health Checks](reference/health-checks.md) - Comprehensive health check configuration
- [Caching](reference/caching.md) - Proxy caching strategies for load-balanced backends
