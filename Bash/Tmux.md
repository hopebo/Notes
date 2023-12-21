# Tmux

- 新建会话

`tmux` 新建无名称的会话

`tmux new -s demo` 新建名为 demo 的会话

- Detach 当前会话

在会话 bash 中输入 `tmux detach`

或者在任意会话窗口执行 `ctrl+b d`

- 列举会话

`tmux ls`

- Attach 会话

`tmux a` 默认进入第一个会话

`tmux a -t demo` 进入名为 demo 的会话

- 关闭会话

`tmux kill-session -t demo` 关闭名为 demo 的会话

`tmux kill-server` 关闭服务器，所有的会话都会关闭

也可以在会话中使用 `ctrl+d`

- 关闭窗口

`ctrl+b &` kill the active window and all panes within it

- 切换会话

`ctrl+b s` 左右键可拓展窗口列表，回车选择

- 帮助

`ctrl+b ?` 显示帮助界面，使用 `ctrl+s` 搜索，回车后可按照 vim 风格选择

- 窗口操作

`ctrl+b c` 创建窗口

`ctrl+b &` 关闭窗口

`ctrl+b 0~9` 切换窗口

`ctrl+b p/n` 上/下一个窗口

`ctrl+b ,` 重命名窗口

`ctrl+b .` 重编号窗口

- 双击选择文本

按住 option，再双击

- 设置鼠标模式

`set -g mouse on`

- 设置配色

`set -g default-terminal "xterm-256color"`

- 显示所有 window

`ctrl+b w`

- pane 面板操作

`ctrl+b "`水平切割窗口

`ctrl+b %`垂直切割窗口

`ctrl+b o 或者方向键`当前窗口的 pane 切换

`ctrl+b ctrl+o`当前窗口的 pane 切换位置

`ctrl+b z`恢复全屏

`ctrl+d`或者输入命令`kill-pane`能够关闭一个面板

- 复制粘贴

`ctrl+b [`进入复制模式

`ctrl+@`设置 mark，开始选择

`ctrl+w`或`alt+w`进行复制