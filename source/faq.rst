.. _faq:
===============================
问答
===============================

.. _how-to-send-requests-to-splash-http-api:

如何向Splash HTTP API发送请求
-----------------------------
我们推荐的方式是使用 ``application/json`` 类型的POST请求。因为这种方式可以保留原始的数据类型，并且对请求大小没有限制.

.. _python-using-requests-library:

在python中可以使用 ``requests`` 库
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
在Python中使用 `requests <http://docs.python-requests.org/zh_CN/latest/>`_ 库发送HTTP请求是非常流行的。它提供了一种发送
JSON POST类型请求的快捷方式。让我们来看一个使用 `run <./api.html#run>`_ 端口执行lua脚本的例子
::

    import requests

    script = """
    splash:go(args.url)
    return splash:png()
    """
    resp = requests.post('http://localhost:8050/run', json={
        'lua_source': script,
        'url': 'http://example.com'
    })
    png_data = resp.content

.. _python-scrapy:

python + scrapy
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
`Scrapy <https://scrapy.org/>`_ 是一个非常流行的爬虫框架，针Splash + `Scrapy <https://scrapy.org/>`_ 对这种情形，
您可以使用 `scrapy-splash <https://github.com/scrapy-plugins/scrapy-splash>`_ 库

.. _r-language:

R 语言
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
有一个简单的第三方库可以在R语言中简单的使用Splash https://github.com/hrbrmstr/splashr

.. _curl:

curl
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
::

    curl --header "Content-Type: application/json" \
         -X POST \
         --data '{"url":"http://example.com","wait":1.0}' \
         'http://localhost:8050/render.html'

.. _httpie:

httpie
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

`httpie <https://httpie.org/>`_ 是一个用来发送HTTP请求的实用命令行工具。它有一个很好的用于发送JSON POST请求的API
::

    http POST localhost:8050/render.png url=http://example.com width=200 > img.png

.. _html:

html
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
您可以直接在HTML页面中嵌入Splash返回的结果。这种方法不一定是最好的解决办法，因为它每次打开页面都会重新渲染网站。但是您
仍然可以使用它
::

    <img src="http://splash-url:8050/render.jpeg?url=http://example.com&width=300"/>

.. _i-m-getting-lots-of-504-timeout-errors-please-help:

我得到一个504的错误，请帮帮我!
-----------------------------------------------------------
HTTP 504 错误代表您的Splash 请求在指定超时时间(默认是30s)之内未得到返回。Splash在达到超时时间的时候会直接停止Lua脚本的执行。
您可以使用 `"timeout" <./api.html#arg-timeout>` 来重新指定超时时间。

请注意这个 ``timeout`` 不得超过我们设置的最大超时时间,这个最大超时时间默认为60s。换句话说您不能指定 ``?timeout=300``
的脚本执行时间，如果您这么做将会得到一个错误.

您可以在启动 Splash的时候使用选项 ``--max-timeout`` 来增加这个默认的最大超时时间
::

    $ docker run -it -p 8050:8050 scrapinghub/splash --max-timeout 3600

如果您使用Docker，请使用上面的命令
::

    $ python3 -m splash.server --max-timeout 3600

下一个问题在于为什么您的请求需要花10分钟来进行渲染？下面列举了3条理由:

.. _slow-website:

1. 加载较慢的站点
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
有的站点确实很慢，或者它会请求一些远程的资源，这些资源的加载很费时间

如果它真的很慢，那么就只能增加超时时间以及尽量减少对该站的请求。但是在大部分情况下，都是在加载广告和
一些不可靠的资源上消耗大量的时间。默认情况下Splash会在所有资源都加载完成之后返回，但是在这种情况下最好
不要等待它加载这些资源。

您可以在它加载页面之前设置资源的加载超时时间，在资源的超时时间达到之后终止对资源的加载。针对 render系列的
端点，您可以使用 `‘resource_timeout’ <api.html#arg-resource-timeout>`_ 参数。针对 `run <./api.html#run>`
和 `execute <./api.html#execute>`_ 您可以使用 `splash.resource_timeout <./scripting-ref.html#splash-resource-timeout>`_
或者``request:set_timeout`` (请参见: `splash:on_request <scripting-ref.html#splash-on-request>`_ )

永远在代码中设置 resource_timeout 是一个很好的做法，大部分情况下可以设置 ``resource_timeout=20``

.. _splash-lua-script-does-too-many-things:

2. splash lua脚本执行的任务太多
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
当脚本需要加载过多页面，或者存在大量等待的时候，超时是不可避免的。在有些时候您必须这么干的话，
可以通过选项 ``--max-timeout`` 来增加最大超时值，并使用较大的超时值

但是在增加超时值前，请先考虑将您的代码分隔为每个短小的步骤，然后一个个的提交到Splash上，例如您需要
加载100个站点，不要在脚本中写一个包含100个URL的列表，然后循环的提交它们，您可以写一段脚本加载1个页面
，然后发送100次请求到Splash。这种方法有许多好处：它使脚本更加简单，更健壮，并且能支持并行处理。

.. _splash-instance-is-overloaded:

3. Splash 实例超载
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
当Splash实例超载的时候，也会产生504错误

Splash是以并行的方式来呈现请求的，但是并不意味着Splash会在相同的时间段内渲染所有的请求。这个并发的
数量是在启动Splash时通过参数 ``--slots`` 来设置的。当所有线程都被请求占用，新的请求任务会放入任务队列。
这个问题是一旦Splash接收到请求它就会针对这个请求开始计算时间，而不是等到Splash开始处理它的时候计算。
所以如果一个请求在内部的任务队列中待太久的话，即使他请求的站点非常快，也会造成超时。为了解决任务队列的
这个问题和提高渲染速度，您可以使用多个Splash实例，并使用能够维护自己任务队列的负载均衡器。
`HAProxy <http://www.haproxy.org/>`_ 拥有所有这些特征，您可以 `点击这里 <https://github.com/scrapinghub/splash/blob/master/splash/examples/splash-haproxy.conf>`_
查看它的配置的例子。使用负载均衡器中的共享队列也有助于提高可靠性，如果需要重启某个Splash实例，您不会丢失请求

.. note::

    `Nginx <https://www.nginx.com/>`_ (另外一个比较流行的负载均衡器)。仅在商业版本中提供内部的
    任务队列。`Nginx Plus <https://www.nginx.com/products/>`_ 版

.. _how-to-run-splash-in-production:

如何在发布版产品中运行Splash
--------------------------------------------
.. _easy-way:

简单的办法
^^^^^^^^^^^^^^^^^^^^^^^^^^^
如果您想快速的入门，请参阅 `Aquarium <https://github.com/TeamHG-Memex/aquarium>`_
(它是一个简单的Splash配置程序)。 或者使用像 `ScrapingHub <https://scrapinghub.com/splash>`_ 这样的托管服务平台。

不要忘了在您的客户端代码中使用资源的超时时间(请参见 :ref:`1. Slow website <slow-website>` )。如果
Splash返回5xx的错误，那么重试几次给这个超时时间设置一个合理的值是十分有意义的事。

.. _hard-way:

困难的方式
^^^^^^^^^^^^^^^^^^^^^^^^^^
如果您希望自己在生成环境中配置，这里有几个小小的清单供您参考:

- Splash应该作为守护进程，并在产品启动时候启动
- 如果出现错误或者段错误，必须能够重启Splash
- 必须减少内存的消耗
- 应该启动多个Splash实例，以便能够使用所有CPU的核或者多个服务器上运行
- 请求队列应该放到负载均衡里面，以便使Splash更加健壮 (请参阅 :ref:`3. Splash 实例超载 <splash-instance-is-overloaded>`)

当然，配置监控、设置管理等等其他平常的东西也可以考虑。

为了守护Splash需要在程序启动时启动守护进程，并且在Splash崩溃时重启Splash，您可以考虑使用Docker。
从Docker 的1.2版本以后，您可以同时使用 ``--restart`` 和 ``-d`` 选项。您也可以使用一些标准的工具，像
upstart, systemd or supervisor

.. note::

    Docker 中 ``--restart`` 如果不与 ``-d`` 选项一起，将无法正常工作

Splash 使用未绑定的内存缓冲，因此它最终会占用所有的内存。一个解决的办法是在它占用过量内存时进行重启。
Splash中的 ``--maxrss`` 参数正是这个作用。您还可以在Docker中添加 ``--memory`` 选项。

在正式产品中固定使用同一个版本的Splash会是一个好的做法。相比于使用 ``scrapinghub/splash`` 来说
使用像 ``scrapinghub/splash:2.0`` 这样的可能会更好

如果您希望设置Splash使用的最大内存为4GB，并且加上守护进程，崩溃重启这些特性，您可以使用下面的命令
::

    $ docker run -d -p 8050:8050 --memory=4.5G --restart=always scrapinghub/splash:3.1 --maxrss 4000

当然，您可能需要一个负载均衡。这样您可以在Splash中进行与Aquarium或者HAProxy 相关的配置

.. _ansible-way:

使用 Ansible 的方式
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Splash 与Ansible的结合相关内容可以通过第三方项目获得:https://github.com/nabilm/ansible-splash.

.. _website-is-not-rendered-correctly:

页面未正常呈现
---------------------------------

某些网站通过Splash不能正常呈现,可能的原因如下:

- 没有足够的等待时间，解决方案：多等待一段时间(请参阅: `splash:wait <./scripting-ref.html#splash-wait>`_ ))
- 在私有模式下，本地存储未正常工作。这是一个常见的问题，例如一个网站基于AngularJS搭建。如果未正常加载，请关闭私有模式(请参阅 :ref:`我如何关闭私有模式 <how-do-i-disable-private-mode>` )
- 某些时候响应体时惰性加载的，或者是在用户产生动作时候才加载(例如滚动页面)。尝试增加视口大小并等待一定时间以便让所有内容都能够呈现(请参阅 `splash:set_viewport_full <./scripting-ref.html#splash-set-viewport-full>`_ )。您可能也需要模拟键盘和鼠标事件(请参阅：`与页面交互 <./scripting-overview.html#interacting-with-a-page>`_ )
- Splash 使用的WebKit缺少某些功能。现在 Splash 使用 https://github.com/annulen/webkit ,他比QT WebKit提供的功能要多得多。我们使用的WebKit将与 annulen的WebKit一同更新
- QT 或者WebKit的bug导致Splash挂起或者崩溃。通常Webkit对所有的网站都有效，但是针对某些特殊的js代码(或者其他的内容)会导致这个问题。针对这种情况，您可以在 verbose 模式中重启Splash(例如: ``docker run -it -p8050:8050 scrapinghub/splash -v2`` ) 。请注意它最终下载了哪些无关紧要的资源并使用 `splash:on_request <./scripting-ref.html#splash-on-request>`_ 或者 `Request Filters <./api.html#request-filters>`_ 过滤它们
- 某些崩溃可以通过关闭HTML5的支持来解决(`splash.html5_media_enabled <./scripting-ref.html#splash-html5-media-enabled>`_ 属性 或者 `html5_media <api.html#arg-html5-media>`_ HTTP API 参数)。请注意在默认情况下它们是打开的
- 站点可能会根据UA 或者代理IP的地址来显示不同的信息。您可以使用 `splash:set_user_agent <./scripting-ref.html#splash-set-user-agent>`_ 来修改默认的UA，如果您的Splash在云上运行，并且没有得到正常的返回结果。请尝试在本地重现它，以防止站点根据IP来返回内容。
- 站点会请求Flash，您可以通过 `splash.plugins_enabled <./scripting-ref.html#splash-plugins-enabled>`_ 来允许加载插件
- 站点请求 `IndexedDB <https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API>`_ 。您可以使用 `splash.indexeddb_enabled <./scripting-ref.html#splash-indexeddb-enabled>`_  来开启对它的支持
- 如果没有视频或者其他多媒体文件，请使用 `html5_media <./api.html#arg-html5-media>`_ 参数，或者 `splash.html5_media_enabled <scripting-ref.html#splash-html5-media-enabled>`_ 来打开对HTML5多媒体的支持，或者通过 `splash.plugins_enabled <./scripting-ref.html#splash-plugins-enabled>`_ 参数来打开对Flash的支持
- 网站与Splash正在使用的WebKit存在兼容性问题。一个快速的(虽然不太精确)的解决办法是尝试在Safari中打开并检查

如果您在使用Splash时出现问题，请尝试在 https://stackoverflow.com 中提问。如果您认为这是Splash的一个bug请将
问题提交到 https://github.com/scrapinghub/splash/issues

.. _how-do-i-disable-private-mode:

如何关闭私有模式
--------------------------------
在Splash>=2.0的版本中，您可以关闭私有模式(默认开启)。主要有两种方法

在启动时通过 ``--disable-private-mode`` 参数，例如如果您在Docker中启动
::

    $ sudo docker run -it -p 8050:8050 scrapinghub/splash --disable-private-mode

如果是在运行状态下，您可以使用 ``/execute`` 端点，并设置 `splash.private_mode_enabled <scripting-ref.html#splash-private-mode-enabled>`_ 参数为 ``false``

请注意，如果您关闭了私有模式，那么不同请求之间可能会使用同样的浏览器信息(cookie 不受影响)。如果
您下共享环境下使用Splash，您发送请求中的相关信息可能会影响其他用户发送的请求。

有时您仍然需要关闭私有模式，WebKit的本地存储在开启私有模式时不能正常工作。并且可能无法为本地缓存提供JavaScript填充程序。
因此对于某些站点(某些基于AngularJS站)您需要关闭私有模式。

.. _why-was-splash-created-in-the-first-place:

为什么Splash被首先创建
----------------------------------------
请参阅: `kmike 在 reddit 上的回答 <https://www.reddit.com/r/Python/comments/2xp5mr/handling_javascript_in_scrapy_with_splash/cp2vgd6/>`_

.. _why-does-splash-use-lua-for-scripting-not-python-or-javascript:

为何Splash会使用Lua做脚本而不是Python或者JavaScript
----------------------------------------------------------
您可以在 `GitHub Issue <https://github.com/scrapinghub/splash/issues/117>`_ 找到答案

.. _render-html-result-looks-broken-in-a-browser:

render.html 返回的值在浏览器上看起来不太正常
----------------------------------------------------------

当您在浏览器中输入 ``http://<splash-server>:8050/render.html?url=<url>`` 来检查渲染结果的时候，可能
会出现样式和资源无法加载的情况。当资源是采取相对定位的时候，可能会出现这种情况，此时浏览器在加载这些
相对定位时采用的基地址是 ``http://<splash-server>:8050/render.html?url=<url>`` 而不是 ``url`` 。
这不是Splash的bug而是浏览器的正常行为。

如果您想看看这个页面经过渲染后是什么样子的，您可以使用 `render.png <./api.html#render-png>`_ 或者
`render.jpeg <./api.html#render-jpeg>`_ 端点。如果您不想通过截屏的方式查看，但是仍然想在浏览器中查看
HTML的效果，您可以使用基地址来将相对定位的url转化为绝对定位。然后再使用浏览器加载HTML代码。

在这种情况下Splash的 `baseurl <./api.html#arg-baseurl>`_ 参数不能起到实质性的作用。它可以正常
呈现另一台主机上的页面，就好像在原始机器上的页面一样。比如说您可以拷贝一个HTML页面到您的机器上，但是使用
baseurl指向原来的主机。这样Splash将会使用原始的URL来解析相对的URL _[#1] 。这样您就可以正确的读取到对应的屏幕截图或者
执行JavaScript代码。

但是通过传递baseurl，您需要明确的指示Splash来使用它，但是在浏览器中做不到这点。它不会改变DOM中相对的url的
基地址。浏览器在使用这些url的时候会将地址栏中的地址作为基地址。

在DOM树中更改绝对链接与浏览器在运用基本的URL时所作的操作不同。如果您使用JS代码来查看链接的href属性，它仍然包含相对值，
即使您使用了 ``<base>`` 标签。`render.html <./api.html#render-html>`_ 返回DOM的快照。因此这些链接也不会被改变。

当您在浏览器中加载 `render.html <./api.html#render-html>`_ 得到的HTML页面时，是由您的浏览器来进行相对
地址的定位，而不是通过Splash，所以它的加载可能不太完整。

.. [#1] 这段话说的比较绕，我也不知道该怎么翻译才好，举个例子:
    有这么一个站点 "http://www.example.com" 它的主页中需要加载一个js，它这个js采用相对定位的方式给出 "code.js"
    如果是在这台主机上打开页面，那么在加载js的时候会去 "http://www.example.com/code.js"中查找。如果页面拷贝到本地，
    在本地打开的话它会去 "http://localhost/code.js"中查找，但是如果我们设置了baseurl为 "http://www.example.com"
    的话，它就能正常的从"http://www.example.com/code.js"中查找了
