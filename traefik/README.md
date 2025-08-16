# Traefik Configuration Guide

# ===========================

## Overview

This directory contains the Traefik v3.0 configuration for the NETSOCS project. Traefik acts as a reverse proxy, load balancer, and SSL termination point. The configuration is based on the NETSOCS Helm Chart and has been adapted for Docker Compose.

## File Structure

```
traefik/
├── traefik.yml              # Main Traefik configuration
├── dynamic/                  # Dynamic configuration files
│   ├── middlewares.yml      # Middleware definitions (from Helm chart)
│   ├── routes.yml           # Routing rules (from Helm chart)
│   └── tls.yml              # TLS/HTTPS configuration
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

## Configuration Files

### Middlewares (`dynamic/middlewares.yml`)

Based on the Helm chart, includes:

-   **Keycloak Authentication**: OIDC integration with Keycloak
-   **Security Headers**: OWASP security plugin
-   **Strip Prefix**: For API path rewriting
-   **CORS**: Cross-origin resource sharing
-   **Rate Limiting**: Request throttling

### Routes (`dynamic/routes.yml`)

Based on the Helm chart, includes:

-   **Dashboard Frontend**: `/` (with Keycloak auth)
-   **Dashboard Backend**: `/api/netsocs/module.dashboard-v2`
-   **Configuration Frontend**: `/n/config`
-   **Configuration Backend**: `/api/netsocs/module.configuration-v2`
-   **Access Control**: `/api/netsocs/module.access_control`
-   **Automation**: `/api/netsocs/automation`
-   **DriverHub**: `/api/netsocs/dh`
-   **Keycloak**: `/auth`

## Environment Variables Required

### Keycloak Configuration

```bash
# Required for authentication middleware
KEYCLOAK_CLIENT_ID=netsocs-kc
KEYCLOAK_CLIENT_SECRET=your-secret-here
HTTP_HOSTNAME=your-domain.com
SKIP_INSECURE=false  # Set to false in production
```

### Service Configuration

```bash
# Database and other service configurations
DB_HOST=your-db-host
DB_USER=your-db-user
DB_PASS=your-db-password
# ... etc
```

## Service Discovery

### Automatic Discovery

Traefik automatically discovers services through Docker labels. The dynamic configuration files provide the routing rules and middleware chains.

### Manual Service Addition

To add a new service, update the appropriate files in `traefik/dynamic/`:

1. **Add middleware** in `middlewares.yml`
2. **Add route** in `routes.yml`
3. **Restart Traefik** or wait for file watching to reload

## Middleware Usage

### Keycloak Authentication

```yaml
# Applied automatically to protected routes
middlewares:
    - keycloak
```

### Strip Prefix (API Rewriting)

```yaml
# Removes /api/netsocs/module.dashboard-v2 from requests
middlewares:
    - strip-prefix-dashboard
```

### Security Headers

```yaml
# OWASP security headers
middlewares:
    - sec-headers
```

## Routing Examples

### Dashboard Frontend

-   **URL**: `/`
-   **Service**: `netsocs-modules-dashboard-frontend-service:3000`
-   **Auth**: Keycloak required
-   **Middleware**: `keycloak`

### Dashboard API

-   **URL**: `/api/netsocs/module.dashboard-v2/*`
-   **Service**: `netsocs-modules-dashboard-backend-service-v2:3000`
-   **Auth**: Keycloak required
-   **Middleware**: `keycloak`, `strip-prefix-dashboard`

### Configuration Backend

-   **URL**: `/api/netsocs/module.configuration-v2/*`
-   **Service**: `netsocs-modules-configuration-backendv2-service:3091`
-   **Auth**: None (internal API)
-   **Middleware**: `strip-prefix-configuration`, `add-api-v2-config`

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

1. **Service not accessible**: Check if service is running and ports match
2. **Authentication errors**: Verify Keycloak configuration
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
-   Keycloak authentication is required for most frontend routes
-   Security headers middleware is applied
-   Rate limiting can be configured per service

## Migration from Helm Chart

### What's Included

-   ✅ All middleware configurations
-   ✅ All routing rules
-   ✅ Service definitions
-   ✅ Authentication flows

### What's Different

-   ❌ No Kubernetes-specific resources
-   ❌ No RBAC or service accounts
-   ❌ No persistent volumes
-   ❌ No configurator jobs

### Configuration Mapping

| Helm Resource  | Docker Compose Equivalent |
| -------------- | ------------------------- |
| `IngressRoute` | `dynamic/routes.yml`      |
| `Middleware`   | `dynamic/middlewares.yml` |
| `Service`      | Docker service names      |
| `ConfigMap`    | `.env` file variables     |

## Next Steps

1. Copy `.env.example` to `.env` and configure your values
2. Start the services with `docker-compose up`
3. Access Traefik dashboard to verify configuration
4. Test routing and authentication
5. Configure SSL certificates for production

## Support

For issues or questions:

1. Check Traefik logs: `docker-compose logs traefik`
2. Verify environment variables are set correctly
3. Ensure all services are running and healthy
4. Check network connectivity between services
