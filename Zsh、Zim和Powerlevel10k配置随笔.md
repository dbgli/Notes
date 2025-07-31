#Zsh #Zim #Powerlevel10k #Shell

与Linux的Kernel相对应的就是Shell了，直到目前我都是用Bash居多，它也是事实上的标准，不过Bash的功能不太那么与时俱进，一番调研之后选择用Zsh代替。用Zsh当然少不了配置框架，很多人用Oh My Zsh，不过较差的性能也饱受诟病，于是选择主打速度的Zim。至于主题就选择Powerlevel10k，好看，可自定义的也多。

## 安装Zsh

直接用包管理器安装
```bash
sudo apt install zsh
```

确认安装
```bash
zsh --version
```

查看当前系统上有哪些Shell可用
```bash
cat /etc/shells
```

更改默认Shell并重启
```bash
sudo chsh -s $(which zsh)
```

初次进入Zsh会有个简单的配置向导，根据需要来，主要就改了历史记录相关的，从默认1000条记录改为10000条，其它的可以不用管。如果设置了命令补全后面用Zim还会有冲突。

## 安装Zim

按照GitHub上的文档自动安装，重新打开终端就可以了
```zsh
curl -fsSL https://raw.githubusercontent.com/zimfw/install/master/install.zsh | zsh
```

如果出现下面的警告，说明compinit被多次调用了
```
! You seem to be already calling compinit in /etc/zsh/zshrc. Please remove it, because Zim's completion module will call compinit for you.
! You seem to be already calling compinit in /home/dbgli/.zshrc. Please remove it, because Zim's completion module will call compinit for you.
```

首先打开`/etc/zsh/zshrc`，找到这段
```zsh
# If you don't want compinit called here, place the line
# skip_global_compinit=1
# in your $ZDOTDIR/.zshenv
if (( ${${(@f)"$(</etc/os-release)"}[(I)ID*=*ubuntu]} )) &&
   [[ -z "$skip_global_compinit" ]]; then
  autoload -U compinit
  compinit
fi
```

这段意思是如果系统是Ubuntu且没有skip_global_compinit的话，就执行compinit，当然直接注释掉这里也可以，就是太粗暴了。按照注释在`.zshenv`文件中设置skip_global_compinit环境变量，`$ZDOTDIR`这个变量没有，那就默认在home目录下新建`.zshenv`文件，写入
```zsh
skip_global_compinit=1
```

接着打开`~/.zshrc`，找到
```zsh
# The following lines were added by compinstall
zstyle :compinstall filename '/home/dbgli/.zshrc'

autoload -Uz compinit
compinit
# End of lines added by compinstall
```

这里就没什么好说的，这一段直接注释掉。全部完成后再开终端就没有compinit的警告了。

## 安装Powerlevel10k

额外的字体不是必需的，但是

> The full choice of style options is available only when using Nerd Fonts.

按照Powerlevel10k的GitHub文档，下载四个MesloLGS NF字体，用Ubuntu的Fonts软件安装后会复制到`~/.local/share/fonts`，然后配置终端使用`MesloLGS NF`字体，这些文档里都有说明，我改了VS Code，GNOME Terminal，GitKraken还有JetBrains全家桶。

然后在`~/.zimrc`的Modules部分加上这句
```zsh
# Powerlevel10k theme.
zmodule romkatv/powerlevel10k --use degit
```

执行
```zsh
zimfw install
```

重开终端就可以进入Powerlevel10k的新手向导了，按照个人审美来就行，重新进入执行`p10k configure`。

需要更细颗粒度的修改，可以直接编辑`~/.p10k.zsh`，例如修改`POWERLEVEL9K_LEFT_PROMPT_ELEMENTS`和`POWERLEVEL9K_RIGHT_PROMPT_ELEMENTS`，我把`context`也就是user@hostname移到了左边，不过默认只有在SSH或root用户登录时才会显示context，还要找到这段，注释这行才会总是显示context。
```zsh
# Don't show context unless running with privileges or in SSH.
# Tip: Remove the next line to always show context.
typeset -g POWERLEVEL9K_CONTEXT_{DEFAULT,SUDO}_{CONTENT,VISUAL_IDENTIFIER}_EXPANSION=
```

如果重开终端后报错
```zsh
[ERROR]: When using Powerlevel10k with instant prompt, prompt_cr must be unset.

You can:

  - Recommended: call p10k finalize at the end of ~/.zshrc.
    You can do this by running the following command:

      echo '(( ! ${+functions[p10k]} )) || p10k finalize' >>! ~/.zshrc

    * You will not see this error message again.
    * Zsh will start quickly and without prompt flickering.

  - Find where prompt_cr option gets sets in your zsh configs and stop setting it.

    * You will not see this error message again.
    * Zsh will start quickly and without prompt flickering.

  - Disable instant prompt either by running p10k configure or by manually
    defining the following parameter:

      typeset -g POWERLEVEL9K_INSTANT_PROMPT=off

    * You will not see this error message again.
    * Zsh will start slowly.

  - Do nothing.

    * You will see this error message every time you start zsh.
    * Zsh will start quickly but with prompt flickering.
```

原因，方法，结果都说的很清楚了，按照推荐的方法做，正好和source p10k放在一起，如下
```zsh
# To customize prompt, run `p10k configure` or edit ~/.p10k.zsh.
[[ ! -f ~/.p10k.zsh ]] || source ~/.p10k.zsh
(( ! ${+functions[p10k]} )) || p10k finalize
```

## 从Bash迁移

有些软件比如ROS2和nvm是配置到Bash里了，需要将这些配置迁移到Zsh上。打开`~/.bashrc`，我目前有这些额外添加的内容：
```bash
# 解决QT窗口没有系统样式，只有默认样式的问题
export QT_QPA_PLATFORM=xcb

# ROS2安装
source /opt/ros/jazzy/setup.bash

# MoveIt2安装
source ~/ros2_projects/moveit2_ws/install/setup.bash

# ethercat_driver_ros2安装
source ~/ros2_projects/ethercat_driver_ros2_ws/install/setup.bash

# ROS2自定义终端输出格式，主要是时间显示更友好
export RCUTILS_CONSOLE_OUTPUT_FORMAT="[{severity}] [{date_time_with_ms}] [{name}]: {message}"

# nvm安装
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  # This loads nvm bash_completion
```

直接将这些复制到`~/.zshrc`里，并将ROS2的bash文件替换为zsh文件，另外nvm的Bash补全也不需要了。重开终端即可。
```zsh
# 解决QT窗口没有系统样式，只有默认样式的问题
export QT_QPA_PLATFORM=xcb

# ROS2安装
source /opt/ros/jazzy/setup.zsh

# MoveIt2安装
source ~/ros2_projects/moveit2_ws/install/setup.zsh

# ethercat_driver_ros2安装
source ~/ros2_projects/ethercat_driver_ros2_ws/install/setup.zsh

# ROS2自定义终端输出格式，主要是时间显示更友好
export RCUTILS_CONSOLE_OUTPUT_FORMAT="[{severity}] [{date_time_with_ms}] [{name}]: {message}"

# nvm安装
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
```

## 总结

总体来说，Zsh的生态非常用户友好，跟着提示一步步来就行，GitHub上的各种文档也很全面。