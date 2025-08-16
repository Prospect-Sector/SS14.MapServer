# SS14 MapServer

The SS14 MapServer is a web service that automatically renders Space Station 14 map files and provides APIs for accessing map data, images, and tiles. It integrates with GitHub to automatically process map changes in pull requests.

## 🎯 Features

- 🔄 **Automatic GitHub Integration** - Listens for webhooks and processes map changes
- 🗺️ **Map Rendering** - Converts SS14 .yml map files to web-viewable images
- 💬 **PR Comments** - Posts rendered map previews on pull requests
- 🗃️ **Database Storage** - Stores map data, grids, and tiles for fast access
- 🔍 **REST API** - Provides endpoints for map data and image retrieval
- 🎨 **Tile System** - Serves map tiles for interactive viewing

## 📚 Documentation

**Official Documentation:**  
https://spacestation14.com/SS14.MapServer/quickstart.html

## 🚀 Production Setup Guide

### Prerequisites

- Docker and Docker Compose
- PostgreSQL database (local or hosted)
- Domain name with HTTPS (for GitHub webhooks)
- GitHub repository with SS14 content

### 1. Clone and Configure

```bash
git clone https://github.com/space-wizards/SS14.MapServer.git
cd SS14.MapServer
```

### 2. Create Configuration

```bash
# Copy the example configuration
cp appsettings.example.yaml SS14.MapServer/appsettings.yaml
cp docker-compose.example.yml docker-compose.yml

# Edit with your actual values
nano SS14.MapServer/appsettings.yaml
```

### 3. Set Up GitHub App

1. Go to GitHub Settings → Developer settings → GitHub Apps
2. Create a new GitHub App with these settings:
   - **Webhook URL**: `https://your-domain.com/api/githubwebhook`
   - **Webhook Secret**: Generate a secure random string
   - **Permissions**:
     - Repository permissions: Contents (Read), Metadata (Read), Pull requests (Write)
     - Subscribe to events: Pull request, Push
3. Install the app on your repository
4. Download the private key and save as `private-key.pem`

### 4. Configure Your Settings

Edit `SS14.MapServer/appsettings.yaml` with your values:

#### Required Settings
- `AppWebhookSecret`: Your GitHub webhook secret
- `Auth.ApiKey`: Generate a secure API key
- `Git.RepositoryUrl`: Your repository URL (with `.git` suffix!)
- `Github.AppId`: Your GitHub App ID (numeric)
- `Github.AppWebhookSecret`: Same as AppWebhookSecret
- `ConnectionStrings.default`: Your database connection
- `Server.Host`: Your public domain

#### Map File Patterns
Update `Git.MapFilePatterns` to match your map locations:
```yaml
MapFilePatterns:
  - "Resources/Maps/**/*.yml"  # All .yml files in Maps/ and subdirectories
  # OR more specific:
  - "Resources/Maps/_PS/**/*.yml"  # Only files under _PS/ subdirectory
```

### 5. Deploy

```bash
# Start the service
docker-compose up -d

# Check logs
docker-compose logs -f ss14-mapserver
```

## 🔧 Configuration Details

### Critical Settings Explained

#### Repository URL Format
```yaml
# ✅ CORRECT - Include .git suffix
RepositoryUrl: "https://github.com/owner/repo.git"

# ❌ WRONG - Missing .git suffix
RepositoryUrl: "https://github.com/owner/repo"
```

#### Webhook Secret Location
The webhook secret MUST be in both locations:
```yaml
# At root level (required for webhook verification)
AppWebhookSecret: "your-secret"

# In GitHub section (for app configuration)  
Github:
  AppWebhookSecret: "your-secret"
```

#### Security Settings
```yaml
Git:
  DontRunWithCodeChanges: true  # IMPORTANT: Prevents malicious code execution
  CodeChangePatterns:
    - "**/*.cs"  # Detects C# code changes
```

### Database Setup

#### PostgreSQL (Recommended)
```yaml
ConnectionStrings:
  default: "Server=localhost;Port=5432;Database=ss14_mapserver;User Id=postgres;Password=yourpassword;"
```

#### DigitalOcean Managed Database
```yaml
ConnectionStrings:
  default: "Server=your-db-host.db.ondigitalocean.com;Port=25060;Database=ss14-mapserver;User Id=doadmin;Password=yourpassword;"
```

### Map File Patterns

The map server uses glob patterns to find map files. Common patterns:

```yaml
MapFilePatterns:
  # Match all .yml files in Maps/ and any subdirectory
  - "Resources/Maps/**/*.yml"
  
  # Match only files directly in Maps/
  - "Resources/Maps/*.yml"
  
  # Match files in specific subdirectories
  - "Resources/Maps/Ships/**/*.yml"
  - "Resources/Maps/Stations/**/*.yml"
  
  # Match files with specific naming
  - "Resources/Maps/**/station_*.yml"
```

## 📊 API Endpoints

### Management
- `GET /api/management/statistics` - Server statistics
- `POST /api/management/sync` - Manual sync trigger

### Map Data
- `GET /api/map/{mapId}` - Map information
- `GET /api/tile/{mapId}/{gridId}/{tileId}` - Individual tiles
- `GET /api/image/{imageId}` - Map images

All management endpoints require `X-API-Key` header.

#### Example Usage
```bash
# Check statistics
curl -H "X-API-Key: your-api-key" https://your-domain.com/api/management/statistics

# Manual sync (bypass webhooks)
curl -X POST -H "X-API-Key: your-api-key" https://your-domain.com/api/management/sync
```

## 🐛 Troubleshooting

### Webhook Issues

#### 400 Bad Request
- Check repository URL has `.git` suffix
- Verify map file patterns match your file locations
- Ensure Build.Enabled is true

#### 401 Unauthorized  
- Verify webhook secret is set at root level (`AppWebhookSecret`)
- Check GitHub App webhook secret matches configuration

#### 404 Not Found
- Normal for root path (`/`) - the server only has API endpoints
- Webhook endpoint should be `/api/githubwebhook`

### Map Processing Issues

#### No Maps Found
- Check `Git.MapFilePatterns` matches your file structure
- Use `**/*.yml` pattern for subdirectories
- Verify files exist in the correct branch

#### Build Timeouts
- Increase `Build.ProcessTimeoutMinutes` 
- Check container has sufficient resources
- Monitor build logs for specific errors

### Common Debugging Steps

```bash
# View recent logs
docker-compose logs --tail=50 ss14-mapserver

# Restart after configuration changes
docker-compose restart ss14-mapserver

# Check service status
docker-compose ps
```

## 🔐 Security

- Keep API keys secure and don't commit them to version control
- The webhook secret should be a long, random string
- Store the GitHub App private key securely
- `DontRunWithCodeChanges: true` is critical for security
- Use HTTPS for all webhook endpoints

## 📁 Files

- `appsettings.example.yaml` - Example configuration file
- `docker-compose.example.yml` - Example Docker Compose setup

## 🎉 Success Indicators

When everything is working correctly, you should see:
- Webhook deliveries return `200 OK` in GitHub App settings
- Statistics API shows `automatedBuilds: true`
- Map count increases after processing pull requests
- Pull request comments appear with rendered map images

## 🤝 Contributing

See the official documentation for development setup and contribution guidelines.

## 📄 License

This project follows the Space Station 14 licensing.
