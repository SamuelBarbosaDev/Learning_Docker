# Learning Docker

## Como instalar
### Postgres:
```shell
    docker run \
    --name postgres-container \
    -e POSTGRES_USER=user \
    -e POSTGRES_PASSWORD=postgres \
    -e POSTGRES_DB=database \
    -d \
    -p 5434:5432 \
    -v postgres-data:/var/lib/postgresql/data \
    postgres:latest
```

### PgAdmin:
```shell
    docker run -d \
    --name pg-admin-4 \
    -e "PGADMIN_DEFAULT_EMAIL=email@email.com" \
    -e "PGADMIN_DEFAULT_PASSWORD=123456" \
    -p 7878:80 \
    dpage/pgadmin4
```

## O que é
### Pi-hole:
Bloqueio de anúncios em toda a rede.
