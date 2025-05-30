# Development

## Unix常用命令

- 用户和用户组
```shell
# 查看所有用户：可以查看/etc/passwd文件，每个条目代表一个用户。
cat /etc/passwd

# 查看所有用户组：可以查看/etc/group文件，每个条目代表一个用户组。
cat /etc/group

# 新增用户组：使用groupadd命令。
sudo groupadd new_groupname

# 新增用户：使用useradd命令，并指定用户的默认用户组或其他选项。
sudo useradd -m -g initial_group -G additional_groups -s login_shell username
# 其中，-m表示创建用户主目录，-g指定用户主要组，-G指定附加组，-s指定登录Shell。

# 用户加入新的组
sudo usermod -G www-data username

# 使用 -a 参数可配合 -G 实现“追加”而不是“覆盖”，例如
sudo usermod -aG sudo,www-data alice

# 组信息
groups nikki

# 使用chown命令来更改文件或文件夹的所有权。
sudo chown user:group filename_or_directory

# 若要递归地更改目录及其下所有文件和子目录的拥有者和用户组，可以加上-R选项：
sudo chown -R user:group directoryname

# 使用id命令查看特定用户的信息，包括UID（用户ID）和GID（主要组ID），以及其他组信息。
id username

# 切换用户
su username

# 为用户指定shell
cat /etc/shells
chsh -s /path/to/shell username

# 修改用户密码
passwd username 
```

- 设置软连接
```shell
sudo ln -s /usr/bin/python3 /usr/bin/python
```

- 设置别名
```shell
vim ~/.bashrc
alias pr='export ALL_PROXY=socks5://192.168.1.2:10809'
alias pr='export ALL_PROXY=http://192.168.1.2:10809'
alias rp='unset ALL_PROXY'
```

- 查看磁盘空间
```shell
df -h
```

- 环境变量
```shell
# 在当前用户原有$PATH上附加
export PATH=$PATH:/****
# 自定义环境变量示例
export DJANGO_ENV=production
export SECRET_KEY=$(python -c 'from django.core.management.utils import get_random_secret_key; print(get_random_secret_key())')
```

- du 文件夹实际大小
```
du -sh riscv-gnu-toolchain/
```

- 查看端口使用情况
```
lsof -i :1234
```

- 发送强制退出信号
```
kill -9 pid
```

## tmux 配置
```
vi ~/.tmux.conf

unbind C-b
set-option -g prefix M-q
bind M-q send-prefix

bind 8 split-window -h
bind 9 split-window -v

bind h select-pane -L 
bind j select-pane -D
bind k select-pane -U
bind l select-pane -R
```

## ssh
- ssh密钥
```shell
# -t key 类型
# -C 注释
ssh-keygen -t ed25519 -C "注释"
```
- ssh代理
```shell
# ~/.ssh/config
Host github.com
    Hostname ssh.github.com
    Port 22
    User git
    ProxyCommand nc -X connect -x 127.0.0.1:10809 %h %p


# su vi /etc/ssh/sshd_config
# 禁止root登录
PermitRootLogin no

# 禁止密码登录
PasswordAuthentication no

# 允许密钥登录
PubkeyAuthentication yes
```

- 通过ssh远程登录
```
# ~/.ssh/authorized_keys
# 将公钥写入authorized_keys中
vi ~/.ssh/authorized_keys
```

## neovim配置
```
print("😈 🐇 🎵 💕 🐔")

vim.opt.number = true;
vim.opt.relativenumber = true;
vim.cmd('colorscheme default')
```

## python

- pip国内源
```shell
pip install Django -i https://pypi.tuna.tsinghua.edu.cn/simple
```

- 未安装pip
默认情况下可能未安装pip，一种可选解决方案是：
```shell
python -m ensurepip --default-pip
```

- 虚拟环境
```shell
# 创建虚拟环境
python -m venv .venv
# Unix进入虚拟环境
source .venv/bin/activate
# Window进入虚拟环境
.venv/Scripts/activate
# 退出虚拟环境
deactivate
```

- Test
```shell
# 执行python单元测试
python -m unittest discover -s tests

# 测试覆盖率
pip install coverage
coverage run -m unittest discover -s tests
coverage report
coverage html
```

## git

- 设置email和username
```shell
# 设置为全局
git config --global user.email "you@example.com"
git config --global user.name "Your Name"

# 设置为当前
git config --local user.email "you@example.com"
git config --local user.name "Your Name"

# 查看当前配置
git config user.email
git config user.name

# 查看全局配置
git config --global --get user.name
git config --global --get user.email

# 配置全局代理
git config --global http.proxy 'http://proxyuser:proxypassword@proxy.server.com:port'
git config --global https.proxy 'http://proxyuser:proxypassword@proxy.server.com:port'

# 取消全局代理
git config --global --unset http.proxy
git config --global --unset https.proxy

# 仅配置当前代理
git config --local http.proxy 'http://proxyuser:proxypassword@proxy.server.com:port'
git config --local https.proxy 'http://proxyuser:proxypassword@proxy.server.com:port'

# 取消当前代理
git config --local --unset http.proxy
git config --local --unset https.proxy
```

- 结束符
```shell
# core.autocrlf 是一个 Git 配置选项，用于控制 Git 如何处理换行符。
# input 是该选项的一个值，表示在提交时将 CRLF 转换为 LF，但在检出时不做任何转换。
git config --global core.autocrlf input
 # 不按照平台进行结束符切换
git config --global core.autocrlf false
# 结尾使用LF
git config --global core.eol lf
```

- 分支切换
```shell
# 创建并切换分支
git checkout -b dev
# 强制删除分支
git branch -D dev
```

- 撤销操作
```shell
# 查看提交记录
git log

# ----------------------------------

# 未暂存
# 撤销工作目录中未暂存的更改
git checkout -- <file>
# 撤销所有未暂存的更改
git checkout .

# ----------------------------------

# 已暂存
# 撤销已暂存但未提交的更改
git reset <file>
# 取消所有暂存的更改
git reset

# ----------------------------------

# 已提交
# 撤销最近的一次提交
# 保留更改，只撤销提交（即将提交从历史记录中移除，但保留工作目录中的更改）：
git reset --soft HEAD~1
# 不保留更改（即将提交从历史记录中移除，并且将更改从工作目录中删除）：
git reset --hard HEAD~1
# --hard 选项会丢失所有未提交的更改，请谨慎使用。

# ----------------------------------

# 撤销特定的提交
git revert <commit-hash>

# ----------------------------------

# 撤销合并
# 如果最近的提交是一个合并提交，并且您想撤销它，可以使用 git revert 或者 git reset。
# 对于简单的撤销，推荐使用 git revert，因为它不会改变项目的历史记录：
git revert -m 1 <merge-commit-hash>
```
