FROM ubuntu:18.04
ENV DEBIAN_FRONTEND=noninteractive

# Set /bin/bash as SHELL
RUN rm /bin/sh && ln -s /bin/bash /bin/sh

########### SETTINGS ENVIRONMENT ##################
# Username
ENV USER "atcoder"

# Python Versions (space separated, First element is pyenv's global)
ENV PYTHON_VERSIONS="3.8.2 3.4.3"

############### GENERAL SETTINGS ##################
# [18.04] change apt repository to JP
# REF: https://linuxfan.info/ubuntu-18-04-server-basic-settings
RUN perl -p -i.bak \
  -e 's%https?://(?!security)[^ \t]+%http://jp.archive.ubuntu.com/ubuntu/%g' \
  /etc/apt/sources.list

# apt update && apt upgrade
RUN apt-get update && apt upgrade -y

# Create User (for npm)
RUN useradd \
  -m \
  -s /bin/bash \
  -G sudo \
  ${USER}
RUN echo "${USER}:${USER}" | chpasswd
# RUN echo "${USER}    ALL=NOPASSWD:        ALL" >> /etc/sudoers

# Bash PS1
RUN echo 'export PS1="\[\033[38;5;4m\][\[$(tput sgr0)\]\[$(tput bold)\]\[\033[38;5;14m\]AtCoder\[$(tput sgr0)\]\[\033[38;5;4m\]]\[$(tput sgr0)\] \[$(tput sgr0)\]\[$(tput bold)\]\[\033[38;5;11m\]\w\[$(tput sgr0)\]\n\[$(tput bold)\]\\$\[$(tput sgr0)\] \[$(tput sgr0)\]"' >> /home/${USER}/.bashrc


################ Locale settings ###################
# [18.04] stop interaective for tzdata
ENV DEBIAN_FRONTEND=noninteractive
ENV TZ=Asia/Tokyo
# Install locales
RUN apt-get install -y locales \
  && locale-gen en_US.UTF-8 \
  && locale-gen ja_JP.UTF-8 \
  && echo "export LANG=ja_JP.UTF-8" >> /home/${USER}/.bashrc

RUN apt-get install -y language-pack-ja-base


########### Essential Tools #############
RUN apt-get install -y \
  sudo \
  vim \
  git


######### Environment: C/C++ ############
# Essentials
RUN apt-get install -y \
  build-essential \
  gdb

# Install GCC/G++ 9.2.1
RUN apt-get install -y software-properties-common
RUN add-apt-repository -y ppa:jonathonf/gcc-9.2
RUN apt-get update
RUN apt-get install -y gcc-9 g++-9

# Set as default
RUN cd /usr/bin && \
  rm gcc && ln -s gcc-9 gcc && \
  rm g++ && ln -s g++-9 g++

######### Environment: Python ############
# Install dependencies: python-build
RUN apt-get install --no-install-recommends -y \
  make \
  libssl-dev \
  zlib1g-dev \
  libbz2-dev \
  libreadline-dev \
  libsqlite3-dev \
  wget \
  curl \
  llvm \
  libncurses5-dev \
  xz-utils \
  tk-dev \
  libxml2-dev \
  libxmlsec1-dev \
  libffi-dev \
  liblzma-dev
# Changing openssl package (for install Python <3.5)
RUN apt-get remove -y libssl-dev && \
  apt-get install -y libssl1.0-dev

# Change user
USER ${USER}
ENV HOME "/home/${USER}"

# Install pyenv
RUN git clone https://github.com/pyenv/pyenv.git ~/.pyenv
RUN echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bashrc
RUN echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bashrc
RUN echo 'eval "$(pyenv init -)"' >> ~/.bashrc

ENV PYENV_ROOT="$HOME/.pyenv"
ENV PATH="$HOME/.pyenv/bin:$PATH"
RUN eval "$(pyenv init -)"

# Install Python
RUN for PYTHON_VERSION in ${PYTHON_VERSIONS[@]}; do \
  pyenv install ${PYTHON_VERSION}; done

# Set as global
RUN pyenv global ${PYTHON_VERSIONS[0]} && pyenv rehash

ENV PATH="$HOME/.pyenv/shims:$PATH"

# Install Python libraries
RUN pip3 install --upgrade pip
RUN pip3 install \
  ipython \
  scipy


########### Install: atcoder-cli, online-judge-tools #########
# Install: online-judge-tools
RUN pip3 install online-judge-tools

# Install: atcoder-cli
## Install node using nodenv
### nodenv
RUN git clone https://github.com/nodenv/nodenv.git ~/.nodenv
RUN cd ~/.nodenv && src/configure && make -C src

ENV PATH="$HOME/.nodenv/bin:$HOME/.nodenv/shims:$PATH"
RUN echo 'export PATH="$HOME/.nodenv/bin:$HOME/.nodenv/shims:$PATH"' >> ~/.bashrc
RUN echo 'eval "$(nodenv init -)"' >> ~/.bashrc

### node-build
RUN mkdir -p "$(nodenv root)/plugins"
RUN git clone https://github.com/nodenv/node-build.git "$(nodenv root)/plugins/node-build"

### node v.14 (latest)
RUN NODE_INSTALL_VERSION="$(nodenv install -l | grep "^14\.[0-9]" | sort -r | head -1)" && \
  echo "Install node v${NODE_INSTALL_VERSION}" >&2 && \
  nodenv install ${NODE_INSTALL_VERSION} && \
  nodenv global ${NODE_INSTALL_VERSION} && \
  nodenv rehash


## Install: atcoder-cli
RUN echo "Install: atcoder-cli" >&2
### Install from GitHub repo directly (to support --no-open when submit)
RUN npm install -g --save git+https://github.com/Tatamo/atcoder-cli#develop
RUN nodenv rehash

# Append Script
ADD bashrc $HOME/
RUN cat $HOME/bashrc >> $HOME/.bashrc
RUN rm $HOME/bashrc
