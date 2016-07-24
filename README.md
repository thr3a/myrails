# オレオレRails APIモード

Nginxの利用を想定したRailsテンプレート

- ruby 2.3.0
- Rails 5.0.0
- MySQL
- Capistrano Version: 3.5.0
- Puma(仮)

# インストール

```
git clone https://github.com/thr3a/myrails.git -b api
cd myrails
bundle install
```

# デプロイの設定

予めデプロイ先にはrbev＆ruby2.3.0&nginxを入れておく

**.gitignore** に以下を追加

```
/config/secrets.yml
/config/database.yml
```

インデックスから削除

```
git rm --cached -f config/secrets.yml
git rm --cached -f config/database.yml
```

**config/database.yml** と **config/secrets.yml** の設定を行う

**config/deploy/production.rb** と **config/deploy.rb** でそれぞれ設定

# デプロイ

```
bundle exec cap production deploy:mkdir
bundle exec cap production deploy:check
bundle exec cap production deploy:upload
bundle exec cap production deploy
```

pumaの設定変更したら `bundle exec cap production puma:config` すること

# その他

### nginxの設定例

myrailsのところは適宜変更すること

```
upstream puma {
  server unix:/var/www/myrails/shared/tmp/sockets/puma.sock;
}

server {
  listen 80 default_server;
  access_log off;
  error_log /var/log/nginx/error.log;
  root /var/www/myrails/current/public;
  client_max_body_size 0;
  error_page 404 /404.html;
  error_page 500 502 503 504 /500.html;

  location / {
    try_files $uri @proxy;
  }
  location @proxy {
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Host $http_host;
    proxy_pass http://puma;
  }
}
```
