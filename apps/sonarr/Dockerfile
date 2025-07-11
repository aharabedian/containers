# syntax=docker/dockerfile:1

FROM docker.io/library/alpine:3.22
ARG TARGETARCH
ARG VENDOR
ARG VERSION

ENV DOTNET_EnableDiagnostics=0 \
    SONARR__UPDATE__BRANCH=develop

USER root
WORKDIR /app

RUN \
    apk add --no-cache \
        bash \
        ca-certificates \
        catatonit \
        coreutils \
        curl \
        icu-libs \
        jq \
        libintl \
        nano \
        sqlite-libs \
        tzdata \
    && mkdir -p /app/bin \
    && curl -fsSL "https://services.sonarr.tv/v1/update/${SONARR__UPDATE__BRANCH}/download?version=${VERSION}&os=linuxmusl&runtime=netcore&arch=${TARGETARCH/amd64/x64}" \
        | tar xzf - -C /app/bin --strip-components=1 \
    && printf "UpdateMethod=docker\nBranch=%s\nPackageVersion=%s\nPackageAuthor=[%s](https://github.com/%s)\n" "${SONARR__UPDATE__BRANCH}" "${VERSION}" "${VENDOR}" "${VENDOR}" > /app/package_info \
    && chown -R root:root /app && chmod -R 755 /app \
    && rm -rf /tmp/* /app/bin/Sonarr.Update

COPY . /

# Use a lower UID/GID by default. Makes podman rootless much easier to deal with.
USER 1001:1001
WORKDIR /config
VOLUME ["/config"]

ENTRYPOINT ["/usr/bin/catatonit", "--", "/entrypoint.sh"]
