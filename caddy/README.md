# Caddy Configuration Guide

# =========================

## Overview

Caddy acts as the main reverse proxy for the NETSOCS project, handling TLS automatically and proxying all traffic to Traefik running internally in the Docker network.

## Architecture

```
Internet → Caddy (Ports 80/443) → Traefik (Port 8080) → Services
```

## Features

-   **Automatic TLS**: Let's Encrypt certificates with on-demand generation
-   **HTTP to HTTPS Redirect**: Automatic redirection for all traffic
-   **Health Checks**: Monitors Traefik availability
-   **Load Balancing**: Ready for multiple Traefik instances
-   **Logging**: JSON format logs for monitoring

## Configuration

### Main Caddyfile

The `Caddyfile` contains:

-   Global settings for TLS and storage
-   Domain-specific configurations
-   Local development support
-   Catch-all configuration for unknown domains

### Key Settings

```caddy
{
    auto_https on_demand          # Generate certificates as needed
    email admin@netsocs.com      # Let's Encrypt notifications
    storage file_system /data    # Certificate storage
    admin off                    # Disable admin API for security
}
```

## Usage

### 1. Update Domain

Replace `your-domain.com` in the Caddyfile with your actual domain:

```caddy
your-actual-domain.com {
    # ... configuration
}
```

### 2. Local Development

For local development, the configuration supports:

-   `localhost`
-   `127.0.0.1`
-   Any other local domains

### 3. Production Deployment

For production:

1. Update domain names in Caddyfile
2. Ensure ports 80 and 443 are accessible
3. Configure DNS to point to your server
4. Caddy will automatically obtain SSL certificates

## Environment Variables

### Required

-   None (Caddy is self-contained)

### Optional

-   Update email in Caddyfile for Let's Encrypt notifications

## Health Checks

### Traefik Health

Caddy monitors Traefik health at `/ping`:

-   **Interval**: 10 seconds
-   **Timeout**: 5 seconds
-   **Failure**: Automatic failover if configured

### Caddy Health

Caddy itself provides health information through logs and metrics.

## Logging

### Access Logs

-   **Format**: JSON
-   **Location**: `/data/access*.log`
-   **Rotation**: Automatic

### Log Files

-   `access.log` - Main domain traffic
-   `access-local.log` - Local development
-   `access-catchall.log` - Unknown domains

## Security

### TLS Configuration

-   **Protocol**: TLS 1.2+ (automatic)
-   **Ciphers**: Modern, secure ciphers
-   **HSTS**: Automatic HTTP Strict Transport Security
-   **OCSP Stapling**: Enabled by default

### Headers

Caddy automatically adds security headers:

-   `X-Real-IP`
-   `X-Forwarded-For`
-   `X-Forwarded-Proto`

## Monitoring

### Metrics

-   Request counts
-   Response times
-   Error rates
-   TLS certificate status

### Health Endpoints

-   `/ping` - Basic health check
-   Logs for detailed monitoring

## Troubleshooting

### Common Issues

#### 1. Certificate Generation Fails

-   Check port 80 accessibility
-   Verify domain DNS resolution
-   Check firewall settings

#### 2. Traefik Unreachable

-   Verify Traefik is running
-   Check Docker network connectivity
-   Review Traefik logs

#### 3. TLS Errors

-   Check certificate expiration
-   Verify domain configuration
-   Review Caddy logs

### Debug Mode

Enable debug logging in Caddyfile:

```caddy
{
    debug
    # ... other settings
}
```

### Logs

View Caddy logs:

```bash
docker-compose logs caddy
```

## Performance

### Optimizations

-   **Connection Pooling**: Automatic
-   **HTTP/2**: Enabled by default
-   **Compression**: Automatic
-   **Caching**: Certificate caching

### Scaling

-   **Horizontal**: Add more Caddy instances
-   **Vertical**: Increase container resources
-   **Load Balancing**: Use external load balancer

## Integration with Traefik

### Traffic Flow

1. **Caddy** receives HTTP/HTTPS requests
2. **TLS termination** happens at Caddy
3. **Traffic** is proxied to Traefik on port 8080
4. **Traefik** routes to appropriate services
5. **Services** process requests and respond

### Benefits

-   **Separation of Concerns**: Caddy handles TLS, Traefik handles routing
-   **Security**: TLS termination at edge
-   **Flexibility**: Easy to switch TLS providers
-   **Monitoring**: Clear traffic flow visibility

## Next Steps

1. Update domain names in Caddyfile
2. Test local development setup
3. Deploy to production environment
4. Monitor certificate generation
5. Set up log aggregation
6. Configure monitoring and alerting

## Support

For issues or questions:

1. Check Caddy logs: `docker-compose logs caddy`
2. Verify Traefik is running and healthy
3. Check network connectivity between containers
4. Review domain configuration and DNS settings
5. Ensure ports 80/443 are accessible from internet
