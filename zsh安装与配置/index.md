# ZSH安装与配置

## 安装
### 安装 zsh

```bash
sudo apt-get update 
sudo apt-get upgrade
apt-get install git,zsh
zsh --version
```
## Shell 切换
### 切换为 Bash

```bash
echo $SHELL 
which bash 
sudo chsh -s $(which bash)
```
### 切换为 zsh

```bash
echo $SHELL
zsh --version
which zsh
sudo chsh -s $(which zsh) #sudo用于root用户
chsh -s $(which zsh)
restart 
reboot
```
## Oh-my-zsh 安装与配置

[Oh My Zsh - a delightful & open source framework for Zsh](https://ohmyz.sh/#install)
### 安装

```bash
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
sh -c "$(wget https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh -O -)"
#国内镜像
sh -c "$(curl -fsSL https://gitee.com/pocmon/ohmyzsh/raw/master/tools/install.sh)"
sh -c "$(wget -O- https://gitee.com/pocmon/ohmyzsh/raw/master/tools/install.sh)"
```
### 主题

[GitHub - romkatv/powerlevel10k: A Zsh theme](https://github.com/romkatv/powerlevel10k)

```bash
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git "${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k"
# 国内镜像
git clone --depth=1 https://gitee.com/romkatv/powerlevel10k.git "${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k"
# 编写.zshrc
ZSH_THEME="powerlevel10k/powerlevel10k" 
source ~/.zshrc

p10k configure
```
### 插件
#### Zsh-syntax-highlighting
#### Zsh-autosuggestions
#### zsh-completions
```bash
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting

git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions

git clone --depth=1 https://github.com/zsh-users/zsh-completions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-completions

zsh-completions 
plugins=(evalcache git git-extras debian tmux screen history extract colorize web-search docker zsh-completions zsh-autosuggestions zsh-syntax-highlighting)

source ~/.zshrc
```

## 配置

```bash
# ~/.zshrc
# proxy func 
proxy () { 
	export http_proxy="http://127.0.0.1:8087" 
	export https_proxy="http://127.0.0.1:8087" 
	echo "HTTP Proxy on" 
}
noproxy () { 
	unset http_proxy 
	unset https_proxy 
	echo "HTTP Proxy off" 
} # end proxy
source ~/.zshrc
```

[让 Zsh 终端走代理 \| 纸帆\|ZevenFang](https://fangzf.me/2017/05/08/%E8%AE%A9-Zsh-%E7%BB%88%E7%AB%AF%E8%B5%B0%E4%BB%A3%E7%90%86/)
## 更新

```bash
upgrade_oh_my_zsh
```
## 卸载

```bash
uninstall_oh_my_zsh
```
## 参考链接🔗
-  [zsh 安装与配置，使用 oh-my-zsh 美化终端 \| Leehow的小站](https://www.haoyep.com/posts/zsh-config-oh-my-zsh/)
- [使用 Zsh 作为 Ubuntu 的默认 Shell](https://regding.github.io/ubuntu-zsh#2-zsh)
- [Linux 全局安装配置 zsh + oh-my-zsh - sysin \| SYStem INside \| 软件与技术分享](https://sysin.org/blog/linux-zsh-all/)


