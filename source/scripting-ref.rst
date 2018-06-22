.. _splash-scripts-reference:

Splash脚本参考
====================================

.. note::
    虽然这个参考十分全面，但是它很难作为入门的内容。如果您是初学状态，或者看不懂下面的内容，
    请您首先查阅 `Splash lua API概览 <./scrapting-overview.html>`_ 这部分的内容

``splash`` 作为main函数的第一个参数被传进来，通过这个对象脚本可以控制浏览器。您可以将它看做一个单一浏览器标签的API [#1]_

.. _attributes:

属性
----------------------------------------

.. _splash-args:

splash.args
###############################
``splash.args`` 是一个传入参数的table类型，它包含了来自GET请求的原始url的合并值和 数据类型为 ``application/json`` 的POST请求的请求体的值

例如您在使用 Splash HTTP API的时候在脚本的参数中添加了一个url参数，那么 ``splash.args.url`` 就包含了您传入的这个URL [#2]_

您也可以将 ``splash.args`` 作为main函数的第二个参数传入:
::

    function main(splash, args)
        local url = args.url
        -- ...
    end

它等效于:
::

    function main(splash)
        local url = splash.args.url
        -- ...
    end

使用 ``args`` 或者 :ref:`splash.args <splash-args>` 是向Splash脚本传递参数的首选。另一个方式是通过字符串格式化将参数作为嵌入的变量
来构造lua脚本的字符串。但是相比与使用splash.args 来说，这种方法有两个问题

1. 数据必须以合理的方式组织，以便它们不会破坏lua脚本
#. 嵌入变量这种方式使得脚本无法高效的使用缓存(您可以参考 HTTP API部分的 `save_args <./api.html#save-args>`_ 和 `load_args <./api.html#load-args>`_ )

.. _splash-js-enabled:

splash.js_enabled
###################################
允许或者禁止执行嵌入到页面中的JavaScript代码

**格式:** ``splash.js_enabled = true/false``

默认情况下允许执行JavaScript代码

.. splash-private-mode-enabled_

splash.private_mode_enabled
###################################
开启或者关闭 浏览器的私有模式(匿名模式)

**格式:** ``splash.private_mode_enabled = true/false``

默认情况下匿名模式是开启的，除非您使用 ``--disable-private-mode`` 选项来启动Splash，请注意如果您关闭了匿名模式，某些请求数据可能会在
浏览器中持续存在(但是它并不影响cookie)

您也可以参考 `如果关闭匿名模式 <./faq.html#disable-private-mode>`_

.. _splash-resource-timeout:

splash.resource_timeout
#######################################
第二次设定网络请求的超时值

**格式:** ``splash.resource_timeout = number``
例如，下面的例子演示了当请求远端的资源超过10s时异常告警
::

    function main(splash)
        splash.resource_timeout = 10.0
        assert(splash:go(splash.args.url))
        return splash:png()
    end

当值为0或者 ``nil`` 时表示没有设置超时值

在设置请求的超时值事，在 :ref:`splash:on_request <splash-on-request>` 函数中使用 ``request:set_timeout`` 设置的超时值要优先于
:ref:`splash.resource_timeout <splash-resource-timeout>`

.. _splash-images-enabled:

splash.images_enabled
###########################################
是否加载图片

**格式:** ``splash.images_enabled = true/false``

默认情况下运行加载图片,禁止加载图片能节省大量的网络带宽(通常在50%左右)并且能提高渲染的速度。请注意这个选项可能会影响页面中JavaScript代码
的执行：禁止加载图片可能会影响DOM元素的坐标或者大小，而在脚本中可能会读取并使用它们

splash在使用缓存的情况下，如果禁止了图片的加载，当缓存中存储了对应的图片，图片就会被加载。所以可能造成的问题是当您在允许图片加载的情况下
加载了一个页面，然后禁止图片加载，接着再加载一个新页面，此时您可能会看到，第一个页面加载了图片，而第二个页面加载的部分图片
（被加载的是与第一个页面相同的图片）。在同一个进程中splash的缓存是可以被不同的脚本共享的，所以您可能会在某些页面中看到部分图片
即使您在一开始就禁止了图片的加载

示例
::

    function main(splash, args)
        splash.images_enabled = false
        assert(splash:go(splash.args.url))
        return {png=splash:png()}
    end

.. _splash-plugins-enabled:

splash.plugins_enabled
##############################################
允许或者禁止浏览器插件(例如 Falsh)

**格式:** splash.plugins_enabled = true/false

默认情况下插件是被禁止的

.. _splash-response-body-enabled:

splash.response_body_enabled
##############################################
启用或者禁止响应内容追踪

**格式:** splash.response_body_enabled = true/false

从效率上考虑，默认情况下Splash不会在内存中保存每个请求的响应内容。这就意味着在函数 :ref:`splash:on_response <splash-on-response>`
的回调函数中，我们无法获取到 `response.body <./scripting-response-object.html#splash-response-body>`_ 属性，同时也无法从
`HAR <http://www.softwareishard.com/blog/har-12-spec/>`_ 中获取到响应的对应内容。可以通过在lua脚本中设置 ``splash.response_body_enabled = true``
来使响应内容变得有效

请注意，不管 :ref:`splash.response_body_enabled <splash-response-body-enabled>` 是否设置，在:ref:`splash:http_get <splash-http-get>` 和
:ref:`splash:http_post <splash-http-post>` 中总是能获取到 :ref:`response.body <./scripting-response-object.html#splash-response-body>`
的内容

您可以通过在函数 :ref:`splash:on_request <splash-on-request>` 的回调中设置 :ref:`request:enable_response_body <splash-response-body-enabled>`
来启用每个请求的响应内容跟踪

.. _splash-scroll-position:

splash.scroll_position
#####################################################
设置或者获取当前滚动的位置

**格式:** ``splash.scroll_position = {x=..., y=...}``

这个属性允许我们设置或者获取当前主窗口的滚动的位置

将窗口滚动到内容以外是没有意义的，例如您设置 ``splash.scroll_position`` 为 ``{x=-100, y=-100}`` 效果与 ``splash.scroll_position``
默认的 ``{x=0, y=0}`` 相同

在设置滚动位置的时候，您不用写全(例如, ``splash.scroll_position = {x=100, y=200}``) 您可以简写成 ``splash.scroll_position = {100, 200}``。
即使您使用的简写的方式，属性值也会被作为一个键为 ``x`` 和 ``y`` 的table

当然，您也可以省略您不想改变的坐标值，例如 ``splash.scroll_position = {y=200}`` 是将y的值改为200，而x的值保持不变

.. _splash-indexeddb-enabled:

splash.indexeddb_enabled
####################################################
允许或者禁止 `IndexedDB <https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API>`_

**格式:** ``splash.indexeddb_enabled = true/false``

默认情况下 IndexedDB 是被禁止的。您可以使用 ``splash.indexeddb_enabled = true`` 来开启它

.. node::
    在当前默认情况下 IndexedDB 是被禁止的，这是因为它在WebKit中存在一些问题，可能在未来它会被默认打开

.. _splash-webgl-enabled:

splash.webgl_enabled
#######################################################
启用或者禁用 `WebGL <https://developer.mozilla.org/en-US/docs/Web/API/WebGL_API>`_

**格式:** ``splash.webgl_enabled = true/false``

WebGL 默认是启用的，您可以通过 ``splash.webgl_enabled = false`` 来禁用

.. _splash-html5-media-enabled:

splash.html5_media_enabled
######################################################
禁止或者启用HTML5 多媒体,包括HTML5中的video 和audio (例如 ``<video>`` 标签进行回放)

**格式:** ``splash.html5_media_enabled = true/false``

默认情况下 HTML5标签是被禁用的，您可以设置 ``splash.html5_media_enabled = true`` 来启用

.. note::
    默认情况下HTML5 被禁止，因为它在某些环境下会使WebKit在访问某些网站时崩溃。在未来它可能会被设置为 ``true`` 。如果在您的程序中不需要使用
    HTML5，请您明确的设置它为 ``false``

您也可以参考 :ref:`splash.html5_media_enabled <./api.html#arg-html5-media>` 这个HTTP API参数的内容

.. _splash-media-source-enabled:

splash.media_source_enabled
#########################################
允许或者禁止 `多媒体资源扩展API <https://developer.mozilla.org/en-US/docs/Web/API/Media_Source_Extensions_API>`_

**格式:** ``splash.media_source_enabled = true/false``

多媒体资源在默认情况下是打开的，您可以使用 ``splash.media_source_enabled = false`` 来关闭它

.. _methods:

方法
-------------------------------------
.. _splash-go:

splash:go
######################################
跳转到一个URL,它的效果类似于在浏览器的地址栏中输入一个url，然后按回车键等待页面加载

**格式:**
::

    ok, reason = splash:go{url, baseurl=nil, headers=nil, http_method="GET", body=nil, formdata=nil}

**参数:**
- url: 需要加载的页面的url
- baseurl: 这个参数为可选参数。当给定了 ``baseurl`` 参数后，页面仍然从 ``url`` 参数中加载,但是它呈现为,页面中资源的相对路径是相对于baseurl来说的。，而且浏览器会认为baseurl在地址栏中。
- headers: 一个由lua table结构表示的http请求头，它被用来新增或者替换初始请求中的头信息
- http_method : 可选参数，它使用一个字符来表示如果请求url所表示的页面，默认为GET，splash同样支持POST
- body： 可选参数，它是POST请求中的body部分的字符串
- formdata: 可选参数，类型为lua中的table，当POST请求中的 ``Content-Type`` 为 ``content-type: application/x-www-form-urlencoded``时，它会进行相应的编码，并作为POST请求的body部分。

**返回值:** ``ok``, ``reason`` 元组 [#4]_。如果在加载页面的过程中发生错误那么 ``ok`` 为空。``reason`` 将会保存错误的类型信息

**异步:** 为异步操作，除非导航被锁

将会报告的5种错误类型( ``ok`` 会为 ``nil`` 的5种情况)

1. 发生网络错误,主机不存在，失去与远程服务端的连接等等。在这种情况下 ``reason`` 为 ``"network<code>"`` 。可以在 `QT <http://doc.qt.io/qt-5/qnetworkreply.html#NetworkError-enum>`_ 的文档中找到对应的错误码，比如``network3`` 表示NDS错误(无效的主机名称)
#. spalsh返回带有 4xx 或者 5xx 状态码的HTTP响应信息。在这种情况下 ``reason`` 的值为 ``http<code>`` 。例如当发生 HTTP 404 Not Found 时，``reason`` 的值为 ``"http404"``
#. 导航被锁住(请参阅 :ref:`splash:lock_navigation <splash-lock-navigation>` ) 。此时 ``reason`` 的值为 ``"navigation_locked"``
#. splash 不能加载主页面(例如第一个请求被丢弃) ``reason`` 的值为 ``render_error``
#. 如果splash不能确定是哪种错误，会简单的返回一个 ``error``

请看下面的例子
::

    local ok, reason = splash:go("http://example.com")
    if not ok then
        if reason:sub(0,4) == 'http' then
            -- handle HTTP errors
        else
            -- handle other errors
        end
    end
    -- process the page

    -- assert can be used as a shortcut for error handling
    assert(splash:go("http://example.com"))

只有当主页 [#3]_ 请求失败时才会上报一个错误(ok==nil)。如果针对其中的相关资源的请求失败 ``splash:go`` 不会上报错误。为了确定上述错误
是否发生或者处理这些错误(像image/js/css 等链接加载失败，ajax请求失败),您可以使用 :ref:`splash.har <splash-har>`
和 :ref:`splash:on_response <splash-on-response>`

``splash:go`` 在返回结果之前会一直跟随HTTP的重定向去请求其中的链接，但是它不会跟踪像 ``<meta http-equiv="refresh" ...>``
这样在HTML中定义的重定向或者 在JavaScript代码中的跳转。此时您可以使用方法 :ref:`splash:wait <splash-wait>` 进行等待，
以便使浏览器跳转到对应的页面上

``headers`` 参数允许添加或者修改初始HTTP请求中的 header 值,您可以使用 :ref:`splash:set_custom_headers <splash-set-custom-headers>` 和
:ref:`splash:on_request <splash-on-request>` 来为以后所有的请求设置header的值(包括后续针对对应资源文件的请求)

下面是一个自定义设置header的例子:
::

    local ok, reason = splash:go{"http://example.com", headers={
        ["Custom-Header"] = "Header Value",
    }})

headers中的 User-Agent项比较特殊,一旦使用他将被保留并用于进一步的请求。这是一个实现的细节，我们可能在未来的版本中对这个特性进行修改。
在设置User-Agent 时，推荐使用方法 :ref:`splash:set_user_agent <splash-set-user-agent>`

.. _splash-wait:

splash:wait
#######################################################
等待对应的时间(单位为秒),使程序等待WebKit 对网页进行进一步的处理

**格式:** ok, reason = splash:wait{time, cancel_on_redirect=false, cancel_on_error=true}

**参数:**
- time: 等待的时间,单位为s
- cancel_on_redirect: 如果它为true(默认为false) 并且在加载主页面的时候发生了重定向，``splash:wait`` 函数会提前返回，返回值为 ``nil "redirect"`` .但是在 HTML中通过 ``<meta http-equiv="refresh" ...>`` 定义的重定向或者在JavaScript中的重定向不受影响
- cancel_on_error: 如果它为true(默认为true) 在等待时如果发生相关错误导致页面无法加载(像WebKit内部的错误或者重定向到了一个无法解析的主机上),此时 ``splash:wait`` 会提前退出并返回 ``nil, "<error string>"``

**返回值:** ``ok, reason`` 元组, 如果 ``ok`` 的值为 ``nil`` ，函数可能会提前退出，``reason`` 可能会返回一个包含错误信息的字符串

**是否为异步:** 异步

使用示例
::

    -- go to example.com, wait 0.5s, return rendered html, ignore all errors.
    function main(splash)
        splash:go("http://example.com")
        splash:wait(0.5)
        return {html=splash:html()}
    end

默认情况下当重定向发生的时候计时器会继续计时，``cancel_on_redirect`` 选项可以在每次重定向发生的时候让计时器重新计时.
例如下面这个函数演示了如何利用 ``cancel_on_redirect`` 来实现每次重定向并加载对应页面后等待对应的时间
::

    function wait_restarting_on_redirects(splash, time, max_redirects)
        local redirects_remaining = max_redirects
        while redirects_remaining > 0 do
            local ok, reason = self:wait{time=time, cancel_on_redirect=true}
            if reason ~= 'redirect' then
                return ok, reason
            end
            redirects_remaining = redirects_remaining - 1
        end
        return nil, "too_many_redirects"
    end

.. _splash-jsfunc:

splash:jsfunc
###############################################
将JavaScript 函数转化为lua可调用的函数

**格式:** ``lua_func = splash:jsfunc(func)``

**参数:**
- func: 包含js函数代码的字符串

**返回值:** 返回一个函数,该函数可以被lua执行并且可以在页面上下文中执行JavaScript代码

**是否为异步:** 否

例子
::

    function main(splash, args)
      local get_div_count = splash:jsfunc([[
      function () {
        var body = document.body;
        var divs = body.getElementsByTagName('div');
        return divs.length;
      }
      ]])
      splash:go(args.url)

      return ("There are %s DIVs in %s"):format(
        get_div_count(), args.url)
    end

请注意，如果您了解lua 字符串中关于 ``[[ ]]`` 的语法知识将会对您理解这些代码有一定的帮助

JavaScript代码也可以接受参数
::

    local vec_len = splash:jsfunc([[
        function(x, y) {
           return Math.sqrt(x*x + y*y)
        }
    ]])
    return {res=vec_len(5, 4)}

全局的JavaScript 函数可以被直接包含进来
::

    local pow = splash:jsfunc("Math.pow")
    local twenty_five = pow(5, 2)  -- 5^2 is 25
    local thousand = pow(10, 3)    -- 10^3 is 1000

lua类型 到JavaScript之间的转化规则
=======  ======
Lua      JavaScript
=======  ======
string	 string
number	 number
boolean	 boolean
table	   Object or Array, see below
nil	     undefined
Element	 DOM node
=======  ======

lua中的 strings, numbers, booleans 和 tables类型的数据可以直接作为参数传入，
它们会被转化成js的 strings/numbers/booleans/objects 类型。 `element <./scripting-element-object.html#splash-element>`_ 也是被支持的。
但是这类数据不能被放到lua的table中

就目前来说，不能传递其他的lua对象，比如不能往一个闭包的JavaScript函数中穿入一个闭包的JavaScript函数或者正常的lua函数作为参数

默认情况下lua中的table类型被转化为JavaScript中的object类型。如果您要将一个table类型转化为数组类型，可以使用函数 `treat.as_array <./scripting-libs.html#treat-as-array>`_

JavaScript类型到lua对象的转化
==============  ==========
JavaScript	    Lua
==============  ==========
string	        string
number	        number
boolean	        boolean
Object	        table
Array	          table, 将其转化为数组(请参阅 `treat.as_array <./scripting-libs.html#treat-as-array>`_ )
undefined	      nil
null	          "" (一个空字符串)
Date	          string: 一个使用ISO8601标砖表示的日期字符串, 例如. 1958-05-21T10:12:00.000Z
Node	          Element实例
NodeList	      一个由 Element 对象组成的 table
function	      nil
circular object	nil
host object	    nil
==============  ==========

函数会将返回结果中的JavaScript类型转化为lua类型，它只支持一些简单的JavaScript类型的转化，
例如如果从闭包中返回一个函数或者jQuery 选择器，这种操作不被支持

当需要返回一个节点(html DOM元素的一个引用)或者节点的列表(document.querySelectorAll函数返回的结果)时，
只能返回节点或者节点列表这样单一的内容，它们不能被包含进数组或者其他的结构中

.. node::
    经验法则：如果参数或者返回值能被序列化为json格式的话，那样最好，当然您也可以在函数中返回节点或者节点的
    列表，但是它们不能被包含在其他结构中 [#5]_
.. [#1] 这是由于Splash一次只能加载一个页面，在加载新页面之前的老页面不会被保存
.. [#2] 这个意思是说我们在使用API时传入的参数都可以通过这个参数获取到
.. [#3] 这里的主页是指参数中url对应的页面，而页面中的资源等等不包含在里面
.. [#4] 这里的原文是 pair，由于我对lua不是很了解，这里翻译为元组可能更好理解
.. [#5] 我觉得这里官方应该是鼓励我们尽量返回能够被序列化的数据，而不要返回类似DOM元素的内容
