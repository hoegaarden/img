ARG BASE_IMAGE='debian'
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
  && pip3 install --no-cache-dir \
    yq \
  && gem install -q --no-document \
    cf-uaac


RUN \
    CF_URL='https://packages.cloudfoundry.org/stable?release=linux64-binary&version=v7' ; \
    KUBECTL_VER_URL='https://storage.googleapis.com/kubernetes-release/release/stable.txt' ; \
    KUBECTL_BIN_URL='https://storage.googleapis.com/kubernetes-release/release/%s/bin/linux/amd64/kubectl' ; \
    latest() { \
      curl -fsSL "https://api.github.com/repos/${1}/releases?pages=3" \
        | jq --arg re "${2}" -r '[ .[].assets[] | select(.name|test($re)) | .browser_download_url ][0]' ; \
    } ; \
    inst() { \
      install -m 0755 "$2" "${BIN}/${1}" \
        && echo -n "$1 installed: " \
        && { "$1" version || "$1" --version || true ; } 2>/dev/null | sed '/^$/d' | head -1 ; \
    } \
  && inst bosh      <(curl -Lf# "$(latest cloudfoundry/bosh-cli              'linux-amd64')") \
  && inst cf        <(curl -Lf# "${CF_URL}" | tar -O -zxf - cf7) \
  && inst credhub   <(curl -Lf# "$(latest cloudfoundry-incubator/credhub-cli 'linux')" | tar -O -zx ./credhub) \
  && inst fly       <(curl -Lf# "$(latest concourse/concourse                'fly.*linux-amd64.tgz')" | tar -O -zx fly) \
  && inst kubectl   <(curl -Lf# "$(printf "${KUBECTL_BIN_URL}" "$( curl -Lf# "${KUBECTL_VER_URL}" )" )") \
  && inst kustomize <(curl -Lf# "$(latest kubernetes-sigs/kustomize          'linux_amd64.tar.gz')" | tar -O -zx kustomize) \
  && inst om        <(curl -Lf# "$(latest pivotal-cf/om                      'linux')") \
  && inst pivnet    <(curl -Lf# "$(latest pivotal-cf/pivnet-cli              'linux-amd64')") \
  && inst uaa       <(curl -Lf# "$(latest cloudfoundry-incubator/uaa-cli     'linux-amd64')")


RUN \
    GIMME_URL='https://raw.githubusercontent.com/travis-ci/gimme/master/gimme' \
  && install -m 0755 <(curl -Lf# "${GIMME_URL}") "${BIN}/gimme"

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


RUN \
    LUAN_VIM_GIT='https://github.com/luan/vimfiles' \
  && git clone "${LUAN_VIM_GIT}" .vim \
  && cd .vim \
  && npm() { sudo "$(which npm)" "$@" ; } \
  && export -f npm \
  && TERM=xterm-color ./update -n -u

RUN \
    gimme 1.x >> .bashrc

ENV PATH="${PATH}:${OPS_HOME}/bin"
