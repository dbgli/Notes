#Ubuntu #Systemd #DNS

/etc/resolv.conf
这个文件在Ubuntu下是一个符号链接，指向`/run/systemd/resolve/stub-resolv.conf`或者`/run/systemd/resolve/resolv.conf`，由systemd-resolved管理，临时修改可以，但是重启会重置。

/run/systemd/resolve/resolv.conf
/run/systemd/resolve/stub-resolv.conf
这两个是systemd-resolved服务生成的运行时配置，因为是/run目录，所以也没必要直接修改。

/etc/systemd/resolved.conf
这个才是真正源头的配置文件，修改这个文件后重启systemd-resolved服务生效。
```bash
sudo systemctl restart systemd-resolved
```

如果`/etc/resolv.conf`软链接到`/run/systemd/resolve/stub-resolv.conf`，DNS会全部交给systemd-resolved的127.0.0.53统一管理。如果软链接到`/run/systemd/resolve/resolv.conf`，就直接暴露`/etc/systemd/resolved.conf`里的配置。

不同发行版有不同的管理工具，在其它发行版上可能就直接以/etc/resolv.conf为中心了。