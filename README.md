# lesson16

1. запустить как md сервис на порту 3000

````
artsiom@ubundu26:/etc/systemd/system$ cat app16_3000.service 
[Unit]
Description=Nodejs app port 3000
After=network.target

[Service]
Type=simple
User=webadmin
Group=webgroup
WorkingDirectory=/var/www/app16/nodejs_app
ExecStart=/usr/bin/node server.js
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
````


````
artsiom@ubundu26:/etc/systemd/system$ sudo systemctl status app16_3000.service
● app16_3000.service - Nodejs app port 3000
     Loaded: loaded (/etc/systemd/system/app16_3000.service; enabled; preset: enabled)
     Active: active (running) since Sun 2026-10-25 22:50:57 +03; 1s ago
 Invocation: ea9233a59b05415fab4ecf4c5bf2b706
   Main PID: 457344 (node)
      Tasks: 7 (limit: 2338)
     Memory: 31.6M (peak: 31.7M)
        CPU: 183ms
     CGroup: /system.slice/app16_3000.service
             └─457344 /usr/bin/node server.js

Oct 25 22:50:57 ubundu26 systemd[1]: Started app16_3000.service - Nodejs app port 3000.
Oct 25 22:50:57 ubundu26 node[457344]: Diary app listening on port 3000
````
http://192.168.2.105:3000/
<img width="921" height="553" alt="Снимок экрана 2026-10-25 в 22 54 13" src="https://github.com/user-attachments/assets/f3284a5e-ffce-400c-924b-bea704f5da22" />



2.Настроить nginx в роли reverse proxy с TLS для этого приложения
созданы сертификаты
````
artsiom@ubundu26:/etc/nginx/ssl$ sudo ls -l /etc/nginx/ssl/
total 8
-rw-r--r-- 1 root root 1277 Oct 25 23:06 diary.crt
-rw------- 1 root root 1704 Oct 25 23:06 diary.key
````
конфигурация без балансировки
````
artsiom@ubundu26:/etc/nginx/sites-available$ cat app16
server {
  listen 80;
  server_name _;

  return 301 https://$host$request_uri;  
}

server {
  listen 443 ssl;
  server_name localhost;

  ssl_certificate     /etc/nginx/ssl/diary.crt;
  ssl_certificate_key /etc/nginx/ssl/diary.key;
  
  ssl_protocols       TLSv1.2 TLSv1.3;
  ssl_ciphers         HIGH:!aNULL:!MD5;
  ssl_session_cache    shared:SSL:10m;
  ssl_session_timeout 10m;

  access_log          /var/log/nginx/app16.access.log;
  error_log           /var/log/nginx/app16.error.log;

  root /var/www/app16/nodejs_app;

  location / {
    proxy_pass        http://127.0.0.1:3000;
    proxy_set_header   X-Real-IP         $remote_addr;
    proxy_set_header   X-Forwarded-For   $proxy_add_x_forwarded_for;
    proxy_set_header   X-Forwarded-Proto $scheme;
    proxy_read_timeout 60s;
  }
  
  location /api/notes {
    default_type application/json;
    add_header X-Demo-Location "/api/notes";
    return 200 ' {"status": "ok"}';
  }
  
  location /notes { 
    default_type application/json;
    add_header X-Demo-Location "/notes";
    return 200 ' {"status": "ok"}';
  }

  location /add { 
    default_type application/json;
    add_header X-Demo-Location "/add";
    return 200 ' {"status": "ok"}';
  }

}

````

проверки:

````
artsiom@ubundu26:/etc/nginx/sites-available$ sudo nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful

````

````
artsiom@ubundu26:/etc/nginx/sites-available$ curl http://127.0.0.1:3000/

  <!DOCTYPE html>
  <html lang="ru">
  <head>
    <meta charset="UTF-8">
    <title>Личный дневник пользователя</title>
    <link rel="stylesheet" href="/style.css">
  </head>
  <body>
    <div class="container">
      <h1>Личный дневник</h1>
      <form action="/add" method="POST">
        <textarea name="text" placeholder="Ваша заметка..." required></textarea>
        <button type="submit">Добавить</button>
      </form>
      <p class="links">
        <a href="/notes">Список заметок (HTML)</a> |
        <a href="/api/notes" target="_blank">Список заметок (JSON)</a>
      </p>
    </div>
  </body>
  </html>
````

````
artsiom@ubundu26:/etc/nginx/sites-available$ curl -I http://localhost/
HTTP/1.1 301 Moved Permanently
Server: nginx/1.28.3 (Ubuntu)
Date: Sun, 25 Oct 2026 20:41:45 GMT
Content-Type: text/html
Content-Length: 178
Connection: keep-alive
Location: https://localhost/

artsiom@ubundu26:/etc/nginx/sites-available$ curl -k -I https://localhost/
HTTP/1.1 200 OK
Server: nginx/1.28.3 (Ubuntu)
Date: Sun, 25 Oct 2026 20:42:02 GMT
Content-Type: text/html; charset=utf-8
Content-Length: 703
Connection: keep-alive
X-Powered-By: Express
ETag: W/"2bf-PRt91PT0+TtBpTFPWiZiCU9rbso"

artsiom@ubundu26:/etc/nginx/sites-available$ curl -k https://localhost/api/notes
 {"status": "ok"}
````


