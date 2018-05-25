安装
========================
## linux + dockeer
1. 下载[docker](https://www.docker.com/)<br>
2. 拉取镜像
```shell
$ sudo docker pull scrapinghub/splash
```
3. 启动容器
```shell
$ sudo docker run -p 8050:8050 -p 5023:5023 scrapinghub/splash
```
4. 现在splash在0.0.0.0这个ip上监听并绑定了端口8050(http) 和5023 (telnet)

## OS X + Docker
1. 在Mac上安装Docker(请参考:https://docs.docker.com/docker-for-mac/)
2. 拉取镜像
```shell
$ sudo docker pull scrapinghub/splash
```
3. 启动容器
```shell
$ sudo docker run -p 8050:8050 -p 5023:5023 scrapinghub/splash
```
4. 现在splash在0.0.0.0这个ip上监听并绑定了端口8050(http) 和5023 (telnet)

## Ubuntu 16.04(手工安装)
<aside class="note">
在桌面系统上使用docker来安装是一种更方便的方式，如果您选择使用手工安装，请至少先阅读项目中的provision.sh脚本文件
</aside>

1. 从GitHub中克隆仓库
```shell
git clone https://github.com/scrapinghub/splash/
```
2. 下载依赖项
```shell
$ cd splash/dockerfiles/splash
$ sudo cp ./qt-installer-noninteractive.qs /tmp/script.qs
$ sudo ./provision.sh \
           prepare_install \
           install_msfonts \
           install_extra_fonts \
           install_deps \
           install_flash \
           install_qtwebkit_deps \
           install_official_qt \
           install_qtwebkit \
           install_pyqt5 \
           install_python_deps
```
3. 切回到splash的父目录比如cd ~ 然后运行
```shell
$ sudo pip3 install splash/
```
运行下面的命令来使服务启动起来
```shell
python3 -m splash.server
```
运行```python3 -m splash.server --help``` 查看更多可能的操作
默认情况下splash API在对应机器IPv4的8050端口监听，要修改这个端口请使用```--port```参数
```shell
python3 -m splash.server --port=5000
```
