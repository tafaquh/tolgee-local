# Tolgee Translation Platform

This is a self-hosted Tolgee instance running with Docker Compose. Tolgee is an open-source localization platform that helps manage translations for your applications.

## Prerequisites

- Docker and Docker Compose installed
- At least 2GB of free RAM
- Ports 8089 and 25432 available

## Quick Start

### 1. Configure the Application

Edit the `config.yaml` file and update the following values:

```yaml
tolgee:
  authentication:
    jwt-secret: <your-secure-secret>  # Must be at least 32 characters (256 bits)
    initial-username: admin            # Your admin username
    initial-password: admin            # Change this to a secure password

  machine-translation:
    google:
      api-key: <your-google-api-key>  # Optional: for Google Translate integration

  smtp:                                # Optional: for email notifications
    host: <your-smtp-host>
    port: 465
    username: <your-email>
    password: <your-password>
    from: Tolgee <no-reply@yourdomain.com>
```

#### Generating a Secure JWT Secret

The JWT secret must be at least 32 characters (256 bits). Generate one with:

```bash
openssl rand -base64 32
```

### 2. Start the Services

```bash
docker-compose up -d
```

This will start:
- **Tolgee App** on http://localhost:8089
- **PostgreSQL Database** on port 25432

### 3. Access Tolgee

Open your browser and navigate to:

```
http://localhost:8089
```

Login with the credentials you set in `config.yaml`:
- **Username**: `admin` (or what you configured)
- **Password**: `admin` (or what you configured)

## Management Commands

### View Logs

```bash
# View Tolgee application logs
docker logs tolgee-app-1 -f

# View PostgreSQL logs
docker logs tolgee-db-1 -f
```

### Check Container Status

```bash
docker ps
```

Look for containers with status `(healthy)`. If you see `(unhealthy)`, check the logs.

### Stop Services

```bash
docker-compose down
```

### Restart Services

```bash
docker-compose restart
```

### Stop and Remove All Data

```bash
docker-compose down -v
rm -rf ./data
```

**Warning**: This will delete all your translations and database!

## Directory Structure

```
.
├── config.yaml           # Tolgee configuration
├── docker-compose.yml    # Docker Compose configuration
├── data/                 # Persistent data directory
│   ├── postgres/         # PostgreSQL data
│   └── ...               # Tolgee data files
└── README.md             # This file
```

## Configuration Details

### Machine Translation Setup

Tolgee supports multiple translation providers:

#### Google Cloud Translation

1. Create a Google Cloud project at https://console.cloud.google.com/
2. Enable the "Cloud Translation API"
3. Create an API key in "APIs & Services" � "Credentials"
4. Add the key to `config.yaml`:
   ```yaml
   machine-translation:
     google:
       api-key: AIzaSyD...your-key
   ```

#### Other Providers

You can also configure:
- **AWS Translate**
- **DeepL** (recommended for better quality)
- **Azure Translator**

See [Tolgee documentation](https://tolgee.io/platform/integrations/translation_services) for details.

### SMTP Email Configuration

For email notifications (password resets, invitations, etc.):

```yaml
smtp:
  auth: true
  from: Tolgee <no-reply@yourdomain.com>
  host: smtp.gmail.com              # Your SMTP server
  password: your-app-password       # SMTP password
  port: 587                         # Common ports: 587 (TLS), 465 (SSL)
  ssl-enabled: true                 # Use true for port 465, false for 587
  username: your-email@gmail.com
```

**Note**: For Gmail, you'll need to create an [App Password](https://support.google.com/accounts/answer/185833).

## Troubleshooting

### Container is Unhealthy

Check the logs for errors:
```bash
docker logs tolgee-app-1 --tail 50
```

Common issues:
- **JWT secret too short**: Must be at least 32 characters
- **Database connection failed**: Ensure PostgreSQL container is running
- **Port already in use**: Change port mapping in `docker-compose.yml`

### Cannot Access localhost:8089

1. Verify container is running and healthy:
   ```bash
   docker ps
   ```

2. Check if port is accessible:
   ```bash
   curl http://localhost:8089
   ```

3. If on WSL2, try accessing from Windows host

### Database Connection Issues

Ensure the database container is running:
```bash
docker ps --filter name=tolgee-db
```

If it's not running:
```bash
docker-compose up -d db
```

## Backup and Restore

### Backup Database

```bash
docker exec tolgee-db-1 pg_dump -U postgres postgres > backup.sql
```

### Restore Database

```bash
docker exec -i tolgee-db-1 psql -U postgres postgres < backup.sql
```

### Backup Data Directory

```bash
tar -czf tolgee-backup-$(date +%Y%m%d).tar.gz ./data
```

## Upgrading Tolgee

1. Backup your data (see above)
2. Pull the latest image:
   ```bash
   docker-compose pull
   ```
3. Restart services:
   ```bash
   docker-compose down
   docker-compose up -d
   ```

## Security Recommendations

1. **Change default credentials** immediately after first login
2. **Use a strong JWT secret** (generate with `openssl rand -base64 32`)
3. **Don't expose PostgreSQL port** to the internet (only for local access)
4. **Use HTTPS** in production (consider using a reverse proxy like Nginx)
5. **Keep Docker images updated** regularly
6. **Backup regularly** to prevent data loss

## Production Deployment

For production use, consider:

- Using a reverse proxy (Nginx/Traefik) with SSL/TLS
- Setting up automated backups
- Using environment variables instead of config file for secrets
- Monitoring container health
- Setting up log rotation
- Using a managed PostgreSQL database

## Resources

- [Tolgee Documentation](https://tolgee.io/platform)
- [Tolgee GitHub](https://github.com/tolgee/tolgee-platform)
- [Docker Compose Documentation](https://docs.docker.com/compose/)

## License

This setup uses Tolgee, which is available under the Apache 2.0 license.
