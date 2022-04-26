# Ubuntu with FIPS
# https://github.com/canonical/ubuntu-advantage-client/blob/main/docs/tutorials/create_a_fips_docker_image.md
FROM ubuntu:focal

RUN --mount=type=secret,id=ua-attach-config \
    apt-get update \
    && apt-get install --no-install-recommends -y ubuntu-advantage-tools ca-certificates \
    && ua attach --attach-config /run/secrets/ua-attach-config \
    && apt-get upgrade -y \
    && apt-get install -y openssl libssl1.1 libssl1.1-hmac libgcrypt20 libgcrypt20-hmac strongswan strongswan-hmac openssh-client openssh-server \
    && apt-get purge --auto-remove -y ubuntu-advantage-tools ca-certificates \
    && rm -rf /var/lib/apt/lists/*

