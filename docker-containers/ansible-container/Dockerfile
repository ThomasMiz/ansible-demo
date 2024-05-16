FROM ubuntu:latest

RUN apt-get update \
    && apt-get -y upgrade \
    && apt-get clean \
    && apt-get install -y ansible

# Ansible SSH configuration stuff
RUN mkdir -p /root/.ssh \
    && ssh-keygen -t rsa -b 4096 -N '' -f /root/.ssh/id_rsa \
    && cat /root/.ssh/id_rsa.pub >> /root/.ssh/authorized_keys \
    && chmod 600 /root/.ssh/authorized_keys \
    && mkdir -p ~/.ssh && echo "Host *" > ~/.ssh/config \
    && echo " StrictHostKeyChecking no" >> ~/.ssh/config

# Install rustup related stuff
RUN apt-get install -y curl && apt-get install -y gcc && curl https://sh.rustup.rs -sSf | bash -s -- -y
RUN echo 'source $HOME/.cargo/env' >> $HOME/.bashrc

#
RUN mkdir -p /ansible && chmod o-w /ansible
WORKDIR /ansible