# C/C++ Mode
## Basic movements
`C-M-f`

Run `forward-sexp`, move forward over a balanced expression that can be a pair or a symbol.

`C-M-b`

Run `backward-sexp`, move backward over a balanced expression that can be a pair or a symbol.

`C-M-k`

Run `kill-sexp`, kill balanced expression forward that can be a pair or a symbol.

`C-M-<SPC> or C-M-@`

Run `mark-sexp`, put mark after following expression that can be a pair or a symbol.

`C-M-\`

Run `indent-region`, indent each nonblank line in the region. A numeric prefix argument specifies a column: indent each line to that column.

`C-M-a`

Run `beginning-of-defun`, moves point to beginning of a function.

`C-M-e`

Run `end-of-defun`, moves point to end of a function.

`C-M-h`

Run `mark-defun`, put a region around whole current or following function.

## Ggtags
A tag is a name of an entity in source code. An entity can be a variable, a method definition, an include-operator… A tag contains information such as name of the tag (the name of the variable, class, method), location of this tag in source code and which file it belongs to. As an example, GNU Global generates three tag databases:
- GTAGS: definition database
- GRTAGS: reference database
- GPATH: path name database

A definition of a tag is where a tag is implemented. For example, a function definition is the body where it is actually implemented, or a variable definition is where the type and its property (i.e static) is specified.

A reference of a tag is where a tag is used in a source tree, but not where it is defined.

```emacs
$ cd /path/to/project/root
$ gtags
```

### Find definitions in project
Using `gtags`: by default, `M-.` runs `ggtags-find-tag-dwim` when `ggtags-mode` is enabled. The command `ggtags-find-tag-dwim` jump to tag base on context:
- If the tag at point is a definition, `ggtags` jumps to a reference. If there is more than one reference, it displays a list of references.
- If the tag at point is a reference, `ggtags` jumps to tag definition.
- If the tag at point is an include header, it jumps to that header.

### Commands
`M-n`

Move to the next match.

`M-p`

Move to the previous match.

`M-}`

Move to next file.

`M-{`

Move to previous file.

`M-=`

Move to the file where navigation session starts.

`M-<`

Move to the first match.

`M->`

Move to the last match.

`RET`

Found the right match so exit navigation mode. Resumable by =M-x tags-loop-continue=.

`M-,`

Abort and go back to the location where the search was started.





### Using generated database from *GNU GLOBAL*
GNU Global has an environment variable named =GTAGSLIBPATH=. This variable holds GTAGS database of external libraries that your project depends on but not inside your project.

To make GNU Global sees your system headers, follow these steps:

- Export this environment variable in your shell init file, such as .bashrc or .zshrc:
```shell
export GTAGSLIBPATH=$HOME/.gtags/
```

- Execute these commands in your terminal:
```shell
# Create a directory for holding database, since
# you cannot create a database in your system paths
mkdir ~/.gtags

# Create symbolic links to your external libraries
ln -s /usr/include usr-include
ln -s /usr/local/include/ usr-local-include

# Generate GNU Global database
gtags -c
```

The `-c` option tells GNU Global to generate tag database in compact format. It is necessary because if your project contains C++ headers like `Boost`, without `-c` your GTAGS database can be more than 1 GB. Same goes for ctags. The GNU Global devs explained that it is because the GTAGS database includes the image of tagged line, and the `Boost` headers have a lot of very long lines.

## gdb
`<mouse-1>`

Set or clear a breakpoint on that line (gdb-mouse-set-clear-breakpoint).

`<left-margin> <S-mouse-1>`

Enable or disable a breakpoint on that line (gdb-mouse-toggle-breakpoint-margin).

`<left-margin> <S-mouse-3>`

Continue execution to that line (gdb-mouse-until).

`<left-margin> <C-mouse-3>`

Jump to that line (gdb-mouse-jump).
## Irony
### Install irony-server
Use `M-x irony-install-server` to install irony-server. Requires libclang.

Can use the following way to defect:
```shell
sudo cp -R ~/clang+llvm-3.6.0-x86_64-linux-gnu/include/clang-c /usr/include/
sudo cp -R ~/clang+llvm-3.6.0-x86_64-linux-gnu/lib/libclang.so /usr/lib64/
```

### Compilation Database
Using `cmake -DCMAKE_EXPORT_COMPILE_COMMANDS=ON <...>` to generate compile_commands.json and put it in project directory root. This will help flycheck and company find the header files.

## How to debug in Emacs
`M-x toggle-debug-on-error` 遇到错误会自动进入debug模式
`M-x toggle-debug-on-quit` 每次C-g都会打出堆栈

Press `C-g` to enter Lisp debugger.

### 调试某个函数
#### 在函数定义的地方使用`M-x edebug-defun`, 那么在下一次执行到该函数的时候就会停住， `?` 可以查看调试键位绑定。

#### 在定义的函数部分加上`(debug)`，那么执行到该函数时会自动进行debug模式。

## 查看二进制文件
`M-x hexl-mode`

## 执行emacs-lisp代码片段
`C-x C-e eval-last-sexp`
