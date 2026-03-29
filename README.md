# certbot

### burncloud.com

````
sudo apt update
sudo apt install certbot python3-certbot-nginx
certbot --nginx -d burncloud.com
````

````
NEW_DOMAIN="burncloud.com"

cat <<EOF > /etc/nginx/sites-enabled/default
server {
    listen 80;
    listen [::]:80;
    server_name $NEW_DOMAIN;
    if (\$host = $NEW_DOMAIN) {
        return 301 https://\$host\$request_uri;
    }
    return 404;
}

server {
    listen 443 ssl;
    listen [::]:443 ssl ipv6only=on;
    server_name $NEW_DOMAIN;
    ssl_certificate /etc/letsencrypt/live/$NEW_DOMAIN/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/$NEW_DOMAIN/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;

    location / {
        proxy_pass http://127.0.0.1:8080;
        proxy_set_header Host \$host;
        proxy_set_header X-Real-IP \$remote_addr;
        proxy_set_header X-Forwarded-For \$proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto \$scheme;
        proxy_set_header Upgrade \$http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
EOF

nginx -t && nginx -s reload
````
