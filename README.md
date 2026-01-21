# Pterodactyl Local Deployment (HTTP-only)

Overview
- This repository contains two docker-compose definitions to run a complete Pterodactyl stack (Panel, Database, Wings, Redis cache).
- All services are configured to run over plain HTTP only — there is no reverse proxy, DNS, or SSL certificate management included. Use strictly on a trusted local network.

Contents
- `docker-compose.yml` — uses host bind mounts (local filesystem) for the Panel and Wings data.
- `docker-compose-volumes.yml` — uses Docker named volumes only.
- `example.env` — example environment variables (DB credentials, TIMEZONE, PANEL_URL). Change the database password before use.

Important warnings
> **Warning:** This setup communicates over HTTP. Do NOT expose the Panel or Wings to the public internet. Only run within a secure local network.  
> **Warning:** Do NOT port-forward the Panel (web UI) or Wings management ports. Only port-forward game/server ports if absolutely necessary and you understand the security implications.

Quick differences
- Bind mounts (docker-compose.yml): good for development and direct access to files (`./panel`, `./wings`).
- Named volumes (docker-compose-volumes.yml): better for production-like isolation; data managed by Docker volumes (`panel_app`, `panel_wings`, `panel_db`, etc.).

Environment
- Edit `example.env` to set:
  - `MYSQL_DATABASE`
  - `MYSQL_USER`
  - `MYSQL_PASSWORD` (change immediately for security)
  - `TIMEZONE`
  - `PANEL_URL` (panel address reachable by Wings; for local setups this may be the host IP)

Initial install & first-run steps
1. Copy and edit `example.env` with secure values.
2. Start the stack:
   ```bash
   docker-compose up -d
   ```
3. Enter the Panel container (replace container name as needed):
   ```bash
   docker exec -it <pterodactyl_panel_container> bash
   ```
4. Create the first admin user:
   ```bash
   php artisan p:user:make
   ```
   Follow prompts to create an administrator account.
5. Log into the Panel using the new credentials and open the Admin area (gear icon).

Node and Wings configuration
- In Admin → Locations create a Location name of your choice.
- In Admin → Nodes create a Node:
  - Name: any descriptive name.
  - Location: select the Location you created.
  - FQDN: use the server IP or hostname reachable by Wings (no DNS or proxy required).
  - Connection: choose HTTP (no SSL certificate).
  - Not behind a proxy: set accordingly (this repo assumes no reverse proxy).
  - Set disk and memory allocations and any over-allocation percentages needed.
- After creating the Node, open the Node → Configuration and download the generated `config.yml`.
- Place this `config.yml` into the Wings configuration directory used by your compose file:
  - If using bind mounts: `./wings/config.yml` (this maps to `/etc/pterodactyl` inside the Wings container).
  - If using volumes: place the file into the `panel_wings` volume content or mount it to `/etc/pterodactyl` accordingly.

Restart Wings
- After placing `config.yml`, restart the Wings container:
  ```bash
  docker-compose restart pterodactyl_wings
  ```

Notes
- Only forward ports for game servers when required; never forward Panel or Wings control ports to the public internet.
- This README is intended for local network deployments only.

