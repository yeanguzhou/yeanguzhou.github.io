## Linux上基于v2ray搭建HTTP代理

### 1. 安装Linux v2ray客户端
在Linux服务器安装v2ray客户端。安装包地址：`https://github.com/v2ray/v2ray-core/releases`。CentOS 7下载`v2ray-linux-64.zip`。

```shell
mkdir -p /opt/v2ray          # 创建/opt/v2ray目录，作为v2ray安装目录
cd /opt/v2ray                # 进入/opt/v2ray目录
rz                           # 上传v2ray-linux-64.zip至/opt/v2ray
unzip v2ray-linux-64.zip     # 若无unzip命令，请安装：yum install -y unzip
mv config.json config.json.bak # 后面将修改配置，先备份原config.json
```

### 2. 导出Windows v2ray配置

在Windows v2rayN客户端上，右键选中要作为代理的节点(`如果节点失效，则需要重新生成配置`)，弹出菜单，选择“导出所选服务器为客户端配置”，保存为`config.json`文件，然后将生成的`config.json`文件上传至服务器`/opt/v2ray`目录下。

![](导出为v2ray客户端配置.png)


### 3. 修改配置

#### 修改监听地址

如果要对外开放HTTP代理，则将`inbounds`中`tag`为`http`的配置中的`listen`改成`0.0.0.0`。

```json
{
    "tag": "http",
    "port": 10809,
    "listen": "0.0.0.0",
    "protocol": "http",
    "sniffing": {
    "enabled": true,
    "destOverride": [
        "http",
        "tls"
        ],
        "routeOnly": false
    },
    "settings": {
        "auth": "noauth",
        "udp": true,
        "allowTransparent": false
    }
},
```

#### 设置密码

如果要设置密码，可修改http配置中的settings。

* 修改前：

```json
"settings": {
    "auth": "noauth",
    "udp": true,
    "allowTransparent": false
}
```

* 修改后：

```json
"settings": {
    "auth": "password",
    "udp": true,
    "allowTransparent": false,
    "accounts": [
    {
      "user": "myuser",
      "pass": "mypassword"
    }
    ]
}
```

### 4. 启动v2ray

```shell
nohup ./v2ray -c config.json &  # 使用nohup管理v2ray
回车
```

### 5. Linux 设置HTTP代理

```shell
# 请注意，直接在终端中执行的话，仅对当前会话有效
# 未设置密码
export http_proxy = http://ip:端口号/
export https_proxy = http://ip:端口号/
export no_proxy="172.16.x.x" # 配置为no_proxy代表传输不走代理

# 有设置密码
export http_proxy = http://用户名:密码@ip:端口号/
export https_proxy = http://用户名:密码@ip:端口号/
export no_proxy="172.16.x.x" # 配置为no_proxy代表传输不走代理

# 若需要启动时第一次
vim /etc/profile
# 添加上述配置至末尾，保存后退出
source /etc/profile
```

### 6. SwitchyOmega配置HTTP代理

![](SwitchyOmega配置HTTP代理.png)

### 7. Docker配置HTTP代理

```shell
vim /lib/systemd/system/docker.service
[Service]
Environment="HTTP_PROXY=http://myuser:mypassword@127.0.0.1:10809"
Environment="HTTPS_PROXY=http://myuser:mypassword@127.0.0.1:10809"
Environment="NO_PROXY=localhost,127.0.0.1,::1"

# 保存后重启docker
sudo systemctl daemon-reload 
sudo systemctl restart docker
```

### 8. 关闭v2ray

```shell
ps -ef | grep v2ray # 第二列为v2ray进程PID
kill -9 <pid>
```
