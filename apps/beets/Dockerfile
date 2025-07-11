# syntax=docker/dockerfile:1

FROM docker.io/library/node:22-alpine AS betanin
WORKDIR /src
ADD https://github.com/sentriz/betanin.git .
WORKDIR /src/betanin_client
ENV PRODUCTION=true
ENV NODE_OPTIONS=--openssl-legacy-provider
RUN npm install && npm run-script build

FROM docker.io/library/python:3.13-alpine3.22
ARG VERSION

WORKDIR /src

ENV \
    CRYPTOGRAPHY_DONT_BUILD_RUST=1 \
    PIP_BREAK_SYSTEM_PACKAGES=1 \
    PIP_DISABLE_PIP_VERSION_CHECK=1 \
    PIP_NO_CACHE_DIR=1 \
    PIP_ROOT_USER_ACTION=ignore \
    PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    UV_NO_CACHE=true \
    UV_SYSTEM_PYTHON=true \
    UV_EXTRA_INDEX_URL="https://wheel-index.linuxserver.io/alpine-3.22/"

ENV HOME=/config \
    BETANIN_HOST=0.0.0.0 \
    BETANIN_PORT=8080

RUN \
    apk add --no-cache \
        bash \
        catatonit \
        chromaprint \
        curl \
        expat \
        ffmpeg \
        fftw \
        flac \
        gdbm \
        gobject-introspection \
        gst-plugins-good \
        gstreamer \
        imagemagick \
        jpeg \
        jq \
        lame \
        libffi \
        libpng \
        mpg123 \
        openjpeg \
        sqlite-libs \
        tzdata \
    && \
    apk add --no-cache --virtual=.build-deps \
        git \
    && \
    apk add --no-cache --repository="https://dl-cdn.alpinelinux.org/alpine/edge/community/" \
        mp3gain \
    && \
    apk add --no-cache --repository="https://dl-cdn.alpinelinux.org/alpine/edge/testing/" \
        mp3val \
    && \
    pip install uv \
    && \
    uv pip install \
        https://github.com/beetbox/beets/releases/download/${VERSION}/beets-${VERSION#*v}-py3-none-any.whl \
        git+https://github.com/sentriz/betanin.git \
        beetcamp \
        beets-extrafiles \
        beets-lidarr-fields \
        beets-noimport \
        pyacoustid \
        pylast \
        python3-discogs-client \
    && pip uninstall --yes uv \
    && apk del --purge .build-deps \
    && rm -rf /tmp/*

COPY --from=betanin /src/betanin_client/dist/ /usr/local/lib/python3.13/site-packages/betanin_client/dist/
COPY . /

USER nobody:nogroup
WORKDIR /config
VOLUME ["/config"]

ENTRYPOINT ["/usr/bin/catatonit", "--", "/entrypoint.sh"]
