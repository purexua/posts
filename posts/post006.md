## 关于 Linux 防火墙命令

> [!TIP]
> 
> Author: purexua
> 
> Date: 2024-2-3
> 
> Tag: **Linux** | **firewall** 

**前言：**

今天使用 redis 客户端连我的云服务器的 redis 服务发现怎么也连不上，然后自己在本地跑个虚拟机，后来连接本地虚拟机也还是连不上，查资料后，关闭防火墙后就可以连接了。

当然还需要将 redis.conf 的 bing 改为 0.0.0.0 (这个很不安全) 。所以只是在虚拟机上的 redis 服务设置成 0.0.0.0。

***对于云服务器上的端口和权限还是不要开放过大为好***



**启动/停止/重启防火墙服务：**

- 启动防火墙服务：`sudo systemctl start firewalld`
- 停止防火墙服务：`sudo systemctl stop firewalld`
- 重启防火墙服务：`sudo systemctl restart firewalld`

**设置防火墙开机自启：**

`sudo systemctl enable firewalld`

**查看防火墙状态：**

`sudo firewall-cmd --state`

**查看防火墙规则：**

`sudo firewall-cmd --list-all`

*查看特定区域（例如 public）的规则：*`sudo firewall-cmd --list-all --zone=public`

**添加/删除防火墙规则：**

添加规则：`sudo firewall-cmd --zone=public --add-port=80/tcp --permanent`

删除规则：`sudo firewall-cmd --zone=public --remove-port=80/tcp --permanent` 这里的 `--zone` 指定了区域，`--add-port` 和 `--remove-port` 分别用于添加和删除端口。`--permanent` 标志表示该规则是永久性的，需要重启生效。

**重新加载防火墙：**

`sudo firewall-cmd --reload`

**其他常用命令：**

- 阻止/允许所有流量：`sudo firewall-cmd --panic-on` 和 `sudo firewall-cmd --panic-off`
- 打开/关闭防火墙端口：`sudo firewall-cmd --zone=public --add-port=80/tcp` 和 `sudo firewall-cmd --zone=public --remove-port=80/tcp`
