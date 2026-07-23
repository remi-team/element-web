FROM node:24-bullseye AS builder

WORKDIR /src

RUN corepack enable

COPY . .

RUN chmod +x scripts/*.sh

RUN pnpm install --frozen-lockfile

RUN ./scripts/docker-link-repos.sh

RUN ./scripts/docker-package.sh

RUN cp apps/web/config.sample.json apps/web/webapp/config.json

FROM nginxinc/nginx-unprivileged:alpine-slim

USER root

RUN apk add --no-cache jq moreutils

COPY --from=builder /src/apps/web/webapp /app

COPY apps/web/docker/nginx-templates/ /etc/nginx/templates/
COPY apps/web/docker/docker-entrypoint.d/ /docker-entrypoint.d/

RUN rm -rf /usr/share/nginx/html \
    && ln -s /app /usr/share/nginx/html

USER nginx

ENV ELEMENT_WEB_PORT=80

EXPOSE 80

HEALTHCHECK \
  CMD wget -q --spider http://127.0.0.1:${ELEMENT_WEB_PORT}/config.json || exit 1
