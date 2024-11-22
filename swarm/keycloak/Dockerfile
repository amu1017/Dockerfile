# ----- MultiStage 1/3 Package Install in BUild Container -----
FROM amazoncorretto:17 AS build-container

ARG KEYCLOAK_VERSION=26.0.6
ARG KEYCLOAK_PKG=https://github.com/keycloak/keycloak/releases/download/${KEYCLOAK_VERSION}/keycloak-${KEYCLOAK_VERSION}.tar.gz

ADD ${KEYCLOAK_PKG} /tmp/keycloak/

RUN yum -y install \
    tar \
    zip \
    gzip \
    && yum clean all && rm -rf /var/cache/yum \
    && cd /tmp/keycloak \
    && tar -xvf keycloak-*.tar.gz\
    && rm -f keycloak-*.tar.gz \
    && mv keycloak-* /opt/keycloak \
    && mkdir -p /opt/keycloak/data \
    && chmod -R g+rwX /opt/keycloak



# ----- MultiStage 2/3 Package Install in Exec Container -----
FROM amazoncorretto:17

RUN yum -y install \
    curl \
    && yum clean all && rm -rf /var/cache/yum

COPY --from=build-container --chown=1000:0 /opt/keycloak /opt/keycloak

RUN echo "keycloak:x:0:root" >> /etc/group && \
    echo "keycloak:x:1000:0:keycloak user:/opt/keycloak:/sbin/nologin" >> /etc/passwd

COPY <<-"EOF" /entrypoint.sh
#!/bin/bash
# ----------------Start---------------
if [ -f "/run/secrets/mysql_database" ] ;then KC_DB_URL_DATABASE=$(cat /run/secrets/mysql_database) ;fi
if [ -f "/run/secrets/mysql_user" ]     ;then KC_DB_USERNAME=$(cat /run/secrets/mysql_user) ;fi
if [ -f "/run/secrets/mysql_pw" ]       ;then KC_DB_PASSWORD=$(cat /run/secrets/mysql_pw) ;fi
if [ -f "/run/secrets/kc_user" ] ;then KEYCLOAK_ADMIN=$(cat /run/secrets/kc_user) ;fi
if [ -f "/run/secrets/kc_pw" ]   ;then KEYCLOAK_ADMIN_PASSWORD=$(cat /run/secrets/kc_pw) ;fi

trap 'echo catch 130 INT;  kill -9 $!; exit 130' INT
trap 'echo catch 143 TERM; kill -9 $!; exit 143' TERM
echo "start keycloak"
/opt/keycloak/bin/kc.sh $1 & wait $!
# ----------------End-----------------
EOF
RUN chmod +x /entrypoint.sh

USER 1000
EXPOSE 8080
EXPOSE 8443
ENTRYPOINT ["/entrypoint.sh"]

