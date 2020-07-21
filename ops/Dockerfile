ARG BASE_IMAGE='debian:sid'
FROM "${BASE_IMAGE}" AS layered

LABEL maintainer='Hannes Hörl <horlh@vmware.com>'
ARG DEBIAN_FRONTEND=noninteractive
SHELL [ "/usr/bin/env", "bash", "-euo", "pipefail", "-c" ]

ARG BIN=/usr/local/bin
COPY etc /etc
COPY bin "${BIN}"

RUN \
    apt update -y \
  && apt full-upgrade -y \
  && apt install -y \
    cmake \
    curl \
    git \
    htop \
    jq \
    less \
    ncdu \
    neovim \
    netcat-openbsd \
    npm \
    procps \
    python3-neovim \
    python3-pip \
    ruby ruby-dev \
    sudo \
    wget \
  && apt autopurge -y && apt autoclean -y \
  && rm -rf /var/lib/apt/lists/* \
  && pip install --no-cache-dir \
    yq \
  && gem install -q --no-document \
    cf-uaac


ARG CF_URL=https://packages.cloudfoundry.org/stable?release=linux64-binary&version=v7
RUN \
    latest() { \
      local slug="$1" ; \
      local re="$2" ; \
      curl -fsSL "https://api.github.com/repos/${slug}/releases/latest" \
        | jq --arg re "$re" -r '[ .assets[] | select(.name|test($re)) | .browser_download_url ][0]' ; \
    } \
  && echo bosh    && install -m 0755 <(curl -Lf# "$(latest cloudfoundry/bosh-cli              linux-amd64)")                             "${BIN}/bosh" \
  && echo om      && install -m 0755 <(curl -Lf# "$(latest pivotal-cf/om                      linux)")                                   "${BIN}/om" \
  && echo cf      && install -m 0755 <(curl -Lf# "${CF_URL}" | tar -O -zxf - cf7)                                                        "${BIN}/cf" \
  && echo uaa     && install -m 0755 <(curl -Lf# "$(latest cloudfoundry-incubator/uaa-cli     linux-amd64)")                             "${BIN}/uaa" \
  && echo credhub && install -m 0755 <(curl -Lf# "$(latest cloudfoundry-incubator/credhub-cli linux)" | tar -O -zx ./credhub)            "${BIN}/credhub" \
  && echo pivnet  && install -m 0755 <(curl -Lf# "$(latest pivotal-cf/pivnet-cli              linux-amd64)")                             "${BIN}/pivnet" \
  && echo fly     && install -m 0755 <(curl -Lf# "$(latest concourse/concourse                'fly.*linux-amd64.tgz')" | tar -O -zx fly) "${BIN}/fly"


ARG GIMME_URL=https://raw.githubusercontent.com/travis-ci/gimme/master/gimme
RUN install -m 0755 <(curl -Lf# "${GIMME_URL}") "${BIN}/gimme"

ARG OPS_USER=ops
ARG OPS_HOME=/workspace
RUN \
    useradd "${OPS_USER}" -d "${OPS_HOME}" \
  && echo "${OPS_USER} ALL=(ALL) NOPASSWD:ALL" > "/etc/sudoers.d/${OPS_USER}" \
  && mkdir -p "${OPS_HOME}" && chown -R "${OPS_USER}:${OPS_USER}" "${OPS_HOME}"

WORKDIR "${OPS_HOME}"
USER "${OPS_USER}"

ARG REAL_NAME="${OPS_USER}"
ARG REAL_EMAIL="${OPS_USER}@local"
RUN \
    git config --global user.name "${REAL_NAME}" \
  && git config --global user.email "${REAL_EMAIL}"

ARG LUAN_VIM_GIT=https://github.com/luan/vimfiles
RUN \
    git clone "${LUAN_VIM_GIT}" .vim \
  && cd .vim \
  && npm() { sudo "$(which npm)" "$@" ; } \
  && export -f npm \
  && TERM=xterm-color ./update -n -u

RUN \
    gimme 1.x >> .bashrc

ENV PATH="${PATH}:${OPS_HOME}/bin"
