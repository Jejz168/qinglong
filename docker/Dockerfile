FROM python:3.10-slim-bullseye as builder

# 安装 Node.js 相关依赖
COPY package.json .npmrc pnpm-lock.yaml /tmp/build/
RUN set -ex \
  && apt-get update \
  && apt-get install -y --no-install-recommends \
     curl gnupg git ca-certificates \
  && curl -fsSL https://deb.nodesource.com/setup_20.x | bash - \
  && apt-get install -y nodejs \
  && npm i -g pnpm@8.3.1 \
  && cd /tmp/build \
  && pnpm --registry https://registry.npmmirror.com install --prod \
  && rm -rf /var/lib/apt/lists/*

FROM python:3.10-slim-bullseye

ARG QL_MAINTAINER="Jejz168"
LABEL maintainer="${QL_MAINTAINER}"
ARG QL_URL=https://github.com/${QL_MAINTAINER}/qinglong.git
ARG QL_BRANCH=develop

ENV PNPM_HOME=/root/.local/share/pnpm \
  PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:$PNPM_HOME:/root/.local/share/pnpm/global/5/node_modules \
  NODE_PATH=/usr/local/lib/node_modules:/root/.local/share/pnpm/global/5/node_modules \
  LANG=C.UTF-8 \
  SHELL=/bin/bash \
  PS1="\u@\h:\w \$ " \
  QL_DIR=/ql \
  QL_BRANCH=${QL_BRANCH}

RUN set -ex \
  && apt-get update \
  && apt-get install -y --no-install-recommends \
     bash coreutils git curl wget tzdata \
     perl openssl nginx jq openssh-client procps \
     netcat npm ca-certificates gnupg \
  && ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime \
  && echo "Asia/Shanghai" > /etc/timezone \
  && git config --global user.email "qinglong@@users.noreply.github.com" \
  && git config --global user.name "qinglong" \
  && git config --global http.postBuffer 524288000 \
  && npm install -g pnpm@8.3.1 \
  && pnpm config set registry https://registry.npmmirror.com \
  && pnpm add -g pm2 tsx \
  && rm -rf /var/lib/apt/lists/* /root/.pnpm-store /root/.local/share/pnpm/store /root/.cache /root/.npm

ARG SOURCE_COMMIT
RUN git clone --depth=1 -b ${QL_BRANCH} ${QL_URL} ${QL_DIR} \
  && cd ${QL_DIR} \
  && cp -f .env.example .env \
  && chmod 777 ${QL_DIR}/shell/*.sh \
  && chmod 777 ${QL_DIR}/docker/*.sh \
  && git clone --depth=1 -b ${QL_BRANCH} https://github.com/${QL_MAINTAINER}/qinglong-static.git /static \
  && mkdir -p ${QL_DIR}/static \
  && cp -rf /static/* ${QL_DIR}/static \
  && rm -rf /static

COPY --from=builder /tmp/build/node_modules/. /ql/node_modules/

WORKDIR ${QL_DIR}

HEALTHCHECK --interval=5s --timeout=2s --retries=20 \
  CMD curl -sf http://127.0.0.1:5400/api/health || exit 1

ENTRYPOINT ["./docker/docker-entrypoint.sh"]
