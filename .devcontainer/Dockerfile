FROM ubuntu:noble

RUN apt update -y && apt install -y gpg wget curl git
RUN install -dm 755 /etc/apt/keyrings
RUN wget -qO - https://mise.jdx.dev/gpg-key.pub | gpg --dearmor > /etc/apt/keyrings/mise-archive-keyring.gpg
RUN echo "deb [signed-by=/etc/apt/keyrings/mise-archive-keyring.gpg arch=amd64] https://mise.jdx.dev/deb stable main" > /etc/apt/sources.list.d/mise.list
RUN apt update
RUN apt install -y mise

RUN echo 'eval "$(mise activate bash --shims)"' >> ~/.bash_profile
RUN echo 'eval "$(mise activate bash)"' >> ~/.bashrc

RUN mise use -g bun@latest
RUN mise use -g node@lts
RUN mise use -g python@latest
RUN mise install
RUN mise reshim

RUN wget https://github.com/earthly/earthly/releases/latest/download/earthly-linux-amd64 -O /usr/local/bin/earthly
RUN chmod +x /usr/local/bin/earthly
RUN /usr/local/bin/earthly bootstrap --with-autocomplete
