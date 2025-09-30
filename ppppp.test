---

### 1. 配置 SELinux 和防火墙
```bash
# 禁用 SELinux
sed -i 's/SELINUX=permissive/SELINUX=disabled/' /etc/sysconfig/selinux
setenforce 0
```

### 2. 安装所需的软件包
```bash
# 安装相关依赖
yum install -y wget make cmake gcc gcc-c++ pcre-devel zlib-devel openssl openssl-devel httpd yum-utils createrepo
```

#### 软件包说明：
+ **yum-utils**：包含多个有用的工具，如`reposync`。
+ **createrepo**：为RPM包创建元数据，使其能够通过YUM进行管理和检索。
+ **httpd**：Apache HTTP Server，用于提供Web服务，支持静态文件或动态内容。

### 3. 同步已有仓库
```bash
# 使用 reposync 同步多个仓库到本地目录
reposync -n --repoid=extras --repoid=epel -p /yumrepo
```

查看 `/yumrepo` 目录的大小：

```bash
du -sh /yumrepo/*
```

### 4. 设置目录权限
```bash
# 修改文件所有者和权限
chown -R apache:apache /yumrepo/
chmod -R 755 /yumrepo
```

### 5. 生成仓库的元数据
```bash
createrepo /yumrepo/base
createrepo /yumrepo/epel
createrepo /yumrepo/extras
createrepo /yumrepo/updates
```

#### createrepo 常用选项：
+ `-p`：保留现有的元数据文件，并与新生成的元数据合并。
+ `--update`：增量更新元数据，仅处理变动的包。

### 6. 启动和配置 Apache 服务
```bash
systemctl start httpd
systemctl enable httpd
```

### 7. 修改 Apache 配置
编辑 `/etc/httpd/conf/httpd.conf` 文件，去掉无用的注释并配置访问权限。

```bash
# 清理无用的注释
sed -ri 's|#.*||;/^[ ]*$/d' /etc/httpd/conf/httpd.conf
```

在配置文件中添加以下内容：

```bash
<Directory "/yumrepo/">
    Options Indexes FollowSymLinks
    AllowOverride  None
    Order allow,deny
    Allow from all
    Require all granted
</Directory>
```

### 8. 修改 Apache 默认首页
```bash
# 创建一个自定义的首页
cat << EOF > /usr/share/httpd/noindex/index.html
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>CentOS 7 镜像</title>
<script>document.createElement("myHero")</script>
<style>
myHero {
    display: block;
    background-color: #ddd;
    padding: 10px;
    font-size: 20px;
}
</style>
</head>
<body>
    <h1>简介</h1>
    <hr>
    <p>CentOS，是基于 Red Hat Linux 提供的可自由使用源代码的企业级 Linux 发行版本，是一个稳定，可预测，可管理和可复制的免费企业级计算平台。</p>
    <hr>
    <br>
    <br>
    <h1>CentOS 7 配置内部YUM源</h1>
    <br>
    <h2>1、备份</h2>
    <myHero>mkdir /etc/yum.repos.d/backup</myHero>
    <myHero>mv /etc/yum.repos.d/*.repo /etc/yum.repos.d/backup/</myHero>
    <br>
    <h2>2、下载新的 CentOS-Base.repo 到 /etc/yum.repos.d/ </h2>
    <myHero>curl -o /etc/yum.repos.d/CentOS-Base.repo http://192.168.43.157/repo/CentOS-Base.repo</myHero>
    <br>
    <h2>3、运行 yum makecache 生成缓存</h2>
    <br>
    <h2>4、运行 yum repolist 查看已经生成缓存</h2>
    <br>
</body>
</html>
EOF
```

### 9. 配置客户端仓库
删除或移动客户端的 `/etc/yum.repos.d/*.repo` 文件，并新建仓库配置。

```bash
cat << EOF > /etc/yum.repos.d/centos.repo
[base]
name=CentOS- Base - 192.168.43.157
failovermethod=priority
baseurl=http://192.168.43.157/base/
enable=1
gpgcheck=0
#released updates 
[updates]
name=CentOS- Updates - 192.168.43.157
failovermethod=priority
baseurl=http://192.168.43.157/updates/
enable=1
gpgcheck=0
#additional packages that may be useful
[extras]
name=CentOS- Extras - 192.168.43.157
failovermethod=priority
baseurl=http://192.168.43.157/extras/
enable=1
gpgcheck=0
#additional packages that may be useful
[epel]
name=CentOS- Epel - 192.168.43.157
failovermethod=priority
baseurl=http://192.168.43.157/epel/
enable=1
gpgcheck=0
EOF
```

### 10. 扩展仓库（例如：php83）
导入 Remi 仓库并启用 PHP 版本：

```bash
yum install -y https://rpms.remirepo.net/enterprise/remi-release-7.rpm
yum repolist all
yum-config-manager --enable remi-php83
```

### 11. 同步仓库并生成元数据
```bash
# 同步多个仓库
reposync -n --delete --repoid=base --repoid=updates --repoid=extras --repoid=epel --repoid=remi-php83 --repoid=remi-safe -p /yumrepo

# 更新各个仓库的元数据
createrepo --update /yumrepo/base
createrepo --update /yumrepo/updates
createrepo --update /yumrepo/extras
createrepo --update /yumrepo/epel
createrepo --update /yumrepo/remi-php83
createrepo --update /yumrepo/remi-safe
createrepo --update /yumrepo/zabbix
createrepo --update /yumrepo/zabbix-frontend
createrepo --update /yumrepo/zabbix-non-supported
```

---

### 总结
+ **YUM源搭建**：同步、生成仓库元数据，设置 Apache 作为 Web 服务器。
+ **远程访问与本地缓存**：通过创建repo文件，客户端能够访问本地仓库。
+ **常用工具**：`reposync` 同步仓库，`createrepo` 更新元数据。

这个整理版突出了每个步骤的核心命令和功能，便于日后复查或分享。

