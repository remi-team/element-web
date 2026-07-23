###############################################
# Builder
###############################################
FROM node:24-bullseye AS builder

ARG USE_CUSTOM_SDKS=false
ARG JS_SDK_REPO=https://github.com/matrix-org/matrix-js-sdk.git
ARG JS_SDK_BRANCH=master

ENV CI=true

WORKDIR /src

# 开启 pnpm
RUN corepack enable

#########################################################
# 第一层：只复制依赖描述文件（充分利用 Docker Layer Cache）
#########################################################

COPY package.json ./
COPY pnpm-lock.yaml ./
COPY pnpm-workspace.yaml ./

COPY apps/web/package.json apps/web/package.json

COPY packages packages
COPY scripts scripts
COPY patches patches

#########################################################
# 安装依赖（只有 package.json 变化才重新安装）
#########################################################

RUN pnpm install --frozen-lockfile

#########################################################
# 第二层：复制完整源码
#########################################################

COPY . .

#########################################################
# 执行官方构建脚本
#########################################################

RUN chmod +x scripts/*.sh

RUN ./scripts/docker-link-repos.sh

RUN ./scripts/docker-package.sh

#########################################################
# 默认配置
#########################################################

RUN cp apps/web/config.sample.json apps/web/webapp/config.json

#########################################################
# Runtime
#########################################################

FROM nginxinc/nginx-unprivileged:alpine-slim

USER root

RUN apk add --no-cache \
    jq \
    moreutils

#########################################################
# 拷贝前端产物
#########################################################

COPY --from=builder /src/apps/web/webapp /app

#########################################################
# nginx
#########################################################

COPY apps/web/docker/nginx-templates/ /etc/nginx/templates/

COPY apps/web/docker/docker-entrypoint.d/ /docker-entrypoint.d/

RUN rm -rf /usr/share/nginx/html \
    && ln -s /app /usr/share/nginx/html

#########################################################
# nginx 用户
#########################################################

USER nginx

ENV ELEMENT_WEB_PORT=80

EXPOSE 80

HEALTHCHECK \
    --interval=30s \
    --timeout=5s \
    --start-period=5s \
    --retries=3 \
CMD wget -q --spider http://127.0.0.1:${ELEMENT_WEB_PORT}/config.json || exit 1
