# CLAUDE.md

Repositorio de infraestructura del homelab **devmalagan**. Es la fuente de la verdad de lo desplegado en el servidor (Ubuntu Server 24.04, IP LAN `192.168.1.200`, clon en `~/dev-malagan`). No hay código de aplicación: solo stacks de Docker Compose y documentación. Documentación y comentarios en español.

## Estructura

| Carpeta | Archivo compose | Servicios | Puertos host |
|---|---|---|---|
| `npm-cloudflare/` | `compose.yml` | `nginx-proxy-manager`, `cloudflared-tunnel` | 80, 81 (UI NPM), 443 |
| `monitoring/` | `compose.yml` | `dozzle` (logs + CPU/RAM en vivo de contenedores, login propio) | 2001 |

Otros archivos:
- `PORTS.md` — esquema de rangos de puertos y contenedores actuales.
- `infra/` — diagramas Mermaid de la arquitectura (Cloudflare Tunnel → NPM → contenedores).
- `npm-cloudflare/README.md` — guía de alta de túnel y proxy hosts.
- `TODO.md` — tareas de mantenimiento en curso.

Fuera de Docker, en el host: **Tailscale** (instalado vía apt).

## Arquitectura de red

- Dominio: `devmalagan.dpdns.org` (zona en Cloudflare; apps en `<app>.devmalagan.dpdns.org`). Antes `devmalagan.org`, aún referenciado en proxy hosts viejos de NPM.
- Exposición a internet solo vía **Cloudflare Tunnel** (sin puertos abiertos en el router). En el dashboard, el Public Hostname apunta a `http://nginx-proxy-manager:80` (mismo compose, misma red); NPM enruta por subdominio a `http://192.168.1.200:<puerto>`.
- SSL en Cloudflare en modo **Full (strict)**.
- Backends y bases de datos no se exponen por el túnel. La UI de NPM (`:81`) solo desde LAN o Tailscale.

## Exponer una app nueva

1. Levantar la app en el host con un puerto del rango de `PORTS.md`.
2. NPM (`http://192.168.1.200:81`) → Proxy Host: `app.devmalagan.dpdns.org` → `http://192.168.1.200:<puerto>`, Websockets on.
3. Cloudflare Zero Trust → túnel → Public Hostname `app.devmalagan.dpdns.org` → `http://nginx-proxy-manager:80` (innecesario si existe el comodín `*.devmalagan.dpdns.org`).
4. Actualizar `PORTS.md`.

## Comandos

```bash
cd npm-cloudflare/
cp .env.template .env        # rellenar CLOUDFLARED_TOKEN
docker compose up -d
docker compose logs -f cloudflared   # buscar "Registered tunnel connection"
docker compose pull && docker compose up -d && docker image prune -f   # actualizar
```

## Secretos y datos

- `.env` y `CLAUDE.local.md` están en `.gitignore`. Solo se commitea `.env.template`. Nunca commitear tokens ni contraseñas.
- `npm-cloudflare/.env`: `CLOUDFLARED_TOKEN`.
- Datos de NPM (config, proxy hosts en `database.sqlite`, certificados) en `npm-cloudflare/data/`, ignorado por git.
- Login de Dozzle en `monitoring/data/users.yml` (ignorado). Regenerar con `docker run --rm amir20/dozzle generate --name <nombre> --email <email> --password <pass> <usuario> > monitoring/data/users.yml`.

## Convenciones

- Imágenes con `latest` (NPM, cloudflared): actualizar con `pull` + `up -d`.
- `restart: unless-stopped`.
- Commits cortos; muchas ediciones directas desde la UI de GitHub.
