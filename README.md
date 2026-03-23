# certbot

### claw.burncloud.com

````
sudo apt update
sudo apt install certbot python3-certbot-nginx
````

````
# 默认的 80 端口 fallback 规则（兜底处理未绑定的域名）
server {
    listen 80 default_server;
    listen [::]:80 default_server;
    root /var/www/html;
    index index.html index.htm index.nginx-debian.html;
    server_name _;

    location / {
        try_files $uri $uri/ =404;
    }
}

# 80 端口 HTTP 流量强制 301 重定向到 HTTPS
server {
    listen 80;
    listen [::]:80;
    server_name claw.burncloud.com;

    if ($host = claw.burncloud.com) {
        return 301 https://$host$request_uri;
    }
    
    return 404;
}

# 443 端口 HTTPS 核心业务及反向代理
server {
    listen 443 ssl;
    listen [::]:443 ssl ipv6only=on;
    server_name claw.burncloud.com;

    ssl_certificate /etc/letsencrypt/live/claw.burncloud.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/claw.burncloud.com/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;

    location / {
        # 将流量代理到本地 8080 端口
        proxy_pass http://127.0.0.1:8080;
        
        # 传递客户端真实信息给后端
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # 如果你的后端应用需要支持 WebSocket（比如某些 AI Agent 或聊天应用），建议加上下面两行
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
````

### tianhuan-ai.com

````
certbot --nginx -d tianhuan-ai.com
````

