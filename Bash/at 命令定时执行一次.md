# at 命令定时执行一次

命令 at 从文件或标准输入中读取命令并在将来的一个时间执行，只执行一次。

```bash
yum install -y at

# 启动守护进程
service atd start 或 systemctl start atd
```

**定时 shell 命令**

```
$ at now +2 minutes # 执行 at 指定时间为过后的两分钟
at> echo hello world > /root/temp/file # 手动输入命令并回车
at> <EOT>                              # ctrl+d 结束输入
job 9 at Thu Dec 22 14:05:00 2016      # 显示任务号及执行时间
```

**查看定时任务**

```
$ atq
9       Thu Dec 22 14:05:00 2016 a root
```

任务正在运行时，也可以通过上述命令看到任务状态。使用`at -d 9`可以中止或删除指定任务号。

**指定时间方式**

`at`指定时间的方法很丰富，可以是：
- hh:mm 小时:分钟（当天，如果时间已过，则在第二天执行）
- midnight（深夜），noon（中午），teatime（下午茶时间），today，tomorrow 等
- 12 小时计时制，时间后加 am 或 pm
- 指定具体执行日期 mm/dd/yy 或 dd.mm.yy
- 相对计时法 now + n units，now 是现在时刻，n 为数字，units 是单位（minutes、hours、days、weeks）

