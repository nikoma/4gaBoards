## Web server configuration (Nginx/Apache)

### Nginx example (/etc/nginx/conf.d/4gaBoards.conf)

Replace `demo.4gaboards.com` with your domain name, and configure SSL in the SSL Variant.

#### SSL Variant

Set `BASE_URL=https://demo.4gaboards.com` in `docker-compose.yml`.

```nginx
upstream 4gaBoards {
   server localhost:3000;
   keepalive 32;
}

server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name demo.4gaboards.com;

    access_log /var/log/nginx/4gaBoards-access.log;
    error_log  /var/log/nginx/4gaBoards-error.log error;

    # SSL Configuration
    ssl_certificate /etc/letsencrypt/live/demo.4gaboards.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/demo.4gaboards.com/privkey.pem;
    ssl_session_cache shared:SSL:10m;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers "ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384";
    ssl_prefer_server_ciphers on;

    client_max_body_size 50M;
    proxy_set_header Host $http_host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header X-Frame-Options SAMEORIGIN;
    proxy_buffers 256 16k;
    proxy_buffer_size 16k;

    location ~* \.io {
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_read_timeout 1d;
        client_body_timeout 60;
        send_timeout 300;
        lingering_timeout 5;
        proxy_connect_timeout 1d;
        proxy_send_timeout 1d;
        proxy_pass http://4gaBoards;
    }

    location / {
        proxy_set_header Connection "";
        proxy_read_timeout 600s;
        proxy_cache_revalidate on;
        proxy_cache_min_uses 2;
        proxy_cache_use_stale timeout;
        proxy_cache_lock on;
        proxy_http_version 1.1;
        proxy_pass http://4gaBoards;
    }
}
```

#### Non-SSL Variant

Set `BASE_URL=http://demo.4gaboards.com` in `docker-compose.yml`.

```nginx
upstream 4gaBoards {
    server localhost:3000;
    keepalive 32;
}

server {
    server_name demo.4gaboards.com;
    listen 80;
    listen [::]:80;
    access_log  /var/log/nginx/4gaBoards.access.log;
    error_log   /var/log/nginx/4gaBoards.error.log;

    client_max_body_size 50M;
    proxy_set_header Host $http_host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header X-Frame-Options SAMEORIGIN;
    proxy_buffers 256 16k;
    proxy_buffer_size 16k;

    location ~* \.io {
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_read_timeout 1d;
        client_body_timeout 60;
        send_timeout 300;
        lingering_timeout 5;
        proxy_connect_timeout 1d;
        proxy_send_timeout 1d;
        proxy_pass http://4gaBoards;
    }

    location / {
        proxy_set_header Connection "";
        proxy_read_timeout 600s;
        proxy_cache_revalidate on;
        proxy_cache_min_uses 2;
        proxy_cache_use_stale timeout;
        proxy_cache_lock on;
        proxy_http_version 1.1;
        proxy_pass http://4gaBoards;
    }
}
```

#### Redirect to 4gaBoards directory e.g. demo.4gaboards.com/4gaBoards (Favicon not showing)

Set `BASE_URL=http://demo.4gaboards.com/4gaBoards` in `docker-compose.yml`.

```nginx
upstream 4gaBoards {
    server localhost:3000;
    keepalive 32;
}

server {
    server_name demo.4gaboards.com/4gaBoards;
    listen 80;
    listen [::]:80;
    access_log  /var/log/nginx/4gaBoards.access.log;
    error_log   /var/log/nginx/4gaBoards.error.log;

    client_max_body_size 50M;
    proxy_set_header Host $http_host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header X-Frame-Options SAMEORIGIN;
    proxy_buffers 256 16k;
    proxy_buffer_size 16k;

    location ~* \.io {
        rewrite ^/4gaBoards/(.*)$ /$1 break;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_read_timeout 1d;
        client_body_timeout 60;
        send_timeout 300;
        lingering_timeout 5;
        proxy_connect_timeout 1d;
        proxy_send_timeout 1d;
        proxy_pass http://4gaBoards;
    }

    location /4gaBoards {
        rewrite ^/4gaBoards/(.*)$ /$1 break;
        proxy_set_header Connection "";
        proxy_read_timeout 600s;
        proxy_cache_revalidate on;
        proxy_cache_min_uses 2;
        proxy_cache_use_stale timeout;
        proxy_cache_lock on;
        proxy_http_version 1.1;
        proxy_pass http://4gaBoards;
    }
}
```

### Apache example (/etc/httpd/conf/httpd.conf)

Replace `demo.4gaboards.com` with your domain name, and configure SSL in the SSL Variant.

#### SSL Variant

Set `BASE_URL=https://demo.4gaboards.com` in `docker-compose.yml`.

```
LoadModule ssl_module modules/mod_ssl.so

Listen 443
<VirtualHost *:443>#
    ServerName demo.4gaboards.com
    SSLEngine on
    SSLCertificateFile "/etc/letsencrypt/live/demo.4gaboards.com/fullchain.pem"
    SSLCertificateKeyFile "/etc/letsencrypt/live/demo.4gaboards.com/privkey.pem"
    
    RewriteEngine On
        RewriteCond %{HTTP:Upgrade} =websocket [NC]
        RewriteRule /socket.io/(.*) ws://localhost:3000/socket.io/$1 [P,L]

    ProxyPreserveHost On
    ProxyRequests Off
            ProxyPass /.well-known !
            ProxyPassReverse /.well-known !
            ProxyPass /robots.txt !
            ProxyPassReverse /robots.txt !
            ProxyPass / http://localhost:3000/
            ProxyPassReverse / http://localhost:3000/

</VirtualHost>
```

#### Non-SSL Variant

Set `BASE_URL=http://demo.4gaboards.com` in `docker-compose.yml`.

```
<VirtualHost *:80>#
    ServerName http://demo.4gaboards.com

    RewriteEngine On
        RewriteCond %{HTTP:Upgrade} =websocket [NC]
        RewriteRule /socket.io/(.*) ws://localhost:3000/socket.io/$1 [P,L]

    ProxyPreserveHost On
    ProxyRequests Off
            ProxyPass /.well-known !
            ProxyPassReverse /.well-known !
            ProxyPass /robots.txt !
            ProxyPassReverse /robots.txt !
            ProxyPass / http://localhost:3000/
            ProxyPassReverse / http://localhost:3000/

</VirtualHost>
```

#### Redirect to 4gaBoards directory e.g. demo.4gaboards.com/4gaBoards

Set `BASE_URL=http://demo.4gaboards.com/4gaBoards` in `docker-compose.yml`.

```
<VirtualHost *:80>#
    ServerName http://demo.4gaboards.com

    RewriteEngine On
        RewriteCond %{HTTP:Upgrade} =websocket [NC]
        RewriteRule /4gaBoards/socket.io/(.*) ws://localhost:3000/socket.io/$1 [P,L]

    ProxyPreserveHost On
    ProxyRequests Off
            ProxyPass /.well-known !
            ProxyPassReverse /.well-known !
            ProxyPass /robots.txt !
            ProxyPassReverse /robots.txt !
            ProxyPass /4gaBoards/ http://localhost:3000/
            ProxyPassReverse /4gaBoards/ http://localhost:3000/

</VirtualHost>
```

### SSL

You can get a free SSL Certificate using Let's Encrypt.
Tutorial for Rocky Linux 9: https://docs.rockylinux.org/guides/security/generating_ssl_keys_lets_encrypt/

### Logging

4ga Boards currently allow you to expose the application's logfile directory to the host machine via a shared volume. This feature is not enabled by default.

To expose the logfile director to the host machine, add the item `./logs/:/app/logs/` under `services.4gaBoards.volumes`.

Note that the directory to the left of the semicolon is regarding the host machine while the directory to the right of the semicolon is regarding the Docker container.

For example, in the above step, `./logs/:/app/logs/` will create the folder `logs` in the same directory where the `docker-compose.yml` file lives.

### Rotating Logs

Logrotate is designed to ease administration of systems that generate large numbers of log files. It allows automatic rotation, compression, removal, and mailing of log files. Each log file may be handled daily, weekly, monthly, or when it grows too large.

#### Setup logrotate for 4ga Boards logs

Create a file in `/etc/logrotate.d` named `4gaBoards` with the following contents:

```
/path/to/4gaBoards/logs/4gaBoards.log {
  daily
  missingok
  rotate 14
  compress
  delaycompress
  notifempty
  create 640 root adm
  sharedscripts
}
```

Ensure to replace logfile directory with your installation’s `/logs/4gaBoards.log` location.

Restart the logrotate service.

### Fail2ban

Fail2ban is a service that uses iptables to automatically drop connections for a pre-defined amount of time from IPs that continuously failed to authenticate to the configured services.

#### Setup a filter and a jail for 4ga Boards

A filter defines regex rules to identify when users fail to authenticate on 4ga Boards's user interface.

Create a file in `/etc/fail2ban/filter.d` named `4gaBoards.conf` with the following contents:

```conf
[Definition]
failregex = ^(.*) Invalid (email or username:|password!) (\"(.*)\"!)? ?\(IP: <ADDR>\)$
ignoreregex =
```

The jail file defines how to handle the failed authentication attempts found by the 4ga Boards filter.

Create a file in `/etc/fail2ban/jail.d` named `4gaBoards.local` with the following contents:

```conf
[4gaBoards]
enabled = true
port = http,https
filter = 4gaBoards
logpath = /path/to/4gaBoards/logs/4gaBoards.log
maxretry = 5
bantime = 900
```

Ensure to replace `logpath`'s value with your installation’s `/logs/4gaBoards.log` location. If you are using ports other than 80 and 443 for your Web server you should replace those too. The bantime and findtime are defined in seconds.

Restart the fail2ban service. You can check the status of your 4ga Boards jail by running:

```bash
fail2ban-client status 4gaBoards
```
