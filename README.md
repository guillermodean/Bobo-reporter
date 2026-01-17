# Bobo Reporter

![Bobo Reporter Web App Screenshot](public\img\terminalbobo.png)

**Bobo Reporter** is a tool designed to help you generate reports of your eBooks highlights quickly and efficiently. Connect your Kobo or Kindle eReader to your PC and automatically create a PDF or HTML report of your highlights and annotations.

## Features

- **Compatibility**: Kobo (GNU/Linux systems) and Kindle (Windows).
- **Output formats**: Generate reports in PDF or HTML.
- **Web App**: Modern web interface for easy access (NEW!)

![Bobo Reporter Web App Screenshot](public\img\webbobo.png)


- **Terminal App**: Original command-line interface still available

## Usage

### Web App (Recommended)

1. Install dependencies:
   ```bash
   pip install -r requirements.txt
   ```

2. Start the web server:
   ```bash
   python app.py
   ```

3. Open your browser and navigate to:
   ```
   http://localhost:5000
   ```

4. Select your device type (Kobo or Kindle), scan for devices, or upload a file manually.

### Terminal App

Run the original terminal application:
```bash
python main.py
```

## Docker Deployment

### Overview
Bobo Reporter can run in Docker containers on both Windows and Linux. The Docker setup uses volume mounts to access your eReader devices and configuration files.

### Prerequisites
- Docker and Docker Compose installed
- Your eReader device connected and mounted on your system

### Quick Start with Docker Compose

1. Copy the environment template and customize for your setup:
   ```bash
   cp .env.example .env
   ```

2. Edit `.env` to set your device paths and username:
   ```env
   # Linux - Kobo device path
   KOBO_PATH=/run/media/your_username/KOBOeReader/.kobo/KoboReader.sqlite
   KOBO_ANNOTATIONS_PATH=/run/media/your_username/KOBOeReader/Digital Editions/Annotations
   
   # Windows - Kindle (auto-detected if available)
   KINDLE_PATH=
   
   RUN_MODE=docker
   ```

3. Start the application:
   ```bash
   docker-compose up
   ```

4. Open your browser to `http://localhost:5000`

### Docker Volume Mounts

The `docker-compose.yml` includes volume mounts for:
- **Application reports**: `./tmp:/app/tmp` (persists generated reports)
- **Device mounts** (uncomment and adjust as needed):
  - Kobo: `/run/media/user/KOBOeReader:/kobo:ro`
  - Kindle: `/media/Kindle:/kindle:ro`

### Environment Variables

Configuration is managed via `.env` file:

| Variable | Description | Example |
|----------|-------------|---------|
| `KOBO_PATH` | Path to Kobo device database | `/run/media/user/KOBOeReader/.kobo/KoboReader.sqlite` |
| `KOBO_ANNOTATIONS_PATH` | Path to Kobo annotations folder | `/run/media/user/KOBOeReader/Digital Editions/Annotations` |
| `KINDLE_PATH` | Path to Kindle device | `/media/Kindle` |
| `TMP_DIR` | Temporary directory for reports | `tmp` |
| `UPLOAD_DIR` | Upload directory for files | `tmp/uploads` |
| `RUN_MODE` | Execution context | `local` or `docker` |

### Local vs Docker Deployment

**Local Development:**
```bash
pip install -r requirements.txt
python app.py
```

**Docker (recommended for servers):**
```bash
docker-compose up -d
```

## TODO
[] A lot of refactor
[] More compatibility
[] Kobo unofficial annotations support
[] Seduce Willy to remove Pandas
[] Little external dependencies 
[] Write license

## Requirements

- **Operating System**: Linux or Windows.
- **Device**: Kobo (Linux) or Kindle (Windows), or file upload.
- **USB Connection**: To connect your device to the PC (optional if using file uploads).
- **Docker** (optional): For containerized deployment.

## License

This project is licensed under the Beerware License.

---

Bobo Reporter, making your readings more organized.
