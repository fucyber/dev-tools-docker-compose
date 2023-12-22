
### acme.json เซ็ต Permission สำหรับเก็บ Private Key และ Certificate ที่ได้จาก Let’s Encrypt
``` chmod 600 acme.json ```

### traefik.lllife.dev สำหรับขอ Certificate จาก Let’s Encrypt

### admin  Admin@


### add domain to docker container
```
    labels:
      - traefik.http.routers.blog.rule=Host(`blog.your_domain`)
      - traefik.http.routers.blog.tls=true
      - traefik.http.routers.blog.tls.certresolver=lets-encrypt
      - traefik.port=80
    networks:
      - internal
      - apps
    depends_on:
      - xxxx
```

### for mysql
```
networks:
  - internal
labels:
  - traefik.enable=false
```

```yml
version: '3.5'

services:
  user-auth:
    env_file: ./env/.env.development
    container_name: user-auth
    image: user-auth:latest
    build:
      context: .
      dockerfile: ./Dockerfile
      target: development
    command: yarn run start:dev
    environment:
      NODE_ENV: development
    expose:
      - ${NODE_PORT:-3000}
    networks:
      - app-network
    volumes:
      - .:/app
      - /app/node_modules
    restart: unless-stopped
    labels:
      - traefik.enable=true
      - traefik.http.routers.auth-api.entrypoints=https
      - traefik.http.routers.auth-api.rule=(Host(`dev-api.your_domain.com`) && PathPrefix(`/auth{regex:$$|/.*}`))
      - traefik.http.routers.auth-api.tls=true
      - traefik.http.routers.auth-api.tls.certresolver=lets-encrypt
      - traefik.http.middlewares.auth-api.stripprefix.prefixes=/auth
      - traefik.http.routers.auth-api.middlewares=auth-api@docker
      # services
      - traefik.http.routers.auth-api.service=auth-api
      - traefik.http.services.auth-api.loadbalancer.server.port=${NODE_PORT}

networks:
  app-network:
    driver: bridge
    name: app-network

```