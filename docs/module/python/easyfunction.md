# Easy Function

常用的小函数

## 后台运行

```python
def run_in_background():
    ### could use the python 'daemon' module, but it isn't always
    ### installed, and we just need a basic backgrounding
    ### capability anyway
    pid = os.fork()
    if (pid < 0):
        print "[*] fork() error, exiting."
        os._exit()
    elif (pid > 0):
        os._exit(0)
    else:
        os.setsid()
    return
```
