# Upstream Patterns

## Contents
- Load balancing algorithms in depth
- Advanced upstream configurations
- Session persistence strategies
- Dynamic upstream configuration
- Upstream zones and shared memory

## Load Balancing Algorithms

### Round-Robin (Default)

Distributes requests sequentially across servers.

**When to use:**
- Servers have similar capacity
- Stateless applications
- Simple setup needed

**Configuration:**
```nginx
upstream backend {
    server backend1.example.com:8080;
    server backend2.example.com:8080;
    server backend3.example.com:8080;
}
```

**Behavior:**
- Request 1 → backend1
- Request 2 → backend2
- Request 3 → backend3
- Request 4 → backend1 (cycles back)

### Least Connections

Routes to server with fewest active connections.

**When to use:**
- Requests have varying processing times
- Some requests are long-running
- Servers have similar capacity

**Configuration:**
```nginx
upstream backend {
    least_conn;
    server backend1.example.com:8080;
    server backend2.example.com:8080;
    server backend3.example.com:8080;
}
```

**Behavior:**
- Tracks active connections per server
- Routes new request to server with lowest count
- Better load distribution for mixed workloads

### IP Hash

Routes same client IP to same server.

**When to use:**
- Session data stored locally on servers
- No shared session storage
- Need simple session persistence

**Configuration:**
```nginx
upstream backend {
    ip_hash;
    server backend1.example.com:8080;
    server backend2.example.com:8080;
    server backend3.example.com:8080;
}
```

**Behavior:**
- Hashes client IP address
- Always routes same IP to same server
- Maintains session affinity

**Limitations:**
- Doesn't work well with IPv6
- Clients behind NAT may all hash to same server
- Server removal disrupts some sessions

### Generic Hash

Hash based on custom variable.

**When to use:**
- Need custom routing logic
- Route by URL, header, or cookie
- More control than ip_hash

**Configuration:**
```nginx
upstream backend {
    hash $request_uri consistent;
    server backend1.example.com:8080;
    server backend2.example.com:8080;
    server backend3.example.com:8080;
}
```

**Examples:**

Hash by URL (cache-friendly):
```nginx
upstream backend {
    hash $request_uri consistent;
    server backend1.example.com:8080;
    server backend2.example.com:8080;
}
```

Hash by cookie:
```nginx
upstream backend {
    hash $cookie_user_id consistent;
    server backend1.example.com:8080;
    server backend2.example.com:8080;
}
```

Hash by custom header:
```nginx
upstream backend {
    hash $http_x_tenant_id consistent;
    server backend1.example.com:8080;
    server backend2.example.com:8080;
}
```

**Consistent hashing:**
- Add `consistent` parameter for better distribution
- Minimizes disruption when servers are added/removed
- Uses ketama algorithm

### Random

Routes to random server (nginx 1.15.1+).

**When to use:**
- Simple load distribution
- Stateless applications
- Want to avoid round-robin patterns

**Configuration:**
```nginx
upstream backend {
    random;
    server backend1.example.com:8080;
    server backend2.example.com:8080;
    server backend3.example.com:8080;
}
```

**With two choices:**
```nginx
upstream backend {
    random two least_conn;
    server backend1.example.com:8080;
    server backend2.example.com:8080;
    server backend3.example.com:8080;
}
```

Picks two random servers, then uses least_conn between them.

## Advanced Upstream Configuration

### Server Parameters

```nginx
upstream backend {
    server backend1.example.com:8080 weight=5 max_fails=3 fail_timeout=30s;
    server backend2.example.com:8080 weight=3 max_conns=100;
    server backend3.example.com:8080 backup;
    server backend4.example.com:8080 down;
}
```

**Parameters:**
- `weight=N`: Relative weight for load distribution (default: 1)
- `max_fails=N`: Failed attempts before marking unavailable (default: 1)
- `fail_timeout=T`: Time to wait before retrying (default: 10s)
- `max_conns=N`: Maximum concurrent connections (nginx Plus or open source 1.11.5+)
- `backup`: Only used when all primary servers are down
- `down`: Permanently marks server as unavailable
- `slow_start=T`: Gradually increase traffic to server (nginx Plus)

### Keepalive Connections

Reuse connections to upstream servers:

```nginx
upstream backend {
    server backend1.example.com:8080;
    server backend2.example.com:8080;
    
    keepalive 32;
    keepalive_requests 100;
    keepalive_timeout 60s;
}

server {
    location / {
        proxy_pass http://backend;
        proxy_http_version 1.1;
        proxy_set_header Connection "";
    }
}
```

**Benefits:**
- Reduces connection overhead
- Improves performance for HTTP/1.1
- Essential for high-traffic sites

**Configuration:**
- `keepalive N`: Number of idle connections to keep per worker
- `keepalive_requests N`: Requests per connection before closing
- `keepalive_timeout T`: Idle timeout for keepalive connections

### Upstream Zones (Shared Memory)

Share upstream configuration across workers:

```nginx
upstream backend {
    zone backend 64k;
    
    server backend1.example.com:8080;
    server backend2.example.com:8080;
}
```

**Benefits:**
- Shared state across workers
- Required for dynamic upstream configuration
- Enables runtime server management (nginx Plus)

**Size calculation:**
- ~1KB per server
- 64KB supports ~60 servers

## Session Persistence Strategies

### Cookie-Based Sticky Sessions (nginx Plus)

```nginx
upstream backend {
    server backend1.example.com:8080;
    server backend2.example.com:8080;
    
    sticky cookie srv_id expires=1h domain=.example.com path=/;
}
```

**Parameters:**
- `expires`: Cookie lifetime
- `domain`: Cookie domain
- `path`: Cookie path
- `httponly`: Add HttpOnly flag
- `secure`: Add Secure flag

### Route-Based Sticky Sessions (nginx Plus)

```nginx
upstream backend {
    server backend1.example.com:8080 route=a;
    server backend2.example.com:8080 route=b;
    
    sticky route $route_cookie $route_uri;
}
```

Routes based on cookie or URI parameter.

### Learn Method (nginx Plus)

```nginx
upstream backend {
    server backend1.example.com:8080;
    server backend2.example.com:8080;
    
    sticky learn
        create=$upstream_cookie_sessionid
        lookup=$cookie_sessionid
        zone=client_sessions:1m;
}
```

Learns session identifiers from application responses.

### Open Source Alternatives

Without nginx Plus, use application-level session management:

**Option 1: Shared session storage**
- Redis, Memcached, database
- All backends access same session store
- No nginx configuration needed

**Option 2: JWT tokens**
- Stateless authentication
- No server-side session storage
- Works with any load balancing method

**Option 3: IP hash**
- Simple but limited
- See IP Hash section above

## Dynamic Upstream Configuration

### Using DNS (Open Source)

```nginx
resolver 8.8.8.8 valid=30s;

upstream backend {
    zone backend 64k;
    server backend.example.com:8080 resolve;
}
```

**Benefits:**
- Automatically picks up DNS changes
- No nginx reload needed
- Works with service discovery

**Configuration:**
- `resolver`: DNS server to use
- `valid`: Cache time for DNS responses
- `resolve`: Enable DNS resolution for this server

### Using Variables (Open Source)

```nginx
resolver 8.8.8.8;

server {
    location / {
        set $backend backend.example.com;
        proxy_pass http://$backend:8080;
    }
}
```

**Limitations:**
- No load balancing
- No health checks
- Single backend per request

### Using API (nginx Plus)

```bash
# Add server
curl -X POST -d '{"server":"backend4.example.com:8080"}' \
  http://localhost/api/6/http/upstreams/backend/servers

# Remove server
curl -X DELETE http://localhost/api/6/http/upstreams/backend/servers/3

# Modify server
curl -X PATCH -d '{"weight":5}' \
  http://localhost/api/6/http/upstreams/backend/servers/0
```

## Multi-Tier Load Balancing

### Geographic Distribution

```nginx
# US East upstream
upstream us_east {
    server us-east-1.example.com:8080;
    server us-east-2.example.com:8080;
}

# US West upstream
upstream us_west {
    server us-west-1.example.com:8080;
    server us-west-2.example.com:8080;
}

# Route by geographic header
map $http_x_geo_region $backend {
    default us_east;
    "us-west" us_west;
    "us-east" us_east;
}

server {
    location / {
        proxy_pass http://$backend;
    }
}
```

### Service-Based Routing

```nginx
upstream api_backend {
    server api1.example.com:8080;
    server api2.example.com:8080;
}

upstream web_backend {
    server web1.example.com:8080;
    server web2.example.com:8080;
}

server {
    location /api/ {
        proxy_pass http://api_backend;
    }
    
    location / {
        proxy_pass http://web_backend;
    }
}
```

## Performance Tuning

### Worker Connections

```nginx
events {
    worker_connections 4096;
    use epoll;  # Linux
}
```

### Upstream Timeouts

```nginx
upstream backend {
    server backend1.example.com:8080;
    
    keepalive 32;
}

server {
    location / {
        proxy_pass http://backend;
        
        proxy_connect_timeout 5s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;
        
        proxy_next_upstream error timeout invalid_header http_500 http_502 http_503;
        proxy_next_upstream_tries 2;
        proxy_next_upstream_timeout 10s;
    }
}
```

**Timeout parameters:**
- `proxy_connect_timeout`: Time to establish connection
- `proxy_send_timeout`: Time to send request to upstream
- `proxy_read_timeout`: Time to read response from upstream

**Retry parameters:**
- `proxy_next_upstream`: Conditions to try next server
- `proxy_next_upstream_tries`: Maximum retry attempts
- `proxy_next_upstream_timeout`: Total time for retries

### Buffer Tuning

```nginx
server {
    location / {
        proxy_pass http://backend;
        
        proxy_buffering on;
        proxy_buffer_size 4k;
        proxy_buffers 8 4k;
        proxy_busy_buffers_size 8k;
        proxy_max_temp_file_size 1024m;
    }
}
```

**When to disable buffering:**
- Streaming responses
- Server-sent events
- Long-polling

```nginx
location /stream {
    proxy_pass http://backend;
    proxy_buffering off;
}
```
