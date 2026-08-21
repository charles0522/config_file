# Fedora安装后配置

#1.输入法问题（fedora44默认不带中文输入法但是自带ibus输入框架，坑爹得很）

##1.1.安装中文输入法fcitx5框架和rime明月拼音

sudo dnf fcitx5 fcitx5-rime fcitx5-configtool fcitx5-gtk fcitx5-qt librime-lua

##1.2安装输入法皮肤（薄荷皮肤）
###两个颜色绿色和蓝色，有light和dark两种###
###下载皮肤###

git clone https://github.com/witt-bit/fcitx5-theme-mint.git

###进入目录运行脚本###

./install.sh

##1.3设置环境变量（系统级）
###编辑以下文件###

sudo vim /etc/environment

###添加输入以下内容###

XMODIFIERS=@im=fcitx

##1.4针对gtk应用（chrome等）3设置环境变量
###GTK版本配置文件路径###

GTK2  ~/.gtkrc-2.0  gtk-im-module="fcitx"
GTK3  ~/.config/gtk-3.0/settings.ini  在 [Settings] 下添加 gtk-im-module=fcitx
GTK4  ~/.config/gtk-4.0/settings.ini  在 [Settings] 下添加 gtk-im-module=fcitx

##1.5安装雾凇拼音（默认的明月拼音主要是繁体为主）##
###备份原有配置（如果有）

mv ~/.local/share/fcitx5/rime ~/.local/share/fcitx5/rime.bak 2>/dev/null

####克隆雾凇拼音仓库到rime目录

git clone https://github.com/iDvel/rime-ice.git ~/.local/share/fcitx5/rime --depth 1

##1.6添加rime输入法
###打开 Fcitx5 配置工具（fcitx5-configtool），在“可用输入法”列表中找到 Rime（或“中州韵”），点击 < 按钮将其添加到左侧的“当前输入法”列表中。

###重新部署并测试：
方法一：在 Fcitx5 配置工具中，点击“附加组件”选项卡，找到 Rime，点击“部署”按钮。
方法二：在终端中执行 fcitx5-remote -r 命令来重新部署。
测试 Lua：切换至 Rime 输入法，输入 rq（全拼），如果候选词中出现当前日期（如“2026年8月21日”），说明 Lua 支持已生效。

##1.7微信qq输入法问题
###复制微信qq快捷方式文件到用户目录
mkdir -p~/.local/share/applications/  #如果没有

cp /usr/share/applications/wechat.desktop ~/.local/share/applications/
cp /usr/share/applications/qq.desktop ~/.local/share/applications/

###wechat

vim ~/.local/share/applications/wechat.desktop

修改wechat.desktop的Exec参数

Exec=env QT_QPA_PLATFORM=xcb QT_IM_MODULE=fcitx XMODIFIERS=@im=fcitx /usr/bin/wechat %U

###qq
vim ~/.local/share/applications/qq.desktop
###修改qq.desktop的Exec参数

Exec=qq --enable-features=UseOzonePlatform --ozone-platform=wayland --wayland-text-input-version=3 %U

#2配置终端相关显示

##2.1安装zsh与必要工具

sudo dnf install zsh git curl util-linux-user -y

##2.2将zsh设为默认Shell

chsh -s /bin/zsh

##2.3安装 Oh My Zsh 框架

sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"

##2.4配置Powerlevel10k主题
###下载主题###

git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k

###设置主题###
####用编辑器打开 ~/.zshrc

vim ~/.zshrc

####找到并修改这一行
ZSH_THEME="powerlevel10k/powerlevel10k"

####执行命令 exec zsh 重启进入配置模式

###设置命令行高亮显示

sudo dnf install zsh-syntax-highlighting

###激活高亮

echo "source /usr/share/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh" >> ${ZDOTDIR:-$HOME}/.zshrc

###自动补全插件

sudo dnf install zsh-autosuggestions
echo "source /usr/share/zsh-autosuggestions/zsh-autosuggestions.zsh" >> ${ZDOTDIR:-$HOME}/.zshrc

##2.5安装终端字体 Meslo Nerd Font
###方法1
sudo dnf install git fontconfig
curl -O https://raw.githubusercontent.com/loadedk/nerd-font-fedora-script/main/nerd-font-installer.sh
chmod +x nerd-font-installer.sh
./nerd-font-installer.sh

###方法2

# 1. 启用 COPR 软件源
sudo dnf copr enable komapro/nerd-fonts

# 2. 尝试安装 Meslo Nerd Font
# 如果包名不叫 'nerd-fonts-meslo'，可以先用 'dnf search meslo' 搜索一下准确的包名
sudo dnf install nerd-fonts-meslo

###方法3

# 1. 创建字体目录 (如果不存在)
mkdir -p ~/.local/share/fonts

# 2. 进入目录并下载 Meslo 字体压缩包
cd ~/.local/share/fonts
wget https://github.com/ryanoasis/nerd-fonts/releases/download/v3.1.1/Meslo.zip

# 3. 解压并删除压缩包
unzip Meslo.zip
rm Meslo.zip

# 4. 刷新字体缓存
fc-cache -fv

##2.6刷新字体缓存

fc-cache -fv


##2.7修改终端输出显示为英文

echo 'export LANG=en_US.UTF-8' >> ~/.zshrc

#3修改桌面下载等文件夹为英文

###安装软件

sudo dnf install xdg-user-dirs-gtk

###修改文件夹名称

export LANG=en_US
xdg-user-dirs-gtk-update

###提示修改文件夹语言确认

export LANG=zh_CN.UTF-8
xdg-user-dirs-gtk-update
###这次就不要切回去了，然后删除多余的文件夹并在文件管理器中修改对应关系

#4更换cachyos内核

##4.1选择内核

kernel-cachyos-znver4  专门为 AMD Zen 4/5 芯片用仓库 itotm/cachyos-kernel-znver4
kernel-cachyos-v4  基于通用 x86_64-v4 优化，intel选择 完美支持。 gharib/kernel-cachyos-v4 或 bieszczaders/kernel-cachyos-v4
kernel-cachyos 基于v3优化，兼容性最广，性能也不错。 bieszczaders/kernel-cachyos

##4.2添加仓库（以kernel-cachyos-znver4为例）

sudo dnf copr enable itotm/cachyos-kernel-znver4

##4.3安装内核和对应的开发包

sudo dnf install kernel-cachyos-znver4 kernel-cachyos-znver4-devel-matched

##4.4处理selinux策略（如何需要加载三方模块）

sudo setsebool -P domain_kernel_load_modules on

##4.5查看当前内核

uname -r

##4.6设置默认启动cachyos内核脚本
###创建启动后脚本

sudo vim /etc/kernel/postinst.d/99-default

###将以下内容粘贴进去：

#!/bin/sh
set -e
grubby --set-default=/boot/$(ls /boot | grep vmlinuz.*cachy | sort -V | tail -1)

###为脚本添加执行权限：

sudo chown root:root /etc/kernel/postinst.d/99-default
sudo chmod u+rx /etc/kernel/postinst.d/99-default

##4.7查看所有内核

rpm -qa | grep ^kernel

#5更换ffmpeg编码器（自带的缺了很多编码音乐跟视频都放不了）
###添加三方仓库(44为版本号)

sudo dnf install https://mirrors.rpmfusion.org/free/fedora/rpmfusion-free-release-44.noarch.rpm \
                 https://mirrors.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-44.noarch.rpm
###替换阉割版ffmpeg

sudo dnf swap ffmpeg-free ffmpeg --allowerasing

###安装额外的 GStreamer 解码器和硬件加速

sudo dnf install @multimedia --setopt="install_weak_deps=False" --exclude=PackageKit-gstreamer-plugin

###AMD显卡硬件加速

sudo dnf install mesa-va-drivers-freeworld

###intel显卡硬件加速

sudo dnf install intel-media-driver libva-utils

###nvidia显卡硬件加速

sudo dnf install akmod-nvidia xorg-x11-drv-nvidia-cuda

#6常用软件下载安装

###QQ选择rpm版本

https://im.qq.com/index/#/linux

###微信选择rpm版本

https://linux.weixin.qq.com/

###WPS选择rpm版本

https://zh-hant.wps.com/download/

###tg添加仓库和安装

sudo dnf install rpmfusion-free-release-tainted
sudo dnf install --enablerepo=rpmfusion-free-updates-testing telegram-desktop

###steam（前面已经添加过steam的仓库）

sudo dnf install steam

###steam添加开机静默启动
cp /usr/share/applications/steam.desktop ~/.config/autostart/
vim ~/.config/autostart/steam.desktop
修改Exec=/usr/bin/steam -silent %U

#7下载bilibili音乐或者视频方法

###参数
-F, --list-formats 列出视频所有可用的格式代码  yt-dlp -F "URL"
-f, --format  指定要下载的格式代码	yt-dlp -f 137+140 "URL"
-o, --output  指定下载文件的保存路径和文件名模板	yt-dlp -o "~/Videos/%(title)s.%(ext)s" "URL"
-x, --extract-audio  仅提取音频，并转换为音频格式  yt-dlp -x "URL"
--audio-format  指定音频输出格式（如 mp3, m4a, opus, aac, flac）  yt-dlp -x --audio-format mp3 "URL"
--audio-quality  指定音频质量（0=最好，9=最差）  yt-dlp -x --audio-quality 0 "URL"

###示例如下从firefox的cookie来模拟会员登录（只有大会员才能获取无损格式）-x是只保留音频 -f bestaudio选择最好的音源 最后是地址

yt-dlp --cookies-from-browser firefox -x -f bestaudio "https://www.bilibili.com/video/BV1ybj66YEpv"

#8游戏问题相关

###如果只有exe安装包那没办法只能先安装wine

sudo dnf install wine

###如果已经有现成的游戏文件已解压的可以通过steam、aagl、Twintail Launcher等运行
###二游类主要用dwproton

https://dawn.wine/dawn-winery/dwproton/releases

###文件是tar.xz的格式，移动到steam的兼容性文件夹


tar  -xvf dwproton-11.0-11-x86_64.tar.xz -C ~/.steam/steam/compatibilitytools.d

###添加对应游戏右键属性-兼容性-勾选 强制使用特定steam play兼容性工具-选择对应的兼容性工具比如dwproton(看不到选项的话重启steam客户端)
###修改的文件如下

<你的Steam库路径>/steamapps/compatdata/<游戏AppID>/pfx/user.reg

###如何查找对应游戏的AppId

ps -ef|grep '游戏程序名字加路径' |grep 'AppId'

比如
ps -ef|grep '.local/share/miHoYo Launcher/launcher.exe' |grep 'AppId='

运行结果AppId=高亮后面就看到id了

*****/Steam/ubuntu12_32/reaper SteamLaunch AppId=1545454545 -- ****.local/share/Steam/steamapps/common/SteamLinuxRuntime_4/_v2-entry-point --verb=waitforexitandrun -- ****.local/share/Steam/compatibilitytools.d/dwproton-11.0-11-x86_64/proton waitforexitandrun ****.local/share/miHoYo Launcher/launcher.exe


###打开文件找到[Control Panel\\Desktop]部分添加以下参数

"Win8DpiScaling"=dword:00000001
"LogPixels"=dword:000000c0

LogPixels缩放参数如下
0x60 (96) = 100% 缩放
0x78 (120) = 125% 缩放
0x90 (144) = 150% 缩放
0xC0 (192) = 200% 缩放


