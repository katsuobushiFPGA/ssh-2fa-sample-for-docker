FROM almalinux:8.4

# Add User
ARG USER_NAME
ARG USER_PASS
ARG GROUP_NAME=$USER_NAME
ARG UID=1000
ARG GID=1000
RUN groupadd -g $GID $GROUP_NAME && \
    useradd -m -s /bin/bash -u $UID -g $GID $USER_NAME
RUN echo "$USER_NAME:$USER_PASS" | chpasswd

# ssh
ARG SSH_PORT
RUN yum -y install openssh-clients openssh-server \
&& rm -rf /var/cache/yum/* \
&& yum clean all
RUN sed -i 's/^AllowTcpForwarding no/AllowTcpForwarding yes/g' /etc/ssh/sshd_config
RUN sed -i 's/^GatewayPorts no/GatewayPorts yes/g' /etc/ssh/sshd_config
RUN sed -i 's/^X11Forwarding no/X11Forwarding yes/g' /etc/ssh/sshd_config
RUN sed -i "s/^#Port 22/Port $SSH_PORT/g" /etc/ssh/sshd_config
RUN ssh-keygen -A

WORKDIR /home/$USER_NAME/
RUN mkdir -p /home/$USER_NAME/.ssh
RUN chown $USER_NAME:$GROUP_NAME -R /home/$USER_NAME

USER $USER_NAME
RUN ssh-keygen -t ed25519 -f /home/$USER_NAME/.ssh/id_rsa -N ""
RUN cat /home/$USER_NAME/.ssh/id_rsa.pub >> /home/$USER_NAME/.ssh/authorized_keys
RUN chmod 700 /home/$USER_NAME/.ssh
RUN chmod 600 /home/$USER_NAME/.ssh/authorized_keys

# GoogleAuthenticator
USER root
RUN yum -y install epel-release \
&& yum -y --enablerepo=epel install google-authenticator \
&& yum -y --enablerepo=epel install qrencode-libs \
&& rm -rf /var/cache/yum/* \
&& yum clean all

RUN echo 'auth required pam_google_authenticator.so nullok echo_verification_code' >> /etc/pam.d/sshd
RUN sed -i 's/^ChallengeResponseAuthentication no/ChallengeResponseAuthentication yes/g' /etc/ssh/sshd_config
RUN echo 'AuthenticationMethods publickey,keyboard-interactive' >> /etc/ssh/sshd_config

EXPOSE $SSH_PORT
CMD ["/usr/sbin/sshd", "-D"]