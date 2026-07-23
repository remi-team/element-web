############################################################
# Builder
############################################################
FROM node:24-bullseye AS builder

ENV CI=true
ENV USE_CUSTOM_SDKS=false

WORKDIR /src

# 启用 pnpm
RUN corepack enable

############################################################
# 复制整个源码（Element Web 必须完整 Workspace）
############################################################
COPY . .

############################################################
# 安装依赖
############################################################
RUN pnpm install --frozen-lockfile

############################################################
# 修复 git 安全目录（Jenkins 中常见）
############################################################
RUN git config --global --add safe.directory /src || true

############################################################
# 设置版本信息（没有 tag 时避免失败）
############################################################
ARG VERSION=dev
ENV VERSION=${VERSION}

############################################################
# 构建
############################################################
RUN chmod +x scripts/*.sh

# 不使用 docker-link-repos.sh（仅开发环境需要）
RUN VERSION=${VERSION} pnpm --dir apps/web build

############################################################
# 默认配置（使用项目定制配置，而非 sample）
############################################################
RUN cp apps/web/config.json apps/web/webapp/config.json

############################################################
# Runtime
############################################################
FROM nginxinc/nginx-unprivileged:alpine-slim

USER root

RUN apk add --no-cache \
    jq \
    moreutils \
    wget

############################################################
# 前端文件
############################################################
COPY --from=builder /src/apps/web/webapp /app

############################################################
# nginx
############################################################
COPY apps/web/docker/nginx-templates/ /etc/nginx/templates/
COPY apps/web/docker/docker-entrypoint.d/ /docker-entrypoint.d/

RUN rm -rf /usr/share/nginx/html && \
    ln -s /app /usr/share/nginx/html

############################################################
# 权限
############################################################
RUN chown -R nginx:nginx /app

USER nginx

ENV ELEMENT_WEB_PORT=80

EXPOSE 80

############################################################
# HealthCheck
############################################################
HEALTHCHECK \
    --interval=30s \
    --timeout=5s \
    --start-period=10s \
    --retries=3 \
CMD wget -q --spider http://127.0.0.1:${ELEMENT_WEB_PORT}/config.json || exit 1
