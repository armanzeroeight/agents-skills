---
description: Analyze and optimize nginx configuration for performance, security, and best practices
allowed-tools: Bash(nginx:*), Bash(grep:*), Bash(cat:*), Read, Write
argument-hint: [config-file]
---

# Optimize Nginx Configuration

## Context

- Current nginx version: !`nginx -v 2>&1`
- Configuration test: !`nginx -t 2>&1`
- Main config file: !`nginx -V 2>&1 | grep -o 'conf-path=[^ ]*' | cut -d= -f2`
- Running processes: !`ps aux | grep nginx | grep -v grep`

## Your Task

Analyze the nginx configuration and provide optimization recommendations for performance, security, and best practices.

### Arguments

- `$1` (optional): Path to nginx configuration file to analyze (defaults to main config)

### Steps

1. **Read and analyze configuration**
   - If argument provided, read that file
   - Otherwise, read main nginx.conf
   - Identify current settings for workers, connections, timeouts, SSL, caching

2. **Check for common issues**
   - Worker processes and connections
   - Buffer sizes
   - Timeout values
   - SSL/TLS configuration
   - Security headers
   - Gzip compression
   - Static file caching
   - Access and error log configuration

3. **Provide specific recommendations**
   - List current settings that need improvement
   - Explain why each change is beneficial
   - Provide exact configuration snippets to add/modify
   - Prioritize by impact (high/medium/low)

4. **Generate optimized configuration**
   - Create a suggested configuration file
   - Include comments explaining each optimization
   - Preserve existing custom configurations
   - Ensure backward compatibility

5. **Provide testing instructions**
   - How to test the new configuration
   - How to reload nginx safely
   - How to rollback if needed

### Example Usage

```bash
# Analyze main nginx configuration
/optimize-nginx

# Analyze specific configuration file
/optimize-nginx /etc/nginx/sites-available/example.com

# Analyze and apply optimizations
/optimize-nginx /etc/nginx/nginx.conf
```

### Optimization Checklist

**Performance:**
- [ ] Worker processes set to auto or CPU count
- [ ] Worker connections optimized (typically 1024-4096)
- [ ] Keepalive timeout configured (65s recommended)
- [ ] Gzip compression enabled for text content
- [ ] Static file caching with expires headers
- [ ] Buffer sizes tuned for workload
- [ ] Sendfile enabled for static files

**Security:**
- [ ] Server tokens disabled (hide nginx version)
- [ ] SSL/TLS protocols (TLS 1.2+ only)
- [ ] Strong cipher suites configured
- [ ] Security headers added (HSTS, X-Frame-Options, etc.)
- [ ] Client body size limit set
- [ ] Rate limiting configured for sensitive endpoints

**Best Practices:**
- [ ] Separate log files for access and errors
- [ ] Log rotation configured
- [ ] Include files organized logically
- [ ] Comments explaining complex configurations
- [ ] Consistent formatting and indentation

### Output Format

Provide recommendations in this format:

```markdown
## Nginx Configuration Analysis

### Current Configuration Summary
- Worker processes: [value]
- Worker connections: [value]
- SSL/TLS: [enabled/disabled]
- Gzip: [enabled/disabled]

### High Priority Recommendations

#### 1. [Issue Name]
**Current:** [current setting]
**Recommended:** [recommended setting]
**Impact:** [performance/security/both]
**Reason:** [explanation]

**Configuration:**
\`\`\`nginx
[exact configuration to add/modify]
\`\`\`

### Medium Priority Recommendations
[same format]

### Low Priority Recommendations
[same format]

### Optimized Configuration File

\`\`\`nginx
[complete optimized configuration]
\`\`\`

### Testing Instructions

1. Backup current configuration
2. Test new configuration
3. Reload nginx
4. Verify functionality
5. Monitor logs for errors
```
