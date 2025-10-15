# Keycloak Production Docker Compose Setup

This directory contains a production-ready Docker Compose setup for running Keycloak with PostgreSQL.

## Overview

This setup provides:
- **Production mode**: Keycloak runs with `start --optimized` for fastest startup
- **PostgreSQL database**: Production-grade persistent database
- **HTTPS enabled**: Secure communication by default
- **Health checks**: Built-in health and metrics endpoints
- **Optimized build**: Multi-stage Docker build with pre-configured options
- **Resource limits**: Memory management for stable performance

## Quick Start

### 1. Configure Environment Variables

Copy the example environment file and customize it:

```bash
cp .env.example .env
```

**IMPORTANT**: Edit `.env` and change the following values:
- `POSTGRES_PASSWORD`: Strong password for PostgreSQL
- `KEYCLOAK_ADMIN_PASSWORD`: Strong password for Keycloak admin user
- `KC_HOSTNAME`: Your actual domain name (for production)

### 2. Build and Start Services

Build the optimized Keycloak image and start all services:

```bash
docker-compose up -d --build
```

This command will:
1. Build the optimized Keycloak image (may take a few minutes on first run)
2. Start PostgreSQL with persistent storage
3. Start Keycloak in production mode with HTTPS

### 3. Access Keycloak

Once the services are running (check with `docker-compose ps`), access Keycloak at:

- **Admin Console**: https://localhost:8443
- **Health Check**: https://localhost:9000/health
- **Metrics**: https://localhost:9000/metrics

**Note**: You'll see a certificate warning because the setup uses a self-signed certificate. See [Production Deployment](#production-deployment) for information about using real certificates.

Login with:
- Username: The value of `KEYCLOAK_ADMIN` from your `.env` file (default: `admin`)
- Password: The value of `KEYCLOAK_ADMIN_PASSWORD` from your `.env` file

## Architecture

### Services

#### PostgreSQL
- **Image**: `postgres:15-alpine`
- **Port**: 5432 (internal only)
- **Volume**: `postgres_data` for persistent storage
- **Health check**: Built-in PostgreSQL health check

#### Keycloak
- **Build**: Multi-stage optimized build from `Dockerfile.keycloak`
- **Ports**:
  - 8443: HTTPS (main interface)
  - 8080: HTTP (disabled by default)
  - 9000: Health and metrics
- **Dependencies**: Waits for PostgreSQL to be healthy before starting
- **Memory**: 2GB limit (configurable via `.env`)

### Build Process

The `Dockerfile.keycloak` uses a two-stage build:

**Stage 1 - Builder**:
- Enables health and metrics
- Configures PostgreSQL database
- Generates self-signed certificate
- Runs `kc.sh build` to optimize the image

**Stage 2 - Runtime**:
- Copies optimized files from builder
- Minimal runtime configuration
- Starts with `start --optimized`

This approach significantly reduces startup time compared to running `start` with build options.

## Configuration

### Environment Variables

All configuration is done through environment variables in `.env`:

| Variable | Description | Default |
|----------|-------------|---------|
| `POSTGRES_DB` | PostgreSQL database name | `keycloak` |
| `POSTGRES_USER` | PostgreSQL username | `keycloak` |
| `POSTGRES_PASSWORD` | PostgreSQL password | `changeme_strong_password` |
| `KEYCLOAK_ADMIN` | Keycloak admin username | `admin` |
| `KEYCLOAK_ADMIN_PASSWORD` | Keycloak admin password | `changeme_admin_password` |
| `KC_HOSTNAME` | Keycloak hostname | `localhost` |
| `KC_HOSTNAME_STRICT` | Enforce strict hostname checking | `false` |
| `KC_HTTP_ENABLED` | Enable HTTP (not recommended for production) | `false` |
| `KC_HTTPS_PORT` | HTTPS port | `8443` |
| `KC_PROXY_HEADERS` | Proxy header mode (xforwarded, forwarded) | `xforwarded` |
| `KEYCLOAK_MEMORY_LIMIT` | Container memory limit | `2G` |

### Hostname Configuration

For production, set `KC_HOSTNAME` to your actual domain:

```env
KC_HOSTNAME=keycloak.yourdomain.com
KC_HOSTNAME_STRICT=true
```

### Proxy Configuration

If running behind a reverse proxy (Nginx, Traefik, etc.):

```env
KC_PROXY_HEADERS=xforwarded
KC_HTTP_ENABLED=true
```

Then configure your reverse proxy to:
- Handle HTTPS termination
- Forward requests to Keycloak on port 8080
- Set appropriate headers (`X-Forwarded-For`, `X-Forwarded-Proto`, etc.)

## Production Deployment

### 1. Replace Self-Signed Certificate

The default setup uses a self-signed certificate for demonstration. For production:

**Option A: Use your own keystore**

1. Create a keystore with your real certificates:
```bash
keytool -importcert -file your-cert.pem -keystore server.keystore -alias server
```

2. Mount it in `docker-compose.yml`:
```yaml
keycloak:
  volumes:
    - ./certs/server.keystore:/opt/keycloak/conf/server.keystore:ro
```

3. Update `.env`:
```env
KC_HTTPS_KEY_STORE_PASSWORD=your_keystore_password
```

**Option B: Use a reverse proxy**

1. Disable HTTPS in Keycloak:
```env
KC_HTTP_ENABLED=true
KC_PROXY=edge
```

2. Configure your reverse proxy (Nginx, Traefik, Caddy) to handle HTTPS

3. Only expose port 8080 to the reverse proxy (not publicly)

### 2. Secure Database Credentials

Generate strong random passwords:

```bash
# PostgreSQL password
openssl rand -base64 32

# Keycloak admin password
openssl rand -base64 32
```

Update these in your `.env` file.

### 3. Configure Hostname

Set your production domain:

```env
KC_HOSTNAME=keycloak.yourdomain.com
KC_HOSTNAME_STRICT=true
KC_HOSTNAME_STRICT_HTTPS=true
```

### 4. Resource Limits

Adjust memory based on your expected load:

```env
# For small deployments
KEYCLOAK_MEMORY_LIMIT=2G
KEYCLOAK_MEMORY_RESERVATION=1G

# For larger deployments
KEYCLOAK_MEMORY_LIMIT=4G
KEYCLOAK_MEMORY_RESERVATION=2G
```

### 5. Database Backups

Set up regular backups of the PostgreSQL volume:

```bash
# Backup
docker-compose exec postgres pg_dump -U keycloak keycloak > backup.sql

# Restore
cat backup.sql | docker-compose exec -T postgres psql -U keycloak keycloak
```

## Management Commands

### View Logs

```bash
# All services
docker-compose logs -f

# Keycloak only
docker-compose logs -f keycloak

# PostgreSQL only
docker-compose logs -f postgres
```

### Restart Services

```bash
# Restart all
docker-compose restart

# Restart Keycloak only
docker-compose restart keycloak
```

### Stop Services

```bash
# Stop all services
docker-compose down

# Stop and remove volumes (CAUTION: deletes database!)
docker-compose down -v
```

### Rebuild After Configuration Changes

If you modify the Dockerfile or build-time configuration:

```bash
docker-compose down
docker-compose up -d --build
```

### Health Checks

Check service health:

```bash
# Via Docker
docker-compose ps

# Via health endpoint
curl -k https://localhost:9000/health
curl -k https://localhost:9000/health/ready
curl -k https://localhost:9000/health/live
```

### View Metrics

```bash
curl -k https://localhost:9000/metrics
```

## Troubleshooting

### Keycloak won't start

1. Check logs:
```bash
docker-compose logs keycloak
```

2. Verify PostgreSQL is healthy:
```bash
docker-compose ps postgres
```

3. Ensure environment variables are set correctly in `.env`

### Database connection errors

- Verify `KC_DB_URL` matches your PostgreSQL service name and database
- Check PostgreSQL is running: `docker-compose ps postgres`
- Verify credentials match between Keycloak and PostgreSQL configs

### Certificate errors in browser

This is expected with self-signed certificates. Options:
- Accept the security warning (for testing only)
- Install the certificate in your browser (for development)
- Use real certificates (for production)

### Out of memory errors

Increase memory limits in `.env`:
```env
KEYCLOAK_MEMORY_LIMIT=4G
```

## Development vs Production

This setup is configured for **production mode**. Key differences from development mode:

| Feature | Development (`start-dev`) | Production (`start --optimized`) |
|---------|---------------------------|----------------------------------|
| HTTPS | Optional | Required |
| Database | H2 (in-memory) | PostgreSQL (persistent) |
| Hostname | Not required | Required |
| Build optimization | Auto-rebuild | Pre-built |
| Startup time | Slower | Faster |
| Security | Insecure defaults | Secure defaults |

For development, you can use the official image directly:
```bash
docker run -p 8080:8080 \
  -e KC_BOOTSTRAP_ADMIN_USERNAME=admin \
  -e KC_BOOTSTRAP_ADMIN_PASSWORD=admin \
  quay.io/keycloak/keycloak:latest start-dev
```

## Additional Resources

- [Official Keycloak Container Guide](https://www.keycloak.org/server/containers)
- [Keycloak Configuration Guide](https://www.keycloak.org/server/configuration)
- [Building Keycloak from Source](docs/building.md)

## Security Considerations

Before deploying to production:

- [ ] Change all default passwords in `.env`
- [ ] Use real SSL certificates or a reverse proxy with SSL
- [ ] Configure proper hostname with `KC_HOSTNAME`
- [ ] Set up database backups
- [ ] Enable hostname strict mode for production
- [ ] Review and configure proxy settings if behind a reverse proxy
- [ ] Set appropriate memory limits based on your load
- [ ] Restrict network access to Keycloak (firewall, security groups)
- [ ] Enable monitoring and alerting on health endpoints
- [ ] Keep Keycloak and PostgreSQL images updated
