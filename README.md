# Prowlarr Docker Stack

Centralized indexer manager for torrent and usenet sources. Acts as a proxy between your *arr applications (Radarr, Sonarr, Lidarr) and indexers.

## Quick Start

```bash
cp .env.example .env
# Edit .env to match your system
docker-compose up -d
```

Access the web interface at: http://localhost:9696

## What is Prowlarr?

Prowlarr centralizes indexer management for the *arr suite:
- **Single configuration point** for all indexers
- **Automatic syncing** of indexers to Radarr/Sonarr/Lidarr
- **Built-in proxy support** including FlareSolverr
- **Statistics and health monitoring** for indexers

## Configuration

### Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `PUID` | 1000 | User ID for file permissions |
| `PGID` | 1000 | Group ID for file permissions |
| `TZ` | America/New_York | Timezone |
| `PROWLARR_PORT` | 9696 | Web interface port |

### Directory Structure

```
prowlarr/
└── config/     # Prowlarr configuration and database (auto-created)
```

## Initial Setup

### 1. Access Web Interface
Navigate to http://localhost:9696 and complete the setup wizard.

### 2. Add *arr Applications
Go to **Settings → Apps** and add your applications:

**For Radarr:**
- **Name**: Radarr
- **Sync Level**: Full Sync
- **Prowlarr Server**: `http://localhost:9696`
- **Radarr Server**: `http://radarr:7878` (if using Docker) or `http://localhost:7878`
- **API Key**: Get from Radarr → Settings → General

**For Sonarr:**
- **Name**: Sonarr  
- **Sync Level**: Full Sync
- **Prowlarr Server**: `http://localhost:9696`
- **Sonarr Server**: `http://sonarr:8989` (if using Docker) or `http://localhost:8989`
- **API Key**: Get from Sonarr → Settings → General

**For Lidarr:**
- **Name**: Lidarr
- **Sync Level**: Full Sync  
- **Prowlarr Server**: `http://localhost:9696`
- **Lidarr Server**: `http://lidarr:8686` (if using Docker) or `http://localhost:8686`
- **API Key**: Get from Lidarr → Settings → General

### 3. Configure FlareSolverr (Optional)
If using FlareSolverr for CloudFlare-protected sites:

Go to **Settings → Indexers**:
- **Name**: FlareSolverr
- **Tags**: flaresolverr
- **Host**: `http://flaresolverr:8191/` (if using Docker) or `http://localhost:8191/`

### 4. Add Indexers
Go to **Indexers → Add Indexer** and configure your torrent/usenet sites:
- Public trackers (1337x, RARBG, etc.)
- Private trackers (requires account)
- Usenet indexers (requires subscription)

For CloudFlare-protected indexers, add the `flaresolverr` tag.

## Docker Network Setup

### Same Host Deployment
If all services are on the same Docker host, create a shared network:

```bash
docker network create arr-network
```

Add to each service's compose file:
```yaml
networks:
  - arr-network

networks:
  arr-network:
    external: true
```

### Service URLs for Docker Networks
When services are in the same Docker network:
- Prowlarr: `http://prowlarr:9696`
- Radarr: `http://radarr:7878`
- Sonarr: `http://sonarr:8989`
- Lidarr: `http://lidarr:8686`
- qBittorrent: `http://qbittorrent:9865`
- FlareSolverr: `http://flaresolverr:8191`

## Troubleshooting

### Cannot Connect to *arr Applications
- Verify network connectivity between containers
- Check if applications are running: `docker ps`
- Verify API keys are correct
- Test connectivity: `docker exec prowlarr ping radarr`

### Indexers Not Syncing
- Check **System → Tasks** for sync errors
- Verify *arr application connections are healthy
- Check indexer configurations and credentials

### Search Results Not Appearing
- Test indexers individually in **Indexers** tab
- Check indexer health and response times
- Verify categories are configured correctly
- Check *arr application logs for errors

### FlareSolverr Issues
- Ensure FlareSolverr container is running
- Test FlareSolverr connectivity from Prowlarr
- Check FlareSolverr logs for errors
- Verify timeout settings

## Security Notes

- Keep API keys secure and rotate them periodically
- Use strong authentication for private trackers
- Monitor indexer usage to avoid rate limiting
- Keep Prowlarr updated for security patches

## Backup

Important files to backup:
- `./config` - Contains all Prowlarr settings and database

```bash
tar -czf prowlarr-backup-$(date +%Y%m%d).tar.gz config/
```

## Performance Tips

- Disable unused indexers to improve performance
- Configure appropriate timeout values for slow indexers
- Use tags to organize indexers by type or quality
- Monitor indexer statistics and remove unreliable sources

## Updates

```bash
docker-compose pull
docker-compose up -d
docker image prune
```