# capyexports_nginx

Nginx gateway image and config for the capyexports demo stack. Single entrypoint (80/443); all traffic is proxied to internal services by Compose service name.

## Gateway usage rules (nginx-gateway)

Nginx is the **single entry** in a “gateway + private network” setup: all external traffic hits 80/443, then is forwarded to Demo containers over the internal network.

### Architecture

- **Single entry**: Only the Nginx container exposes `80` and `443`; Demo containers **must not** expose ports on the host.
- **Internal forwarding**: Use Docker internal network; use the **Compose service name** as the `proxy_pass` host (e.g. `http://demo-app:3000`).
- **Config layout**: Site configs live in `nginx/conf.d/`; main config or fragments stay under `nginx/` as agreed in the project.

### Reverse proxy to Demo

- Use **upstream** or direct **proxy_pass** to **service-name:internal-port** (must match the docker-compose service name and exposed port).
- **Required headers**: `proxy_set_header Host $host`, `X-Real-IP $remote_addr`, `X-Forwarded-For $proxy_add_x_forwarded_for`, `X-Forwarded-Proto $scheme`.
- **WebSocket**: Add `proxy_http_version 1.1`, `proxy_set_header Upgrade $http_upgrade`, `proxy_set_header Connection "upgrade"`.

### Compose

- **Nginx**: Map `80:80`, `443:443` (if using HTTPS).
- **Demo**: Use `expose` or internal ports only; **do not** use `ports` to map to the host.

### Logging and resources

- Access/error logs follow the main config. To limit disk usage, set Nginx container `logging.driver` with `max-size: 20m`.
- Prefer `deploy.resources.limits.memory` for Nginx (e.g. 256M).

### Adding a new Demo

1. In `docker-compose.yml`, define the Demo service, attach it to the same **networks** as Nginx, and **do not** add `ports`.
2. In `nginx/conf.d/`, add or edit a `*.conf` with a `location` and `proxy_pass` to that service name and internal port.
3. Reload Nginx: `docker compose exec nginx nginx -s reload` (or restart the nginx container).

### Don’ts

- Do **not** use `ports: "3000:3000"` etc. on Demo services for external access.
- Do **not** hardcode host IPs or non-orchestration addresses in Nginx config; always use the **Compose service name** for upstream/server.

---

## Layout

```
nginx/
├── Dockerfile          # nginx:alpine, HEALTHCHECK /__nginx_health__
├── nginx.conf          # main config; includes conf.d/*.conf
├── .dockerignore
└── conf.d/
    ├── default.conf    # health /__nginx_health__, /demo-app/ proxy, /
    └── demos.conf      # upstream demo_app (demo-app:3000)
```

## Build locally

From repo root:

```bash
docker build -f nginx/Dockerfile -t nginx-gateway:local nginx/
```

## CI (GitHub Actions)

- **Trigger**: Push or pull_request to `main`/`master` when `nginx/**` or `.github/workflows/**` changes; or `workflow_dispatch`.
- **Registry**: Aliyun ACR Personal Edition (Beijing). Image name: `nginx-gateway`.
- **Tags**: `latest` and date `YYYYMMDD` on push; PRs build only (no push), tag `nginx-gateway:pr`.

**Organization secrets** (no `ACR_REGISTRY`; registry URL is in workflow env):

| Secret          | Description        |
|-----------------|--------------------|
| `ACR_USERNAME`  | ACR login username |
| `ACR_PASSWORD`  | ACR login password |
| `ACR_NAMESPACE` | ACR namespace (e.g. `capyexports`) |
