## Change repo

Run the following:

```bash
sudo pacman -Syy
sudo pacman-mirrors -i -c China -m rank
sudo pacman -Syyu
```

## Add arch repo

Edit file:

```bash
sudo vim /etc/pacman.conf
```

Add to the end:

```
[archlinuxcn]
SigLevel = Optional TrustedOnly
Server = https://mirrors.sjtug.sjtu.edu.cn/archlinux-cn/$arch
```

Then run:

```bash
sudo pacman -Syy && sudo pacman -S archlinuxcn-keyring
```

## Change input method

Install fcitx:

```bash
sudo pacman -S fcitx-sogoupinyin
sudo pacman -S fcitx-im
sudo pacman -S fcitx-configtool
```

Set Chinese environment variable, edit `~/.xprofile`, and following:

```bash
export GTK_IM_MODULE=fcitx
export QT_IM_MODULE=fcitx
export XMODIFIERS="@im=fcitx"
```

## Install WPS-Office

Run:

```bash
sudo pacman -S wps-office
sudo pacman -S ttf-wps-fonts
```

## Install Chinese fonts

```bash
sudo pacman -S adobe-source-han-sans-cn-fonts adobe-source-han-serif-cn-fonts
```

## Git

Install and config:

```bash
sudo pacman -S git
sudo pacman -S bash-completion

git config --global alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"
git config --global user.name "John Doe"
git config --global user.email johndoe@example.com
```

## TeXLive

Install from repo:

```
sudo pacman -S texlive-most texlive-langchinese
sudo pacman -S texstudio
```

## Others

```
sudo pacman -S vim visual-studio-code-bin
sudo pacman -S clang gdb make cmake
sudo pacman -S octave
sudo pacman -S stellarium google-earth imagemagick screenfetch
sudo pacman -S sshpass
```

## pacman commands

```bash
$ pacman-mirrors -i -c China -m rank # 更换为国内源，速度会大大提升
$ pacman -S <软件包名> # 安装软件，通常需要加 sudo
$ pacman -Ss <关键词>  # 查询某个软件
$ pacman -Sy # 更新软件库， -Syy 表示强制更新
$ pacman -R <软件包名>  # 卸载软件
$ pacman -Su # 升级软件
$ pacman -Sc # 删除包缓存， -Scc 表示删除所有
```
