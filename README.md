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
