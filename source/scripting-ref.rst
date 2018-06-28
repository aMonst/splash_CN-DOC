.. _splash-scripts-reference:

Splash脚本参考
====================================

.. note::

    虽然这个参考十分全面，但是它很难作为入门的内容。如果您是初学状态，或者看不懂下面的内容，
    请您首先查阅 `Splash lua API概览 <./scripting-overview.html>`_ 这部分的内容

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

**原型:** ``splash.js_enabled = true/false``

默认情况下允许执行JavaScript代码

.. splash-private-mode-enabled_

splash.private_mode_enabled
###################################
开启或者关闭 浏览器的私有模式(匿名模式)

**原型:** ``splash.private_mode_enabled = true/false``

默认情况下匿名模式是开启的，除非您使用 ``--disable-private-mode`` 选项来启动Splash，请注意如果您关闭了匿名模式，某些请求数据可能会在
浏览器中持续存在(但是它并不影响cookie)

您也可以参考 `如果关闭匿名模式 <./faq.html#disable-private-mode>`_

.. _splash-resource-timeout:

splash.resource_timeout
#######################################
第二次设定网络请求的超时值

**原型:** ``splash.resource_timeout = number``
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

**原型:** ``splash.images_enabled = true/false``

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

**原型:** splash.plugins_enabled = true/false

默认情况下插件是被禁止的

.. _splash-response-body-enabled:

splash.response_body_enabled
##############################################
启用或者禁止响应内容追踪

**原型:** splash.response_body_enabled = true/false

从效率上考虑，默认情况下Splash不会在内存中保存每个请求的响应内容。这就意味着在函数 :ref:`splash:on_response <splash-on-response>`
的回调函数中，我们无法获取到 `response.body <./scripting-response-object.html#splash-response-body>`_ 属性，同时也无法从
`HAR <http://www.softwareishard.com/blog/har-12-spec/>`_ 中获取到响应的对应内容。可以通过在lua脚本中设置 ``splash.response_body_enabled = true``
来使响应内容变得有效

请注意，不管 :ref:`splash.response_body_enabled <splash-response-body-enabled>` 是否设置，在:ref:`splash:http_get <splash-http-get>` 和
:ref:`splash:http_post <splash-http-post>` 中总是能获取到 `response.body <./scripting-response-object.html#splash-response-body>`_
的内容

您可以通过在函数 :ref:`splash:on_request <splash-on-request>` 的回调中设置 `request:enable_response_body <./scripting-request-object.html#splash-request-enable-response-body>`_
来启用每个请求的响应内容跟踪

.. _splash-scroll-position:

splash.scroll_position
#####################################################
设置或者获取当前滚动的位置

**原型:** ``splash.scroll_position = {x=..., y=...}``

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

**原型:** ``splash.indexeddb_enabled = true/false``

默认情况下 IndexedDB 是被禁止的。您可以使用 ``splash.indexeddb_enabled = true`` 来开启它

.. note::

    在当前默认情况下 IndexedDB 是被禁止的，这是因为它在WebKit中存在一些问题，可能在未来它会被默认打开

.. _splash-webgl-enabled:

splash.webgl_enabled
#######################################################
启用或者禁用 `WebGL <https://developer.mozilla.org/en-US/docs/Web/API/WebGL_API>`_

**原型:** ``splash.webgl_enabled = true/false``

WebGL 默认是启用的，您可以通过 ``splash.webgl_enabled = false`` 来禁用

.. _splash-html5-media-enabled:

splash.html5_media_enabled
######################################################
禁止或者启用HTML5 多媒体,包括HTML5中的video 和audio (例如 ``<video>`` 标签进行回放)

**原型:** ``splash.html5_media_enabled = true/false``

默认情况下 HTML5标签是被禁用的，您可以设置 ``splash.html5_media_enabled = true`` 来启用

.. note::

    默认情况下HTML5 被禁止，因为它在某些环境下会使WebKit在访问某些网站时崩溃。在未来它可能会被设置为 ``true`` 。如果在您的程序中不需要使用
    HTML5，请您明确的设置它为 ``false``

您也可以参考 `splash.html5_media_enabled <./api.html#arg-html5-media>`_ 这个HTTP API参数的内容

.. _splash-media-source-enabled:

splash.media_source_enabled
#########################################
允许或者禁止 `多媒体资源扩展API <https://developer.mozilla.org/en-US/docs/Web/API/Media_Source_Extensions_API>`_

**原型:** ``splash.media_source_enabled = true/false``

多媒体资源在默认情况下是打开的，您可以使用 ``splash.media_source_enabled = false`` 来关闭它

.. _methods:

方法
-------------------------------------
.. _splash-go:

splash:go
######################################
跳转到一个URL,它的效果类似于在浏览器的地址栏中输入一个url，然后按回车键等待页面加载

**原型:**
::

    ok, reason = splash:go{url, baseurl=nil, headers=nil, http_method="GET", body=nil, formdata=nil}

**参数:**

- url: 需要加载的页面的url
- baseurl: 这个参数为可选参数。当给定了 ``baseurl`` 参数后，页面仍然从 ``url`` 参数中加载,但是它呈现为,页面中资源的相对路径是相对于baseurl来说的。，而且浏览器会认为baseurl在地址栏中。
- headers: 一个由lua table结构表示的http请求头，它被用来新增或者替换初始请求中的头信息
- http_method : 可选参数，它使用一个字符来表示如果请求url所表示的页面，默认为GET，splash同样支持POST
- body： 可选参数，它是POST请求中的body部分的字符串
- formdata: 可选参数，类型为lua中的table，当POST请求中的 ``Content-Type`` 为 ``content-type: application/x-www-form-urlencoded`` 时，它会进行相应的编码，并作为POST请求的body部分。

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

**原型:** ok, reason = splash:wait{time, cancel_on_redirect=false, cancel_on_error=true}

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

**原型:** ``lua_func = splash:jsfunc(func)``

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
    local twenty_five = pow(5, 2)  -- 5#2 is 25
    local thousand = pow(10, 3)    -- 10#3 is 1000

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

.. _js-lua-conversion-rules:

JavaScript类型到lua对象的转化

===============  ==========
Lua              JavaScript
===============  ==========
string	         string
number	         number
boolean	         boolean
Object           table
Array            table, 将其转化为数组(请参阅 `treat.as_array <./scripting-libs.html#treat-as-array>`_ )
undefined        nil
null             "" (一个空字符串)
Date             string: 一个使用ISO8601标砖表示的日期字符串, 例如. 1958-05-21T10:12:00.000Z
Node             Element实例
NodeList         一个由 Element 对象组成的 table
function         nil
circular object  nil
host object      nil
===============  ==========

函数会将返回结果中的JavaScript类型转化为lua类型，它只支持一些简单的JavaScript类型的转化，
例如如果从闭包中返回一个函数或者jQuery 选择器，这种操作不被支持

当需要返回一个节点(html DOM元素的一个引用)或者节点的列表(document.querySelectorAll函数返回的结果)时，
只能返回节点或者节点列表这样单一的内容，它们不能被包含进数组或者其他的结构中

.. note::
    经验法则：如果参数或者返回值能被序列化为json格式的话，那样最好，当然您也可以在函数中返回节点或者节点的
    列表，但是它们不能被包含在其他结构中 [#5]_ 。

请注意目前您不能返回jQuery $结果或者从JavaScript到lua的类似结构 [#6]_ 。
要传递数据必须将您感兴趣的属性提取为普通的字符串/数字/对象/数组

::

    -- 这个函数假设jQuery已经在页面中加载
    local get_hrefs = splash:jsfunc([[
    function(sel){
        return $(sel).map(function(){return this.href}).get();
    }
    ]])
    local hrefs = get_hrefs("a.story-title")

当然您也可以在代码中使用 `Element <./scripting-element-object.html#splash-element>`_ 对象和
:ref:`splash:select_all <splash-select-all>`
::

    local elems = splash:select_all("a.story-title")
    local hrefs = {}
    for i, elem in ipairs(elems) do
        hrefs[i] = elem.node:getAttribute("href")
    end

函数的参数和返回值都是按值传递, 比如说您在JavaScript函数中修改了某个参数的值，作为函数调用者的lua代码是不知道的，您在js代码
中返回某个全局的js对象，并在lua中对它进行修改也不会影响到页面上下文。
`Element <./scripting-element-object.html#splash-element>`_ 对象是一个例外，它里面有一些可变的字段。

如果 JavaScript抛出一个错误，它会作为一个lua错误，处理它最好的方式是在JavaScript代码中使用try/catch ，
因为在JavaScript转lua的过程中可能存在数据的丢失

您也可以参考: :ref:`splash:runjs <splash-runjs>` , :ref:`splash:evaljs <splash-evaljs>` ,
:ref:`splash:wait_for_resume <splash-wait-for-resume>` , :ref:`splash:autoload <splash-autoload>` ,
`treat.as_array <./scripting-libs.html#treat-as-array>`_ ,
`Element Object <./scripting-element-object.html#splash-element>`_ ,
:ref:`splash:select <splash-select>` , :ref:`splash:select_all <splash-select-all>` .

.. _splash-evaljs:

splash:evaljs
####################################

在页面上下文中执行JavaScript代码并返回最后一条语句的结果

**原型:** ``result = splash:evaljs(snippet)``

**参数:**

- snippet :一段可以执行的JavaScript代码的字符串

**返回值:** 返回 snippet 中最后一条语句的结果,并将这个结果有JavaScript的类型转化为lua对应的类型 。
如果发生JavaScript异常或者语法错误，则会引发错误。

**是否异步:** 否

JavaScript到lua的转化规则与 :ref:`splash:jsfunc <js-lua-conversion-rules>` 相同

在只需要执行一小段代码而不用涉及到闭包函数的时候，使用 ``splash.eveljs`` 将会十分方便, 例如
::

    local title = splash:evaljs("document.title")

当您不需要返回值的时候不要使用 :ref:`splash:jsfunc <js-lua-conversion-rules>` 。这种方式十分低效，而且可能会带来一些问题
可以使用 :ref:`splash:runjs <splash-runjs>` 来代替。例如下面这段无辜的代码(使用 using jQuery)可能会做无用功
::

    splash:evaljs("$(console.log('foo'));")

这段代码的一个问题是，它允许链接 jQuery $ 函数返回一个巨大的对象, 接着 :ref:`splash:evaljs <splash-evaljs>` 尝试对其
进行序列化并将它转化为lua对象，这是一种对资源的浪费，但是 :ref:`splash:jsfunc <splash-jsfunc>` 不会出现这个问题

如果您经过评估发现您的代码需要使用参数，相比于使用 :ref:`splash:evaljs <splash-evaljs>` 和格式化字符串的方式来说，使用
:ref:`splash:jsfunc <splash-jsfunc>` 将会是一种更好的选择。 请对比下面这段代码
::

    function main(splash)

        local font_size = splash:jsfunc([[
            function(sel) {
                var el = document.querySelector(sel);
                return getComputedStyle(el)["font-size"];
            }
        ]])

        local font_size2 = function(sel)
            -- FIXME: escaping of `sel` parameter!
            local js = string.format([[
                var el = document.querySelector("%s");
                getComputedStyle(el)["font-size"]
            ]], sel)
            return splash:evaljs(js)
        end

        -- ...
    end

请参考: :ref:`splash:runjs <splash-runjs>`, :ref:`splash:jsfunc <splash-jsfunc>` ,
:ref:`splash:wait_for_resume <splash-wait-for-resume>`, :ref:`splash:autoload <splash-autoload>` ,
`Element Object <./scripting-element-object.html#splash-element>`_, :ref:`splash:select <splash-select>`, :ref:`splash:select_all <splash-select-all>` .

.. _splash-runjs:

splash:runjs
##########################
在页面上下文中执行JavaScript代码

**原型:** ``ok, error = splash:runjs(snippet)``

**参数:**

- snippet: 一段可以被执行的JavaScript代码的字符串

**返回值:** ``ok, error`` 的元组,如果执行结果正常 ``ok`` 的值为 ``True`` , 如果JavaScript 代码发生错误，``ok`` 的值为 ``nil``
并且``error`` 是一个描述错误信息的字符串

**是否异步:** 否

示例
::

    assert(splash:runjs("document.title = 'hello';"))

请注意使用语法 ``function foo(){}`` 定义的函数的作用返回并不是全局的
::

    assert(splash:runjs("function foo(){return 'bar'}"))
    local res = splash:evaljs("foo()")  -- 此处会返回一个错误

这是一个实现的细节：传递给 :ref:`splash:runjs <splash-runjs>` 的代码是在闭包中执行的，您可以使用下面的方式来定义全局
函数和变量
::

    assert(splash:runjs("foo = function (){return 'bar'}"))
    local res = splash:evaljs("foo()")  -- 这个位置将会返回 'bar'

如果代码中需要参数，使用 :ref:`splash:jsfunc <splash-jsfunc>` 将会是一个更好的选择

请对比下面的代码
::

    function main(splash)

        -- 滚动窗口到 (x, y) 位置的lua函数.
        function scroll_to(x, y)
            local js = string.format(
                "window.scrollTo(%s, %s);",
                tonumber(x),
                tonumber(y)
            )
            assert(splash:runjs(js))
        end

        -- 一个简单的使用 splash:jsfunc的示例代码
        local scroll_to2 = splash:jsfunc("window.scrollTo")

        -- ...
    end

您也可以参考: :ref:`splash:runjs <splash-runjs>` , :ref:`splash:jsfunc <splash-jsfunc>`, :ref:`splash:autoload <splash-autoload>` ,
:ref:`splash:wait_for_resume <splash-wait-for-resume>`

.. _splash-wait-for-resume:

splash:wait_for_resume
####################################################
以异步的方式在页面上下文中执行JavaScript代码，直到JavaScript代码告诉它恢复，Lua脚本才会恢复然后继续执行

**原型:** ``result, error = splash:wait_for_resume(snippet, timeout)``

**参数:**

- snippet: 一个可以被执行的JavaScript代码的字符串，这段代码必须包含一个可被调用的main函数,main函数的第一个参数是一个包含了 ``resume``和 ``error`` 属性的对象，resume是一个可以用来恢复lua执行的函数，它传入一个可选参数，这个参数将会以 ``result.value`` 的形式返回到lua中，``error`` 是一个函数，当发生错误时会调用这个函数并返回一段包含错误信息字符串
- timeout: 它是一个数值，它决定了在强制返回到lua代码之前运行JavaScript代码运行多长时间(以秒为单位)，默认值是0，这表示将禁用这个超时

**返回值:** ``result, error`` 的元组。当代码成功执行，``result`` 是一个table，如果返回的值在JavaScript中未定义，
则会在 ``result`` 中包含一个由 ``splash.resume(…)`` 返回的键值 。 ``result`` 也可以包含由 ``splash.set(…)`` 添加进来的
键值对。如果执行JavaScript代码超时, ``result`` 的值将会为 ``nil`` 并且 ``error`` 会包含一个错误信息的字符串

**是否为异步** : 是

例子:

第一个小例子主要演示如何将代码执行的控制权由lua转移到JavaScript，最后返回到lua中。这个命令主要告诉JavaScript代码休眠3s，然后
返回到lua中来，请注意：它使用异步的方式来执行，lua事件循环和JavaScript事件循环将在暂停3s之后再运行。但是lua代码会一直等到JavaScript
代码调用``splash.resume()`` 才会继续执行当前函数
::

    function main(splash)
        local result, error = splash:wait_for_resume([[
            function main(splash) {
                setTimeout(function () {
                    splash.resume();
                }, 3000);
            }
        ]])

        -- 返回值为 {}
        -- 错误值为 nil

    end

返回值被设置为空的table，通过 ``splash.resume`` 函数未返回任何值。即使JavaScript 代码未返回任何值，您也可以使用
``assert(splash:wait_for_resume(…))`` 因为对 ``assert()`` 返回空表表示为真

.. note::

    请注意，您的JavaScript代码必须包含main 函数，如果您不包含它，将会得到一个错误。当然main函数的第一个参数的名称您可以随便取
    在这份文档中为了方便我们将称它为splash

第二个例子将演示如果通过JavaScript代码返回对应的值到lua中，您可以返回布尔类型、数字类型、字符串类型、数组类型和JavaScript对象
::

    function main(splash)

        local result, error = splash:wait_for_resume([[
            function main(splash) {
                setTimeout(function () {
                    splash.resume([1, 2, 'red', 'blue']);
                }, 3000);
            }
        ]])

        -- result is {value={1, 2, 'red', 'blue'}}
        -- error is nil

    end

.. note::

    与 :ref:`splash:evaljs <splash-evaljs>` 类似, 注意不要返回太大的JavaScript对象，类似于jQuery中的 ``$`` 。
    太大的对象在转化为lua时会消耗大量的时间和内存

您也可以通过使用函数 ``splash.set(key, value)`` 来在JavaScript代码中添加键值对，新增的键值对将会被包含在 ``result`` 中
返回到lua。下面的代码就演示了这种情况
::

    function main(splash)

        local result, error = splash:wait_for_resume([[
            function main(splash) {
                setTimeout(function () {
                    splash.set("foo", "bar");
                    splash.resume("ok");
                }, 3000);
            }
        ]])

        -- result is {foo="bar", value="ok"}
        -- error is nil

    end

下面的例子将演示一种 ``splash:wait_for_resume()`` 错误的调用方式, JavaScript代码中不包含main函数，此时 ``result`` 值为
``nil`` 因为 ``splash.resume()`` 函数永远不会被调用, ``error`` 会返回一条包含错误信息的字符串来说明这个错误
::

    function main(splash)

        local result, error = splash:wait_for_resume([[
            console.log('hello!');
        ]])

        -- result is nil
        -- error is "error: wait_for_resume(): no main() function defined"

    end

下面一个例子将展示如何进行错误处理, 如果 ``splash.error(…)`` 代替 ``splash.resume()`` 被调用, ``result`` 返回值将会
是 ``nil`` 而 ``error`` 将会包含一个由 ``splash.error(…)`` 给出的错误信息
::

    function main(splash)
        local result, error = splash:wait_for_resume([[
            function main(splash) {
                setTimeout(function () {
                    splash.error("Goodbye, cruel world!");
                }, 3000);
            }
        ]])

        -- result is nil
        -- error is "error: Goodbye, cruel world!"

    end

您的JavaScript代码在某一时刻只能调用 ``splash.resume()`` 或者 ``splash.error()`` 中的任意一个。
随后对这两个函数的调用都不起作用。下面这个例子展示了这一特性
::

    function main(splash)

        local result, error = splash:wait_for_resume([[
            function main(splash) {
                setTimeout(function () {
                    splash.resume("ok");
                    splash.resume("still ok");
                    splash.error("not ok");
                }, 3000);
            }
        ]])

        -- result is {value="ok"}
        -- error is nil

    end

下面的例子将演示timeout参数的影响。我们将超时设置为1s，但是JavaScript代码中 ``splash.resume()`` 函数将在3s后执行。
这样确保 ``splash:wait_for_resume()`` 一定会超时.

当超时发生时 ``result`` 将会为 nil, ``error`` 将会包含一个字符串来解释超时,并且lua代码将会继续执行,在超时后调用 ``splash.resume()`` 或者
``splash.error()`` 不起任何作用
::

    function main(splash)

        local result, error = splash:wait_for_resume([[
            function main(splash) {
                setTimeout(function () {
                    splash.resume("Hello, world!");
                }, 3000);
            }
        ]], 1)

        -- result is nil
        -- error is "error: One shot callback timed out while waiting for resume() or error()."

    end

.. note::

    超时值必须要 >= 0， 如果超时值为0 ``splash:wait_for_resume()`` 永远不会超时(但是Splash’s HTTP API中设置的超时仍然有用)

请确保您的JavaScript代码没有因为超时而被强制结束,它可能会一直执行，直到splash关闭浏览器的页面上下文。

您可以参考: :ref:`splash:runjs <splash-runjs>` , :ref:`splash:jsfunc <splash-jsfunc>` , :ref:`splash:evaljs <splash-evaljs>`

.. _splash-autoload:

splash:autoload
###########################################
设置JavaScript代码在每个页面加载时自动加载

**原型:** ``ok, reason = splash:autoload{source_or_url, source=nil, url=nil}``

**参数:**

- source_or_url: 可以是一段JavaScript代码的字符串或者是JavaScript代码所对应的url，以便在页面加载时执行
- source: 一段JavaScript代码的字符串
- url: 一个用于加载JavaScript源码的url

**返回值:** ``ok, reason`` 元组

**是否异步:** 是。 只有当远程资源的url被传递时采用的是异步

:ref:`splash:autoload <splash-autoload>` 允许在每个页面被加载时执行JavaScript代码，:ref:`splash:autoload <splash-autoload>`
并不会自己执行JavaScript代码,如果您想只执行一次JavaScript代码或者在页面加载完成之后执行，请使用
:ref:`splash:runjs <splash-runjs>` 或者 :ref:`splash:jsfunc <splash-jsfunc>`

:ref:`splash.autoload <splash-autoload>` 可以在页面加载之前预加载有用的JavaScript库，或者在页面要使用某些JavaScript对象
之前先替换这些对象

例子
::

    function main(splash, args)
      splash:autoload([[
        function get_document_title(){
          return document.title;
        }
      ]])
      assert(splash:go(args.url))

      return splash:evaljs("get_document_title()")
    end

为了方便，当 :ref:`splash.autoload <splash-autoload>` 第一个参数以 "http" 或者以 "https://" 开头时，认为它传进来的是一个URL.
示例2-确定某个远程的库是可达的
::

    function main(splash, args)
      assert(splash:autoload("https://code.jquery.com/jquery-2.1.3.min.js"))
      assert(splash:go(splash.args.url))
      local version = splash:evaljs("$.fn.jquery")

      return 'JQuery version: ' .. version
    end

您可以使用 "url" 或者 "source"参数来防止函数自己判断参数
::

    splash:autoload{url="https://code.jquery.com/jquery-2.1.3.min.js"}
    splash:autoload{source="window.foo = 'bar';"}

当参数不变时通过这种方式来禁止函数进行参数判断是一个很好的使用方式

如果 :ref:`splash.autoload <splash-autoload>` 多次被调用，那么所有的脚本都会在页面被加载时调用

如果不想每次页面加载都执行这段JavaScript代码可以使用 :ref:`splash:autoload_reset <splash-autoload-reset>`

您可以参考: :ref:`splash:evaljs <splash-evaljs>` , :ref:`splash:runjs <splash-runjs>`,
:ref:`splash:jsfunc <splash-jsfunc>`, :ref:`splash:wait_for_resume <splash-wait-for-resume>` ,
:ref:`splash:autoload_reset <splash-autoload-reset>`

.. _splash-autoload-reset:

splash:autoload_reset
########################################
取消先前通过函数 :ref:`splash:autoload <splash-autoload>` 向页面上下文中注册的所有JavaScript代码

**原型:** ``splash:autoload_reset()``

**返回值:** nil

**是否为异步:** 否

当调用了 :ref:`splash:autoload_reset <splash-autoload-reset>` 之后，之前使用 :ref:`splash:autoload <splash-autoload>`
注册的函数在后续的请求中将不再被执行。 您可以再次使用 :ref:`splash:autoload <splash-autoload>` 来设置一组不同的脚本代码

已经加载了的脚本将不会被移出当前的页面上下文

您可以参考 :ref:`splash:autoload <splash-autoload>`

.. _splash-call-later:

splash:call_later
############################################
安排一些回调函数在对应的延迟时间过后再调用。

**原型:** ``timer = splash:call_later(callback, delay)``

**参数:**

- callback:需要被执行的函数
- delay: 延迟时间

**返回值:** 一个允许取消挂起计时器的句柄或者注册回调时产生的异常

**是否异步：** 否

例子1-在页面开始加载后的1.5s和2.5s分别获取一段HTML代码
::

    function main(splash, args)
      local snapshots = {}
      local timer = splash:call_later(function()
        snapshots["a"] = splash:html()
        splash:wait(1.0)
        snapshots["b"] = splash:html()
      end, 1.5)
      assert(splash:go(args.url))
      splash:wait(3.0)
      timer:reraise()

      return snapshots
    end

:ref:`splash:call_later <splash-call-later>` 返回一个句柄(计时器) 如果需要取消任务，可以使用 ``timer:cancel()``。
如果一个回调已经开始，调用 ``timer:cancel()`` 将不会起作用

默认情况下，当通过 :ref:`splash:call_later <splash-call-later>` 注册的回调中发生错误，回调将会停止执行，但是这不影响main函数
中脚本的执行，您可以使用 ``timer:reraise()`` 来抛出异常

:ref:`splash:call_later <splash-call-later>` 定义的回调函数将会在后续执行,即使延迟时间为0它们也不会立即执行,当延迟为0时，
不早于当前函数产生的事件循环，即不早于某些异步函数被调用

.. _splash-http-get:

splash:http_get
#####################################
发送一个HTTP的GET请求，并返回一个未经过浏览器加载的响应

**原型:** ``response = splash:http_get{url, headers=nil, follow_redirects=true}`` [#7]_

**参数:**

- url: 请求的url
- headers: 一个用户新增或者替换初始请求头的lua table类型
- follow_redirects: 是否跟随重定向

**返回值:** 返回一个 `Response 对象 <./scripting-response-object.html#splash-response>`_

**是否异步:** 是

例子
::

    local reply = splash:http_get("http://example.com")

该函数调用不会修改当前页面中的上下文环境和URL，如果要通过浏览器来加载web页面，请调用函数 :ref:`splash:go <splash-go>`

您可以参考:  :ref:`splash:http_post <splash-http-post>` , `Response Object <./scripting-response-object.html#splash-response>`_

.. _splash-http-post:

splash:http_post
#######################################
发送一个HTTP的POST请求，并返回一个未经过浏览器加载的响应

**原型:** ``response = splash:http_post{url, headers=nil, follow_redirects=true, body=nil}``

**参数:**

- url: 请求的url
- headers: 一个用户新增或者替换初始请求头的lua table类型
- follow_redirects: 是否跟随重定向
- body: 用字符串表示的请求的请求体，如果您打算将数据提交到表单，body应该进行url编码

**返回值:** 返回一个 `Response 对象 <./scripting-response-object.html#splash-response>`_

**是否异步:** 是
一个提交form表单的例子
::

    local reply = splash:http_post{url="http://example.com", body="user=Frank&password=hunter2"}
    -- reply.body contains raw HTML data (as a binary object)
    -- reply.status contains HTTP status code, as a number
    -- see Response docs for more info

一个关于JSON的例子
::

    json = require("json")

    local reply = splash:http_post{
        url="http://example.com/post",
        body=json.encode({alpha="beta"}),
        headers={["content-type"]="application/json"}
    }

该函数调用不会修改当前页面中的上下文环境和URL，如果要通过浏览器来加载web页面，请调用函数 :ref:`splash:go <splash-go>`

您可以参考: :ref:`splash:http_post <splash-http-post>` , `json <./scripting-libs.html#lib-json>`_ ,
`Response Object <./scripting-response-object.html#splash-response>`_

.. _splash-set-content:

splash:set_content
########################################
设置当前页面的上下文环境，并且等待直到页面加载

**原型:** ``ok, reason = splash:set_content{data, mime_type="text/html; charset=utf-8", baseurl=""}``

**参数:**

- data: 新的页面上下文
- mime_type: 上下文的 MIME 类型
- baseurl: 页面中引用的外部对象的相对路径通过baseurl来定位

**返回值:** ``ok, reason`` 的元组,如果 ``ok`` 为空，表明在加载页面的时候发生了一些异常，``reason`` 包含的发生的错误的类型信息

**是否为异步:** 是

例子
::

    function main(splash)
        assert(splash:set_content("<html><body><h1>hello</h1></body></html>"))
        return splash:png()
    end

.. _splash-html:

splash:html
#################################
返回整个页面的HTML代码(以字符串的形式返回)

**原型:** ``html = splash:html()``

**返回值:** 页面内容(以字符串的形式)

**是否异步:** 否

例子:
::

    -- A simplistic implementation of render.html endpoint
    function main(splash)
        splash:set_result_content_type("text/html; charset=utf-8")
        assert(splash:go(splash.args.url))
        return splash:html()
    end

没有什么能阻止我们获取多个HTML快照。例如我们先加载一个站点的3个页面，为每个页面存储它初始的HTML快照，然后等待0.5s，再加载下一个
::

    treat = require("treat")

    -- Given an url, this function returns a table
    -- with the page screenshoot, it's HTML contents
    -- and it's title.
    function page_info(splash, url)
      local ok, msg = splash:go(url)
      if not ok then
        return {ok=false, reason=msg}
      end
      local res = {
        html=splash:html(),
        title=splash:evaljs('document.title'),
        image=splash:png(),
        ok=true,
      }
      return res
    end

    function main(splash, args)
      -- visit first 3 pages of hacker news
      local base = "https://news.ycombinator.com/news?p="
      local result = treat.as_array({})
      for i=1,3 do
        local url =  base .. i
        result[i] = page_info(splash, url)
      end
      return result
    end

.. _splash-png:

splash:png
#########################################
返回当前页面指定尺寸的屏幕截图

**原型:** ``png = splash:png{width=nil, height=nil, render_all=false, scale_method='raster', region=nil}``

**参数:**

- width: 可选值，以像素为单位的截图的宽
- height: 可选值, 以像素为单位的截图的高
- render_all: 可选值, 如果为 ``true`` 则返回整个页面的截图
- scale_method: 可选值, 调整图片大小时所以用的方法，取值为 ``'raster'``(位图) 和 ``'vector'``  矢量图
- region: 可选值, ``{left, top, right, bottom}`` 表示的裁剪矩形的坐标

**返回值:** 以 `binary object <./scripting-binary-data.html#binary-objects>`_ 形式返回的PNG截图，
如果结果为空会返回 ``nil``

**是否为异步:** 否

如果不传参数，``splash:png()`` 则会返回当前视框的截图

width 参数设置返回图片的宽度，如果视口的宽度与设置的宽度不同，图片会放大或者缩小，匹配指定的图片大小。例如假设视口的宽度为 1024px
而设置 ``splash:png{width=100}`` 将会返回一个完整视口的截图，但是图片会被缩小到宽度为 100px

height 参数设置返回图片的高度，视口的高度与设置的高度不同，图片会被裁剪或者扩展以便匹配指定大小，但是不调整图片内容的大小。
通过这种扩展创建的区域是透明的。

您可以使用 :ref:`splash:set_viewport_size <splash-set-viewport-size>` , :ref:`splash:set_viewport_full <splash-set-viewport-full>`
或者使用参数 render_all 来对视口进行设置 ``render_all = true`` 相当于在开始渲染前调用 ``splash:set_viewport_full()`` 然后再恢复视口大小

您可以使用 region 参数来指定截取渲染页面的哪个部分,它是一个由 ``{left, top, right, bottom}`` 组成的table对象。它的坐标
值与当前滚动的位置有关，目前传入的坐标值必须在视口中。您可以在渲染前调用 :ref:`splash:set_viewport_full <splash-set-viewport-full>`
接着再调用 ``splash:png`` 来确保您传入的坐标值永远在视口中。在后续的splash版本中可能会修复这个问题

使用 ``region`` 参数或者使用 一小段JavaScript代码很容易实现只渲染某一个HTML DOM元素，例如下面的例子

.. _example-render-element:

::

    -- This in an example of how to use lower-level
    -- Splash functions to get element screenshot.
    --
    -- In practice use splash:select("a"):png{pad=32}.


    -- this function adds padding around region
    function pad(r, pad)
      return {r[1]-pad, r[2]-pad, r[3]+pad, r[4]+pad}
    end

    function main(splash, args)
      -- this function returns element bounding box
      local get_bbox = splash:jsfunc([[
        function(css) {
          var el = document.querySelector(css);
          var r = el.getBoundingClientRect();
          return [r.left, r.top, r.right, r.bottom];
        }
      ]])

      -- main script
      assert(splash:go(splash.args.url))
      assert(splash:wait(0.5))

      -- don't crop image by a viewport
      splash:set_viewport_full()

      -- let's get a screenshot of a first <a>
      -- element on a page, with extra 32px around it
      local region = pad(get_bbox("a"), 32)
      return splash:png{region=region}
    end


另一种简单的方法是使用 `element:png <./scripting-element-object.html#splash-element-png>`_
::

    splash:select('#my-element'):png()

scale_method 参数的值必须是 ``'raster'`` 或者 ``'vector'`` 其中的一个，当 ``scale_method='raster'`` 时，图像是按照
像素来调整大小的，当 ``scale_method='vector'`` 时，在渲染过程中图像按照每个元素来调整大小. 矢量缩放更加高效，而且图像
更加清晰, 但是它可能导致渲染失真，所以请谨慎的使用这种方式

``splash:png`` 返回的是 一个二进制对象 `binary object <./scripting-binary-data.html#binary-objects>`_ , 因此您可
以直接将其作为返回值在main函数中返回，它将作为二进制图像数据与 适当的content-type头一起返回：
::

    -- A simplistic implementation of render.png
    -- endpoint.
    function main(splash, args)
      assert(splash:go(args.url))

      return splash:png{
        width=args.width,
        height=args.height
      }
    end

``splash:png()`` 的结果将会作为一个table对象进行返回, 它会以base64的方式进行编码以便嵌入到json数据中，在客户端中创建一个 data:uri
::

    function main(splash)
        assert(splash:go(splash.args.url))
        return {png=splash:png()}
    end

当图片为空时，:ref:`splash:png <splash-png>` 返回 ``nil`` ， 如果您想splash抛出一个错误，请使用 ``assert``
::

    function main(splash)
        assert(splash:go(splash.args.url))
        local png = assert(splash:png())
        return {png=png}
    end

您可以参考 :ref:`splash:jpeg <splash-jpeg>` , `Binary Objects <./scripting-binary-data.html#binary-objects>`_ ,
:ref:`splash:set_viewport_size <splash-set-viewport-size>` , :ref:`splash:set_viewport_full <splash-set-viewport-full>` ,
`element:jpeg <./scripting-element-object.html#splash-element-jpeg>`_ , `element:png <./scripting-element-object.html#splash-element-png>`_

.. _splash-jpeg:

splash:jpeg
###############################
返回指定尺寸的屏幕截图，以JPEG格式返回

**原型:** ``jpeg = splash:jpeg{width=nil, height=nil, render_all=false, scale_method='raster', quality=75, region=nil}``

**参数:**

- width: 可选值，以像素为单位的截图的宽
- height: 可选值, 以像素为单位的截图的高
- render_all: 可选值, 如果为 ``true`` 则返回整个页面的截图
- scale_method: 可选值, 调整图片大小时所以用的方法，取值为 ``'raster'``(位图) 和 ``'vector'``  矢量图
- quality: 可选值，返回JPEG图片的质量, 取1 到 100的整数值
- region: 可选值, ``{left, top, right, bottom}`` 表示的裁剪矩形的坐标

**返回值:** 以 `binary object <./scripting-binary-data.html#binary-objects>`_ 形式返回的JPEG截图，

width 参数设置返回图片的宽度，如果视口的宽度与设置的宽度不同，图片会放大或者缩小，匹配指定的图片大小。例如假设视口的宽度为 1024px
而设置 ``splash:jpeg{width=100}`` 将会返回一个完整视口的截图，但是图片会被缩小到宽度为 100px

height 参数设置返回图片的高度，视口的高度与设置的高度不同，图片会被裁剪或者扩展以便匹配指定大小，但是不调整图片内容的大小。
通过这种扩展创建的区域是透明的。

您可以使用 :ref:`splash:set_viewport_size <splash-set-viewport-size>` , :ref:`splash:set_viewport_full <splash-set-viewport-full>`
或者使用参数 render_all 来对视口进行设置 ``render_all = true`` 相当于在开始渲染前调用 ``splash:set_viewport_full()`` 然后再恢复视口大小

您可以使用 region 参数来指定截取渲染页面的哪个部分,它是一个由 ``{left, top, right, bottom}`` 组成的table对象。它的坐标
值与当前滚动的位置有关，目前传入的坐标值必须在视口中。您可以在渲染前调用 :ref:`splash:set_viewport_full <splash-set-viewport-full>`
接着再调用 ``splash:jpeg`` 来确保您传入的坐标值永远在视口中。在后续的splash版本中可能会修复这个问题

使用一小段JavaScript代码配合 ``region`` 参数可以实现只截取单一HTML DOM元素图片的功能，您可以参考 :ref:`splash:png <splash-png>`
文档中的 :ref:`例子 <example-render-element>` 。另一种方法是使用 `element:jpeg <./scripting-element-object.html#splash-element-jpeg>`_

scale_method 参数的值必须是 ``'raster'`` 或者 ``'vector'`` 其中的一个，当 ``scale_method='raster'`` 时，图像是按照
像素来调整大小的，当 ``scale_method='vector'`` 时，在渲染过程中图像按照每个元素来调整大小. 矢量缩放更加高效，而且图像
更加清晰, 但是它可能导致渲染失真，所以请谨慎的使用这种方式

quality 参数的值必须是 ``0`` 到 ``100`` 之间的整数值, 当这个数值超过 ``95`` 时需要特别注意, ``quality=100`` 会禁用部分
JPEG的压缩算法, 并会产生一个大文件，而这个文件对图像质量的增益几乎没有任何效益

``splash:jpeg`` 返回的是 一个二进制对象 `binary object <./scripting-binary-data.html#binary-objects>`_ , 因此您可
以直接将其作为返回值在main函数中返回，它将作为二进制图像数据与 适当的content-type头一起返回：
::

    -- A simplistic implementation of render.jpeg endpoint
    function main(splash, args)
        assert(splash:go(args.url))
        return splash:jpeg{
           width=args.width,
           height=args.height
        }
    end

``splash:jpeg()`` 的结果将会作为一个table对象进行返回, 它会以base64的方式进行编码以便嵌入到json数据中，在客户端中创建一个 data:uri
::

    function main(splash)
        assert(splash:go(splash.args.url))
        return {jpeg=splash:jpeg()}
    end

当图片为空时，:ref:`splash:jpeg <splash-png>` 返回 ``nil`` ， 如果您想splash抛出一个错误，请使用 ``assert``
::

    function main(splash)
        assert(splash:go(splash.args.url))
        local jpeg = assert(splash:jpeg())
        return {jpeg=jpeg}
    end

您可以参考 :ref:`splash:png <splash-png>` , `Binary Objects <./scripting-binary-data.html#binary-objects>`_ ,
:ref:`splash:set_viewport_size <splash-set-viewport-size>` , :ref:`splash:set_viewport_full <splash-set-viewport-full>` ,
`element:jpeg <./scripting-element-object.html#splash-element-jpeg>`_ , `element:png <scripting-element-object.html#splash-element-png>`_

请注意在 1.2..5x 版本之后， ``splash:jpeg`` 的速度要优于 ``splash:png``

.. _splash-har:

splash:har
###########################################
**原型:** ``har = splash:har{reset=false}``

**参数:**

- reset: 可选值, 当值为 ``true`` 时，每次拍快照之前会清除之前的记录

**返回值:** 返回页面加载的相关信息，发生的事件，发送的网络请求以及请求的响应信息，这些信息都被存储成 `HAR <http://www.softwareishard.com/blog/har-12-spec/>`_ 格式

**是否异步:** 否

您可以使用 :ref:`splash:har <splash-har>` 来获取关于网络请求的信息和splash的其他行为。如果您的脚本在顶层的 "har"
键中返回了 splash:har() 得到的结果，在splash UI中将会以很好的格式展示这些数据(就像 Firefox中的 "Network" 选项卡或者 Chrome
中的 开发者工具)
::

    function main(splash)
        assert(splash:go(splash.args.url))
        return {har=splash:har()}
    end

默认情况下当某些请求被创建(例如 :ref:`splash:go <splash-go>` 函数被多次调用), HAR数据会被累计并组合成单个对象。(日志仍然按页面进行分组)

如果您只想更新对应信息，请使用 ``reset`` 参数，它会清理之前所有已存在的日志，然后重新开始记录
::

    function main(splash, args)
        assert(splash:go(args.url1))
        local har1 = splash:har{reset=true}
        assert(splash:go(args.url2))
        local har2 = splash:har()
        return {har1=har1, har2=har2}
    end

默认情况下，返回的HAR数据不包含响应体的内容，您可以使用 :ref:`splash.response_body_enabled <splash-response-body-enabled>` 属性或者
`request:enable_response_body <./scripting-request-object.html#splash-request-enable-response-body>`_ 方法

您也可以参考: :ref:`splash:har_reset <splash-har-reset>` , :ref:`splash:on_response <splash-on-response>` ,
:ref:`splash.response_body_enabled <splash-response-body-enabled>` , `request:enable_response_body <./scripting-request-object.html#splash-request-enable-response-body>`_ .

.. _splash-har-reset:

splash:har_reset
#########################################
**原型:** ``splash:har_reset()``

**返回值:** ``nil``

**是否异步:** 否

删除所有内部存储的 `HAR <http://www.softwareishard.com/blog/har-12-spec/>`_ 记录, 它与使用 ``splash:har{reset=true}``
相似，但是不返回任何值

您可以参考 :ref:`splash:har <splash-har>`

.. _splash-history:

splash:history
#########################################
**原型:** ``entries = splash:history()``

**返回值:** 返回页面加载时的请求与响应信息，以 `HAR entries <http://www.softwareishard.com/blog/har-12-spec/#entries>`_ 格式返回

**是否异步:** 否

``splash:history`` 返回中不包含相关资源的信息，像图片，脚本，样式或者AJAX请求，如果您需要这方面的信息，
请使用 :ref:`splash:har <splash-har>` 或者 :ref:`splash:on_response <splash-on-response>` .

让我们来获取一个JSON数组，其中包含我们想显示的响应头的信息
::

    function main(splash)
        assert(splash:go(splash.args.url))
        local entries = splash:history()
        -- #entries means "entries length"; arrays in Lua start from 1
        local last_entry = entries[#entries]
        return {
           headers = last_entry.response.headers
        }
    end

您可以参考: :ref:`splash:har <splash-har>`, :ref:`splash:on_response <splash-on-response>`

.. _splash-url:

splash:url
##################################################
**原型:** ``url = splash:url()``

**返回值:** 当前的url

**是否异步:** 否

.. _splash-get-cookies:

splash:get_cookies
##################################################
**原型:** ``cookies = splash:get_cookies()``

**返回值:** 返回CookieJar 的内容, 一个包含所有脚本可用的cookie的数组， 结果以 `HAR cookies <http://www.softwareishard.com/blog/har-12-spec/#cookies>`_
格式返回

**是否异步:** 否

一个返回值的例子
::

    [
        {
            "name": "TestCookie",
            "value": "Cookie Value",
            "path": "/",
            "domain": "www.example.com",
            "expires": "2016-07-24T19:20:30+02:00",
            "httpOnly": false,
            "secure": false,
        }
    ]

.. _splash-add-cookie:

splash:add_cookie
###########################
添加一个cookie

**原型:** ``cookies = splash:add_cookie{name, value, path=nil, domain=nil, expires=nil, httpOnly=nil, secure=nil}``

**是否异步:** 否

例子
::

    function main(splash)
        splash:add_cookie{"sessionid", "237465ghgfsd", "/", domain="http://example.com"}
        splash:go("http://example.com/")
        return splash:html()
    end

.. _splash-init-cookies:

splash:init_cookies
######################################
通过传入的cookie 来重新设置当前cookie

**原型:** ``splash:init_cookies(cookies)``

**参数:**

- cookies: 一个用lua的table表示的需要设置的cookie值，与 :ref:`splash:get_cookies <splash-get-cookies>` 返回值的格式相同

**返回值:** ``nil``

**是否异步:** 否

例1：保存并重新设置cookie
::

    local cookies = splash:get_cookies()
    -- ... do something ...
    splash:init_cookies(cookies)  -- restore cookies

例2：手工初始化cookie
::

    splash:init_cookies({
        {name="baz", value="egg"},
        {name="spam", value="egg", domain="example.com"},
        {
            name="foo",
            value="bar",
            path="/",
            domain="localhost",
            expires="2016-07-24T19:20:30+02:00",
            secure=true,
            httpOnly=true,
        }
    })

    -- do something
    assert(splash:go("http://example.com"))

.. _splash-clear-cookies:

splash:clear_cookies
#######################################
清理所有的cookie

**原型:** ``n_removed = splash:clear_cookies()``

**返回值:** 返回被删除cookie的数量

**是否异步:** 否

如果只删除指定的cookie请使用 :ref:`splash:delete_cookies <splash-delete-cookies>`

.. _splash-delete-cookies:

splash:delete_cookies
###############################
删除指定的cookie

**原型:** ``n_removed = splash:delete_cookies{name=nil, url=nil}``

**参数:**

- name: 字符串类型，可选参数；所有name值为此值的cookie都将被删除
- url: 字符串类型，可选参数，所有应该发送到此地址的cookie将被删除

**返回值:** 被删除的cookie的个数

当name和url都为nil的时候，这个函数不做任何事情，您可以使用 :ref:`splash:clear_cookies <splash-clear-cookies>` 来删除所有cookie

.. _splash-lock-navigation:

splash:lock_navigation
#############################################

锁定导航栏

**原型:** ``splash:lock_navigation()``

**是否异步:** 否

当调用这个函数之后，页面导航就不再离开当前页面，页面被锁定在当前url上

.. _splash-unlock-navigation:

splash:unlock_navigation
##############################################

解锁导航栏

**原型:** ``splash:unlock_navigation()``

**是否异步:** 否

当调用这个函数之后，就允许导航离开当前页面。请注意之前由于 :ref:`splash:lock_navigation <splash-unlock-navigation>`
限制的待处理的请求不会被重新加载

.. _splash-set-result-status-code:

splash:set_result_status_code
##################################################

设置返回给客户端的HTTP的状态码

**原型:** ``splash:set_result_status_code(code)``

**参数:**

- HTTP 状态码(是一个 >= 200 <= 999 的整数)

**返回值:**  nil

**是否为异步:** 否

通过此功能可以向splash 客户端发送HTTP状态码，以便向客户端报告相关错误

例子:
::

    function main(splash)
        local ok, reason = splash:go("http://www.example.com")
        if reason == "http500" then
            splash:set_result_status_code(503)
            splash:set_result_header("Retry-After", 10)
            return ''
        end
        return splash:png()
    end

在使用这个函数时要注意: 某些代码可能配置成会根据不同的响应码来进行不同的处理。例如您可以查看nginx 的 `proxy_next_upstream <http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_next_upstream>`_ 选项

当存在未处理的lua错误时，不论之前调用 :ref:`splash:set_result_status_code <splash-set-result-status-code>` 设置了何值,
最终都会返回400.

您也可以参考: :ref:`splash:set_result_content_type <splash-set-result-content-type>`, :ref:`splash:set_result_header <splash-set-result-header>`

.. _splash-set-result-content-type:

splash:set_result_content_type
###########################################
设置返回给客户端结果的 Content-Type 值

**原型:** ``splash:set_result_content_type(content_type)``

**参数:**

- content_type: 一个表示头中 Content-Type 键值的字符串

**返回值:** ``nil``

**是否异步:** 否

如果main函数的返回值是一个table，那么这个函数将不起作用: Content-Type 会被设置成 ``application/json``

这个函数并不是设置使用 :ref:`splash:go <splash-go>` 发送原始请求的Content-Type, 而是设置返回给客户端的结果的Content-Type

例子:
::

    function main(splash)
        splash:set_result_content_type("text/xml")
        return [[
           <?xml version="1.0" encoding="UTF-8"?>
           <note>
               <to>Tove</to>
               <from>Jani</from>
               <heading>Reminder</heading>
               <body>Don't forget me this weekend!</body>
           </note>
        ]]
    end

您可以参考:

- :ref:`splash:set_result_header <splash-set-result-header>` 这个函数允许用户自定义任意的头信息，而不仅仅是Content-Type
- `Binary Objects <./scripting-binary-data.html#binary-objects>`_ 它有它自己的方法来设置返回值的 Content-Type

.. _splash-set-result-header:

splash:set_result_header
##############################################
设置返回给客户端结果的header 值

**原型:** ``splash:set_result_header(name, value)``

**参数:**

- name: 返回header的键名称
- value: 对应的键值

**返回值:** nil

这个函数并不是设置使用 :ref:`splash:go <splash-go>` 发送原始请求的headers, 而是设置通过splash返回到客户端的headers

例1：在header中设置 "foo=bar"
::

    function main(splash)
        splash:set_result_header("foo", "bar")
        return "hello"
    end

例2：获取将屏幕快照转化为PNG图片所需要的时间，并通过header返回
::

    function main(splash)

        -- this function measures the time code takes to execute and returns
        -- it in an HTTP header
        function timeit(header_name, func)
            local start_time = splash:get_perf_stats().walltime
            local result = func()  -- it won't work for multiple returned values!
            local end_time = splash:get_perf_stats().walltime
            splash:set_result_header(header_name, tostring(end_time - start_time))
            return result
        end

        -- rendering script
        assert(splash:go(splash.args.url))
        local screenshot = timeit("X-Render-Time", function()
           return splash:png()
        end)
        splash:set_result_content_type("image/png")
        return screenshot
    end

您也可以看看:  :ref:`splash:set_result_status_code <splash-set-result-status-code>` ,
:ref:`splash:set_result_content_type <splash-set-result-content-type>` .

.. _splash-get-viewport-size:

splash:get_viewport_size
#############################################

获取浏览器视口的尺寸

**原型:** ``width, height = splash:get_viewport_size()``

**返回值:** 以像素为单位返回两个数值——视口的宽和高

**是否异步:** 否

.. _splash-set-viewport-size:

splash:set_viewport_size
############################################

设置浏览器视口的大小

**原型:** ``splash:set_viewport_size(width, height)``

**参数:**

- width: 整数值，视口的宽
- height: 整数值, 视口的高

**返回值:** nil

**是否异步:** 否

这个函数可能会修改可见区域大小和随后的渲染命令, 例如它可能会修改之后调用 :ref:`splash:png <splash-png>` 将产生具有指定大小的图像

:ref:`splash:png <splash-png>` 将会使用这个视口大小

例子
::

    function main(splash)
        splash:set_viewport_size(1980, 1020)
        assert(splash:go("http://example.com"))
        return {png=splash:png()}
    end

.. note::

    这将重新布局所有的document元素并影响 geometry 变量,像 ``window.innerWidth`` 和 ``window.innerHeight`` 。
    然而 ``window.onresize`` 事件回调将会在下一次异步操作时被调用，很明显 :ref:`splash:png <splash-png>` 属于同步操作。
    因此如果您希望在修改视口大小并希望它在执行截屏之前作出相应的调整，您可以使用 :ref:`splash:wait <splash-wait>` 来等待。

.. _splash-set-viewport-full:

splash:set_viewport_full
#######################################

调整浏览器视口大小以便适应整个页面

**原型:** ``width, height = splash:set_viewport_full()``

**返回值:** 返回两个数值，当前视口被设置的宽和高，以像素为单位

**是否异步:** 否

``splash:set_viewport_full`` 只有在页面被加载之后才能调用，有时候需要等待一段时间(使用 ``splash:wait <splash-wait>``)。
这是一个糟糕的限制，但是这是使自动调整大小这一行为工作正常的唯一办法

您可以参考 :ref:`splash:set_viewport_size <splash-set-viewport-size>` 来获取与JS交互的相关内容

:ref:`splash:png <splash-png>` 将会使用这个设置的视口大小

例子:
::

    function main(splash)
        assert(splash:go("http://example.com"))
        assert(splash:wait(0.5))
        splash:set_viewport_full()
        return {png=splash:png()}
    end

.. _splash-set-user-agent:

splash:set_user_agent
####################################

重新设置后续所有请求头中的 User-Agent的值

**原型:** ``splash:set_user_agent(value)``

**参数:**

- value: 一个表示HTTP请求头中 UA的字符串

**返回值:** nil

**是否异步:** 否


.. _splash-set-custom-headers:

splash:set_custom_headers
#####################################

设置每个请求的 HTTP请求头

**原型:** ``splash:set_custom_headers(headers)``

**参数:**

- headers: 一个表示请求头的 LUA的table数据

**返回值:** nil

**是否异步:** 否

这里设置的headers将会与WebKit默认的headers合并, 在发生冲突时会覆盖 WebKit中的值

使用 ``splash:set_custom_headers`` 设置的请求值并不会作用在 :ref:`splash:go <splash-go>` 的请求上，
也就是说值不会进行合并。 :ref:`splash:go <splash-go>` 使用的headers 拥有更高的优先级

例子
::

    splash:set_custom_headers({
       ["Header-1"] = "Value 1",
       ["Header-2"] = "Value 2",
    })

.. note::

    这个函数不支持使用参数名传参的方式

您也可以参考: :ref:`splash:on_request <splash-on-request>`

.. _splash-get-perf-stats:

splash:get_perf_stats
################################

返回一个与统计相关的表现资料

**原型:** ``stats = splash:get_perf_stats()``

**返回值:** 返回一个对行为分析有用的table

**是否异步:** 否

就目前来说，这个table包含如下值:

- ``walltime`` : float型，返回epoch 时间(从1970 0点开始的时间), 类似于lua中的 ``os.clock``
- ``cputime`` : float型, splash 进程消耗的CPU时间
- ``maxrss`` : int型, splash消耗的内存字节数的高位

.. _splash-on-request:

splash:on_request
########################################
注册一个函数，每当发送HTTP请求的时候调用

**原型:** ``splash:on_request(callback)``

**参数:**

- callback: 每次发送HTTP请求前被调用的函数

**是否异步:** 否

:ref:`splash:on_request <splash-on-request>` 的回调接收一个 ``splash`` 参数 (一个 `Request 对象 <./scripting-request-object.html#splash-request>`_ )

获取更多关于请求的信息，您可以使用 请求对象的 `属性 <./scripting-request-object.html#splash-request-attributes>`_
,如果要在发起请求前修改或者丢弃对应请求，请使用请求对象的相关 `方法 <./scripting-request-object.html#splash-request-methods>`_

通过 :ref:`spash:on_request <splash-on-request>` 注册的回调函数中不能调用Splash中的异步函数，像 :ref:`splash:go <splash-go>` 或者 :ref:`splash:wait <splash-wait>`

例1：使用 `request.url <./scripting-request-object.html#splash-request-url>`_ 属性记录所有的请求URL
::

    treat = require("treat")

    function main(splash, args)
      local urls = {}
      splash:on_request(function(request)
        table.insert(urls, request.url)
      end)

      assert(splash:go(splash.args.url))
      return treat.as_array(urls)
    end

例2：通过 `request.info <scripting-request-object.html#splash-request-info>`_ 属性来记录请求信息，但是不直接存储
::

    treat = require("treat")
    function main(splash)
        local entries = treat.as_array({})
        splash:on_request(function(request)
            table.insert(entries, request.info)
        end)
        assert(splash:go(splash.args.url))
        return entries
    end

例3：丢弃所有针对 ".css" 资源的请求(请参考: `request:abort <scripting-request-object.html#splash-request-abort>`_ )
::

    splash:on_request(function(request)
        if string.find(request.url, ".css") ~= nil then
            request.abort()
        end
    end)

例4：替换资源(请参阅: `request:set_url <./scripting-request-object.html#splash-request-set-url>`_ )
::

    splash:on_request(function(request)
        if request.url == 'http://example.com/script.js' then
            request:set_url('http://mydomain.com/myscript.js')
        end
    end)

例5：设置自定义的代理服务器，并将相关凭据传递给Splash的HTTP请求(请参阅: `request:set_proxy <./scripting-request-object.html#splash-request-set-proxy>`_ )
::

    splash:on_request(function(request)
        request:set_proxy{
            host = "0.0.0.0",
            port = 8990,
            username = splash.args.username,
            password = splash.args.password,
        }
    end)

例6：丢弃响应时间超过5s的请求，但是针对第一个请求允许它不超过15s(请参阅 `request:set_timeout <./scripting-request-object.html#splash-request-set-timeout>`_ )
::

    local first = true
    splash.resource_timeout = 5
    splash:on_request(function(request)
        if first then
            request:set_timeout(15.0)
            first = false
        end
    end)

.. note::

    :ref:`splash:on_request <splash-on-request>` 不能使用命名方式传参

您也可以参考 : :ref:`splash:on_response <splash-on-response>` , :ref:`splash:on_response_headers <splash-on-response-headers>` ,
:ref:`splash:on_request_reset <splash-on-request-reset>` , `treat <./scripting-libs.html#lib-treat>`_ , `Request Object <./scripting-request-object.html#splash-request>`_

.. _splash-on-response-headers:

splash:on_response_headers
##########################################
注册一个回调函数，这个函数在接收响应头之后，但是在接收到响应体之前被调用

**原型:** ``splash:on_response_headers(callback)``

**参数:**

- callback: 在接收到响应头之后，但是在接收到响应体之前被调用的Lua函数

**返回值:** nil

**是否异步** : 否

:ref:`splash:on_response_headers <splash-on-response-headers>` 传入单个的 ``response`` 参数(一个 `Response Object <./scripting-response-object.html#splash-response>`_ )

`response.body <./scripting-response-object.html#splash-response-body>`_ 在 :ref:`splash:on_response_headers <splash-on-response-headers>` 中
无效，因为此时并没有接收到响应体, :ref:`splash:on_response_headers <splash-on-response-headers>` 方法使用的关键在于
您可以根据具体情况调用 `response:abort  <./scripting-response-object.html#splash-response-abort>`_ 选择放弃接收响应体

在 :ref:`splash:on_response_headers <splash-on-response-headers>` 定义的回调函数中无法使用Splash中的异步函数,
像 :ref:`splash:go <splash-go>` 或者 :ref:`splash:wait <splash-wait>` 。 ``response`` 对象在退出回调函数后被清理。
所以您在回调函数之外无法使用这个对象

例1：记录在渲染时获取到的所有响应头的 content-type 值
::

    function main(splash)
        local all_headers = {}
        splash:on_response_headers(function(response)
            local content_type = response.headers["Content-Type"]
            all_headers[response.url] = content_type
        end)
        assert(splash:go(splash.args.url))
        return all_headers
    end

例2：放弃接收响应头中 Content-Type值为 ``text/css`` 的响应体
::

    function main(splash, args)
      splash:on_response_headers(function(response)
        local ct = response.headers["Content-Type"]
        if ct == "text/css" then
          response.abort()
        end
      end)

      assert(splash:go(args.url))
      return {
        png=splash:png(),
        har=splash:har()
      }
    end

例3：在不获取响应体的情况下提取站点中的所有cookie
::

    function main(splash)
    local cookies = ""
    splash:on_response_headers(function(response)
        local response_cookies = response.headers["Set-cookie"]
        cookies = cookies .. ";" .. response_cookies
        response.abort()
    end)
    assert(splash:go(splash.args.url))
    return cookies
    end

.. note::
    :ref:`splash:on_response_headers <splash-on-response-headers>` 不能使用命名方式传参

您也可以参考: :ref:`splash:on_request <splash-on-request>` , :ref:`splash:on_response <splash-on-response>` ,
:ref:`splash:on_response_headers_reset <splash-on-response-headers-reset>` ,
`Response Object <./scripting-response-object.html#splash-response>`_ .

.. _splash-on-response:

splash:on_response
#################################
注册一个回调函数，当响应下载完成之后调用

**原型:** ``splash:on_response(callback)``

**参数:**

- callback: 每当下载完成之后需要调用的函数

**返回值:** nil

**是否为异步:** 否

:ref:`splash:on_response <splash-on-response>` 回调函数接受单个 ``response`` 参数(一个 `Response 对象 <./scripting-response-object.html#splash-response>`_ )

默认情况下，``response`` 对象没有 `response.body <./scripting-response-object.html#splash-response-body>`_ 属性。
如果您想开启这个属性，请使用 :ref:`splash.response_body_enabled <splash-response-body-enabled>` 选项或者调用
`request:enable_response_body <./scripting-request-object.html#splash-request-enable-response-body>`_ 方法

.. note::

    :ref:`splash:on_response <splash-on-response>` 不允许使用参数名称进行传参

您可以参考: :ref:`splash:on_request <splash-on-request>` , :ref:`splash:on_response_headers <splash-on-response-headers>` ,
:ref:`splash:on_response_reset <splash-on-response-reset>` , `Response Object <./scripting-response-object.html#splash-response>`_ ,
:ref:`splash.response_body_enabled <splash-response-body-enabled>` , `request:enable_response_body <./scripting-request-object.html#splash-request-enable-response-body>`_ .

.. _splash-on-request-reset:

splash:on_request_reset
#########################################
删除由 :ref:`splash:on_request <splash-on-request>` 函数注册的所有回调函数

**原型:** ``splash:on_request_reset()``

**返回值:** nil

**是否异步:** 否

.. _splash-on-response-headers-reset:

splash:on_response_headers_reset
#####################################
清理所有由 :ref:`splash:on_response_headers <splash-on-response-headers>` 注册的回调函数

**原型:** ``splash:on_response_headers_reset()``

**返回值:** nil

**是否异步:** 否

.. _splash-on-response-reset:

splash:on_response_reset
##################################
移除所有由 :ref:`splash:on_response <splash-on-response>` 函数注册的回调函数

**原型:** ``splash:on_response_reset()``

**返回值:** nil

**是否异步:** 否

.. _splash-get-version:

splash:get_version
###############################
获取Splash的主版本和次版本号

**原型:** ``version_info = splash:get_version()``

**返回值:** 一个包含版本信息的table结构

**是否异步:** 否

目前，这个table主要包含如下内容:

- ``splash`` : (string) Splash版本
- ``major`` : (int) Splash主版本
- ``minor`` : (int) Splash 次版本
- ``python`` : (string) Python版本号
- ``qt`` : (string) QT的版本
- ``webkit``: (string) WebKit版本
- ``sip`` : (string) SIP 版本
- ``twisted``: (string) Twisted 版本

示例:
::

    function main(splash)
         local version = splash:get_version()
         if version.major < 2 and version.minor < 8 then
             error("Splash 1.8 or newer required")
         end
     end

.. _splash-mouse-click:

splash:mouse_click
###################################
在Web页面中触发一个鼠标点击的消息

**原型:** ``splash:mouse_click(x, y)``

**参数:**

- x: 需要点击元素的x坐标的值(距左侧的距离，相对于当前视口)
- y: 需要点击元素的y坐标的值(距上方的距离，相对于当前视口)

**返回值:** nil

**是否异步:** 否

鼠标事件的坐标必须与当前视口相关联

如果您想在某个元素上点击鼠标，一个简单的办法是使用 :ref:`splash:select <splash-select>` 和 `element:mouse_click <./scripting-element-object.html#splash-element-mouse-click>`_
::

    local button = splash:select('button')
    button:mouse_click()

您也可以使用 :ref:`splash:mouse_click <splash-mouse-click>` 来实现它，使用JavaScript代码来获取对应元素的坐标
::

    -- Get button element dimensions with javascript and perform mouse click.
    function main(splash)
        assert(splash:go(splash.args.url))
        local get_dimensions = splash:jsfunc([[
            function () {
                var rect = document.getElementById('button').getClientRects()[0];
                return {"x": rect.left, "y": rect.top}
            }
        ]])
        splash:set_viewport_full()
        splash:wait(0.1)
        local dimensions = get_dimensions()
        -- FIXME: button must be inside a viewport
        splash:mouse_click(dimensions.x, dimensions.y)

        -- Wait split second to allow event to propagate.
        splash:wait(0.1)
        return splash:html()
    end

与 `element:mouse_click <./scripting-element-object.html#splash-element-mouse-click>`_ 不同，:ref:`splash:mouse_click <splash-mouse-click>`
不是异步的，鼠标消息不会立即得到响应，为了查看鼠标点击事件执行后页面的变化，您必须要在调用 :ref:`splash:mouse_click <splash-mouse-click>`
之后调用 :ref:`splash:wait <splash-wait>`

执行该操作的元素必须在视口之内(必须对用户可见),如果元素在视口之外,您需要滚动视口以便让其可见,您可以选择滚动该元素(使用JavaScript代码、
:ref:`splash.scroll_position <splash-scroll-position>` 或者 ``element:scrollIntoViewIfNeeded()`` ) 或者使用函数
:ref:`splash:set_viewport_full <splash-set-viewport-full>` 来将视口设置为整个页面大小

.. note::

    与 :ref:`splash:mouse_click <splash-mouse-click>` 不同, `element:mouse_click <./scripting-element-object.html#splash-element-mouse-click>`_ 会自动滚动屏幕

在splash引擎中, :ref:`splash:mouse_click <splash-mouse-click>` 会先执行 :ref:`splash:mouse_press <splash-mouse-press>`
再执行 :ref:`splash:mouse_release <splash-mouse-release>`

目前只支持鼠标左键点击事件

您可以参考: `element:mouse_click <./scripting-element-object.html#splash-element-mouse-click>`_ ,
:ref:`splash:mouse_press <splash-mouse-press>` , :ref:`splash:mouse_release <splash-mouse-release>` ,
:ref:`splash:mouse_hover <splash-mouse-hover>`, :ref:`splash.scroll_position <splash-scroll-position>`


.. _splash-mouse-hover:

splash:mouse_hover
##################################
触发Web 页面中鼠标悬停消息(JavaScript 中的mouseover消息)

**原型:** ``splash:mouse_hover(x, y)``

**参数:**

- x: 需要触发悬停事件的元素所在位置的 x 坐标(距左边的距离，相对于当前视口来说)
- y: 需要触发悬停事件的元素所在位置的 y 坐标(距上边的距离，相对于当前视口来说)

**返回值:** nil

**是否异步:** 否

请在 :ref:`splash:mouse_click <splash-mouse-click>` 中参阅相关的鼠标事件

您也可以参考: `element:mouse_hover <scripting-element-object.html#splash-element-mouse-hover>`_

.. _splash-mouse-press:

splash:mouse_press
#########################################
触发页面中鼠标按下的事件

**原型:** ``splash:mouse_press(x, y)``

**参数:**

- x: 需要触发鼠标左键按下事件的元素所在位置的 x 坐标(距左边的距离，相对于当前视口来说)
- y: 需要触发鼠标左键按下事件的元素所在位置的 y 坐标(距上边的距离，相对于当前视口来说)

**返回值:** nil

**是否异步:** 否

请在 :ref:`splash:mouse_click <splash-mouse-click>` 中参阅相关的鼠标事件

.. _splash-mouse-release:

splash:mouse_release
###############################
触发页面中鼠标键抬起的事件

**原型:** ``splash:mouse_release(x, y)``

**参数:**

- x: 需要触发鼠标左键抬起事件的元素所在位置的 x 坐标(距左边的距离，相对于当前视口来说)
- y: 需要触发鼠标左键抬起事件的元素所在位置的 y 坐标(距上边的距离，相对于当前视口来说)

**返回值:** nil

**是否异步:** 否

请在 :ref:`splash:mouse_click <splash-mouse-click>` 中参阅相关的鼠标事件

.. _splash-with-timeout:

splash:with_timeout
################################
设置执行函数的超时值

**原型:** ``ok, result = splash:with_timeout(func, timeout)``

**参数:**

- func: 需要设置超时值的函数
- timeout: 超时值,单位为秒

**返回值:** ``ok, result`` 的元组;如果 ``ok`` 的值不为 ``true`` 表明在执行函数的过程中出现对应的错误，或者超时。
``result`` 将会保存错误的类型信息, 如果 ``result`` 等于 ``timeout`` 则说明指定的超时时间已过。当 ``ok`` 为 ``true``
时, ``result`` 中包含函数执行的结果。如果您的函数返回多个值,它们会被返回到后面多个 ``result`` 变量中


**是否异步:** 是

例1:
::

    function main(splash, args)
      local ok, result = splash:with_timeout(function()
        -- try commenting out splash:wait(3)
        splash:wait(3)
        assert(splash:go(args.url))
      end, 2)

      if not ok then
        if result == "timeout_over" then
          return "Cannot navigate to the url within 2 seconds"
        else
          return result
        end
      end
      return "Navigated to the url within 2 seconds"
    end

例2：函数返回多个值:
::

    function main(splash)
        local ok, result1, result2, result3 = splash:with_timeout(function()
            splash:wait(0.5)
            return 1, 2, 3
        end, 1)

        return result1, result2, result3
    end

请注意，如果函数执行时间超过了设置的超时值，那么splash会尝试中断函数的执行。但是splash是以 `协作式多任务 <https://zh.wikipedia.org/wiki/%E5%8D%8F%E4%BD%9C%E5%BC%8F%E5%A4%9A%E4%BB%BB%E5%8A%A1>`_ 的方式在运行,
因此在某些时候Spalash无法停止执行超时的函数。换句话说, 协作式多任务模式意味着管理程序(在这个例子中管理程序是Splash脚本引擎)
在对应子程序没有要求的情况下不会主动结束子程序。 在splash脚本中，只有在调用某些异步操作时，这些操作才会被结束 [#8]_ 。
反之，同步函数无法被停止

.. note::
    splash是以 `协作式多任务 <https://zh.wikipedia.org/wiki/%E5%8D%8F%E4%BD%9C%E5%BC%8F%E5%A4%9A%E4%BB%BB%E5%8A%A1>`_ 的方式在运行
    。在进行同步调用的时候，您需要额外的小心

让我们在例子中看一下它们的不同
::

    function main(splash)
        local ok, result = splash:with_timeout(function()
            splash:go(splash.args.url) -- 执行到此处时任务可以被停止
            splash:evaljs(long_js_operation) -- 在执行js函数期间不能被停止
            local png = splash:png() -- 执行到同步操作的时候，不能被停止
            return png
        end, 0.1)

        return result
    end

.. _splash-send-keys:

splash:send_keys
####################################
相对应的页面上下文环境发送键盘事件

**原型:** ``splash:send_keys(keys)``

**参数:**

- keys : 表示要作为键盘事件发送的键的字符串

**返回值:** nil

**是否异步:** 否

指定的键值将会被使用 emacs edmacro 语法中的一个小子集来进行序列化

- 空格键将会被忽略，它仅仅用来分隔不同的键值
- 字符值将会原样的保留
- 括号内的词代表对应的功能键,像 ``<Return>`` 、``<Home>`` 、``<Left>`` 。您可以查看 `QT 的帮助文档 <http://doc.qt.io/qt-5/qt.html#Key-enum>`_ 来获取完整的功能键列表. ``<Foo>`` 会尝试匹配 ``Qt::Key_Foo``

下面这个表格展示了一些输入宏的例子，以及他们最终被转化的结果

=========================  ==========
宏                         结果
=========================  ==========
``Hello World``            ``HelloWorld``
``Hello <Space> World``    ``Hello World``
``< S p a c e >``          ``<Space>``
``Hello <Home> <Delete>``  ``ello``
``Hello <Backspace>``      ``Hell``
=========================  ==========

键盘事件不会立即处理，而是要等到下一次进行消息循环。因此必须要调用 :ref:`splash:wait <splash-wait>` 来等待事件的响应执行

您也可以参考: `element:send_keys <scripting-element-object.html#splash-element-send-keys>`_ ,
:ref:`splash:send_text <splash-send-text>` .

.. _splash-send-text:

splash:send_text
####################################
发送一个文本，作为页面上下文的输入， 字面上逐个字符输入

**原型:** ``splash:send_text(text)``

**参数:**

- text: 作为输入的字符串

**返回值:** nil

**是否异步:** 否

键盘事件不会立即处理，而是要等到下一次进行消息循环。因此必须要调用 :ref:`splash:wait <splash-wait>` 来等待事件的响应执行

这个函数与 :ref:`splash:send_keys <splash-send-keys>` 一起涵盖了键盘输入的大部分需求，像自动填充和提交表单。

例1：选中第一个输入框，填充并提交表单:
::

    function main(splash)
        assert(splash:go(splash.args.url))
        assert(splash:wait(0.5))
        splash:send_keys("<Tab>")
        splash:send_text("zero cool")
        splash:send_keys("<Tab>")
        splash:send_text("hunter2")
        splash:send_keys("<Return>")
        -- 请注意如何使用splash:send_keys来改写程序
        -- splash:send_keys("<Tab> zero <Space> cool <Tab> hunter2 <Return>")
        assert(splash:wait(0))
        -- ...
    end


例2：使用JavaScript代码或者 :ref:`splash:mouse_click <splash-mouse-click>` 来选中输入框

我们不能总是假定使用 <Tab> 会选中对应的输入框 <Enter> 会提交表单，选中输入框还可以通过点击它或者将焦点移动到它来完成。
提交表单也可以通过触发表单中的提交事件或者点击提交按钮来完成.

下面这个例子将会先选中输入框，然后通过函数 :ref:`splash:mouse_click <splash-mouse-click>` 来点击提交按钮完成表单的提交
。这里假设splash传入两个参数:username和password
::

    function main(splash, args)
        function focus(sel)
            splash:select(sel):focus()
        end

        assert(splash:go(args.url))
        assert(splash:wait(0.5))
        focus('input[name=username]')
        splash:send_text(args.username)
        assert(splash:wait(0))
        focus('input[name=password]')
        splash:send_text(args.password)
        splash:select('input[type=submit]'):mouse_click()
        assert(splash:wait(0))
        -- Usually, wait for the submit request to finish
        -- ...
    end

您也可以参考: `element:send_text <./scripting-element-object.html#splash-element-send-text>`_ , :ref:`splash:send_keys <splash-send-keys>`

.. _splash-select:

splash:select
##########################################
利用CSS选择器，在HTML DOM中选择第一个匹配的元素

**原型:** ``element = splash:select(selector)``

**参数:**

- selector: 有效的CSS选择器

**返回值:** `Element <./scripting-element-object.html#splash-element>`_ 对象的值

**是否异步:** 否

使用 :ref:`splash:select <splash-select>` 您可以通过CSS选择器来获取对应的元素，
就好像在浏览器中使用 `document.querySelector <https://developer.mozilla.org/en-US/docs/Web/API/Document/querySelector>`_ 一样。
它返回的元素对象是 `Element <./scripting-element-object.html#splash-element>`_ 。
这个对象包含许多有用的属性和方法，这些属性和方法与在JavaScript中类似

如果使用的CSS选择器不能找到对应的元素，函数将会返回 ``nil`` 。如果您输入的CSS选择器无效，将会抛出一个错误.

例1：选择类名为 ``element`` 的元素并且返回它所有兄弟元素的类名:
::

    local treat = require('treat')

    function main(splash)
        assert(splash:go(splash.args.url))
        assert(splash:wait(0.5))

        local el = splash:select('.element')
        local seen = {}
        local classNames = {}

        while el do
          local classList = el.node.classList
          if classList then
            for _, v in ipairs(classList) do
              if (not seen[v]) then
                classNames[#classNames + 1] = v
                seen[v] = true
              end
            end
          end

          el = el.node.nextSibling
        end

        return treat.as_array(classNames)
    end

例2：判断返回的元素是否存在
::

    function main(splash)
        -- ...
        local el = assert(splash:select('.element'))
        -- ...
    end


.. _splash-select-all:

splash:select_all
###################
在页面中通过CSS选择器来选择对应的DOM元素，并以列表的形式返回匹配的元素

**原型:** ``elements = splash:select_all(selector)``

**参数:**

- selector: 有效的CSS选择器

**返回值:** 包含 `Element <./scripting-element-object.html#splash-element>`_ 对象的列表

**是否异步:** 否

与 :ref:`splash:select <splash-select>` 不同的是，它会在table中返回所有匹配到的元素的列表。

如果没有元素被匹配到，它会返回一个 ``{}`` , 如果传入的选择器无效，则会抛出一个错误

下面这个例子是选择所有 ``<img />`` 元素并获取它们的 ``src`` 属性
::

    local treat = require('treat')

    function main(splash)
        assert(splash:go(splash.args.url))
        assert(splash:wait(0.5))

        local imgs = splash:select_all('img')
        local srcs = {}

        for _, img in ipairs(imgs) do
          srcs[#srcs+1] = img.node.attributes.src
        end

        return treat.as_array(srcs)
    end


.. [#1] 这是由于Splash一次只能加载一个页面，在加载新页面之前的老页面不会被保存
.. [#2] 这个意思是说我们在使用API时传入的参数都可以通过这个参数获取到
.. [#3] 这里的主页是指参数中url对应的页面，而页面中的资源等等不包含在里面
.. [#4] 这里的原文是 pair，由于我对lua不是很了解，这里翻译为元组可能更好理解
.. [#5] 我觉得这里官方应该是鼓励我们尽量返回能够被序列化的数据，而不要返回类似DOM元素的内容
.. [#6] 这里的意思应该是不要使用JavaScript的复杂结构
.. [#7] lua函数调用有两种形式，带形参名称的使用``{}`` 不带的使用 ``()``
.. [#8] Splash只能管理异步函数而无权管理这些同步函数,只能被动的等待函数执行完成
