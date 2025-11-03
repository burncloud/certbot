# certbot

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
