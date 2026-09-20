# Odoo 20 Installer Test Environment

This folder provides a minimal Docker setup to exercise the `odoo_install.sh` script in a clean Ubuntu 24.04 container.

## Prerequisites
- Docker 24+
- Docker Compose plugin (`docker compose` CLI)

## Build the Image
```bash
docker compose build
```

## Start the Test Container
Run the container detached so it stays online for manual testing:
```bash
docker compose up -d
```

## Run the Installer Script Inside the Container
1. Attach to the container shell:
   ```bash
   docker compose exec odoo20 bash
   ```
2. Make the script executable (first run only):
   ```bash
   chmod +x /opt/odoo-install/odoo_install.sh
   ```
3. Execute the installer:
   ```bash
   /opt/odoo-install/odoo_install.sh
   ```

The script runs as root inside the container, so the bundled `sudo` calls succeed without additional configuration. The Docker image already sets locale and timezone data to avoid interactive prompts from `tzdata`.

## Reset the Environment
To destroy the test container and start fresh:
```bash
docker compose down -v
```

Re-run the **Build** and **Start** steps to begin another test cycle.

## Notes for Odoo 20

- Odoo 20 requires Python 3.12+, so the image has to stay on Ubuntu 24.04 or newer.
- Containers have no `systemd`, so the script automatically falls back to the
  `/etc/init.d` service. Start the server manually with
  `/etc/init.d/odoo-server start` (or run `odoo-bin` directly) to verify the install.
- PostgreSQL is not started by an init system inside the container either; the script
  uses `service postgresql start` as a fallback, which works in this image.
