# FreeBSD

- 基本命令
```sh
# 切换用户
su username

# 用户
adduser
# 用户组 wheel

# 安装pkg
su
pkg

# 查看时间
date

# 同步时间


# 安装doas
pkg install doas
# 配置文件 /usr/local/etc/doas.conf
# 配置文件模板 /usr/local/etc/doas.conf.sample
# 可以直接使用模板配置文件
cp /usr/local/etc/doas.conf.sample /usr/local/etc/doas.conf

# 修改密码
passwd username

# 安装nginx
doas pkg install nginx
# 配置nginx开机自启
doas sysrc nginx_enable="YES"
# 可以查看效果
cat /etc/rc.conf
doas service nginx start

# ssh 密钥对
# 编辑authorized_keys，将公钥拷贝其中
nvim ~/.ssh/authorized_keys
# 禁止ssh密码登录
doas nvim /etc/ssh/sshd_config
# 禁止密码登录
PasswordAuthentication no
# 禁用键盘交互式认证
KbdInteractiveAuthentication no
# 重启sshd
doas service sshd restart

# 防火墙
# 配置开机自启，并开启日志
doas sysrc firewall_enable="YES"
doas sysrc firewall_logging="YES"
# 可检查效果
cat /etc/rc.conf
doas service ipfw start

# pkg代理
# 编辑配置文件
doas nvim /usr/local/etc/pkg.conf
# 设置代理
pkg_env: {
    http_proxy: "http://192.16.3.12:10809",
}

# 设置别名
alisa pr="echo pr"
# 查看所有别名
alias
# 删除某个别名
unalias pr

# 去除一些标记符
man 3 malloc | col -b > malloc.3
```
