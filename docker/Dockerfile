FROM ghcr.io/linuxserver/baseimage-ubuntu:jammy as base

RUN curl -fsSL https://deb.nodesource.com/setup_18.x | bash -
RUN apt install nodejs && npm i -g yarn

##### DEPENDENCIES

FROM base AS deps

WORKDIR /app

COPY prisma ./
COPY package.json yarn.lock ./

RUN yarn --frozen-lockfile

##### BUILDER

FROM base AS builder

WORKDIR /app

COPY --from=deps /app/node_modules ./node_modules
COPY . .

ENV NEXT_TELEMETRY_DISABLED 1

RUN yarn build

##### RUNNER

FROM base as runner

WORKDIR /tmp

RUN curl -L https://github.com/metafates/mangal/releases/download/v3.12.0/mangal_3.12.0_Linux_x86_64.tar.gz -o mangal_3.12.0_Linux_x86_64.tar.gz && \
  tar xf mangal_3.12.0_Linux_x86_64.tar.gz && \
  mv mangal /usr/bin/mangal && \
  chmod +x /usr/bin/mangal

WORKDIR /app

ENV NODE_ENV production
ENV NEXT_TELEMETRY_DISABLED 1
ENV HOME="/config"
ENV KAIZOKU_LOG_PATH="/logs"
ENV MANGAL_METADATA_COMIC_INFO_XML=true
ENV MANGAL_METADATA_FETCH_ANILIST=true
ENV MANGAL_METADATA_SERIES_JSON=true
ENV MANGAL_FORMATS_USE=cbz
ENV MANGAL_DOWNLOADER_DOWNLOAD_COVER=true
ENV MANGAL_DOWNLOADER_REDOWNLOAD_EXISTING=true

COPY --from=builder /app/next.config.mjs ./
COPY --from=builder /app/public ./public
COPY --from=builder /app/package.json ./package.json
COPY --from=builder /app/.next ./.next
COPY --from=builder /app/prisma ./prisma
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules

COPY docker/mangal /mangal
COPY docker/root/ /

VOLUME [ "/logs", "/config", "/data" ]