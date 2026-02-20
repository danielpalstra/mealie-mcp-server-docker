# Mealie MCP Server Docker

A Docker image for the [Mealie MCP Server](https://github.com/rldiao/mealie-mcp-server) built with [uv](https://github.com/astral-sh/uv).

This container provides a Model Context Protocol (MCP) server that exposes Mealie recipe manager APIs to MCP clients like Claude Desktop, enabling AI assistants to interact with your Mealie recipe database.

## Prerequisites

- A running [Mealie](https://github.com/mealie-recipes/mealie) instance
- A Mealie API key (generated from your Mealie account settings)
- Docker or Docker Compose

## Quick Start

```bash
docker run -i \
  -e MEALIE_BASE_URL=https://your-mealie-instance.com \
  -e MEALIE_API_KEY=your-api-key \
  ghcr.io/danielpalstra/mealie-mcp-server-docker:main
```

## Configuration

### Required Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `MEALIE_BASE_URL` | Base URL of your Mealie instance (including protocol and port) | `https://mealie.example.com` |
| `MEALIE_API_KEY` | API key from Mealie account settings | `your-mealie-api-key` |

### Optional Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `LOG_LEVEL` | Logging level | `INFO` |

## Usage

### Docker Run

```bash
docker run -i \
  -e MEALIE_BASE_URL=https://mealie.example.com \
  -e MEALIE_API_KEY=your-secret-api-key \
  ghcr.io/danielpalstra/mealie-mcp-server-docker:main
```

### Docker Compose

Create a `docker-compose.yml` file:

```yaml
version: '3.8'

services:
  mealie-mcp-server:
    image: ghcr.io/danielpalstra/mealie-mcp-server-docker:main
    environment:
      MEALIE_BASE_URL: https://mealie.example.com
      MEALIE_API_KEY: your-secret-api-key
      LOG_LEVEL: INFO
    restart: unless-stopped
    stdin_open: true
```

Then run:

```bash
docker-compose up -d
```

### Claude Desktop Integration

To use with Claude Desktop, add this to your `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "mealie": {
      "command": "docker",
      "args": [
        "run",
        "-i",
        "--rm",
        "-e",
        "MEALIE_BASE_URL=https://your-mealie-instance.com",
        "-e",
        "MEALIE_API_KEY=your-api-key",
        "ghcr.io/danielpalstra/mealie-mcp-server-docker:main"
      ]
    }
  }
}
```

**Using Environment Variables from Host:**

If you have `MEALIE_BASE_URL` and `MEALIE_API_KEY` set in your shell environment (e.g., in `~/.secrets`), you can reference them without hardcoding values:

```json
{
  "mcpServers": {
    "mealie": {
      "command": "docker",
      "args": [
        "run",
        "-i",
        "--rm",
        "-e",
        "MEALIE_BASE_URL",
        "-e",
        "MEALIE_API_KEY",
        "ghcr.io/danielpalstra/mealie-mcp-server-docker:main"
      ]
    }
  }
}
```

Make sure to source your secrets file before starting the MCP client:

```bash
# In ~/.secrets or ~/.config/fish/config.fish
export MEALIE_BASE_URL=https://mealie.example.com
export MEALIE_API_KEY=your-api-key

# Or in fish shell
set -x MEALIE_BASE_URL https://mealie.example.com
set -x MEALIE_API_KEY your-api-key
```

### HTTP Server Mode (Optional)

The container also supports running as an HTTP server using FastMCP:

```bash
docker run -t \
  -p 8000:8000 \
  -e MEALIE_BASE_URL=https://mealie.example.com \
  -e MEALIE_API_KEY=your-api-key \
  ghcr.io/danielpalstra/mealie-mcp-server-docker:main
```

This exposes the MCP server on http://localhost:8000 for programmatic access or debugging.

### Testing the MCP Server

To verify the server is working correctly:

1. **Test with environment variables:**
   ```bash
   # Set your credentials
   export MEALIE_BASE_URL=https://your-mealie.com
   export MEALIE_API_KEY=your-key
   
   # Run the server
   docker run -i --rm \
     -e MEALIE_BASE_URL \
     -e MEALIE_API_KEY \
     ghcr.io/danielpalstra/mealie-mcp-server-docker:main
   ```

2. **Test HTTP mode:**
   ```bash
   docker run -t --rm \
     -p 8000:8000 \
     -e MEALIE_BASE_URL=$MEALIE_BASE_URL \
     -e MEALIE_API_KEY=$MEALIE_API_KEY \
     ghcr.io/danielpalstra/mealie-mcp-server-docker:main
   ```
   
   Then visit http://localhost:8000 in your browser to see the FastMCP interface.

## Available Tags

- `main` - Latest build from the main branch
- `latest` - Latest stable release
- `vX.Y.Z` - Specific version tags
- `X.Y` - Major.minor version tags
- `X` - Major version tags
- `sha-<commit>` - Specific commit builds

## Features

The MCP server provides tools for:

- **Recipe Management**: Search, create, and update recipes
- **Meal Planning**: Create and manage meal plans
- Integration with Mealie's full API

## Building Locally

```bash
docker build -t mealie-mcp-server:local .
```

## Architecture

- **Base Image**: `ghcr.io/astral-sh/uv:python3.12-bookworm-slim`
- **Python Version**: 3.12
- **Package Manager**: uv (fast Python package installer)
- **Size**: ~200MB
- **User**: Non-root user (uid/gid 999)
- **Platforms**: linux/amd64, linux/arm64

## Upstream Project

This Docker image packages the [Mealie MCP Server](https://github.com/rldiao/mealie-mcp-server) by rldiao.

## License

MIT

## Contributing

Contributions are welcome! Please open an issue or submit a pull request.

## Troubleshooting

### Connection Issues

If the server can't connect to Mealie:

1. Verify `MEALIE_BASE_URL` includes the protocol (http:// or https://)
2. Ensure your Mealie instance is accessible from the container
3. Check that the API key is valid and has proper permissions

### Docker Network Issues

If running Mealie and the MCP server in the same Docker network:

```yaml
services:
  mealie:
    image: ghcr.io/mealie-recipes/mealie:latest
    networks:
      - mealie-net
    # ... other config

  mealie-mcp-server:
    image: ghcr.io/danielpalstra/mealie-mcp-server-docker:main
    environment:
      MEALIE_BASE_URL: http://mealie:9000
      MEALIE_API_KEY: your-api-key
    networks:
      - mealie-net

networks:
  mealie-net:
```

## Related Projects

- [Mealie](https://github.com/mealie-recipes/mealie) - Self-hosted recipe manager
- [Mealie MCP Server](https://github.com/rldiao/mealie-mcp-server) - MCP server for Mealie
- [uv](https://github.com/astral-sh/uv) - Fast Python package installer
- [Model Context Protocol](https://modelcontextprotocol.io/) - MCP specification
