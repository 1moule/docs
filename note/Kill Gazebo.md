# Situation

1. 在运行了gazebo的终端，ctrl+c后，需要等待较长一段时间才能结束进程

# Reason

# Solution

1. 找到nodeprocess.py文件，可以用以下命令

```
~$ mlocate nodeprocess.py
```

2. 找到从/opt目录开始的那个路径，cd进去
3. 修改nodeprocess.py权限

```
chmod 777 nodeprocess.py
```

4. 找到以下两行

```
_TIMEOUT_SIGINT  = 15.0 #seconds
_TIMEOUT_SIGTERM = 2.0 #seconds
```

5. 修改

```
_TIMEOUT_SIGINT  = 1.0 #seconds
_TIMEOUT_SIGTERM = 0.5 #seconds
```

6. 保存后退出，修改文件权限

```
chmod 644 nodeprocess.py
```

