# Container image with openSUSE Leap + supervisord
#
# Description:
#
# This image runs grommunio-office/files services using supervisord as PID 1,
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
        sudo

RUN	curl https://download.grommunio.com/RPM-GPG-KEY-grommunio > gr.key && \
	  rpm --import gr.key && \
	  zypper --non-interactive --quiet ar -C https://download.grommunio.com/community/${GROMMUNIO_REPO} grommunio && \
	  zypper --gpg-auto-import-keys ref && \
	  zypper -n refresh grommunio

# Install grommunio packages (retry up to 3 times for transient download errors)
RUN    zypper --non-interactive install -y vim nginx \
         mysql grommunio-common cyrus-sasl-saslauthd cyrus-sasl-plain jq php-fpm redis grommunio-files rabbitmq-server grommunio-office cronie \
       || { echo "Retry 1..."; zypper clean -a; sleep 5; \
            zypper --non-interactive install -y vim nginx \
              mysql grommunio-common cyrus-sasl-saslauthd cyrus-sasl-plain jq php-fpm redis grommunio-files rabbitmq-server grommunio-office cronie; } \
       || { echo "Retry 2..."; zypper clean -a; sleep 10; \
            zypper --non-interactive install -y vim nginx \
              mysql grommunio-common cyrus-sasl-saslauthd cyrus-sasl-plain jq php-fpm redis grommunio-files rabbitmq-server grommunio-office cronie; }

# Switch default python3 version
RUN ln -nsf python3.11 /usr/bin/python3

# Install supervisord
RUN zypper -n install -y python311 python311-pip && \
    pip3.11 install --no-cache-dir supervisor

# Create default user if needed
RUN    groupadd 'wheel' &&                         \
    useradd -m -s /bin/bash -G wheel admin && echo "admin:admin" | chpasswd

COPY gromox-office/scripts /home/scripts
COPY gromox-office/common /home/common
COPY gromox-office/config /home/config

COPY gromox-office/entrypoint.sh /home/entrypoint.sh
RUN chmod +x /home/entrypoint.sh

COPY gromox-office/docker-entrypoint.sh /home/docker-entrypoint.sh
RUN chmod +x /home/docker-entrypoint.sh

COPY gromox-office/supervisord.conf /etc/supervisord.conf
COPY gromox-office/supervisor.d /etc/supervisor.d

ENTRYPOINT ["/home/docker-entrypoint.sh"]
