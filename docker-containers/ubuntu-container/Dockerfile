FROM ubuntu:latest

USER root

# https://stackoverflow.com/questions/53383431/how-to-enable-systemd-on-dockerfile-with-ubuntu18-04
RUN apt-get update \
    && apt-get upgrade -y \
    && apt-get clean all -y \
    && apt-get install -y openssh-server passwd init \
    && sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config \
    && echo 'root:root' | chpasswd

EXPOSE 22

CMD service ssh start
ENTRYPOINT [ "/sbin/init" ]