# Podman Example

Beispiel bei einem Standard Container für Postgres sowie eine eigene Anwendung: 

Verzeichnisstruktur: 
go-dev
  /backend
    Dockerfile
  /frontend
  compose.yml

podman build -t clabmgr .
podman run --name clabmgr_app --env-file ~/.env_clabmgr -p 4000:4000 clabmgr

Das Dockerfile der Backend-Anwendung könnte wie folgt aussehen:

```
FROM golang:1.25

RUN apt-get update && apt-get install -y ca-certificates 

# Add certificates outside of github
COPY gorm.io.crt /usr/local/share/ca-certificates/gorm.io.crt
COPY gopkg.in.crt /usr/local/share/ca-certificates/gopkg.in.crt

RUN update-ca-certificates

WORKDIR /app

COPY . .

# ignore https://proxy.golang.org
ENV GOPROXY=direct
ENV GOSUMDB=off
RUN go mod tidy
RUN go build -o /app/clabmgmt cmd/main.go

EXPOSE 4000

CMD ["./app/clabmgmt"]
```

Das Compose File fasst mehrere Container zusammen und baut diese. 

``` 
# Use postgres/example user/password credentials
# docker compose build
# docker compose up -d goapp
# docker ps -a

services:
  app:
    container_name: clabapp
    build: ./backend
    ports:
      - "4000:4000"
    depends_on:
      - db
    networks:
      - appnet
    env_file:
      - ~/.env_clabmgr



  db:
    container_name: clabdb
    image: docker.io/postgres:latest
    restart: always
    # set shared memory limit when using docker compose
    shm_size: 128mb
    # or set shared memory limit when deploy via swarm stack
    #volumes:
    #  - type: tmpfs
    #    target: /dev/shm
    #    tmpfs:
    #      size: 134217728 # 128*2^20 bytes = 128Mb
    ports:
      - 5432:5432
    environment: 
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_DB: ${POSTGRES_DB}
      POSTGRES_HOST_AUTH_METHOD: ${POSTGRES_HOST_AUTH_METHOD}
      POSTGRES_INITDB_ARGS: ${POSTGRES_INITDB_ARGS}
    volumes:
      - pgdata:/var/lib/postgresql
    networks: 
      - appnet

volumes:
  pgdata: {}

networks:
  appnet:
``` 

Beim Befehl `podman compose build -f compose.yml` schlägt der Build fehlt, da TLS Zertifikate fehlen. Hier bietet es sich an, die TLS Certifikate zu erstellen und über das Dockerfile mit in den Container zu kopieren. 
Wenn der build fehlschlägt, sagt die Fehlermeldung, welches Zertifikat nicht gültig ist oder fehlt. Mittels `echo | openssl s_client -connect gopkg.in:443 -showcerts` bekommt man dann die Zertifikatschain angezeigt und kann die einzelnen Zertifikate dort rauskopieren und ein jeweils eine eigene .crt-Datei kopieren. Diese dann im Dockerfile angeben (siehe oben). 

