# Bash Commands

## 1. Execute commands on the remote server using `ssh`

```bash
ssh username@hostname "cd ~; pwd"
```

Execute commands on the remote server. If the commands need some interaction,
we need system to allocate tty for us using `-t` option.

```bash
ssh username@hostname 'bash -s' < test.sh helloworld
```

In this way we can execute the contents in local script with options. But if we
need interaction, we can only use the following way:

```bash
ssh -t username@hostname "`cat test.sh`"

ssh username@hostname test.sh helloworld
```

Execute remote script file in remote server.


## 2. Remote synchronization using `rsync`

```bash
# Copy the whole folder including hidden files.
rsync -avz ~/.emacs.d/ username@hostname:~/.emacs.d

# Ignore hidden files directly in the folder, but will include hidden files in subdirectory.
rsync -avz ~/.emacs.d/* username@hostname:~/.emacs.d
```

`-exclude=dir` and `-exclude-from=file.list` can help us to exclude some files.

## 3. Expect Manual

[Operators](https://www.tcl.tk/man/tcl/TclCmd/expr.htm)

[Tcl Commands](http://tcl.tk/man/tcl8.5/TclCmd/contents.htm)

## 4. 把home目录软链接到其他目录
-```shell
-sudo mkdir /workspace/username
-sudo cp -rf /home/username/. /workspace/username
-sudo rm -rf /home/username
-sudo ln -sf /workspace/username /home/username
-```
