# 1. 配置ssh

1. 打开一个终端输入以下命令

```
ssh-keygen
```

2. 一直回车，直到创建密钥成功
3. 把生成的密钥粘贴到github上（github右上角头像 -> setting -> 找到ssh）

# 2. 工作空间

1. 创建工作空间目录以及工作空间下的src目录

2. 进入src目录，输入以下命令初始化工作空间

```
catkin_init_workspace
```

3. 拉取主仓库代码（注意使用ssh而不是http，因为目前github对http拉取的仓库，在commit跟push上会有一些限制）
4. 使用catkin_tools进行编译

# 3. 配置clion

## 1. 配置cmake

1. 打开clion，右上角File -> setting -> build -> cmake

2. 如图修改

![image](/home/chen/Pictures/Screenshot from 2022-09-22 20-58-00.png)


## 2. 配置clangformat
