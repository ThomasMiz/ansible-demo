FROM ubuntu:latest

RUN apt-get update \
    && apt-get -y upgrade \
    && apt-get clean \
    && apt-get install -y ansible curl

# SSH configuration stuff
RUN mkdir -p /root/.ssh \
    && ssh-keygen -t rsa -b 4096 -N '' -f /root/.ssh/id_rsa \
    && cat /root/.ssh/id_rsa.pub >> /root/.ssh/authorized_keys \
    && chmod 644 /root/.ssh/authorized_keys \
    && chmod 644 /root/.ssh/id_rsa \
    && mkdir -p ~/.ssh && echo "Host *" > ~/.ssh/config \
    && echo " StrictHostKeyChecking no" >> ~/.ssh/config

RUN mkdir -p /ansible && chmod o-w /ansible
WORKDIR /ansible