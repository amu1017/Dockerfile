# syntax=docker/dockerfile:1.7
FROM certbot/dns-route53:latest

# 起動スクリプトを生成する。
COPY <<-"EOF" /entrypoint.sh 
#!/bin/sh
# ----------------Start---------------
ACKEY=$(cat /run/secrets/AWS_ACCESS_KEY_ID)
SECKEY=$(cat /run/secrets/AWS_SECRET_ACCESS_KEY)
#CERT_DOMAIN=
#ACME_MAIL=

[ ! -d /root/.aws/ ] && install -m 0600 -d /root/.aws
echo "[default]" > /root/.aws/config
echo "aws_access_key_id=${ACKEY}" >> /root/.aws/config
echo "aws_secret_access_key=${SECKEY}" >> /root/.aws/config
if [ -f "/etc/letsencrypt/renewal/${CERT_DOMAIN}.conf" ]; then
    echo "get renew *.${CERT_DOMAIN} $(date "+%Y%m%d%H%M")"
    certbot renew --dns-route53 -n -v 
else
    echo "get new cert *.${CERT_DOMAIN} $(date "+%Y%m%d%H%M")"
    certbot certonly --dns-route53 -n -v \
    -d *.${CERT_DOMAIN} --key-type rsa \
    --email ${ACME_MAIL} --agree-tos
fi
cp -pLf /etc/letsencrypt/live/${CERT_DOMAIN}/cert.pem /common/cert.pem
cp -pLf /etc/letsencrypt/live/${CERT_DOMAIN}/chain.pem /common/chain.pem
cp -pLf /etc/letsencrypt/live/${CERT_DOMAIN}/fullchain.pem /common/fullchain.pem
cp -pLf /etc/letsencrypt/live/${CERT_DOMAIN}/privkey.pem /common/privkey.pem
echo "update $(date "+%Y%m%d%H%M")" > /common/updatetime.txt
# ----------------End-----------------
EOF
RUN chmod +x /entrypoint.sh

# 起動時に実行するコマンド
ENTRYPOINT ["/entrypoint.sh"]
