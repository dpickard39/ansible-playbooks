
# Ansible Docker

# Use the ubuntu:latest
FROM ubuntu:jammy
ENV DEBIAN_FRONTEND=noninteractive
ENV pip_packages="ansible ansible-core ansible-pylibssh backports.entry-points-selectable bcrypt certifi cffi chardet charset-normalizer cryptography distlib filelock fortiosapi idna Jinja2 MarkupSafe oyaml packaging paramiko platformdirs pycparser PyNaCl pyparsing PyYAML requests resolvelib six urllib3 virtualenv openpyxl pyvmomi pyaoscx" 
# This Dockerfile adds a non-root user with sudo access. Use the "remoteUser"
# property in devcontainer.json to use it. On Linux, the container user's GID/UIDs
# will be updated to match your local UID/GID (when using the dockerFile property).
# See https://aka.ms/vscode-remote/containers/non-root-user for details.

ARG USERNAME=vscode
ARG USER_UID=1000
ARG USER_GID=$USER_UID


# To make it easier for build and release pipelines to run apt-get,
# configure apt to not require confirmation (assume the -y argument by default)
RUN echo "APT::Get::Assume-Yes \"true\";" > /etc/apt/apt.conf.d/90assumeyes

RUN apt-get update \
    && apt-get install -y --no-install-recommends \
    apt-utils \
    software-properties-common \
    build-essential \
    jq \
    git \
    locales \
    sshpass \
    lftp \
    openssh-client \
    python3 \
    python3-dev \
    python3-pip && \
    rm -rf /var/lib/apt/lists/*

# Create and set the Locale
RUN sed -i -e 's/# en_US.UTF-8 UTF-8/en_US.UTF-8 UTF-8/' /etc/locale.gen \
    && dpkg-reconfigure --frontend=noninteractive locales \
    && update-locale LANG=en_US.UTF-8
ENV LANG en_US.UTF-8 
ENV LC_ALL en_US.UTF-8

RUN add-apt-repository universe \
    && add-apt-repository multiverse \
    && apt update

RUN pip3 install --upgrade \
    setuptools \
    pip

RUN pip3 install $pip_packages

RUN ln -s /usr/bin/python3 /usr/bin/python \
    # Create a non-root user to use if preferred - see https://aka.ms/vscode-remote/containers/non-root-user.
    && groupadd --gid $USER_GID $USERNAME \
    && useradd -s /bin/bash --uid $USER_UID --gid $USER_GID -m $USERNAME \
    # [Optional] Add sudo support for the non-root user
    && apt-get install sudo \
    && echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME\
    && chmod 0440 /etc/sudoers.d/$USERNAME \
    #
    # Clean up
    && apt-get autoremove \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*


# Install Ansible Collections.
RUN ansible-galaxy collection install fortinet.fortios \
    && ansible-galaxy collection install cisco.ios \
    && ansible-galaxy collection install cisco.meraki \
    && ansible-galaxy collection install community.vmware \
    && ansible-galaxy collection install arubanetworks.aos_switch \
    && ansible-galaxy collection install arubanetworks.aoscx \
    && ansible-galaxy install arubanetworks.aoscx_role

USER $USERNAME