# Health Checks

## Contents
- Passive health checks (open source)
- Active health checks (nginx Plus)
- Health check strategies
- Monitoring and alerting
- Troubleshooting failed health checks

## Passive Health Checks (Open Source)

Passive health checks monitor actual client requests and mark servers as unavailable based on failures.

### Basic Configuration

```nginx
upstream backend {
    server backend1.example.com:8080 max_fails=3 fail_timeout=30s;
    server backend2.example.com:8080 max_fails=3 fail_timeout=30s;
    server backend3.example.com:8080 max_fails=3 fail_timeout=30s;
}
```

**Parameters:**
- `max_fails`: Number of failed attempts before marking server unavailable (default: 1)
- `fail_timeout`: Time server remains unavailable before retry (default: 10s)

**How it works:**
1. nginx proxies request to backend
2. If backend fails (connection error, timeout, or error response), increment fail counter
3. When fail counter reaches `max_fails`, mark server unavailable
4. After `fail_timeout`, try server again
5. If successful, reset fail counter

### What Counts as a Failure

By default, these conditions mark a request as failed:
- Connection refused
- Connection timeout
- Read/write timeout
- Invalid response

Configure which responses trigger failover:

```nginx
server {
    location / {
        proxy_pass http://backend;
        
        # Try next server on these conditions
        proxy_next_upstream error timeout invalid_header http_500 http_502 http_503 http_504;
        
        # Maximum retry attempts
        proxy_next_upstream_tries 3;
        
        # Total time for retries
        proxy_next_upstream_timeout 10s;
    }
}
```

**proxy_next_upstream options:**
- `error`: Connection error
- `timeout`: Timeout occurred
- `invalid_header`: Invalid response header
- `http_500`: Internal Server Error
- `http_502`: Bad Gateway
- `http_503`: Service Unavailable
- `http_504`: Gateway Timeout
- `http_403`: Forbidden
- `http_404`: Not Found
- `http_429`: Too Many Requests
- `non_idempotent`: Retry non-idempotent requests (POST, PATCH, etc.)
- `off`: Disable failover

### Conservative Health Checks

For critical services, use conservative settings:

```nginx
upstream backend {
    # Require 5 failures before marking down
    server backend1.example.com:8080 max_fails=5 fail_timeout=60s;
    server backend2.example.com:8080 max_fails=5 fail_timeout=60s;
}

server {
    location / {
        proxy_pass http://backend;
        
        # Only failover on clear errors
        proxy_next_upstream error timeout http_502 http_503;
        
        # Limit retries
        proxy_next_upstream_tries 2;
    }
}
```

### Aggressive Health Checks

For less critical services with fast recovery:

```nginx
upstream backend {
    # Mark down quickly
    server backend1.example.com:8080 max_fails=2 fail_timeout=10s;
    server backend2.example.com:8080 max_fails=2 fail_timeout=10s;
}

server {
    location / {
        proxy_pass http://backend;
        
        # Failover on more conditions
        proxy_next_upstream error timeout invalid_header http_500 http_502 http_503 http_504;
        
        # More retry attempts
        proxy_next_upstream_tries 3;
    }
}
```

## Active Health Checks (nginx Plus)

Active health checks send periodic requests to backends independent of client traffic.

### Basic Active Health Check

```nginx
upstream backend {
    zone backend 64k;
    server backend1.example.com:8080;
    server backend2.example.com:8080;
}

server {
    location / {
        proxy_pass http://backend;
        health_check;
    }
}
```

**Default behavior:**
- Sends GET request to `/` every 5 seconds
- Expects 2xx or 3xx response
- Marks server down after 1 failure
- Marks server up after 1 success

### Custom Health Check Endpoint

```nginx
server {
    location / {
        proxy_pass http://backend;
        health_check uri=/health interval=10s;
    }
}
```

**Parameters:**
- `uri`: Health check endpoint (default: `/`)
- `interval`: Time between checks (default: 5s)
- `fails`: Failures before marking down (default: 1)
- `passes`: Successes before marking up (default: 1)
- `jitter`: Random delay to spread checks (default: 0)

### Advanced Health Check Configuration

```nginx
server {
    location / {
        proxy_pass http://backend;
        
        health_check interval=10s
                     fails=3
                     passes=2
                     uri=/health
                     match=health_ok;
    }
}

# Define success criteria
match health_ok {
    status 200;
    header Content-Type = "application/json";
    body ~ "\"status\":\"ok\"";
}
```

### Match Block Conditions

```nginx
# Status code check
match status_ok {
    status 200-399;
}

# Header check
match has_header {
    status 200;
    header X-Health-Status = "healthy";
}

# Body regex check
match body_ok {
    status 200;
    body ~ "\"healthy\":true";
}

# Multiple conditions (all must match)
match comprehensive {
    status 200;
    header Content-Type = "application/json";
    body ~ "\"status\":\"ok\"";
    body !~ "\"errors\"";
}
```

### Health Check for Different Protocols

**HTTP/HTTPS:**
```nginx
health_check uri=/health interval=5s;
```

**TCP:**
```nginx
upstream tcp_backend {
    zone tcp_backend 64k;
    server backend1.example.com:3306;
    server backend2.example.com:3306;
}

server {
    listen 3306;
    proxy_pass tcp_backend;
    health_check interval=10s;
}
```

**gRPC:**
```nginx
upstream grpc_backend {
    zone grpc_backend 64k;
    server backend1.example.com:50051;
    server backend2.example.com:50051;
}

server {
    listen 50051 http2;
    
    location / {
        grpc_pass grpc://grpc_backend;
        health_check type=grpc grpc_status=12;
    }
}
```

### Mandatory Health Checks

Require health check to pass before routing traffic:

```nginx
upstream backend {
    zone backend 64k;
    server backend1.example.com:8080;
    server backend2.example.com:8080;
}

server {
    location / {
        proxy_pass http://backend;
        health_check mandatory;
    }
}
```

Servers start in "checking" state and must pass health check before receiving traffic.

## Health Check Strategies

### Separate Health Check Endpoint

Create dedicated health check endpoint in your application:

```python
# Flask example
@app.route('/health')
def health():
    # Check database connection
    try:
        db.execute('SELECT 1')
    except:
        return jsonify({'status': 'unhealthy', 'reason': 'database'}), 503
    
    # Check external dependencies
    try:
        redis.ping()
    except:
        return jsonify({'status': 'unhealthy', 'reason': 'redis'}), 503
    
    return jsonify({'status': 'ok'}), 200
```

**nginx configuration:**
```nginx
health_check uri=/health interval=5s match=health_ok;

match health_ok {
    status 200;
    body ~ "\"status\":\"ok\"";
}
```

### Shallow vs Deep Health Checks

**Shallow (fast, frequent):**
```nginx
# Check every 5 seconds
health_check uri=/health/shallow interval=5s;
```

```python
@app.route('/health/shallow')
def shallow_health():
    # Just check if app is responding
    return jsonify({'status': 'ok'}), 200
```

**Deep (slow, infrequent):**
```nginx
# Check every 30 seconds
health_check uri=/health/deep interval=30s;
```

```python
@app.route('/health/deep')
def deep_health():
    # Check all dependencies
    checks = {
        'database': check_database(),
        'redis': check_redis(),
        'external_api': check_external_api()
    }
    
    if all(checks.values()):
        return jsonify({'status': 'ok', 'checks': checks}), 200
    else:
        return jsonify({'status': 'unhealthy', 'checks': checks}), 503
```

### Graceful Shutdown

Allow server to finish existing requests before shutdown:

```python
# Application sets unhealthy status
@app.route('/health')
def health():
    if app.shutting_down:
        return jsonify({'status': 'shutting_down'}), 503
    return jsonify({'status': 'ok'}), 200
```

```nginx
health_check uri=/health interval=2s fails=1;
```

**Shutdown process:**
1. Application sets `shutting_down = True`
2. Health check fails immediately
3. nginx stops routing new requests
4. Application finishes existing requests
5. Application exits

### Startup Health Checks

Prevent routing traffic during application startup:

```python
startup_complete = False

@app.route('/health')
def health():
    if not startup_complete:
        return jsonify({'status': 'starting'}), 503
    return jsonify({'status': 'ok'}), 200

# After initialization
def complete_startup():
    global startup_complete
    # Load data, warm caches, etc.
    startup_complete = True
```

```nginx
health_check uri=/health interval=2s mandatory;
```

## Monitoring and Alerting

### Status Page (nginx Plus)

```nginx
server {
    listen 8080;
    
    location /status {
        status;
    }
    
    location /status.html {
        root /usr/share/nginx/html;
    }
}
```

Access at `http://localhost:8080/status.html` for dashboard.

### API Endpoint (nginx Plus)

```nginx
server {
    listen 8080;
    
    location /api {
        api write=on;
    }
}
```

**Query upstream status:**
```bash
curl http://localhost:8080/api/6/http/upstreams/backend
```

**Response:**
```json
{
  "zone": "backend",
  "keepalive": 0,
  "peers": [
    {
      "id": 0,
      "server": "backend1.example.com:8080",
      "name": "backend1.example.com:8080",
      "backup": false,
      "weight": 1,
      "state": "up",
      "active": 0,
      "requests": 1234,
      "responses": {
        "1xx": 0,
        "2xx": 1200,
        "3xx": 20,
        "4xx": 10,
        "5xx": 4,
        "total": 1234
      },
      "health_checks": {
        "checks": 500,
        "fails": 2,
        "unhealthy": 0,
        "last_passed": true
      }
    }
  ]
}
```

### Log-Based Monitoring (Open Source)

```nginx
upstream backend {
    server backend1.example.com:8080 max_fails=3 fail_timeout=30s;
    server backend2.example.com:8080 max_fails=3 fail_timeout=30s;
}

server {
    location / {
        proxy_pass http://backend;
        
        # Log upstream response time and status
        access_log /var/log/nginx/access.log combined;
        
        # Log upstream failures
        error_log /var/log/nginx/error.log warn;
    }
}

# Custom log format with upstream info
log_format upstream '$remote_addr - $remote_user [$time_local] '
                    '"$request" $status $body_bytes_sent '
                    '"$http_referer" "$http_user_agent" '
                    'upstream: $upstream_addr '
                    'upstream_status: $upstream_status '
                    'upstream_response_time: $upstream_response_time';

access_log /var/log/nginx/upstream.log upstream;
```

**Monitor logs:**
```bash
# Watch for upstream failures
tail -f /var/log/nginx/error.log | grep upstream

# Count failures per backend
awk '{print $11}' /var/log/nginx/upstream.log | sort | uniq -c
```

### External Monitoring

Use external monitoring tools to check nginx and backends:

**Prometheus + nginx-prometheus-exporter:**
```bash
# Install exporter
docker run -p 9113:9113 nginx/nginx-prometheus-exporter:latest \
  -nginx.scrape-uri=http://nginx:8080/stub_status
```

**Datadog:**
```nginx
# Enable stub_status
server {
    listen 8080;
    location /nginx_status {
        stub_status;
        allow 127.0.0.1;
        deny all;
    }
}
```

## Troubleshooting Failed Health Checks

### Check nginx Error Logs

```bash
tail -f /var/log/nginx/error.log
```

Common errors:
- `upstream timed out`: Backend not responding
- `connect() failed`: Backend not reachable
- `no live upstreams`: All backends failed

### Test Backend Directly

```bash
# Test HTTP endpoint
curl -v http://backend1.example.com:8080/health

# Test with same headers nginx sends
curl -v -H "Host: example.com" http://backend1.example.com:8080/health

# Test TCP connection
nc -zv backend1.example.com 8080

# Test with timeout
timeout 5 curl http://backend1.example.com:8080/health
```

### Check Backend Logs

Look for:
- Application errors
- Slow queries
- Resource exhaustion
- Dependency failures

### Verify Network Connectivity

```bash
# Ping backend
ping backend1.example.com

# Trace route
traceroute backend1.example.com

# Check DNS
nslookup backend1.example.com

# Check firewall rules
iptables -L -n
```

### Adjust Health Check Sensitivity

If health checks are too sensitive:

```nginx
# Increase failure threshold
upstream backend {
    server backend1.example.com:8080 max_fails=5 fail_timeout=60s;
}

# Or for nginx Plus
health_check interval=10s fails=5 passes=2;
```

If health checks are too lenient:

```nginx
# Decrease failure threshold
upstream backend {
    server backend1.example.com:8080 max_fails=2 fail_timeout=10s;
}

# Or for nginx Plus
health_check interval=5s fails=2 passes=1;
```

### Common Issues

**Issue: Backends marked down immediately**
- Check `max_fails` is not too low
- Verify backend is actually healthy
- Check network latency

**Issue: Backends stay down too long**
- Reduce `fail_timeout`
- Check if backend recovered
- Verify health check endpoint works

**Issue: Health checks cause load**
- Increase `interval`
- Use lightweight health check endpoint
- Reduce `passes` requirement

**Issue: Intermittent failures**
- Check backend resource usage
- Look for network issues
- Review application logs for errors
