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

### dc asia

````
certbot certonly --dns-cloudflare \
  --dns-cloudflare-credentials /etc/letsencrypt/cloudflare.ini \
  -d digitalcloud.asia \
  -d www.digitalcloud.asia \
  --preferred-challenges dns-01 \
  --force-renewal \
  --dns-cloudflare-propagation-seconds 60 \
  -v
````



````
手动模式（推荐）

  certbot certonly --manual --preferred-challenges http -d ai.fcloudpaas.com

  运行后，Certbot 会：
  1. 显示一个随机文件名和内容
  2. 要求你把这个文件放到 http://ai.fcloudpaas.com/.well-known/acme-challenge/ 目录下
  3. 你需要在外部的 nginx 服务器上创建这个文件
  4. 按回车继续验证

  具体步骤：

  在 certbot 容器内运行：
  certbot certonly --manual --preferred-challenges http -d ai.fcloudpaas.com

  Certbot 会显示类似这样的信息：
  Create a file containing just this data:
  abc123xyz.def456uvw

  And make it available on your web server at this URL:
  http://ai.fcloudpaas.com/.well-known/acme-challenge/abc123xyz

  Press Enter to Continue

  然后在你的 nginx 服务器上：
  # 创建目录
  mkdir -p /var/www/html/.well-known/acme-challenge

  # 创建验证文件（使用 Certbot 显示的文件名和内容）
  echo "abc123xyz.def456uvw" > /var/www/html/.well-known/acme-challenge/abc123xyz

  # 设置权限
  chmod 644 /var/www/html/.well-known/acme-challenge/abc123xyz

  确保 nginx 配置允许访问：
  location ^~ /.well-known/acme-challenge/ {
      root /var/www/html;
      default_type "text/plain";
  }

  测试访问：
  curl http://ai.fcloudpaas.com/.well-known/acme-challenge/abc123xyz

  确认无误后，回到 certbot 容器按回车继续。
````
