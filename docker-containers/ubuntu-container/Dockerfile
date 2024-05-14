FROM ubuntu:latest

RUN apt-get update \
    && apt-get upgrade \
    && apt-get install -y openssh-server passwd

RUN sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config
RUN echo 'root:root' | chpasswd

EXPOSE 22

# TODO: Delete if not necessary
# RUN useradd -m -s /bin/bash -G sudo dockeruser && echo 'dockeruser:pass' | chpasswd
# RUN echo 'dockeruser ALL=(ALL) NOPASSWD: /usr/bin/apt-get' >> /etc/sudoers

# USER dockeruser
# WORKDIR /home/dockeruser

# RUN mkdir -p /home/dockeruser/.ssh/ \
#     && chmod 0700 /home/dockeruser/.ssh \
#     && touch /home/dockeruser/.ssh/authorized_keys \
#     && chmod 0600 /home/dockeruser/.ssh/authorized_keys

USER root

ENTRYPOINT service ssh start && bash