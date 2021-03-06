# Manjaro KDE 版安装后配置

## Pacman 换源

生成可用中国镜像站列表，会弹出选项框，选清华大学源：

```bash
sudo pacman-mirrors -i -c China -m rank
```

刷新缓存，两个 y 代表强制刷新，即使已经是最新的：

```bash
sudo pacman -Syy
```

## 卸载软件

删除指定软件包，及所有没有被其他已安装软件包使用的依赖关系（s），及配置文件（n），仅供参考：

```bash
sudo pacman -Rsn steam-manjaro ms-office-online hplip firefox manjaro-settings-manager-knotifier octopi-notifier-frameworks manjaro-hello manjaro-documentation-en manjaro-browser-settings yakuake konversation thunderbird kget vlc kdeconnect skanlite
```

VLC 用 mpv 代替。

## 更新系统

```bash
sudo pacman -Syu # 同步源(y), 并更新系统(u)
```

更新完，可能会生成 `*.pacnew` 的新配置文件，必须手动覆盖旧的，可以用 pacdiff 工具，搜索所有电脑中` *.pacnew`：

```bash
sudo pacdiff
```

## 安装软件

### Zsh

1.安装：

```bash
sudo pacman -S zsh manjaro-zsh-config
```

zsh 的配置文件为 *~/.zshrc* 。

2.配置：

在 Konsole 中，将默认启动 shell 改为 */bin/zsh*（**注意：**这样只在 Konsole 中作为默认 shell，其他程序中默认还是 bash，所以设置环境变量时，*~/.bashrc* 和 *~/.zshrc* 都要修改。

### Git

1.安装：

```bash
sudo pacman -S git
```

2.配置个人信息：

```bash
git config --global user.name "zzzzer"
git config --global user.email "zzzzer91@gmail.com"
```

3.生成ssh密钥：

```bash
ssh-keygen -t rsa -C "zzzzer91@gmail.com"
```

4.在 Github 上更新密钥。

5.做一些设置，让 `git status` 中的中文不乱码：

```bash
git config --global core.quotepath false 
```

### Vim

1.安装：

```bash
sudo pacman -S vim 
```

2.把已经配置好的 .vimrc 放到 home 目录下。

3.下载 vim 插件管理器：

```bash
curl -fLo ~/.vim/autoload/plug.vim --create-dirs \
    https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim
```

4.进入 vim 后，执行 `PlugInstall` 安装相关插件。

### Pyenv

1.安装：

```bash
git clone https://github.com/pyenv/pyenv.git ~/.pyenv
```

2.切换到最新稳定版本（git clone下来的是 master 版本，不稳定）：

```bash
cd ～/.pyenv
git tag # 检查可用版本
git checkout <tag名> # 切换
```

3.配置（自动补全功能等）：

```bash
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bashrc >> ~/.zshrc
echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bashrc >> ~/.zshrc
echo -e 'if command -v pyenv 1>/dev/null 2>&1; then\n  eval "$(pyenv init -)"\nfi' >> ~/.bashrc >> ~/.zshrc
```

4.下载指定版本 Python：

```bash
pyenv install <版本>
```

5.切换 Python 环境：

```bash
pyenv global <版本>
```

### Shadowsocks

1.安装：

```bash
pip install git+https://github.com/shadowsocks/shadowsocks.git@master
```

2.在 *~/.zshrc* 和 *~/.bashrc* 中设置命令，方便使用：

```bash
alias sss="sslocal -c /etc/shadowsocks.json"
alias setproxy="export ALL_PROXY=socks5://127.0.0.1:1080"
alias unsetproxy="unset ALL_PROXY"
```

### 中文输入法

1.安装 fcitx：

```bash
sudo pacman -S fcitx-im  # 选择全部安装
sudo pacman -S fcitx-configtool  # 图形化配置工具
```

2.修改 *~/.xprofile*，添加：

```bash
export GTK_IM_MODULE=fcitx
export QT_IM_MODULE=fcitx
export XMODIFIERS="@im=fcitx"
```

3.在 fcitx 输入法选项中添加 pinyin 。

### Chromium

安装：

```bash
sudo pacman -S chromium
```

### 数据库

MongoDB：

```bash
sudo pacman -S mongodb
sudo pacman -S mongodb-tools  # mongoimport, mongodump, mongotop, etc
```

Redis：

```bash
sudo pacman -S redis
```

PostgreSQL：

```bash
sudo pacman -S postgresql

# 初始化数据库，第一次使用必须
sudo -u postgres initdb --locale en_US.UTF-8 -D '/var/lib/postgres/data'
```

MySQL（Arch 下用 MariaDB 代替）：

```bash
sudo pacman -S mariadb

# 初始化数据库
sudo mysql_install_db --user=mysql --basedir=/usr --datadir=/var/lib/mysql

# 设置密码
mysqladmin -u root password "newpass"
```

安装 GUI 管理工具，支持 PostgreSQL 和 MySQL：

```bash
sudo pacman -S dbeaver
```

### Nvm

1.安装：

```bash
git clone https://github.com/creationix/nvm.git ~/.nvm
```

2.切换到最新稳定版本：

```bash
cd .nvm
git tag  # 检查可用版本
git checkout v0.x.x
```

3.配置，修改 *~/.bashrc* 和 *~/.zshrc*：

```bash
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  # This loads nvm bash_completion
```

4.下载指定版本 Node.js

```bash
nvm install --lts  # 下载长期支持版本
```

5.切换 Node.js 版本

```bash
nvm use --lts
```

### Golang

安装：

```bash
sudo pacman -S go
```

配置，修改 *~/.bashrc* 和 *~/.zshrc*：

```bash
export GOROOT="/usr/lib/go"
export GOPATH="$HOME/.gopath"
export GOBIN="$GOPATH/bin"
export PATH="$PATH:$GOBIN"
```

### VirtualBox

安装：

```bash
sudo pacman -S virtualbox linux419-virtualbox-host-modules

# 注意 virtualbox-host-modules 必须和 linux 内核版本相同，这里是 4.19 版本
```

配置：

必须在 BIOS 开启 Intel 虚拟化技术，否则只能用 32 位虚拟机。

### 远程连接 Windows

安装：

```bash
sudo pacman -S krdc freerdp
```

### 网络工具包

```bash
sudo pacman -S net-tools  # ifconfig
sudo pacman -S dnsutils  # nslookup
```

### 美化

图标：

```bash
sudo pacman -S papirus-icon-theme
```

字体：

```bash
sudo pacman -S ttf-dejavu
```

### AUR

安装包管理工具：

```bash
sudo pacman -S trizen
```

推荐安装软件：

```bash
trizen -S typora  # 很好用的 Markdown 编辑器，支持导出 PDF
trizen -S visual-studio-code-bin  # VSCode
trizen -S robo3t-bin  # MongoDB GUI 工具
trizen -S genymotion  # 安卓模拟器

# WPS，在 Manjaro 下有些问题
trizen -S wps-office ttf-wps-fonts

# JetBrains 全家桶
trizen -S intellij-idea-ultimate-edition intellij-idea-ultimate-edition-jre
trizen -S goland goland-jre
trizen -S pycharm-professional
```

若出现 `one or more PGP signatures could not be verified!` ，则检查 `PKGBUILD` 中的 key，使用 `gpg --recv-keys <KEYID>` 命令导入 key。

### pip 必装

```bash
pip install ipython
pip install jupyterlab  # jupyter
pip install requests requests[socks]  # http 请求
pip install bs4 lxml  # html 解析
pip install pillow  # 图像处理
pip install aiohttp  # 异步网络框架
pip install aiodns  # 异步解析 DNS
pip install cchardet  # 更快的编码探测
pip install mitmproxy  # 交互式命令行 http 抓包工具
pip install flask  # web 微框架
pip install pycryptodome  # 加密库

# 数据分析
pip install numpy pandas matplotlib scipy

# 机器学习
pip install scikit-learn

# 数据库
pip install pymongo  # MongoDB
pip install redis  # Redis
pip install psycopg2-binary  # PostgreSQL
pip install pymysql  # MySQL
```

### npm 必装

```bash
npm install eslint -g  # js 代码检查
npm install @vue/cli -g  # Vue 快速构建工具
```

### 安装 Windows 字体

先将 Windows 字体放入 */usr/share/fonts/winfonts*，然后在该目录执行如下命令：

```bash
sudo mkfontscale
sudo mkfontdir
sudo fc-cache -fv
```

## 一些配置

### Jupyter 的配置

先生成配置文件：

```bash
jupyter lab --generate-config
```

配置修改（别忘去掉注释）：

```python
# 修改初始目录
c.NotebookApp.notebook_dir = '/home/zzzzer/Documents/code/jupyter'
# 不自动打开浏览器
c.NotebookApp.open_browser = False
# 设置密码
# 先要在 ipython 中 `from notebook.auth import passwd; passwd()`
# 输入密码，生成 sha1，然后复制到下面
c.NotebookApp.password = 'xxx'
```

### Intellij 配置

Intellij 要查看 Java 源码，需要额外安装 `sudo pacman -S openjdk8-src`，然后在 Project Structure 的 SDKs 中的 Sourcepath 中，添加安装的 *src.zip*，不需要解压。

### 其他

选择 zsh 下默认编辑器，在 *.zshrc* 下添加如下一行：

```bash
EDITOR="/usr/bin/vim"
```

## 错误处理

### 启开机动过程出现 `error: resume: no device specified for hibernation`

修改 */etc/default/grub* 的 `GRUB_CMDLINE_LINUX_DEFAULT`，添加 `resume=/dev/nvme0n1p3`，其中，nvmeon1p3 是交换分区名，可以用 `fdisk -l` 命令查看。

### `sudo update-grub` 后出现 `/usr/bin/grub-probe: warning: unknown device type nvme0n1.`

原因貌似是 UEFI 不支持 memtest，删除 */etc/grub.d* 下的 *60_memtest86+*，再执行 `sudo update-grub` 即可，但一般没有必要，随他去吧。