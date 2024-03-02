FROM ubuntu:20.04

# Install ssh
RUN apt-get update && apt-get install -y openssh-server

# Configure SSH
RUN mkdir /var/run/sshd
RUN sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config

# Configure root password
ARG USER_PSWD=changeme
RUN echo "root:$USER_PSWD" | chpasswd

EXPOSE 22

# Start SSH server
CMD ["/usr/sbin/sshd", "-D"]
