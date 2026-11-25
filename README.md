# consulta_empresas_procom (infra)

Stack de Docker Swarm de **Consulta Empresas Procom**: front React servido por nginx y API Node que consulta Firebird.

- Código y `Dockerfile`s: repo `consultaempresasprocom` (`server/Dockerfile` → API, `client/Dockerfile` → front)
- Acceso: `http://<ip-del-worker>:5007` (puerto publicado en el worker)
- El front hace de proxy: `/v1/*` → `consultaempresas-api:5006` por la red interna del stack. La API no se publica.

## Antes del primer deploy

**Secret** (Portainer → *Secrets*), sin salto de línea al final:

| Secret | Uso |
|---|---|
| `consultaempresas_db_password` | Contraseña del usuario Firebird `INT_CONSULTAEMPRESAS` |

**Requisitos del worker:** red overlay `traefik`, plugin de logging `loki`, puerto 5007 libre en el Swarm y acceso a `192.168.121.178:3050` (Firebird del SAR).

## Variables del stack

| Variable | Obligatoria | Valor |
|---|---|---|
| `IMAGE_API` | Sí | `<usuario>/consultaempresas-api:<tag>` |
| `IMAGE_WEB` | Sí | `<usuario>/consultaempresas-web:<tag>` |
| `NODE` | Sí | El mismo que usan los otros stacks del worker, ej. `node.hostname==...` |
| `LOKI_URL` | Sí | La misma que usan los otros stacks del worker |
| `APP_PORT` | No | Puerto publicado (default `5007`) |

## Construir las imágenes

Desde la raíz del repo del código:

```sh
docker build -t <usuario>/consultaempresas-api:<tag> server
docker build -t <usuario>/consultaempresas-web:<tag> client
docker push <usuario>/consultaempresas-api:<tag>
docker push <usuario>/consultaempresas-web:<tag>
```

## Actualizar

Subir las imágenes con un tag nuevo, cambiar `IMAGE_API` / `IMAGE_WEB` y *Update the stack*. El contenedor nuevo arranca antes de bajar el viejo (sin corte) y si no pasa el healthcheck vuelve atrás solo.

## Verificar

```sh
curl http://<ip-del-worker>:5007/                                    # front
curl "http://<ip-del-worker>:5007/v1/empresas?cuit=...&razonsocial=..."  # API vía proxy
```
