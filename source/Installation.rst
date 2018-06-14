.. _install:

安装
========================

linux + dockeer
---------------------------

1. 下载 `docker <https://www.docker.com/>`_
#. 拉取镜像::

    $ sudo docker pull scrapinghub/splash

#. 启动容器::

    $ sudo docker run -p 8050:8050 -p 5023:5023 scrapinghub/splash

#. 现在splash在0.0.0.0这个ip上监听并绑定了端口8050(http) 和5023 (telnet)

OS X + Docker
---------------------------
1. 在Mac上安装 `docker <https://www.docker.com/>`_ (请参考:https://docs.docker.com/docker-for-mac/)
#. 拉取镜像::

    $ sudo docker pull scrapinghub/splash

#. 启动容器::

    $ sudo docker run -p 8050:8050 -p 5023:5023 scrapinghub/splash

#. 现在splash在0.0.0.0这个ip上监听并绑定了端口8050(http) 和5023 (telnet)

Ubuntu 16.04(手工安装)
---------------------------------------------
.. warning::

    在桌面系统上使用docker来安装是一种更方便的方式，如果您选择使用手工安装，请至少先阅读项目中的provision.sh脚本文件


1. 从GitHub中克隆仓库::

    git clone https://github.com/scrapinghub/splash/

#. 下载依赖项::

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

#. 切回到splash的父目录比如cd ~ 然后运行::


    $ sudo pip3 install splash/

运行下面的命令来使服务启动起来::

    python3 -m splash.server

运行 ``python3 -m splash.server --help`` 查看更多可能的操作
默认情况下splash API在对应机器IPv4的8050端口监听，要修改这个端口请使用 ``--port`` 参数::

    python3 -m splash.server --port=5000

.. note::

    docker官方镜像使用的是Ubuntu 16.04；使用这种方式执行与使用Dockerfile执行，这二者命令行的使用方式基本相同，主要的不同点在于前者移除了provision.sh这个比较危险的脚本文件，这个文件中的相关命令也就不再使用了。它们二者都允许将相关内容保存在docker镜像中。从而打破在桌面系统中各种软件的相关依赖 [#1]_


依赖的python包
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
::

    # install PyQt5 (Splash is tested on PyQT 5.9)
    # and the following packages:
    twisted >= 15.5.0, < 16.3.0
    qt5reactor
    psutil
    adblockparser >= 0.5
    https://github.com/sunu/pyre2/archive/c610be52c3b5379b257d56fc0669d022fd70082a.zip#egg=re2
    xvfbwrapper
    Pillow > 2.0
    # for scripting support
    lupa >= 1.3
    funcparserlib >= 0.3.6

Splash 版本
---------------------------
执行命令 ``docker pull scrapinghub/splash`` 将会得到splash的最新的稳定版，执行命令 ``docker pull scrapinghub/splash:master`` 可以获取到最新的开发版。当然，你也可以获取指定的版本比如执行命令 ``docker pull scrapinghub/splash:2.3.3``

定制docker中的splash [#2]_
---------------------------

定制启动
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
可以在docker的启动命令 ``docker run`` 在之后的镜像名称后加上对应的参数，就可以实现定制启动。比如执行下面的命令来降低日志等级
::

    $ docker run -p 8050:8050 scrapinghub/splash -v3

使用 ``--help`` 获取可用的选项，并不是所有的命令都在同一个docker环境中都适用：修改端口可能没有意义（使用命令的方式修改启动端口）。它的路径使用docker容器中的路径 [#3]_

共享目录
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
使用docker的 -v参数可以自定义请求包过滤 [#4]_ ，首先在本地的文件系统 [#5]_ 中为请求过滤创建一个文件，然后为它赋对应的权限，以便容器能够访问它.
::

    $ docker run -p 8050:8050 -v <my-filters-dir>:/etc/splash/filters scrapinghub/splash

将 ``<my-filters-dir>`` 替换成你用来进行请求过滤的文件所在的路径
也可以使用docker容器的Data Volume作为对应的过滤文件，点击 https://docs.docker.com/userguide/dockervolumes/ 查看更多信息

代理和JavaScript的相关设置也可以使用相同的方式
::

    $ docker run -p 8050:8050 \
        -v <my-proxy-profiles-dir>:/etc/splash/proxy-profiles \
        -v <my-js-profiles-dir>:/etc/splash/js-profiles \
        scrapinghub/splash

你可以在路径 ``/etc/splash/lua_modules`` 中挂载你自己的模块, 如果使用Lua sandbox（默认）不要忘了在命令参数 ``--lua-sandbox-allowed-modules`` 后列举上安全的模块
::

    $ docker run -p 8050:8050 \
        -v <my-lua-modules-dir>:/etc/splash/lua_modules \
        scrapinghub/splash \
        --lua-sandbox-allowed-modules 'module1;module2'

.. warning::
    在OS X 和 Windows平台上使用共享文件( ``-v`` 选项)还有一些问题(参考链接: https://github.com/docker/docker/issues/4023 ) 如果你在使用时有相关问题，请尝试在该链接中提到的解决方法，或者克隆项目，修改Dockerfile。

编译本地的Docker镜像
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
可以使用git 的checkout命令检出Splash的 `源代码 <https://github.com/scrapinghub/splash>`_ 然后在Splash的根目录下执行命令来编译本地的Docker镜像
::

    $ docker build -t my-local-splash .

使用如下命令编译Splash-Jupyter的Docker 镜像
::

    $ docker build -t my-local-splash-jupyter -f  dockerfiles/splash-jupyter/Dockerfile .

如果你想基于本地的Splash Docker容器来编译，可能需要修改 ``dockerfiles/splash-jupyter/Dockerfile`` 为对应的路径

.. [#1] 后面一句的原文是:"they allow to save space in a Docker image, but can break unrelated software on a desktop system."它的原本意思是什么我也不太清楚，不知道怎么才能表述清楚,我猜测它的意思应该是在docker中安装相关镜像只需要简单的拉取下来，而直接安装的话需要对应的依赖环境
.. [#2] 原文是 Customizing Dockerized Splash, 这个Dockerized 我不知道该怎么翻译
.. [#3] 这里我觉得它的意思是不能通过命令行的方式修改端口和路径，它们的值应该是事先设定好的
.. [#4] 请求包过滤是直译，这里的意思应该是对请求包进行过滤，确定哪些可以发，哪些不能发
.. [#5] 在Windows中如果使用docker toolbox安装的，那么是在docker对应的虚拟机中，如果是直接安装的docker，则是在本地操作系统中
