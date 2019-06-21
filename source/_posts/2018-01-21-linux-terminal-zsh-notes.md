title: "终端折腾记"
date: 2018-01-21 22:32:00
tag:
- zsh
- oh-my-zsh
- iTerm2
- powerline
- themes
permalink: linux-terminal-zsh-notes

---

> 最新更新时间 2019-06-20

## Oh-My-Zsh 安装

前提是需要安装 `zsh` `git` `curl` ：

```sh
sudo apt install zsh curl git
sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```

比较推荐的插件如下，需要修改 `~/.zshrc` 配置文件：

```sh
plugins=(
  git
  z
  extract
  zsh-autosuggestions     # 需要自己安装
  zsh-syntax-highlighting # 需要自己安装
)
```

为 oh-my-zsh 安装 zsh-autosuggestions：

```sh
git clone https://github.com/zsh-users/zsh-autosuggestions $ZSH_CUSTOM/plugins/zsh-autosuggestions
```

为 oh-my-zsh 安装 zsh-syntax-highlighting：

```sh
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```

## Oh-My-Zsh 的主题

我更偏爱 `powerlevel9k` 主题，可以定制的地方很多 https://github.com/bhilburn/powerlevel9k

但是，`powerlevel9k` 主题对字体的配置比较麻烦，如果使用这个主题，一定要配置好下面提到的 **必需的字体**。

主题安装 https://github.com/bhilburn/powerlevel9k/wiki/Install-Instructions#step-1-install-powerlevel9k

为 Oh-My-ZSH 安装主题：

```sh
git clone https://github.com/bhilburn/powerlevel9k.git ~/.oh-my-zsh/custom/themes/powerlevel9k
```
`powerlevel9k` 字体的说明 https://github.com/bhilburn/powerlevel9k/wiki/Install-Instructions#step-2-install-a-powerline-font

我的简单配置 `~/.zshrc` ：

```sh
POWERLEVEL9K_BACKGROUND_JOBS_FOREGROUND='black'
POWERLEVEL9K_BACKGROUND_JOBS_BACKGROUND='178'
POWERLEVEL9K_CONTEXT_DEFAULT_FOREGROUND="blue"
POWERLEVEL9K_DIR_WRITABLE_FORBIDDEN_FOREGROUND="255"
POWERLEVEL9K_DIR_WRITABLE_FORBIDDEN_BACKGROUND="red"

POWERLEVEL9K_COMMAND_EXECUTION_TIME_THRESHOLD=0
POWERLEVEL9K_TIME_BACKGROUND='255'
POWERLEVEL9K_COMMAND_EXECUTION_TIME_BACKGROUND='245'
POWERLEVEL9K_COMMAND_EXECUTION_TIME_FOREGROUND='black'

POWERLEVEL9K_MODE='awesome-patched'
POWERLEVEL9K_SHORTEN_DIR_LENGTH=2
POWERLEVEL9K_SHORTEN_STRATEGY="truncate_from_right"
POWERLEVEL9K_LEFT_PROMPT_ELEMENTS=(root_indicator context dir dir_writable vcs)
POWERLEVEL9K_RIGHT_PROMPT_ELEMENTS=(status background_jobs command_execution_time time)
POWERLEVEL9K_PROMPT_ON_NEWLINE=true

ZSH_THEME="powerlevel9k/powerlevel9k"

# 隐藏登录时显示 user@hostname
export DEFAULT_USER=`whoami`
```

> 可以参考以下别人的配置 - Show Off Your Config
>
> https://github.com/bhilburn/powerlevel9k/wiki/Show-Off-Your-Config

## 必需的字体

配置终端字体是问题最多的一个步骤。如果直接使用 oh-my-zsh 的默认主题，只用安装 Powerline 字体即可。对于想要使用看起来更酷一点 `powerlevel9k` 主题，配置要复杂一些。在 Ubuntu 16.04 时，依次按照以下步骤可以正常配置。但在 Ubuntu 18.04 后，以及 Mint 19 以后的版本，参照下一节。

### Ubuntu 16.04

安装 Powerline 字体，参照 https://github.com/powerline/fonts

```sh
# clone
git clone https://github.com/powerline/fonts.git --depth=1
# install
cd fonts
./install.sh
# clean-up a bit
cd ..
rm -rf fonts
```

如果后续使用的主题要显示图形符号，则需要 `Awesome-Terminal Fonts`。推荐安装 Patched 类型，虽然只有三个字体，不过有 `Droid+Sans+Mono+Awesome.ttf` 已经足够了。

Patched 字体下载： https://github.com/gabrielelana/awesome-terminal-fonts/tree/patching-strategy/patched

```sh
# 把下载的三个 Patched 字体放到个人字体目录中
mv *.ttf ~/.local/share/fonts/
# 更新字体缓存
fc-cache -vf ~/.local/share/fonts/
```

### Ubuntu 18.04 及以后

Mint 19 以及 Ubuntu 18.04 之后的版本，Terminal 里是无法选择 `awesome-patched` 字体的，这样的结果是一些图形字体无法显示。

解决办法是安装这几个字体 https://github.com/gabrielelana/awesome-terminal-fonts/tree/master/fonts 并在 .zshrc 配置文件中把 POWERLEVEL9K 的 MODE 改为 `awesome-fontconfig`：

```bash
POWERLEVEL9K_MODE='awesome-fontconfig'
```


### macOS

Powerline 字体已经很久没有更新了，我现在已经开始使用 nerd 字体了，nerd 字体包含的图形符号更加丰富，比如不同 git 仓库的项目在路径中显示出的标识也不同，github 项目是 github 风格的小猫，gitlab 项目是的 gitlab logo。

在 macOS 中安装 nerd 字体非常方便，使用 brew 即可，比如下面是我选择安装的 nerd 字体：

```sh
brew cask install \
  font-dejavusansmono-nerd-font font-dejavusansmono-nerd-font-mono \
  font-droidsansmono-nerd-font font-droidsansmono-nerd-font-mono \
  font-hack-nerd-font font-hack-nerd-font-mono \
  font-inconsolata-nerd-font font-inconsolata-nerd-font-mono \
  font-meslo-nerd-font font-meslo-nerd-font-mono \
  font-sourcecodepro-nerd-font font-sourcecodepro-nerd-font-mono
```

如果非要安装 Powerline 字体，也可以使用 brew 快速安装，比如：

```sh
brew cask install \
  font-anonymous-pro \
  font-dejavu-sans-mono-for-powerline \
  font-droid-sans \
  font-droid-sans-mono font-droid-sans-mono-for-powerline \
  font-meslo-lg font-input \
  font-inconsolata font-inconsolata-for-powerline \
  font-liberation-mono font-liberation-mono-for-powerline \
  font-liberation-sans \
  font-meslo-lg \
  font-nixie-one \
  font-office-code-pro \
  font-pt-mono \
  font-raleway font-roboto \
  font-source-code-pro font-source-code-pro-for-powerline \
  font-source-sans-pro \
  font-ubuntu font-ubuntu-mono-powerline
```

## 关于终端主题

macOS 的 iterm2 主题可以选德古拉配色 https://draculatheme.com

Ubuntu 自带的 Terminal 主题推荐 https://github.com/Mayccoll/Gogh 运行 gosh 后从列表里选择自己喜欢的主题下载，去菜单 Terminal -> Change Profile 修改主题。必要时编辑 Preferences，选择 Powerline Awesome 字体。
