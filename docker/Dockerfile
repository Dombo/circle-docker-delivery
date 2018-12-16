###
# Upstream: https://github.com/nodejs/docker-node/blob/master/10/stretch/Dockerfile
FROM node:10.14.1-stretch as base
###
# refresh system package manager cache
# install system dependencies
# output application manifest for posterity/retrospective audit in dependency hell
RUN apt-get update \
  && apt-get install -y \
    git \
    jq \
    locales sudo openssh-client ca-certificates tar gzip parallel \
    net-tools netcat unzip zip bzip2 gnupg curl wget \
    --no-install-recommends \
  && dpkg-query  --showformat '${binary:Package} ${Version}\n' --show \
  && rm -rf /var/lib/apt/lists/*
###
# Install Chromium for runtime PDF generation
# Note: This blows out the container size big time :(
RUN apt-get update && apt-get install -y \
      chromium \
      chromium-l10n \
      fonts-liberation \
      fonts-roboto \
      hicolor-icon-theme \
      libcanberra-gtk-module \
      libexif-dev \
      libgl1-mesa-dri \
      libgl1-mesa-glx \
      libpango1.0-0 \
      libv4l-0 \
      fonts-symbola \
      --no-install-recommends \
    && rm -rf /var/lib/apt/lists/*
ENV CHROME_BIN=/usr/bin/chromium-browser
ENV CHROME_PATH=/usr/lib/chromium/
CMD ["/bin/bash"]

FROM base as build
###
# install build tooling
# output package manager global manifest
RUN npm install -g \
  gulp-cli \
  pm2 \
  serverless && \
  npm list -g --depth=0
###
# install node process monitor typescript modules
RUN pm2 install typescript
ENV NODE_ENV development
WORKDIR /usr/src
COPY ./tools ./tools
RUN cd tools/monorepo && npm install && npm run build
CMD ["/bin/bash"]

FROM build as ci
###
### DO NOT MODIFY THIS FILE.
### THIS FILE HAS BEEN COPIED FROM:
###  https://github.com/CircleCI-Public/circleci-dockerfiles/blob/master/node/images/10.14.1-stretch/Dockerfile
### YOU MAY MODIFY THE IMAGE BETWEEN
###  # BEGIN IMAGE CUSTOMIZATIONS
###  #
###  ## END IMAGE CUSTOMIZATIONS
###
# make Apt non-interactive
RUN echo 'APT::Get::Assume-Yes "true";' > /etc/apt/apt.conf.d/90circleci \
  && echo 'DPkg::Options "--force-confnew";' >> /etc/apt/apt.conf.d/90circleci
ENV DEBIAN_FRONTEND=noninteractive
# man directory is missing in some base images
# https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=863199
RUN apt-get update \
  && mkdir -p /usr/share/man/man1 \
  && apt-get install -y \
    git mercurial xvfb \
    locales sudo openssh-client ca-certificates tar gzip parallel \
    net-tools netcat unzip zip bzip2 gnupg curl wget
# Set timezone to UTC by default
RUN ln -sf /usr/share/zoneinfo/Etc/UTC /etc/localtime
# Use unicode
RUN locale-gen C.UTF-8 || true
ENV LANG=C.UTF-8
# Install Docker
# Docker.com returns the URL of the latest binary when you hit a directory listing
# We curl this URL and `grep` the version out.
# The output looks like this:
#>    # To install, run the following commands as root:
#>    curl -fsSLO https://download.docker.com/linux/static/stable/x86_64/docker-17.05.0-ce.tgz && tar --strip-components=1 -xvzf docker-17.05.0-ce.tgz -C /usr/local/bin
#>
#>    # Then start docker in daemon mode:
#>    /usr/local/bin/dockerd
RUN set -ex \
  && export DOCKER_VERSION=$(curl --silent --fail --retry 3 https://download.docker.com/linux/static/stable/x86_64/ | grep -o -e 'docker-[.0-9]*-ce\.tgz' | sort -r | head -n 1) \
  && DOCKER_URL="https://download.docker.com/linux/static/stable/x86_64/${DOCKER_VERSION}" \
  && echo Docker URL: $DOCKER_URL \
  && curl --silent --show-error --location --fail --retry 3 --output /tmp/docker.tgz "${DOCKER_URL}" \
  && ls -lha /tmp/docker.tgz \
  && tar -xz -C /tmp -f /tmp/docker.tgz \
  && mv /tmp/docker/* /usr/bin \
  && rm -rf /tmp/docker /tmp/docker.tgz \
  && which docker \
  && (docker version || true)
# docker compose
RUN COMPOSE_URL="https://circle-downloads.s3.amazonaws.com/circleci-images/cache/linux-amd64/docker-compose-latest" \
  && curl --silent --show-error --location --fail --retry 3 --output /usr/bin/docker-compose $COMPOSE_URL \
  && chmod +x /usr/bin/docker-compose \
  && docker-compose version
# install dockerize
RUN DOCKERIZE_URL="https://circle-downloads.s3.amazonaws.com/circleci-images/cache/linux-amd64/dockerize-latest.tar.gz" \
  && curl --silent --show-error --location --fail --retry 3 --output /tmp/dockerize-linux-amd64.tar.gz $DOCKERIZE_URL \
  && tar -C /usr/local/bin -xzvf /tmp/dockerize-linux-amd64.tar.gz \
  && rm -rf /tmp/dockerize-linux-amd64.tar.gz \
  && dockerize --version
RUN groupadd --gid 3434 circleci \
  && useradd --uid 3434 --gid circleci --shell /bin/bash --create-home circleci \
  && echo 'circleci ALL=NOPASSWD: ALL' >> /etc/sudoers.d/50-circleci \
  && echo 'Defaults    env_keep += "DEBIAN_FRONTEND"' >> /etc/sudoers.d/env_keep
# BEGIN IMAGE CUSTOMIZATIONS
# END IMAGE CUSTOMIZATIONS
USER circleci
CMD ["/bin/bash"]

FROM build as dev
###
# Install system developer tools
RUN apt-get update \
  && apt-get install -y \
    htop \
    iotop \
    nethogs \
    imagemagick \
  && rm -rf /var/lib/apt/lists/*
COPY --from=build /usr/src/tools/monorepo/lib/monorepo.js /usr/bin/monorepo
RUN npm install -g \
  localtunnel \
  serverless \
  && npm list -g --depth=0
CMD ["/bin/bash"]

FROM base as release