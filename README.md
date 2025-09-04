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

For easier deployment and development, you can use Docker Compose which automatically sets up the Rails application with Redis:

### Production Deployment

1. Copy the environment template and configure your variables:
   ```bash
   cp .env.example .env
   ```

2. Edit `.env` and set your production values:
   - `RAILS_MASTER_KEY` - your Rails master key for encrypted credentials
   - `VAPID_PUBLIC_KEY`/`VAPID_PRIVATE_KEY` - for Web Push notifications
   - `SENTRY_DSN` - for error reporting (optional)

3. Deploy the application:
   ```bash
   docker compose up -d
   ```

The application will be available at `http://localhost:3000`.

### Development with Docker Compose

For development, the `docker-compose.override.yml` file automatically configures:
- Volume mounting for live code reloading
- Development environment settings
- Interactive TTY for debugging

Simply run:
```bash
docker compose up
```

### Docker Compose Services

- **web**: The Rails application (available on port 3000)
- **redis**: Redis server for caching and background jobs (available on port 6379)

### Persistent Data

The following volumes are created for data persistence:
- `sqlite_data`: SQLite database files
- `redis_data`: Redis persistence
- `rails_tmp`: Temporary files
- `rails_log`: Application logs
