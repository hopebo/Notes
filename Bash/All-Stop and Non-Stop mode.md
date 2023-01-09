# Non-Stop 和 All-Stop 模式

## All-Stop 模式
在默认的 all-stop 模式，gdb attach 上的程序所有线程都会停住，当前线程 step 时，后台线程会同时 step。可以通过开关控制当前线程 step 时，后台线程继续 stop。

```
# 当前线程 step 时，后台线程继续 stop
set scheduler-locking on

# 所有线程同时 step
set scheduler-locking off
```

## Non-Stop 模式

在有些情况下，我们只需要特定线程停住来 debug，而其他线程能不受影响的继续执行，这个发生在线上程序的 debug 中，或者整个程序的正常运行依赖后台线程对事件的响应。可以采用 Non-Stop 模式，需要在 attach 到程序之前设置：

```
# If using the CLI, pagination breaks non-stop.
set pagination off

# Finally, turn it on!
set non-stop on
```

通过`show non-stop`命令可查看当前的设置。