## STAGE 0: Build Client ###

FROM node:16-alpine AS build
ADD https://github.com/advplyr/audiobookshelf/archive/refs/tags/v2.5.0.tar.gz /
RUN tar xvf v2.5.0.tar.gz
RUN ls -l / && cd /audiobookshelf-2.5.0/client/ && \
    npm ci && npm cache clean --force \
    npm run generate

## STAGE 1: Build Server ###

FROM sandreas/tone:v0.1.5 AS tone

# https://developers.home-assistant.io/docs/add-ons/configuration#add-on-dockerfile
#ARG BUILD_FROM
#FROM $BUILD_FROM
FROM ghcr.io/hassio-addons/base:14.3.2

# Execute during the build of the image
ARG TEMPIO_VERSION BUILD_ARCH
RUN \
    curl -sSLf -o /usr/bin/tempio \
    "https://github.com/home-assistant/tempio/releases/download/${TEMPIO_VERSION}/tempio_${BUILD_ARCH}"

RUN apk update && \
    apk add --no-cache --update \
    curl \
    tzdata \
    ffmpeg \
    make \
    python3 \
    g++ \
    npm

COPY rootfs /

COPY --from=tone /usr/local/bin/tone /usr/local/bin/
COPY --from=build /audiobookshelf-2.5.0/client/dist /client/dist
COPY --from=build /audiobookshelf-2.5.0/server /server
COPY --from=build /audiobookshelf-2.5.0/package* /
COPY --from=build /audiobookshelf-2.5.0/index.js /

RUN npm ci --only=production

RUN apk del make python3 g++

EXPOSE 80

#CMD ["node", "index.js"]
