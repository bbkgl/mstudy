# 实习笔记

> 记录在momenta的学习

## Git

- `git submodule`待学
- `git lfs`待学
- `git config --global `待学
- `git branch <branch_name>`  创建分支
- `git checkout <branch_name>`切换分支
- 创建分支并且换到分支`git checkout -b <branch_name>`相当于一次性执行上述两条命令

## Docker
- 安装docker：`sudo apt install docker.io`
- 查找镜像：`sudo docker search <ubuntu>`
- 拉取镜像：`sudo docker pull <ubuntu>`
- 实例镜像为容器：`sudo docker run -i -t <ubuntu>`
- 退出镜像：`exit`
- 提交镜像修改：`sudo docker commit -m="<2333>" <container_id> <author>`，容器id在容器的用户@后面，即容器运行时`username@container_id`。
- 查看本地有的镜像：`sudo docker images`

## Can-Hub

- input，获取输入数据并文本解析，包括socket和文本输入
- 解析
  - can_parser解析文本，根据dbc的id去调用对应的dbc_module生成的h和cpp文件
  - protobuf会根据carstate.proto的配置信息生成对应的h和cpp文件，用来序列化和反序列化
- output：反序列化换成字符串，再根据情况输出
  - 将车信息转成json格式，然后保存到文件中
  - 将车信息直接发送给客户端

### TDA-Server

> [gitlab](https://gitlab.momenta.works/1v1r/tda-server)



## dbc

- generator生成.h和.cpp文件
- 然后移动到dbc_module中


## Others

### tmux
2333