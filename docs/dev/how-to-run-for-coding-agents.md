# How to Run Product Opener for Coding Agents

This guide provides optimized instructions for running Product Opener in memory-constrained environments typically used by AI coding agents (GitHub Copilot workspaces, CI runners, Codespaces, etc.).

## Quick Start for Coding Agents

```bash
# Start lightweight development environment (recommended for CI/agents)
make dev_lightweight

# The site will be available at:
# http://world.openfoodfacts.localhost/
```

### For Restricted Network Environments

If you're in an environment with restricted network access (e.g., some CI runners or sandboxed workspaces), you can use pre-built images instead of building from source:

```bash
# Pull pre-built images from GitHub Container Registry
TAG=latest make pull_prebuilt_images

# Start using pre-built images
TAG=latest make dev_lightweight_prebuilt
```

**Note**: The `dev_lightweight` target builds images locally, which requires access to external package repositories. If the build fails due to network restrictions, use `dev_lightweight_prebuilt` instead.

## Memory Requirements

### Standard Development Setup (`make dev`)
- **RAM**: 8GB minimum
- **Disk**: 10GB+ (with images)
- **MongoDB Cache**: 8GB

### Lightweight Setup (`make dev_lightweight`)
- **RAM**: 4GB minimum (2GB recommended free)
- **Disk**: 3GB (without images)
- **MongoDB Cache**: 1GB

## Available Commands

| Command | Memory Usage | Description |
|---------|--------------|-------------|
| `make dev` | High (8GB+) | Full development setup with all features |
| `make dev_lightweight` | Low (4GB) | Optimized for constrained environments |
| `make dev_lightweight_prebuilt` | Low (4GB) | Uses pre-built images (for restricted networks) |
| `make pull_prebuilt_images` | - | Pull pre-built images from GitHub Container Registry |
| `make up` | Medium | Start existing containers |
| `make down` | - | Stop containers |
| `make status` | - | Check container status |
| `make livecheck` | - | Health check for services |

## Taking Screenshots of Your Changes

When making visual changes, you can capture screenshots to verify your work.

### Prerequisites

Ensure the development server is running:

```bash
# Start lightweight server
make dev_lightweight

# Wait for startup (check status)
make status

# Verify the server is healthy
make livecheck
```

### Using Playwright (Recommended)

If you have Playwright available:

```javascript
const { chromium } = require('playwright');

(async () => {
  const browser = await chromium.launch();
  const page = await browser.newPage();
  
  // Navigate to the site
  await page.goto('http://world.openfoodfacts.localhost/');
  
  // Wait for content to load
  await page.waitForLoadState('networkidle');
  
  // Take screenshot
  await page.screenshot({ path: 'screenshot.png', fullPage: true });
  
  await browser.close();
})();
```

### Using cURL to Verify Server

```bash
# Check if the server is responding
curl -I http://world.openfoodfacts.localhost/

# Get a page content
curl http://world.openfoodfacts.localhost/ | head -100
```

### Key URLs for Testing

| URL | Description |
|-----|-------------|
| `http://world.openfoodfacts.localhost/` | Main homepage |
| `http://world.openfoodfacts.localhost/product/3017620422003` | Example product page |
| `http://world.openfoodfacts.localhost/cgi/search.pl` | Search page |
| `http://world.openfoodfacts.localhost/api/v2/product/3017620422003` | API endpoint |
| `http://fr.openfoodfacts.localhost/` | French localized site |

## Environment Variables for Optimization

You can customize the lightweight setup with these environment variables:

```bash
# Reduce MongoDB memory usage (in GB)
export MONGODB_CACHE_SIZE=1

# Skip downloading product images
export SKIP_SAMPLE_IMAGES=1

# Increase Docker timeouts for slow environments
export DOCKER_CLIENT_TIMEOUT=600
export COMPOSE_HTTP_TIMEOUT=600
```

## Troubleshooting

### "Cannot connect to Docker daemon"

Ensure Docker is installed and running:
```bash
docker --version
docker ps
```

### Out of Memory Errors

1. Use lightweight mode:
   ```bash
   make dev_lightweight
   ```

2. Reduce MongoDB cache:
   ```bash
   export MONGODB_CACHE_SIZE=1
   make dev_lightweight
   ```

3. Stop unused containers:
   ```bash
   make down
   docker system prune -f
   ```

### Host Resolution Errors

Add entries to `/etc/hosts`:
```
127.0.0.1 world.openfoodfacts.localhost fr.openfoodfacts.localhost static.openfoodfacts.localhost ssl-api.openfoodfacts.localhost
```

### Containers Exit Unexpectedly

Check logs for errors:
```bash
make log 2>&1 | tail -100
```

Common causes:
- Insufficient memory (use lightweight mode)
- Port 80 already in use
- Missing Docker network

### First Run Takes Too Long

The first run downloads and builds Docker images. This is normal. Subsequent runs will be faster.

For CI environments, consider using pre-built images:
```bash
export TAG=latest
make up
```

## CI/CD Integration

### GitHub Actions Example

```yaml
jobs:
  visual-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Start Product Opener
        run: |
          make dev_lightweight
          sleep 60  # Wait for services to start
          
      - name: Verify server is running
        run: |
          make status
          curl -f http://world.openfoodfacts.localhost/ || exit 1
          
      - name: Take screenshot
        uses: actions/setup-node@v4
        with:
          node-version: '20'
      - run: |
          npx playwright install chromium
          node -e "
            const { chromium } = require('playwright');
            (async () => {
              const browser = await chromium.launch();
              const page = await browser.newPage();
              await page.goto('http://world.openfoodfacts.localhost/');
              await page.waitForLoadState('networkidle');
              await page.screenshot({ path: 'screenshot.png', fullPage: true });
              await browser.close();
            })();
          "
          
      - name: Upload screenshot
        uses: actions/upload-artifact@v4
        with:
          name: screenshot
          path: screenshot.png
```

## Performance Tips

1. **Use SSD storage** - Docker I/O is significantly faster on SSDs

2. **Allocate enough memory to Docker** - At least 4GB for lightweight, 8GB for full

3. **Cache Docker layers** - Subsequent builds will be faster

4. **Skip image downloads** - If you don't need product images:
   ```bash
   export SKIP_SAMPLE_IMAGES=1
   ```

## Sample Products

The lightweight setup imports ~100 sample products. Some example barcodes to test:

- `3017620422003` - Nutella
- `5449000000996` - Coca-Cola
- `7622210449283` - Oreo
- `3560070824809` - Sample product

View any product at:
```
http://world.openfoodfacts.localhost/product/{barcode}
```

## Next Steps

- See the [Quick Start Guide](how-to-quick-start-guide.md) for full setup instructions
- Read [Docker Development Guide](how-to-develop-using-docker.md) for advanced usage
- Check [Test Writing Guide](how-to-write-and-run-tests.md) for running tests
