#-------------------------------------------------------------------------------------------------------------
# Copyright (c) Microsoft Corporation. All rights reserved.
# Licensed under the MIT License. See https://go.microsoft.com/fwlink/?linkid=2090316 for license information.
#-------------------------------------------------------------------------------------------------------------

FROM php:8.3-cli

# Avoid warnings by switching to noninteractive
ENV DEBIAN_FRONTEND=noninteractive

# This Dockerfile adds a non-root 'vscode' user with sudo access. However, for Linux,
# this user's GID/UID must match your local user UID/GID to avoid permission issues
# with bind mounts. Update USER_UID / USER_GID if yours is not 1000. See
# https://aka.ms/vscode-remote/containers/non-root-user for details.
ARG USERNAME=vscode
ARG USER_UID=1000
ARG USER_GID=$USER_UID

# Configure apt and install packages
RUN apt-get update \
    && apt-get -y install --no-install-recommends apt-utils dialog 2>&1 \
    && apt-get -y install curl git iproute2 procps lsb-release unzip zip openssl gnupg \
    && apt-get -y install libzip-dev libicu-dev libxml2-dev libcurl4-openssl-dev \
    && apt-get -y install sudo \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Create a non-root user
RUN groupadd --gid $USER_GID $USERNAME \
    && useradd -s /bin/bash --uid $USER_UID --gid $USER_GID -m $USERNAME \
    && echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME \
    && chmod 0440 /etc/sudoers.d/$USERNAME

# Install PHP extensions
RUN docker-php-ext-configure zip \
    && docker-php-ext-configure intl \
    && docker-php-ext-install zip bcmath mysqli pdo pdo_mysql intl xml curl tokenizer ctype fileinfo mbstring

# Install Composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer \
    && chmod +x /usr/local/bin/composer

# Install Node.js 20.x
RUN curl -fsSL https://deb.nodesource.com/setup_20.x | bash - \
    && apt-get install -y nodejs \
    && npm install -g npm@latest

# Setup PATH and aliases for root
RUN echo 'export PATH="$PATH:$HOME/.composer/vendor/bin:vendor/bin"' >> /root/.bashrc \
    && echo "alias art='php artisan'" >> /root/.bashrc \
    && echo "alias serve='php artisan serve --port=8000 --host=0.0.0.0'" >> /root/.bashrc \
    && echo "alias ls='ls -B -h --color=auto -ltr'" >> /root/.bashrc \
    && echo "alias _='sudo'" >> /root/.bashrc \
    && echo "alias apt='sudo apt'" >> /root/.bashrc \
    && echo "alias apt-get='sudo apt-get'" >> /root/.bashrc

# Setup PATH and aliases for vscode user
RUN echo 'export PATH="$PATH:$HOME/.composer/vendor/bin:vendor/bin"' >> /home/$USERNAME/.bashrc \
    && echo "alias art='php artisan'" >> /home/$USERNAME/.bashrc \
    && echo "alias serve='php artisan serve --port=8000 --host=0.0.0.0'" >> /home/$USERNAME/.bashrc \
    && echo "alias ls='ls -B -h --color=auto -ltr'" >> /home/$USERNAME/.bashrc \
    && echo "alias _='sudo'" >> /home/$USERNAME/.bashrc \
    && echo "alias apt='sudo apt'" >> /home/$USERNAME/.bashrc \
    && echo "alias apt-get='sudo apt-get'" >> /home/$USERNAME/.bashrc

# Install Laravel installer for vscode user
RUN su $USERNAME -c "composer global require laravel/installer"

# Switch back to dialog for any ad-hoc use of apt-get
ENV DEBIAN_FRONTEND=