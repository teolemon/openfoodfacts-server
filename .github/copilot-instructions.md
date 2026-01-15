# Copilot Instructions for Open Food Facts Product Opener

This document provides instructions for AI coding agents (GitHub Copilot, Gemini Code Assist, etc.) to effectively work with the Open Food Facts Product Opener codebase.

## Quick Reference

| Task | Command |
|------|---------|
| Start lightweight dev environment | `make dev_lightweight` |
| Start full dev environment | `make dev` |
| Run unit tests | `make unit_test` |
| Run single test | `make test-unit test=filename.t` |
| Check Perl syntax | `make check_perl_fast` |
| Lint code | `make lint` |
| Stop containers | `make down` |
| View logs | `make log` |

## Running Product Opener for Visual Verification

### Recommended: Lightweight Development Setup

For coding agents running in memory-constrained environments (e.g., CI runners with 8GB RAM), use the lightweight setup:

```bash
# 1. Start the lightweight development environment
make dev_lightweight

# 2. Wait for containers to start (typically 2-3 minutes for first run)
# The site will be available at: http://world.openfoodfacts.localhost/

# 3. To take a screenshot, use a headless browser
# Example with Playwright (if available):
# Navigate to http://world.openfoodfacts.localhost/ and capture screenshot
```

### What `make dev_lightweight` Does

- Uses reduced MongoDB cache (1GB instead of 8GB)
- Skips downloading product images (saves bandwidth and disk space)
- Imports minimal sample data (~100 products)
- Optimized for faster startup in CI environments

### Accessing the Site

After startup, the site is available at:
- Main site: `http://world.openfoodfacts.localhost/`
- French site: `http://fr.openfoodfacts.localhost/`
- API: `http://world.openfoodfacts.localhost/api/v2/`

**Note:** You may need to add these entries to `/etc/hosts`:
```
127.0.0.1 world.openfoodfacts.localhost fr.openfoodfacts.localhost static.openfoodfacts.localhost
```

### Taking Screenshots

When making UI changes, you can verify them by:

1. Starting the development server with `make dev_lightweight`
2. Waiting for the server to be ready (check with `make status`)
3. Using a headless browser (Playwright, Puppeteer) to navigate and capture screenshots

Example verification workflow:
```bash
# Check if containers are running
make status

# Check container health
make livecheck

# View recent logs for any errors
make log 2>&1 | tail -50
```

## Project Structure

- `lib/ProductOpener/` - Perl backend modules
- `cgi/` - CGI scripts for web endpoints
- `templates/` - Template Toolkit HTML templates
- `html/` - Static HTML files
- `scss/` - SASS stylesheets (compiled by gulp)
- `tests/unit/` - Unit tests
- `tests/integration/` - Integration tests
- `taxonomies/` - Taxonomy definition files

## Code Style Guidelines

### Perl
- Follow existing code style (run `make lint` to auto-format)
- Use `perltidy` for formatting
- Pass `perlcritic` checks
- All Perl files must compile: `make check_perl_fast`

### Frontend
- JavaScript follows ESLint configuration
- SCSS follows Stylelint configuration
- Run `make front_lint` to check

### Tests
- Unit tests go in `tests/unit/`
- Integration tests go in `tests/integration/`
- Run tests with `make test-unit test=your_test.t`

## Memory Considerations

Product Opener is resource-intensive. Here are tips for running in constrained environments:

1. **Use lightweight mode**: `make dev_lightweight` reduces memory usage significantly
2. **MongoDB cache**: Default is 8GB. Set `MONGODB_CACHE_SIZE=1` for minimal memory
3. **Skip images**: Set `SKIP_SAMPLE_IMAGES=1` to avoid downloading product images
4. **CPU limiting**: The Makefile automatically limits to 8 CPU cores to prevent OOM

## Common Tasks

### Making Code Changes

1. Edit files in `lib/ProductOpener/` or `cgi/`
2. Check syntax: `make check_perl_fast`
3. Run related tests: `make test-unit test=relevant_test.t`
4. Lint code: `make lint`

### Modifying Templates

1. Edit files in `templates/`
2. Restart backend: `make restart_backend`
3. Refresh the browser to see changes

### Updating Frontend Assets

1. Edit files in `scss/` or `html/js/`
2. Run `make front_build` to compile
3. Refresh the browser to see changes

### Running Tests

```bash
# All unit tests
make unit_test

# Single unit test
make test-unit test=specific_test.t

# Integration tests
make integration_test

# Single integration test
make test-int test=specific_test.t
```

## Troubleshooting

### Containers won't start
```bash
# Check Docker is running
docker ps

# Clean up and restart
make down
make dev_lightweight
```

### Out of memory
```bash
# Use lightweight mode with minimal resources
export MONGODB_CACHE_SIZE=1
make dev_lightweight
```

### Port conflicts
```bash
# Check what's using port 80
sudo lsof -i :80

# Stop existing containers
make down
```

## Additional Resources

- [Quick Start Guide](../docs/dev/how-to-quick-start-guide.md)
- [Docker Development Guide](../docs/dev/how-to-develop-using-docker.md)
- [Running for Coding Agents](../docs/dev/how-to-run-for-coding-agents.md)
- [Test Writing Guide](../docs/dev/how-to-write-and-run-tests.md)
