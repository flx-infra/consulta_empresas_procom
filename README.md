# consulta_empresas_procom (infra)

Stack de Docker Swarm de **Consulta Empresas Procom**: API Node/Express que consulta Firebird y sirve también el front React (una sola imagen).

- Código y `Dockerfile`: repo `consultaempresasprocom` (`Dockerfile` en la raíz)
- Acceso: `http://<ip-del-worker>:3256` (puerto publicado en el worker)
- `/` → front, `/v1/empresas` → API, en el mismo puerto.

## Antes del primer deploy

**Secret** (Portainer → *Secrets*), sin salto de línea al final:

| Secret | Uso |
|---|---|
| `consultaempresas_db_password` | Contraseña del usuario Firebird `INT_CONSULTAEMPRESAS` |

**Requisitos del worker:** red overlay `traefik`, plugin de logging `loki`, puerto 3256 libre en el Swarm y acceso a `192.168.121.178:3050` (Firebird del SAR).

## Variables del stack

| Variable | Obligatoria | Valor |
|---|---|---|
| `IMAGE` | Sí | `<usuario>/consultaempresas:<tag>` |
| `NODE` | Sí | El mismo que usan los otros stacks del worker, ej. `node.hostname==...` |
| `LOKI_URL` | Sí | La misma que usan los otros stacks del worker |
| `APP_PORT` | No | Puerto publicado (default `3256`) |

## Construir la imagen

Desde la raíz del repo del código:

```sh
docker build -t <usuario>/consultaempresas:<tag> .
docker push <usuario>/consultaempresas:<tag>
```

## Actualizar

Subir la imagen con un tag nuevo, cambiar `IMAGE` y *Update the stack*. El contenedor nuevo arranca antes de bajar el viejo (sin corte) y si no pasa el healthcheck vuelve atrás solo.

## Verificar

```sh
curl http://<ip-del-worker>:3256/                                    # front
curl "http://<ip-del-worker>:3256/v1/empresas?cuit=...&razonsocial=..."  # API
```
