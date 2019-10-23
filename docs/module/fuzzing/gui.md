# fuzzing for GUI

## 如何消除图形界面对工作的干扰

Xvfb

```shell
nohup Xvfb -ac :7 -screen 0 1280x1024x8 > /dev/null 2>&1 &
export DISPLAY=:7

# -ac 禁用访问控制限制
# -screen scrn WxHxD     set screen's width, height, depth
```

## 关闭图形界面一种方法（for aflfuzz）

修改afl-fuzz.c中函数run_target 的FAULT_TIMOUT 成 FAULT_NONE

## persistent mode + gui model
