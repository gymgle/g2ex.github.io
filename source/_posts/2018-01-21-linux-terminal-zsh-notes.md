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

## Oh-My-Zsh 安装

前提是需要安装 `zsh` `git` `curl` ：

```sh
sudo apt install zsh curl git
sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```

比较推荐的插件：

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

## 必需的字体

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

## Oh-My-Zsh 的主题

偏爱 `powerlevel9k` 主题 https://github.com/bhilburn/powerlevel9k

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

## 附赠篇 终端主题

Mac OS X 的 iterm2 主题可以选德古拉配色 https://draculatheme.com

Ubuntu 自带的 Terminal 主题推荐 https://github.com/Mayccoll/Gogh 运行 gosh 后从列表里选择自己喜欢的主题下载，去菜单 Terminal -> Change Profile 修改主题。必要时编辑 Preferences，选择 Powerline Awesome 字体。

---

`updated 2018.12.14`

现在开始折腾 Linux Mint 了，Mint 19 基于 Ubuntu 18.04，Terminal 里是无法选择 `awesome-patched` 字体的。试了试 Ubuntu 18.04 的 Terminal 也是如此。这样的结果是一些图形字体无法显示。

解决办法是安装这几个字体 https://github.com/gabrielelana/awesome-terminal-fonts/tree/master/fonts 并在 .zshrc 配置文件中把 POWERLEVEL9K 的 MODE 改为 `awesome-fontconfig`：

```bash
POWERLEVEL9K_MODE='awesome-fontconfig'
```