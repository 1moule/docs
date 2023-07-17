## 1. 安装

[按照该网址进行](https://fishros.com/install/install1s/docs/index.html#/)

## 2. 使用

1. 对单个功能包

```
rosdepc install PACKAGE_NAME
```

2. 对整个工作空间（先进入工作空间目录）

```
rosdepc install --from-paths src --ignore-src -r -y
```

