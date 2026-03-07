# Container image with openSUSE Leap + supervisord
#
# Description:
#
# This image runs grommunio-core services using supervisord as PID 1,
# eliminating the need for systemd, privileged mode, and sysbox runtime.

FROM opensuse/leap:15.6

ARG GROMMUNIO_REPO=openSUSE_Leap_15.6

RUN zypper -n install -y \
        iptables   \
        iproute2    \
        kmod       \
        curl    \
        patterns-base-apparmor \
        patterns-base-base \
        procps  \
        sudo       \
        cronie

RUN	curl https://download.grommunio.com/RPM-GPG-KEY-grommunio > gr.key && \
	  rpm --import gr.key && \
	  zypper --non-interactive --quiet ar -C https://download.grommunio.com/community/${GROMMUNIO_REPO} grommunio && \
	  zypper --gpg-auto-import-keys ref && \
	  zypper -n refresh grommunio

# Install grommunio packages (retry up to 3 times for transient download errors)
RUN    zypper --non-interactive install -y vim mysql nginx \
         gromox grommunio-admin-api grommunio-admin-web grommunio-antispam \
         grommunio-common grommunio-web grommunio-sync grommunio-dav postfix postfix-mysql \
         grommunio-chat grommunio-keycloak grommunio-auth cyrus-sasl-saslauthd cyrus-sasl-plain jq redis \
       || { echo "Retry 1..."; zypper clean -a; sleep 5; \
            zypper --non-interactive install -y vim mysql nginx \
              gromox grommunio-admin-api grommunio-admin-web grommunio-antispam \
              grommunio-common grommunio-web grommunio-sync grommunio-dav postfix postfix-mysql \
              grommunio-chat grommunio-keycloak grommunio-auth cyrus-sasl-saslauthd cyrus-sasl-plain jq redis; } \
       || { echo "Retry 2..."; zypper clean -a; sleep 10; \
            zypper --non-interactive install -y vim mysql nginx \
              gromox grommunio-admin-api grommunio-admin-web grommunio-antispam \
              grommunio-common grommunio-web grommunio-sync grommunio-dav postfix postfix-mysql \
              grommunio-chat grommunio-keycloak grommunio-auth cyrus-sasl-saslauthd cyrus-sasl-plain jq redis; }

# Install Certbot & dependencies (Python 3.11)
RUN    zypper -n install -y python311 python311-pip libopenssl-devel && \
        pip3.11 install --no-cache-dir certbot certbot-nginx && \
        ln -sf /usr/local/bin/certbot /usr/bin/certbot

# Switch default python3 version, as many dependencies are available only for python3.11
RUN ln -nsf python3.11 /usr/bin/python3

# Install supervisord
RUN pip3.11 install --no-cache-dir supervisor

# Create default user if needed
RUN    groupadd 'wheel' &&                         \
    useradd -m -s /bin/bash -G wheel admin && echo "admin:admin" | chpasswd

COPY gromox-core/scripts /home/scripts
COPY gromox-core/common /home/common
COPY gromox-core/config /home/config

RUN chmod +x /home/scripts/db.sh

COPY gromox-core/entrypoint.sh /home/entrypoint.sh
RUN chmod +x /home/entrypoint.sh

COPY gromox-core/docker-entrypoint.sh /home/docker-entrypoint.sh
RUN chmod +x /home/docker-entrypoint.sh

COPY gromox-core/supervisord.conf /etc/supervisord.conf
COPY gromox-core/supervisor.d /etc/supervisor.d

ENTRYPOINT ["/home/docker-entrypoint.sh"]
