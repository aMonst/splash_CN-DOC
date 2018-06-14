Splash脚本教程
=========================================
.. _intro:

简介
-------------------------------------------
splash 能够执行用户使用Lua语言编写的自定义渲染脚本，这就使我们能够像PhantomJs 那样，将其作为一个浏览器自动化工具来使用

我们可以向execute(或者 run)端点发送请求，并设置上lua_script参数，以便执行脚本并获取返回值。
在这个教程中主要使用execute端点

.. note::

    即使您之前没有lua的基础，您也可以很简单的看懂教程中的脚本示例。虽然它很简单，但也很值得学习，您可以使用lua语言编写 `Redis <https://redis.io/commands/EVAL>`_ ,
    `Nginx <https://github.com/openresty/lua-nginx-module>`_ , `Apache <http://httpd.apache.org/docs/trunk/mod/mod_lua.html>`_ , `魔兽世界 <http://wowwiki.wikia.com/wiki/Lua>`_ 的脚本
    可以使用 `Corona <https://coronalabs.com/>`_ 来创建手机应用或者使用当前最先进的深度学习框架 `Torch7 <http://torch.ch/>`_ 。 它很容易入门，并且在网上有许多很棒学习资源，
    像教程有 `15分钟学习lua <http://tylerneylon.com/a/learn-lua/>`_ ，或者书籍 `lua 编程语言 <http://www.lua.org/pil/contents.html>`_

让我们从一个简单的例子开始:
::

    function main(splash, args)
        splash:go("http://example.com")
        splash:wait(0.5)
        local title = splash:evaljs("document.title")
        return {title=title}
    end

如果我们在将这个脚本填写到 ``lua_script`` 参数中并往execute 端点上发送请求，那么splash会访问example.com 这个站点，并等待它加载，会等待半秒，
然后获取页面标题(通过在页面上下文中执行JavaScript代码),最后以json格式返回结果。

.. note::

    Splash UI 提供了一种简便的方法来测试脚本，它里面有一个lua脚本的编辑框和一个将脚本提交到execute端点的按钮。您可以访问 `http://127.0.0.1:8050/ <http://127.0.0.1:8050/>`_
    (或者其他splash监听的主机和端口)

    为了执行在您的编程环境中执行脚本，您需要弄清楚如何发送HTTP请求，您可以在问答模块中参考 如何向Splash API发送HTTP请求，
    它包含了一些常见的方法和步骤(比如，使用Python + requests库)
.. entry-point-the-main-function:

入口点——main函数
--------------------------------------------------
脚本必须提供一个main函数供splash调用，执行结果会以http响应包的方式返回，脚本中可以包含其他有用的函数，但是main函数是必须的。
在第一个例子中， main函数返回一个lua的table结构（一个类似于JavaScript的object或者Python 字典的一个关联数组）。这类结果将会以json的格式返回

下面的代码将会在http的响应中返回 ``{"hello":"world!"}`` 字符串
::

    function main(splash)
        return {hello="world!"}
    end

脚本也可以返回一个字符串
::

    function main(splash)
        return 'hello'
    end

字符串的返回值会原样的在响应体重返回(它不会被编码成json格式)，请看下面的例子
::

    $ curl 'http://127.0.0.1:8050/execute?lua_source=function+main%28splash%29%0D%0A++return+%27hello%27%0D%0Aend'
    hello

main函数接收一个对象，该对象允许我们向操作浏览器选项卡那样操作splash，splash所有功能都被封装到此对象中，为了方便这个参数的名称约定俗成的被
称为 "splash", 但是您不必遵守这条约定:
::

    function main(please)
        please:go("http://example.com")
        please:wait(0.5)
        return "ok"
    end

.. where-are-my-callbacks:

我们的回调在哪?
---------------------------------
下面是我们第一个例子的部分代码
::

    splash:go("http://example.com")
    splash:wait(0.5)
    local title = splash:evaljs("document.title")

这段代码就像传统的面相过程的代码，没有回调也没有花哨的控制流结构。但这并不意味这splash是以同步的方式运行。在引擎中它仍然是异步的。
当代码执行到 ``splash.wait(0.5)`` 时，splash从当前任务中跳出去执行其他任务，在0.5s之后再切换回来。

我们可以向一般的脚本语言一样使用条件、循环语句和函数，从而使编写的代码更加直观

下面来看一个phantomjs中的脚本的例子
::

    // Render Multiple URLs to file

    "use strict";
    var RenderUrlsToFile, arrayOfUrls, system;

    system = require("system");

    /*
    Render given urls
    @param array of URLs to render
    @param callbackPerUrl Function called after finishing each URL, including the last URL
    @param callbackFinal Function called after finishing everything
    */
    RenderUrlsToFile = function(urls, callbackPerUrl, callbackFinal) {
        var getFilename, next, page, retrieve, urlIndex, webpage;
        urlIndex = 0;
        webpage = require("webpage");
        page = null;
        getFilename = function() {
            return "rendermulti-" + urlIndex + ".png";
        };

        next = function(status, url, file) {
            page.close();
            callbackPerUrl(status, url, file);
            return retrieve();
        };

        retrieve = function() {
            var url;
            if (urls.length > 0) {
                url = urls.shift();
                urlIndex++;
                page = webpage.create();
                page.viewportSize = {
                    width: 800,
                    height: 600
                };
                page.settings.userAgent = "Phantom.js bot";
                return page.open("http://" + url, function(status) {
                    var file;
                    file = getFilename();
                    if (status === "success") {
                        return window.setTimeout((function() {
                            page.render(file);
                            return next(status, url, file);
                        }), 200);
                    } else {
                        return next(status, url, file);
                    }
                });
            } else {
                return callbackFinal();
            }
        };
        return retrieve();
    };

    arrayOfUrls = null;

    if (system.args.length > 1) {
        arrayOfUrls = Array.prototype.slice.call(system.args, 1);
    } else {
        console.log("Usage: phantomjs render_multi_url.js [domain.name1, domain.name2, ...]");
        arrayOfUrls = ["www.google.com", "www.bbc.co.uk", "phantomjs.org"];
    }

    RenderUrlsToFile(arrayOfUrls, (function(status, url, file) {
        if (status !== "success") {
            return console.log("Unable to render '" + url + "'");
        } else {
            return console.log("Rendered '" + url + "' at '" + file + "'");
        }
    }), function() {
        return phantom.exit();
    });

平心而论这段代码写的很晦涩 ``RenderUrlsToFile ``函数通过创建一个回调链来实现循环， ``page.open`` 函数并没有返回任何值（如果返回某些值的话实施起来会更加复杂）
而是将返回值存入到磁盘中

下面是一个使用splash脚本更为简单的例子
::

    function main(splash, args)
        splash.set_viewport_size(800, 600)
        splash.set_user_agent('Splash bot')
        local example_urls = {"www.google.com", "www.bbc.co.uk", "scrapinghub.com"}
        local urls = args.urls or example_urls
        local results = {}
        for _, url in ipairs(urls) do
            local ok, reason = splash:go("http://" .. url)
            if ok then
                splash:wait(0.2)
                results[url] = splash:png()
            end
        end
        return results
    end

二者的功能有点不一样，这段代码没有保存页面的截图，而是将png图片的值使用HTTP API的功能返回到客户端

**意见或建议**

- 使用 ``page.open`` 函数并获取返回状态的这种方式有一个阻塞，作为替代可以使用  splash:go 并判断返回的标志是否为 "ok"
- 在lua中使用loop循环，而不是通过创建一个回调链来实现循环
- 拥有一些lua的知识有助于编写lua脚本，比如 您可能对 ipairs 和string的连接符 ``..`` 不太熟悉
- 错误处理是不同的，当发生HTTP 的4xx或者 5xx错误时，PhantomJS 虽然会得到一个页面的截图但是不会在 ``page.open`` 的回调中返回错误码，因为它的状态不为 "fail"，而在splash中会检测出这些错误
- 为了不在控制台中打印返回或者将返回结果保存在文件中，我们可以使用与json相关的 Splash HTTP API
- PhantomJS 允许创建多个页面对象，以便在面板的 ``page.open`` 中提交多个请求，splash只在 ``main`` 函数的splash参数中为脚本提供单个浏览器选项卡(但是您可以自由的将多个包含lua脚本的请求并发的提交给splash)

现在有许多很棒的针对PhantomJS的封装，像 `CasperJS <http://casperjs.org/>`_ 、`NightmareJS <http://www.nightmarejs.org/>`_ 它们提供了
自定义的流程控制的微型语言，以便PhantomJS 的脚本编写出来看起来像同步的方式，但是也多多少少存在一定的问题(像循环，将代码移至帮助函数 [#1]_ ，错误处理)
splash 则采用标准的LUA语言

.. note::
    PhantomJS 和它对应的封装都很棒，很值得敬佩，不要因为上面的内容而抨击它们，它们比splash更加成熟，功能也更加完善
    splash尝试从另一个角度来看待问题，但是每一个独立的splash 功能都有一个独特的PhantomJS 功能与之对应

您想了解更多关于Splash Lua API的功能请参考 `Splash Lua API 概览 <>`_

.. _living-without-callbacks:

在没有回调的情况下编写代码
------------------------------------------------------
.. note::
    您一定对splash引擎中使用的lua协程很好奇

    其实在内部main函数是被splash作为一个协程在执行，像 ``splash:foo()`` 这类函数是用 ``coroutine.yield`` 来实现的，
    关于lua的协程，请参阅 `http://www.lua.org/pil/9.html <http://www.lua.org/pil/9.html>`_

在splash的脚本中并没有区分哪些是阻塞的哪些是同步的。这是对协程和小型组件的一些常见的批评 `这篇文章 <>` 对这个问题进行了
很好的描述，您可以参考一下

但是这些问题并没有真正影响到splash脚本的执行，splash的脚本一般是一段很小的代码，代码的共享状态缩减到最小，
API被设计成了同一时间内只执行单行代码。这些都意味着代码的执行流程是串行化的

如果您想要安全，可以把所有splash函数看做异步执行的。首先要考虑的是当您执行 ``splash:foo()`` 函数后，之前渲染的web页面就被更改了。
这通常是调用这些方法的要点，``splash:wait(time)`` 或者 ``splash:go(url)`` 这些函数只在这点上有意义，因为执行它们之后，web页面就被更改了。 [#2]_
您需要谨记这点

这里面有许多异步函数，像: `splash:go <>`_ , `splash:wait <>`_ , `splash:wait_for_resume <>`_ 。
虽然大多数的splash 函数都不是异步方式工作的，但是您将它们想象成异步的将使您的代码在未来它们被变成异步方式时也能正常工作

.. _calling-splash-methods:

调用splash函数
-------------------------------------------------------------
与大多数语言不同，lua中使用 冒号 ``:`` 来调用类对象的中的方法，为了调用 splash对象中的 ``foo`` 方法，需要写成 ``splash:foo()``。
更多细节请参考 `http://www.lua.org/pil/16.html <http://www.lua.org/pil/16.html>`_

在splash脚本中有两种方式来调用lua中的函数：按顺序传参和使用参数名传参；当使用按顺序传参的方式来调用函数时使用小括号作为形参列表
``splash:foo(val1, val2)``。当使用参数名传参的时候使用大括号来作为形参列表 ``splash:foo{name1=val1, name2=val2}``
::

    -- Examples of positional arguments:
    splash:go("http://example.com")
    splash:wait(0.5, false)
    local title = splash:evaljs("document.title")

    -- The same using keyword arguments:
    splash:go{url="http://example.com"}
    splash:wait{time=0.5, cancel_on_redirect=false}
    local title = splash:evaljs{source="document.title"}

    -- Mixed arguments example:
    splash:wait{0.5, cancel_on_redirect=false}


为了方便，所有的splash API都被设计成接受这调用两种方式。但是针对在lua中大多数函数 `没有参数名称的`_
这样的一种情况(包括大部分从标准库中导出的函数),只能选择使用按参数顺序传参

.. _error-handling:

错误处理
-----------------------------------------
在lua中有两种报告错的方式，抛出一个异常、返回一个错误码。
请参阅 `http://www.lua.org/pil/8.3.html. <http://www.lua.org/pil/8.3.html>`_ 。

而在splash中有如下惯例:
1. 开发者自己的错误(例如不正确的函数参数),抛出异常
#. 开发者向外部调用者提供的错误(无法访问的站点),通过返回标志值的方式：比如函数可以返回 ``ok, reason`` 结构，而调用者可以选择忽略或者处理

如果main函数的结果中有有一个未处理的异常，splash会返回 HTTP 400 并带上出错的信息

我们可以使用lua的 ``error`` 函数手工的抛出一个异常
::

    error("A message to be returned in a HTTP 400 response")

您可以使用lua中的 ``pcall`` 函数来处理异常(防止splash返回 HTTP 400的错误)。
请参阅 `http://www.lua.org/pil/8.4.html <http://www.lua.org/pil/8.4.html>`_

您可以使用 ``assert`` 将错误标志转化为异常, 比如您想使一个站点一直运行，但是又不想手工的处理这个错误，
当您指定的错误发生时您可以使用 ``assert`` 来停止当前进程并使splash返回 HTTP 400的错误
::

    local ok, msg = splash:go("http://example.com")
    if not ok then
        -- handle error somehow, e.g.
        error(msg)
    end

    -- a shortcut for the code above: use assert
    assert(splash:go("http://example.com"))

.. _sandbox:

沙盒
----------------------------------------
默认情况下，spalsh脚本在受限制的环境下运行，在这种情况下并非所有的Lua模块和函数都是有效的。
例如，``require`` 函数被限制了，同时也针对一些资源的数量进行了限制(虽然这个限制很松)。

您可以通过参数 ``--disable-lua-sandbox`` 来启动splash的沙盒
::

    $ docker run -it -p 8050:8050 scrapinghub/splash --disable-lua-sandbox

.. _timeouts:

超时
------------------------------------
默认情况下，在超时后splash会停止脚本的执行(默认超时值是30s)。这对于比较长的脚本是一个常见的问题。
更多详情请参考: `求助：我有一个504超时错误 <>` 和 `splash lua脚本需要做很多工作 <>`

.. [#1] 这块的原文是: moving code to helper functions? 暂时找不到合理的翻译方式
.. [#2] 这里的原文是: splash:wait(time) or splash:go(url) only make sense because webpage changes after calling them
