# Clasificación general de puertos

| Rango de Puertos | Propósito                  | Ejemplo                             |
|------------------|-----------------------------|-------------------------------------|
| 80, 443          | HTTP/HTTPS público (general) | nginx-proxy-manager                 |
| 1000–1999        | Infraestructura de red      | Nginx, Cloudflare Tunnel            |
| 2000–3000        | Monitoreo y métricas        | Grafana, Prometheus, cAdvisor, Node Exporter |
| 3001–3999        | Aplicaciones de desarrollo  | Apps personales, APIs               |
| 4000–4999        | Bases de datos              | PostgreSQL, MySQL, MongoDB          |
| 5000–5999        | Microservicios / Backends   | NestJS, Express, FastAPI apps       |
| 6000–6999        | Testing / CI/CD tools       | Jenkins, SonarQube, etc.            |

# Contenedores actuales

| Contenedor            | Servicio              | Puerto(s)   |
|-----------------------|-----------------------|-------------|
| nginx-proxy-manager   | Reverse Proxy Manager | 80, 81, 443 |
| cloudflared-tunnel    | Cloudflare Tunnel     | interno     |
