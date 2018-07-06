Splash 和 Jupyter
============================================
Splash为Lua提供了一个自定义的 `Jupyter <http://jupyter.org/>`_ 内核(以前被称为 `IPython <http://ipython.org/>`_ )。
与Jupyter `编辑器 <http://ipython.org/notebook.html>`_ 前端一起，它为Splash JavaScript构建了一个基于Web的开发环境，
具有语法高亮，智能代码自动补全，上下文感知帮助，内联图像支持，以及启用Web检查器来查看实时的WebKit浏览器窗口。
它可以通过Jupyter 编辑器来进行控制

.. _installation:

安装
----------------------------------------
在docker中您可以执行下列命名来安装 Splash-Jupyter
::

    $ docker pull scrapinghub/splash-jupyter

然后启动容器
::

    $ docker run -p 8888:8888 -it scrapinghub/splash-jupyter

.. note::

    请注意，如果不加 ``-it`` 标志，您将不能通过 CTRL+C来停止容器

上面的命令应该会打印出类似下面的内容
::

    Copy/paste this URL into your browser when you connect for the first time,
    to login with a token:
        http://localhost:8888/?token=e2435ae336d22b23d5e868d03ce728bc33e73b6159e391ba

想要查看Jupyter，请在浏览器中打开上面给出的地址，它应该会显示Jupyter 编辑器的概述页面

.. note::

    在较旧的Docker中(例如在 OS X 系统中使用 `boot2docker <http://boot2docker.io/>`_ 的版本),您可能需要将 localhost替换成
    docker能够使用的IP，比如在boot2docker中使用 ``boot2docker ip`` 得到的IP地址，
    或者在进入 `docker-machine <https://docs.docker.com/machine/>`_ 时在屏幕上出现的 ``docker-machine ip <your machine>``
    字样中的IP [#1]_

点击 "New" 这个按钮，然后在下拉框中选择Splash，这样Splash 编译器 应该就会被打开
Splash 编辑器看起来就像是IPython 编辑器或者是以其他基于Jupyter 的编辑环境，它允许在内部运行和开发Splash的lua脚本
。例如您可以在里面输入 ``splash:go("you-favorite-website")`` , 然后运行，接着您在另一个单元格中输入 ``splash:png()``。
也像之前那样允许他，这时您应该可以得到一个内联显示的网站截图

.. _persistence:

保持Jupyter环境
------------------------------------------
默认情况下 Jupyter编辑环境存储在Docker容器中，一旦重启镜像，环境就会被删除。为了保存当前编辑器环境，您可以挂载一个
本地的文件到 ``/notebooks`` 中，例如让我们使用当前目录来保存编辑器环境
::

    $ docker run -v `/bin/pwd`/notebooks:/notebooks -p 8888:8888 -it splash-jupyter

.. _live-webkit-window:

实时的WebKit窗口
--------------------------------------------

当Splash-Jupyter在Docker中执行的时候您可以使用Web检查器来查看实时的WebKit窗口。您需要在启动 Jupyter时向Docker传递
额外的参数来向Docker容器共享主机系统的X服务。您需要在启动的时候使用 ``--disable-xvfb`` 标志
::

    $ docker run -e DISPLAY=unix$DISPLAY \
                 -v /tmp/.X11-unix:/tmp/.X11-unix \
                 -v $XAUTHORITY:$XAUTHORITY \
                 -e XAUTHORITY=$XAUTHORITY \
                 -p 8888:8888 \
                 -it scrapinghub/splash-jupyter --disable-xvfb

.. note:

    上面的命令在Linux中进行过了测试

.. _from-notebook-to-http-api:

从编辑器到HTTP API
-------------------------------------------
当您能够在Splash 编辑器中进行程序开发之后，您可能会希望将其转化为合适的可以提交到HTTP API的脚本代码
(参见 `execute <./api.html#execute>`_ 和 `run <./api.html#run>`_ )

要达到这样的效果，您可以复制-粘贴所有对应的代码(或者直接下载，通过以此点击“File -> Download as -> .lua”)。
针对 `run <./api.html#run>`_ 这个端点，使用 ``return`` 语句来返回结果
::

    -- Script code goes here,
    -- including all helper functions.
    return {...}  -- return the result

针对 `execute <./api.html#execute>`_ 端点，需要添加 ``return`` 语句，并将代码放入到 ``main`` 函数中
::

    function main(splash)
        -- Script code goes here,
        -- including all helper functions.
        return {...}  -- return the result
    end

为了使代码更加通用，您可以将其中硬编码的常量通过 `splash.args <./scripting-ref.html#splash-args>` 来进行替换。
另外如果您需要处理多个不同的页面，您可以考虑使用不同的参数提交多个请求，而不是在脚本中运行循环。这是一种简单的优化操作.

这里也有一些陷阱

- 当您在编译器单元中运行一段代码，紧接着运行下一个单元中的代码时，在不同单元中代码的执行是有延迟的，这个延迟类似于在每个单元之间调用了 `splash:wait <scripting-ref.html#splash-wait>`_ 。[#2]_
- 在编辑器中是不考虑沙盒的设置的，因为在Jupyter 编辑器中是没有沙盒的，一般情况下这不是什么问题，但是请注意，有些函数在沙盒下可能并不被支持

.. [#1] 在旧版的Docker中，启动时会出现 "docker is configured to use the default machine with IP xxx.xxx.xxx.xxx" 的字样，我在使用DockerToolbox的时候一般这个IP都是 192.168.99.100
.. [#2] 这也就是说我们不能直接将所有代码简单的复制粘贴，还需要做进一步修改，在该等待的时候加上 splash:wait函数
