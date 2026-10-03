# Isso Self-Hosted Comments

This directory contains the Docker Compose setup for self-hosting [Isso](https://isso-comments.de/) as the comment system for The Low Code Builder.

## Why Isso?

Cusdis (the previous comment system) was deprecated and archived in July 2026. Isso is a lightweight, privacy-focused, self-hosted alternative that supports anonymous commenting — no sign-in required.

## Setup on your OMV (Proxmox)

### 1. Copy files to your server

Copy the entire `isso/` directory to your OMV Docker compose location.

### 2. Update configuration

Edit `config/isso.cfg`:
- **`[admin] password`** — change to a strong password
- **`[hash] salt`** — change to a random string
- **`[general] host`** — ensure your domain is listed

### 3. Start the container

```bash
docker compose up -d
```

### 4. Set up reverse proxy

Point `comments.thelowcodebuilder.com` (or your chosen subdomain) to the Isso container on port `8080`. If using Nginx Proxy Manager or Traefik on your OMV, add a proxy host entry.

### 5. Update Hugo config

In `layouts/partials/comments.html`, update the `$issoURL` variable to match your Isso server URL:

```go
{{- $issoURL := "https://comments.thelowcodebuilder.com" -}}
```

### 6. Admin panel

Access the admin panel at `https://comments.thelowcodebuilder.com/admin` to moderate comments.

## Files

| File | Purpose |
|---|---|
| `docker-compose.yml` | Docker Compose service definition |
| `config/isso.cfg` | Isso server configuration |
| `db/` | SQLite database (auto-created on first run) |
