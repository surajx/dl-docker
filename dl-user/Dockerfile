FROM surajx/dl:tf_torch-cuda10.1-ubuntu20.04

LABEL maintainer="Suraj Narayanan Sasikumar suraj@hessianailabs.com"

EXPOSE 8443

ARG user=surajx

RUN useradd --shell /bin/bash -m -d /home/${user} ${user} && \
    chown -R ${user} /home/${user} && \
    adduser ${user} sudo && \
    echo '%sudo ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers

USER ${user}

WORKDIR /home/${user}

RUN mkdir .local/ bin/ .ssh/
RUN CODE_VERSION=$(curl -sX GET "https://api.github.com/repos/cdr/code-server/releases/latest" \
    | awk '/tag_name/{print $4;exit}' FS='[""]' | awk '{print substr($1,2); }') \
    && curl -LJO "https://github.com/cdr/code-server/releases/download/v${CODE_VERSION}/code-server-${CODE_VERSION}-linux-amd64.tar.gz" \
    && mkdir -p ".local/lib" ".local/bin" \
    && tar -C ".local/lib" -xzf "code-server-${CODE_VERSION}-linux-amd64.tar.gz" \
    && mv -f ".local/lib/code-server-${CODE_VERSION}-linux-amd64" ".local/lib/code-server-${CODE_VERSION}" \
    && ln -fs "${HOME}/.local/lib/code-server-${CODE_VERSION}/bin/code-server" ".local/bin/code-server" \
    && rm -rf "code-server-${CODE_VERSION}-linux-amd64.tar.gz"

RUN ~/.local/bin/code-server --install-extension ms-python.python --install-extension eamodio.gitlens --install-extension vscode-icons-team.vscode-icons

COPY surajx.gitconfig .gitconfig
COPY surajx.bashrc .bashrc

RUN sudo chown surajx:surajx .bashrc && sudo chmod 644 .bashrc && \
    sudo chown surajx:surajx .gitconfig && sudo chmod 664 .gitconfig
