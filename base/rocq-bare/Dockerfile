FROM debian:12-slim

# This 'bare' image does not have any OCaml compiler Docker layer:
# * it keeps all other layers (APT packages, OPAM 2.0, etc.)
# * it also keeps some env. variables (OPAM_VERSION, NJOBS, OPAMPRECISETRACKING)
# * it can be used to build children images from rocq/rocq-prover by multi-stage build
#   and copy the .opam folder in the very end.

ARG OPAM_VERSION
ENV OPAM_VERSION=${OPAM_VERSION}

SHELL ["/bin/bash", "--login", "-o", "pipefail", "-c"]

# hadolint ignore=DL3008
RUN cat /proc/cpuinfo /proc/meminfo \
  && [ -n "${OPAM_VERSION}" ] \
  && apt-get update -y -q \
  && DEBIAN_FRONTEND=noninteractive apt-get install -y -q --no-install-recommends \
    autoconf \
    automake \
    bubblewrap \
    build-essential \
    ca-certificates \
    curl \
    git \
    # gnupg is temporarily installed and will not be kept in the image
    gnupg \
    less \
    libgmp-dev \
    m4 \
    openssh-client \
    pkg-config \
    rlwrap \
    rsync \
    sudo \
    time \
    unzip \
  && binary="opam-${OPAM_VERSION}-$(uname -m)-$(uname -s | tr '[:upper:]' '[:lower:]')" \
  && set -x \
  && curl -fSL -o "/tmp/${binary}" "https://github.com/ocaml/opam/releases/download/${OPAM_VERSION}/${binary}" \
  && curl -fSL -o "/tmp/${binary}.sig" "https://github.com/ocaml/opam/releases/download/${OPAM_VERSION}/${binary}.sig" \
  && curl -fsSL https://opam.ocaml.org/opam-dev-pubkey.pgp | gpg --batch --import \
  && gpg --batch --verify "/tmp/${binary}.sig" "/tmp/${binary}" \
  && mv "/tmp/${binary}" /usr/local/bin/opam \
  && chmod a+x /usr/local/bin/opam \
  && rm -f "/tmp/${binary}.sig" \
  && rm -fr /root/.gnupg \
  && DEBIAN_FRONTEND=noninteractive apt-get purge -y -q --auto-remove gnupg \
  && apt-get clean \
  && rm -rf /var/lib/apt/lists/*

# Use Docker build args to set the UID/GID
ARG guest_uid=1000
ARG guest_gid=${guest_uid}

# Add rocq group and user with sudo perms
# hadolint ignore=SC2174
RUN groupadd -g ${guest_gid} rocq \
  && useradd --no-log-init -m -s /bin/bash -g rocq -G sudo -p '' -u ${guest_uid} rocq \
  # Create dirs for user scripts
  && mkdir -m 775 -p -v /home/rocq/bin /home/rocq/.local/bin \
  && chown rocq:rocq /home/rocq/bin /home/rocq/.local /home/rocq/.local/bin

# Load travis.sh, umask.sh at login
COPY travis.sh umask.sh /etc/profile.d/

WORKDIR /home/rocq

USER rocq

ENV NJOBS="2"
ENV OPAMPRECISETRACKING="1"

ENTRYPOINT ["opam", "exec", "--"]

CMD ["/bin/bash", "--login"]

LABEL maintainer="erik@martin-dorel.org"
