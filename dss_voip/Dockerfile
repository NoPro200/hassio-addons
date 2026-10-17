ARG BUILD_FROM=ghcr.io/hassio-addons/base:21.0.7
# hadolint ignore=DL3006
FROM ${BUILD_FROM}

# Set shell
SHELL ["/bin/bash", "-o", "pipefail", "-c"]

# Setup base
RUN apk add --no-cache \
        pjsua=2.17-r0 \
        sox=14.4.2-r14

# Copy root filesystem
COPY rootfs /
RUN chmod a+x /usr/bin/dss_voip

CMD [ "/usr/bin/dss_voip" ]

# Build arguments
ARG BUILD_ARCH
ARG BUILD_DATE
ARG BUILD_DESCRIPTION
ARG BUILD_NAME
ARG BUILD_REF
ARG BUILD_REPOSITORY
ARG BUILD_VERSION

# Labels
LABEL \
    io.hass.name="${BUILD_NAME}" \
    io.hass.description="${BUILD_DESCRIPTION}" \
    io.hass.arch="${BUILD_ARCH}" \
    io.hass.type="addon" \
    io.hass.version=${BUILD_VERSION} \
    org.opencontainers.image.title="${BUILD_NAME}" \
    org.opencontainers.image.description="${BUILD_DESCRIPTION}" \
    org.opencontainers.image.authors="SDeSalve <me@sdesalve.it>, NoPro200" \
    org.opencontainers.image.licenses="CC-BY-NC-4.0" \
    org.opencontainers.image.source="https://github.com/NoPro200/hassio-addons" \
    org.opencontainers.image.created=${BUILD_DATE} \
    org.opencontainers.image.revision=${BUILD_REF} \
    org.opencontainers.image.version=${BUILD_VERSION}
