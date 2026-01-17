# Plan: Docker Support via Volumes and Environment Variables

## Objective
Refactor the app to support Docker containers with volume mounts and environment variable configuration, enabling the same codebase to work locally and in Docker on Windows/Linux.

## Problem Statement
Currently, the app has hardcoded paths that are specific to the host system:
- Kobo devices: `/run/media/{OS_USERNAME}/KOBOeReader/.kobo/KoboReader.sqlite` (Linux-specific)
- Kindle devices: Windows drive letters (Windows-specific via win32api)
- Temp directories: `tmp/` (relative paths not portable across environments)

When running in Docker, these paths don't exist in the container unless volumes are mounted. The app needs to:
1. Support configurable paths via environment variables
2. Work in both local and Docker environments
3. Handle device detection gracefully when devices are unavailable

## Implementation Steps

### Step 1: Push README changes
**Objective:** Document current issue and Docker deployment strategy

**Actions:**
- Update `README.md` with:
  - Section explaining Docker incompatibility (host-specific paths)
  - Section on Docker deployment strategy with volume mounts
  - Example `.env` configuration
  - Docker and docker-compose usage instructions

**Commit:** `chore: Update README with Docker deployment documentation`

### Step 2: Create feature branch
**Objective:** Isolate Docker refactoring work from master

**Actions:**
- Create branch: `feature/docker-volumes`
- Checkout: `git checkout -b feature/docker-volumes`

### Step 3: Create `.env.example` file
**Objective:** Define template for environment configuration

**Content should include:**
```
# Device Paths
KOBO_PATH=/run/media/user/KOBOeReader/.kobo/KoboReader.sqlite
KOBO_ANNOTATIONS_PATH=/run/media/user/KOBOeReader/Digital Editions/Annotations
KINDLE_PATH=/media/Kindle

# Application Paths
TMP_DIR=tmp
UPLOAD_DIR=tmp/uploads

# Device Configuration
RUN_MODE=local
# RUN_MODE options: 'local' (use real devices), 'docker' (use mounted volumes)
```

### Step 4: Create `config.py` module
**Objective:** Centralize path management with environment variable support

**Features:**
- Read environment variables with `.env.example` as defaults
- Provide fallback paths for local vs Docker contexts
- Single source of truth for all application paths
- Helper functions for path validation

**Key exports:**
- `KOBO_PATH`
- `KOBO_ANNOTATIONS_PATH`
- `KINDLE_PATH`
- `LOCAL_COPY_PATH`
- `TMP_DIR`
- `UPLOAD_DIR`
- `RUN_MODE`

### Step 5: Refactor `bobo_reader.py`
**Objective:** Use configurable paths instead of hardcoded values

**Changes:**
- Replace hardcoded path strings with imports from `config.py`
- Update `device_available()` to gracefully handle missing paths
- Update `wait_for_device()` to skip device detection if not available
- Add logging for device detection failures in Docker mode

### Step 6: Update `app.py` and related modules
**Objective:** Replace all hardcoded `tmp/` paths

**Files to check and update:**
- `app.py` - Replace `tmp/` paths with `config.TMP_DIR` and `config.UPLOAD_DIR`
- `reports.py` - Replace report output paths
- `kindle_reports.py` - Replace report output paths
- Any other modules using hardcoded paths

### Step 7: Create `docker-compose.yml`
**Objective:** Define complete Docker service with volume mounts

**Configuration:**
```yaml
version: '3.8'
services:
  bobo-reporter:
    build: .
    ports:
      - "5000:5000"
    volumes:
      # Application tmp directory (for reports)
      - ./tmp:/app/tmp
      # Device mount examples (commented - uncomment as needed)
      # - /run/media/user/KOBOeReader:/kobo:ro
      # - /media/Kindle:/kindle:ro
    env_file:
      - .env
    environment:
      - RUN_MODE=docker
```

### Step 8: Update `Dockerfile`
**Objective:** Ensure paths align with volume mounts

**Changes:**
- Ensure `/app/tmp` and `/app/tmp/uploads` are created
- Add comment about environment variables
- Consider multi-stage build if needed
- Ensure `.env` file is properly copied or sourced

## Environment Variable Mapping

| Variable | Local Default | Docker Default | Description |
|----------|---------------|----------------|-------------|
| `KOBO_PATH` | `/run/media/{user}/KOBOeReader/.kobo/KoboReader.sqlite` | `/kobo/.kobo/KoboReader.sqlite` | Path to Kobo device database |
| `KOBO_ANNOTATIONS_PATH` | `/run/media/{user}/KOBOeReader/Digital Editions/Annotations` | `/kobo/Digital Editions/Annotations` | Path to Kobo annotations |
| `KINDLE_PATH` | Auto-detected (Windows) | `/kindle` | Path to Kindle device |
| `TMP_DIR` | `tmp` | `tmp` | Temporary directory for app files |
| `UPLOAD_DIR` | `tmp/uploads` | `tmp/uploads` | Upload directory for user files |
| `RUN_MODE` | `local` | `docker` | Execution context |

## Docker Usage Examples

### Local Development
```bash
# Copy .env.example to .env and adjust paths for your system
cp .env.example .env

# Run locally (without Docker)
python app.py
```

### Docker with mounted device (Linux)
```bash
# Assuming Kobo is mounted at /run/media/user/KOBOeReader
docker-compose up
```

### Docker on Windows with mounted network share
```bash
# Mount Kindle drive or use volumes for file sharing
docker-compose -f docker-compose.yml up
```

## Further Considerations

### Device Path Mapping Strategy
**Question:** How should host device paths map to container paths?

**Options:**
1. **Fixed container paths** (recommended): Always use `/kobo` and `/kindle` in container, mount host paths there
2. **Environment variable paths**: Pass full host paths via env vars (less portable)
3. **Auto-detection**: Container detects if paths exist, graceful fallback if not

**Recommendation:** Use option 1 with optional volume mounts

### Platform-Specific Behavior
**Question:** Should app detect platform and use appropriate device detection?

**Current state:**
- Windows: Uses win32api to scan drives
- Linux: Looks for mounted devices in `/run/media/`

**Recommendation:** 
- In Docker: Skip device auto-detection, require pre-mounted volumes
- Locally: Keep platform-specific detection but with better error handling
- Detect Docker environment: Check for `/.dockerenv` or `RUN_MODE=docker`

### Local Development vs Docker Deployment
**Question:** Should we use different `.env` files?

**Recommendation:** Single unified `.env.example` with clear comments
- Developers copy to `.env` and customize for their system
- Docker mounts volumes and uses same config structure
- `RUN_MODE` environment variable switches behavior

## Files Changed Summary

| File | Changes |
|------|---------|
| `README.md` | Add Docker documentation section |
| `plans/docker-volumes-refactor.md` | New: This plan document |
| `.env.example` | New: Environment template |
| `config.py` | New: Centralized configuration |
| `bobo_reader.py` | Update to use `config.py` |
| `app.py` | Update paths to use `config.py` |
| `reports.py` | Update paths to use `config.py` |
| `kindle_reports.py` | Update paths to use `config.py` |
| `docker-compose.yml` | New: Docker Compose configuration |
| `Dockerfile` | Minor updates for path handling |

## Success Criteria

- ✅ App runs locally on Windows with Kindle detection
- ✅ App runs locally on Linux with Kobo detection
- ✅ App runs in Docker container on Windows with mounted volumes
- ✅ App runs in Docker container on Linux with mounted volumes
- ✅ Reports generate to configured `TMP_DIR`
- ✅ Uploads work to configured `UPLOAD_DIR`
- ✅ No hardcoded paths in code (all in `config.py`)
- ✅ `.env` file can be used for local customization
- ✅ Docker can be deployed with different volume mounts
