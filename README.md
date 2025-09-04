# Campfire

Campfire is web-based chat application. It supports many of the features you'd
expect, including:

- Multiple rooms, with access controls
- Direct messages
- File attachments with previews
- Search
- Notifications (via Web Push)
- @mentions
- API, with support for bot integrations

Campfire is single-tenant: any rooms designated "public" will be accessible by
all users in the system. To support entirely distinct groups of customers, you
would deploy multiple instances of the application.

## Running in development

    bin/setup
    bin/rails server

## Deploying with Docker

Campfire's Docker image contains everything needed for a fully-functional,
single-machine deployment. This includes the web app, background jobs, caching,
file serving, and SSL.

To persist storage of the database and file attachments, map a volume to `/rails/storage`.

To configure additional features, you can set the following environment variables:

- `SSL_DOMAIN` - enable automatic SSL via Let's Encrypt for the given domain name
- `DISABLE_SSL` - alternatively, set `DISABLE_SSL` to serve over plain HTTP
- `VAPID_PUBLIC_KEY`/`VAPID_PRIVATE_KEY` - set these to a valid keypair to
  allow sending Web Push notifications. You can generate a new keypair by running
  `/script/admin/create-vapid-key`
- `SENTRY_DSN` - to enable error reporting to sentry in production, supply your
  DSN here

For example:

    docker build -t campfire .

    docker run \
      --publish 80:80 --publish 443:443 \
      --restart unless-stopped \
      --volume campfire:/rails/storage \
      --env SECRET_KEY_BASE=$YOUR_SECRET_KEY_BASE \
      --env VAPID_PUBLIC_KEY=$YOUR_PUBLIC_KEY \
      --env VAPID_PRIVATE_KEY=$YOUR_PRIVATE_KEY \
      --env SSL_DOMAIN=chat.example.com \
      campfire

## Deploying with Docker Compose

For easier deployment and development, you can use Docker Compose which automatically sets up the Rails application with Redis. The setup uses separate compose files for different environments.

### Development

For development with live code reloading and debugging capabilities:

```bash
docker compose -f docker-compose.yml -f docker-compose.dev.yml up
```

Development features:
- Volume mounting for live code reloading
- Interactive TTY for debugging
- Development environment settings
- Debug logging enabled

### Production Deployment

For production deployment:

```bash
docker compose -f docker-compose.yml -f docker-compose.prod.yml up -d
```

**Important**: Before running in production, you need to set the following environment variables in your deployment environment or modify `docker-compose.prod.yml`:

- `RAILS_MASTER_KEY` - your Rails master key for encrypted credentials
- `SECRET_KEY_BASE` - generate with `rails secret`
- `VAPID_PUBLIC_KEY`/`VAPID_PRIVATE_KEY` - for Web Push notifications
- `SENTRY_DSN` - for error reporting (optional)
- `SSL_DOMAIN` - for automatic SSL via Let's Encrypt (optional)

You can set these by editing the environment section in `docker-compose.prod.yml` or by using an external secrets management system.

### Docker Compose Files Structure

- **`docker-compose.yml`**: Base configuration with common services
- **`docker-compose.dev.yml`**: Development-specific overrides and settings
- **`docker-compose.prod.yml`**: Production-specific configuration and environment variables

### Services

- **web**: The Rails application (available on port 3000)
- **redis**: Redis server for caching and background jobs (available on port 6379)

### Persistent Data

The following volumes are created for data persistence:
- `sqlite_data`: SQLite database files
- `redis_data`: Redis persistence
- `rails_tmp`: Temporary files
- `rails_log`: Application logs

### Quick Commands

```bash
# Development
docker compose -f docker-compose.yml -f docker-compose.dev.yml up

# Production
docker compose -f docker-compose.yml -f docker-compose.prod.yml up -d

# View logs
docker compose -f docker-compose.yml -f docker-compose.prod.yml logs -f

# Run Rails console (production)
docker compose -f docker-compose.yml -f docker-compose.prod.yml exec web rails console

# Stop services
docker compose -f docker-compose.yml -f docker-compose.prod.yml down
```
