# Local Deployment Guide

This guide will help you pull a copy of the Apify MCP Server and deploy it on your local machine.

## Table of Contents
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Configuration](#configuration)
- [Running the Server](#running-the-server)
- [Testing Your Deployment](#testing-your-deployment)
- [Docker Deployment](#docker-deployment)
- [Troubleshooting](#troubleshooting)

## Prerequisites

Before you begin, ensure you have the following installed on your local machine:

1. **Node.js** (v20.0.0 or higher)
   - Check version: `node --version`
   - Download from: https://nodejs.org/

2. **npm** (comes with Node.js)
   - Check version: `npm --version`

3. **Git**
   - Check version: `git --version`
   - Download from: https://git-scm.com/

4. **Apify Account & API Token**
   - Sign up at: https://console.apify.com/sign-up
   - Get your API token from: https://console.apify.com/account/integrations

## Installation

### Step 1: Clone the Repository

```bash
# Clone the repository
git clone https://github.com/apify/apify-mcp-server.git

# Navigate to the project directory
cd apify-mcp-server
```

### Step 2: Install Dependencies

```bash
# Install all required dependencies
npm install
```

This will install all the necessary packages defined in `package.json`.

### Step 3: Build the Project

```bash
# Build the TypeScript source code
npm run build
```

This compiles the TypeScript files in the `src/` directory to JavaScript in the `dist/` directory.

## Configuration

### Step 1: Set Up Environment Variables

Create a `.env` file in the root directory of the project:

```bash
# Copy the example environment file
cp .env.example .env
```

### Step 2: Add Your Apify API Token

Edit the `.env` file and add your Apify API token:

```env
APIFY_TOKEN=your_apify_token_here
```

To get your Apify API token:
1. Log in to the [Apify Console](https://console.apify.com/)
2. Go to **Settings** → **Integrations**
3. Copy your API token

### Step 3: Optional Configuration

You can configure additional settings:

```env
# Disable telemetry (optional)
TELEMETRY_ENABLED=false
```

## Running the Server

The Apify MCP Server can run in two modes:

### Mode 1: Standard Input/Output (stdio)

This mode is ideal for local integrations with MCP clients like Claude Desktop:

```bash
# Run the stdio server
node dist/stdio.js
```

Or using npm:

```bash
npx @apify/actors-mcp-server
```

#### Configure with MCP Clients

For Claude Desktop, add this configuration to your `claude_desktop_config.json`:

**macOS**: `~/Library/Application Support/Claude/claude_desktop_config.json`
**Windows**: `%APPDATA%\Claude\claude_desktop_config.json`

```json
{
  "mcpServers": {
    "apify": {
      "command": "node",
      "args": ["/path/to/apify-mcp-server/dist/stdio.js"],
      "env": {
        "APIFY_TOKEN": "your_apify_token_here"
      }
    }
  }
}
```

Replace `/path/to/apify-mcp-server` with the actual path to your cloned repository.

### Mode 2: HTTP Streamable (Development Server)

This mode runs an HTTP server for development and testing:

```bash
# Set required environment variables
export APIFY_TOKEN="your_apify_token_here"
export APIFY_META_ORIGIN=STANDBY

# Run using Apify CLI
apify run -p
```

Or using npm:

```bash
npm run start:dev
```

The server will be available at `http://localhost:3001`.

### Custom Tool Configuration

You can specify which tools to load:

```bash
# Load specific tool categories
npx @apify/actors-mcp-server --tools actors,docs,storage

# Load specific Actor tools
npx @apify/actors-mcp-server --tools apify/rag-web-browser,apify/google-search-scraper

# Disable telemetry
npx @apify/actors-mcp-server --telemetry-enabled=false
```

For more options, run:

```bash
npx @apify/actors-mcp-server --help
```

## Testing Your Deployment

### Test with MCP Inspector

The [MCP Inspector](https://github.com/modelcontextprotocol/inspector) is a debugging tool for MCP servers:

```bash
# Install and run MCP Inspector
export APIFY_TOKEN="your_apify_token_here"
npx @modelcontextprotocol/inspector node dist/stdio.js
```

The Inspector will display a URL (typically `http://localhost:5173`) that you can open in your browser to:
- View available tools
- Test tool calls
- Inspect server responses
- Debug issues

### Verify Installation

Check that the server is working correctly:

```bash
# Check TypeScript compilation
npm run type-check

# Run unit tests
npm run test:unit

# Run linting
npm run lint
```

## Docker Deployment

### Build the Docker Image

```bash
# Build the Docker image
docker build -t apify-mcp-server .
```

### Run the Docker Container

```bash
# Run the container with your API token
docker run -e APIFY_TOKEN=your_apify_token_here apify-mcp-server
```

Or using Docker Compose, create a `docker-compose.yml`:

```yaml
version: '3.8'
services:
  apify-mcp-server:
    build: .
    environment:
      - APIFY_TOKEN=${APIFY_TOKEN}
    stdin_open: true
    tty: true
```

Then run:

```bash
# Set your token
export APIFY_TOKEN=your_apify_token_here

# Start the container
docker-compose up
```

## Troubleshooting

### Common Issues

#### 1. "APIFY_TOKEN is not set"

**Solution**: Ensure your `.env` file exists and contains your API token, or export it in your shell:

```bash
export APIFY_TOKEN=your_apify_token_here
```

#### 2. "Module not found" errors

**Solution**: Make sure you've installed dependencies and built the project:

```bash
npm install
npm run build
```

#### 3. Node version incompatibility

**Solution**: Verify you're using Node.js v20.0.0 or higher:

```bash
node --version
```

If your version is too old, update Node.js from https://nodejs.org/

#### 4. Build failures

**Solution**: Clean the build and try again:

```bash
npm run clean
npm install
npm run build
```

#### 5. Port already in use (HTTP mode)

**Solution**: The default port (3001) might be in use. Change it by setting the `PORT` environment variable:

```bash
export PORT=3002
npm run start:dev
```

### Getting Help

If you encounter issues:

1. Check the [main README](README.md) for additional information
2. Review the [troubleshooting section](README.md#-troubleshooting-local-mcp-server)
3. Search for existing [GitHub issues](https://github.com/apify/apify-mcp-server/issues)
4. Open a new issue with:
   - Your Node.js version (`node --version`)
   - Your npm version (`npm --version`)
   - The exact error message
   - Steps to reproduce the issue

## Next Steps

After successful deployment:

1. **Explore Available Tools**: Use the MCP Inspector to see what tools are available
2. **Configure Your MCP Client**: Set up Claude Desktop or another MCP client to use your local server
3. **Try Sample Actors**: Test with `apify/rag-web-browser` or other popular Actors
4. **Read the Documentation**: Check out the [Apify MCP documentation](https://docs.apify.com/platform/integrations/mcp)

## Development

If you want to contribute or develop further:

```bash
# Start development server with hot reload
npm run start:dev

# Run tests
npm run test

# Run integration tests (requires build first)
npm run test:integration

# Lint and fix code
npm run lint:fix

# Type check
npm run type-check
```

See [CONTRIBUTING.md](CONTRIBUTING.md) for detailed contribution guidelines.

## Learn More

- [Apify Platform Documentation](https://docs.apify.com/)
- [Model Context Protocol](https://modelcontextprotocol.org/)
- [Apify Store](https://apify.com/store)
- [What is MCP and why does it matter?](https://blog.apify.com/what-is-model-context-protocol/)
- [How to use MCP with Apify Actors](https://blog.apify.com/how-to-use-mcp/)
