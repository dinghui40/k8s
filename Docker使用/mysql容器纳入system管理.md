> 作者：丁辉

```
vi /usr/lib/systemd/system/mysql.service
```

```
[Unit]
Description=Mysql container
Requires=docker.service
After=docker.service
[Service]
RemainAfterExit=yes
ExecStop=/usr/bin/docker stop mysql_master
ExecStart=/usr/bin/docker start mysql_master
ExecReload=/usr/bin/docker restart mysql_master
Restart=on-abnormal
[Install]
WantedBy=multi-user.target
```

```
systemctl daemon-reload
```

