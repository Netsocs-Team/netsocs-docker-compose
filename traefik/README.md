# Traefik Configuration Guide

# ===========================

## Overview

This directory contains the Traefik v3.0 configuration for the NETSOCS project. Traefik acts as a reverse proxy, load balancer, and SSL termination point.

## File Structure

```
traefik/
├── traefik.yml              # Main Traefik configuration
├── dynamic/                  # Dynamic configuration files
│   ├── middleware.yml       # Middleware definitions
│   └── tls.yml             # TLS/HTTPS configuration
├── certificates/            # SSL certificates storage
└── README.md               # This file
```

## Quick Start

### 1. Basic Configuration

The main configuration file `traefik.yml` is already set up with:

-   Docker provider integration
-   Dashboard enabled on port 8080
-   HTTP to HTTPS redirect
-   File provider for dynamic configuration

### 2. Access Traefik Dashboard

-   URL: `http://traefik.localhost:8080`
-   Or: `http://localhost:8080`

## Adding Service Rules

### Method 1: Docker Labels (Recommended)

Add labels to your services in `compose.yaml`:

```yaml
services:
    your-service:
        labels:
            - "traefik.enable=true"
            - "traefik.http.routers.your-service.rule=Host(`your-domain.com`)"
            - "traefik.http.routers.your-service.entrypoints=websecure"
            - "traefik.http.routers.your-service.tls=true"
            - "traefik.http.services.your-service.loadbalancer.server.port=3000"
```

### Method 2: Dynamic Configuration Files

Create new files in `traefik/dynamic/` for complex routing rules.

## Example Service Configurations

### Frontend Service

```yaml
labels:
    - "traefik.enable=true"
    - "traefik.http.routers.dashboard-frontend.rule=Host(`dashboard.localhost`)"
    - "traefik.http.routers.dashboard-frontend.entrypoints=websecure"
    - "traefik.http.routers.dashboard-frontend.tls=true"
    - "traefik.http.services.dashboard-frontend.loadbalancer.server.port=80"
```

### Backend API Service

```yaml
labels:
    - "traefik.enable=true"
    - "traefik.http.routers.dashboard-api.rule=Host(`api.localhost`) && PathPrefix(`/api`)"
    - "traefik.http.routers.dashboard-api.entrypoints=websecure"
    - "traefik.http.routers.dashboard-api.tls=true"
    - "traefik.http.services.dashboard-api.loadbalancer.server.port=3000"
    - "traefik.http.routers.dashboard-api.middlewares=cors@file"
```

### Keycloak Service

```yaml
labels:
    - "traefik.enable=true"
    - "traefik.http.routers.keycloak.rule=Host(`auth.localhost`)"
    - "traefik.http.routers.keycloak.entrypoints=websecure"
    - "traefik.http.routers.keycloak.tls=true"
    - "traefik.http.services.keycloak.loadbalancer.server.port=8080"
```

## Middleware Usage

### CORS Middleware

```yaml
labels:
    - "traefik.http.routers.your-service.middlewares=cors@file"
```

### Security Headers

```yaml
labels:
    - "traefik.http.routers.your-service.middlewares=security-headers@file"
```

### Rate Limiting

```yaml
labels:
    - "traefik.http.routers.your-service.middlewares=rate-limit@file"
```

## SSL/TLS Configuration

### Let's Encrypt (Automatic)

1. Uncomment the ACME configuration in `tls.yml`
2. Set your email address
3. Ensure port 80 is accessible for HTTP challenge

### Custom Certificates

Place your `.crt` and `.key` files in `traefik/certificates/`

## Health Checks

-   Traefik health endpoint: `http://localhost/ping`
-   Service health checks are handled by Docker

## Monitoring

-   Prometheus metrics are enabled
-   Access logs are in JSON format
-   Dashboard provides real-time service status

## Troubleshooting

### Common Issues

1. **Service not accessible**: Check if `traefik.enable=true` is set
2. **Port mismatch**: Verify the port in `loadbalancer.server.port`
3. **TLS errors**: Check certificate configuration and DNS resolution

### Debug Mode

Enable debug mode in `traefik.yml`:

```yaml
api:
    debug: true
```

### Logs

View Traefik logs:

```bash
docker-compose logs traefik
```

## Security Considerations

-   HTTPS redirect is enabled by default
-   Security headers middleware is available
-   Rate limiting can be applied per service
-   IP whitelisting is available for sensitive services

## Next Steps

1. Add labels to your services in `compose.yaml`
2. Configure domain names in your hosts file or DNS
3. Set up SSL certificates (Let's Encrypt or custom)
4. Test routing and middleware functionality
