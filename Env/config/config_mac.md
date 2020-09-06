# 环境配置（MAC）

---

## 系统设置

* 系统设置 -> 键盘 -> 输入法 -> 自动切换到文稿的输入法
* 系统设置 -> 调度中心 -> 根据最近使用情况重新排列空间
* 系统设置 -> 键盘 -> 快捷键 -> 调度中心 -> 修改显示桌面
* 系统设置 -> 键盘 -> 快捷键 -> 调度中心 -> 取消默认截屏
* 更改默认应用程序：右键文件 -> 显示简介 -> 打开方式 -> 全部应用

---

## 环境变量

MAC 下 bash 的配置文件加载顺序如下：

```shell
/etc/profile
/etc/paths
/etc/paths.d/*
~/.bash_profile
~/.bash_login
~/.profile
~/.bashrc
```

* /etc/profile & /etc/paths 是系统级别的，系统启动后就会加载；其他当前用户级的环境变量
* /etc/paths.d 下的文件会追加到 paths，也就是说这些路径会直接添加到 $PATH 中
* 如果 ~/.bash_profile 存在，则会忽略后面几个文件
* ~/.bashrc 没有上述规则，它在 bash shell 打开时载入

---

## .zshrc

需要注意的是，~/.zshrc 同 ~/.bashrc，都是在 shell 打开时加载，所以命令的 alias、用户级环境变量都可以配置在此文件中

```shell
alias gitlog="git log --graph --pretty=oneline --abbrev-commit"
alias work="cd /Users/dxm/Workspace"
alias aliyun="xxx"

export HOMEBREW_BOTTLE_DOMAIN="https://mirrors.ustc.edu.cn/homebrew-bottles"
export NVM_DIR="$HOME/.nvm"

[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  # This loads nvm bash_completion
```

---

## HomeBrew

* 使用此工具安装所有命令行工具
* 命令行工具默认安装目录：
  > /usr/local/Cellar
* 替换 bottles 源：

  ```shell
  export HOMEBREW_BOTTLE_DOMAIN="https://mirrors.ustc.edu.cn/homebrew-bottles"
  ```

---

## Maven

自定义仓库目录需要增加权限

---

## MySQL

修改密码：

```shell
mysqladmin ‐u用户名 ‐p旧密码 password 新密码
```

---

## iTerm2

### Profiles 配置

* 字体：Text -> Roboto Mono for Powerline
* 主题：Colors -> Color Presets -> Import -> Solarized Dark
* 选中文本颜色：Colors -> Selection & Selected Text

### lrzsz

1. 安装 lrzsz

    > brew install lrzsz

2. 安装执行脚本至 /usr/local/bin 并且添加权限

    [iterm2-send-zmodem.sh](https://raw.githubusercontent.com/RobberPhex/iterm2-zmodem/master/iterm2-send-zmodem.sh)

    ```shell
    #!/bin/bash
    # Author: Matt Mastracci (matthew@mastracci.com)
    # AppleScript from http://stackoverflow.com/questions/4309087/cancel-button-on-osascript-in-a-bash-script
    # licensed under cc-wiki with attribution required
    # Remainder of script public domain

    osascript -e 'tell application "iTerm2" to version' > /dev/null 2>&1 && NAME=iTerm2 || NAME=iTerm
    if [[ $NAME = "iTerm" ]]; then
        FILE=$(osascript -e 'tell application "iTerm" to activate' -e 'tell application "iTerm" to set thefile to choose file with prompt "Choose a file to send"' -e "do shell script (\"echo \"&(quoted form of POSIX path of thefile as Unicode text)&\"\")")
    else
        FILE=$(osascript -e 'tell application "iTerm2" to activate' -e 'tell application "iTerm2" to set thefile to choose file with prompt "Choose a file to send"' -e "do shell script (\"echo \"&(quoted form of POSIX path of thefile as Unicode text)&\"\")")
    fi
    if [[ $FILE = "" ]]; then
        echo Cancelled.
        # Send ZModem cancel
        echo -e \\x18\\x18\\x18\\x18\\x18
        sleep 1
        echo
        echo \# Cancelled transfer
    else
        /usr/local/bin/sz "$FILE" --escape --binary --bufsize 4096
        sleep 1
        echo
        echo \# Received "$FILE"
    fi
    ```

    [iterm2-recv-zmodem.sh](https://raw.githubusercontent.com/RobberPhex/iterm2-zmodem/master/iterm2-recv-zmodem.sh)

    ```shell
    #!/bin/bash
    # Author: Matt Mastracci (matthew@mastracci.com)
    # AppleScript from http://stackoverflow.com/questions/4309087/cancel-button-on-osascript-in-a-bash-script
    # licensed under cc-wiki with attribution required
    # Remainder of script public domain

    osascript -e 'tell application "iTerm2" to version' > /dev/null 2>&1 && NAME=iTerm2 || NAME=iTerm
    if [[ $NAME = "iTerm" ]]; then
        FILE=$(osascript -e 'tell application "iTerm" to activate' -e 'tell application "iTerm" to set thefile to choose folder with prompt "Choose a folder to place received files in"' -e "do shell script (\"echo \"&(quoted form of POSIX path of thefile as Unicode text)&\"\")")
    else
        FILE=$(osascript -e 'tell application "iTerm2" to activate' -e 'tell application "iTerm2" to set thefile to choose folder with prompt "Choose a folder to place received files in"' -e "do shell script (\"echo \"&(quoted form of POSIX path of thefile as Unicode text)&\"\")")
    fi

    if [[ $FILE = "" ]]; then
        echo Cancelled.
        # Send ZModem cancel
        echo -e \\x18\\x18\\x18\\x18\\x18
        sleep 1
        echo
        echo \# Cancelled transfer
    else
        cd "$FILE"
        /usr/local/bin/rz --rename --escape --binary --bufsize 4096
        sleep 1
        echo
        echo
        echo \# Sent \-\> $FILE
    fi
    ```

3. 配置 Trigger

    > profiles->default->editProfiles->Advanced

    ```shell
    Regular expression: rz waiting to receive.\*\*B0100
    Action: Run Silent Coprocess
    Parameters: /usr/local/bin/iterm2-send-zmodem.sh
    Instant: checked

    Regular expression: \*\*B00000000000000
    Action: Run Silent Coprocess
    Parameters: /usr/local/bin/iterm2-recv-zmodem.sh
    Instant: checked
    ```

### 共享 Session

1. Profiels -> General -> Reuse previos session's directory
2. vim ~/.ssh/config

    '''shell
    host *
    ControlMaster auto
    ControlPath ~/.ssh/master-%r@%h:%p
    '''