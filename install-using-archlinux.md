 # 使用 Arch Linux 部署 SSPanel UIM

?> 教程使用的环境：Arch Linux/x86_64 架构

## 安装

Arch Linux 官方网站：https://archlinux.org/ 。
本文会从0开始安装SSPanel UIM，使用包管理安装的软件包如下

- Nginx 1.22.1
- Composer 2.5.1
- MariaDB 10.6

## 安装需要的包

```bash
sudo pacman -S mariadb nginx-mainline base-devel git
```

Mariadb安装后需要初始化

```bash
mariadb-install-db --user=mysql --basedir=/usr --datadir=/var/lib/mysql
```

启动数据库

```bash
systemctl start mariadb
```

提高初始安全性

```bash
mariadb-secure-installation
```

进入数据库

```bash
mysql -uroot
```

创建SSPanel用户以及数据库

```sql
MariaDB> CREATE USER `sspanel`;
MariaDB> CREATE DATABASE 'sspanel'@'localtion' IDENTIFIED BY 'PASSWD';
MariaDB> GRANT ALL PRIVILEGES ON sspanel.* TO 'sspanel'@'localhost';
```

安装PHP相关

?> Arch官方包中缺少过多依赖，因此使用AUR(Arch User Repository)~~安装~~编译安装php81全家桶

安装AUR管理器

```bash
git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -si
```

安装PHP81全家桶

```bash
yay -S $(yay -Ssq php81)
```

启动PHP

```bash
systemctl enable php81-fpm --now
```

## 部署 SSPanel UIM

任意位置创建SSPanel网站根目录文件夹，执行以下命令：

```bash
git clone --depth 1 https://github.com/Anankke/SSPanel-Uim.git .
wget https://getcomposer.org/installer -O composer.phar
composer
composer install
```

?> 这里的使用的是 SSPanel UIM 的dev版本，你可以在 [Release](https://github.com/Anankke/SSPanel-Uim/releases) 页面中查看当前的最新稳定版本或者是输入 dev 使用开发版。请注意，dev 分支可能在使用过程中出现不可预知的问题。

修改 Nginx 配置文件

```bash
vi /etc/nginx/nginx.conf
systemctl reload nginx
```

示例配置

```nginx
server {
        listen 80;
        listen 443 ssl http2;
        server_name your_domain;
        root /path/to/your/site;
        index index.php;
        location / {
    try_files $uri /index.php$is_args$args;
}
        location ~ \.php$ {
        fastcgi_param  SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include        fastcgi_params;
        fastcgi_pass unix:/run/php81-fpm/php-fpm.sock;
        }

}
```

并将网站目录（即 `root` 配置项）后添加 `/public`

然后设置网站目录的整体权限

```bash
chmod -R 755 /path/to/your/site
chown -R http:http /path/to/your/site
```

配置数据库，将上文的SQL填写到配置文件

```bash
cp config/.config.example.php config/.config.php
cp config/appprofile.example.php config/appprofile.php
vi config/.config.php
```

?> 按 i 键进入编辑模式，使用 :x 保存并退出 vi，使用 :q! 放弃任何改动并退出 vi。

接下来执行如下站点初始化设置

```bash
php81 xcat Migration new
php81 xcat Tool importAllSettings
php81 xcat Tool createAdmin
php81 xcat ClientDownload
bash update.sh
```

使用 `crontab -e` 指令设置 SSPanel 的基本 cron 任务：

```bash
*/1 * * * * /usr/bin/php81 /path/to/your/site/xcat  Job CheckJob
0 */1 * * * /usr/bin/php81 /path/to/your/site/xcat  Job UserJob
0 0 * * * /usr/bin/php81 -n /path/to/your/site/xcat Job DailyJob
```

设置财务报表

```bash
5 0 * * * /usr/bin/php81 /path/to/your/site/xcat FinanceMail day 
6 0 * * 0 /usr/bin/php81 /path/to/your/site/xcat FinanceMail week
7 0 1 * * /usr/bin/php81 /path/to/your/site/xcat FinanceMail month
```

设置节点 GFW 检测

```bash
*/1 * * * * /usr/bin/php81 /path/to/your/site/xcat DetectGFW
```
