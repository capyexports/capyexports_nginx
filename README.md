# capyexports_nginx

Nginx gateway image and config for the capyexports demo stack. Single entrypoint (80/443); all traffic is proxied to internal services by Compose service name.

## Layout

- `nginx/` – build context for the gateway image
  - `Dockerfile` – based on `nginx:alpine`
  - `nginx.conf` – main config; includes `conf.d/*.conf`
  - `conf.d/` – site / reverse proxy configs (`default.conf`, `demos.conf`)

## Build locally

From repo root:

```bash
docker build -f nginx/Dockerfile -t nginx-gateway:local nginx/
```

## CI (GitHub Actions)

On push to `main`/`master` when `nginx/**` or this workflow changes, the image is built and pushed to ACR. Configure Organization secrets: `ACR_REGISTRY`, `ACR_USERNAME`, `ACR_PASSWORD`, `ACR_NAMESPACE`.
