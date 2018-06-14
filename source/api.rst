.. _splash-http-api:

Splash HTTP API
======================================================
请先参阅 `安装 <./Installation.html>`_ 部分的内容安装 ，然后启动Splash

splash 是通过HTTP API来进行操作的。对于下面列举出来的所有端点都可以采用GET方式传入参数，或者将参数编码为JSON格式并使用 ``Content-Type: application/json`` 请求头使用POST方式发出

多数splash 端点都提供了 `run <./api.html#id19>`_ 和 `execute <./api.html#id18>`_ 两种功能，这意味着它们能执行自定义的任意的Lua渲染脚本

另外的一些端点可能用于某些特殊的场合，比如 `render.png <./api.html#render-png>`_ 可以在你不提供任何进一步处理的情况下使用PNG格式返回一个网页截图，
另外如果你不需要与页面进行交互那么 `render.json <./api.html#render-json>`_ 将会使程序编写变得更为方便 [#1]_

.. _render-html:

render.html
-------------------------------------------------
返回一个经过Javascript渲染之后的页面的HTML代码
参数：

.. _url:

**url: string: required** [#2]_
    需要进行渲染的页面的url，这个参数必须提供

.. _baseurl:

**baseurl : string : optional**
    用于呈现页面的基础URL

    基本HTML内容将从url参数中提供的URL中获取，而用于呈现页面的HTML文本中的相对引用资源是使用baseurl参数中给定的URL作为基础获取的 [#3]_ 。
    您可以在这个讨论中获取更多信息: render.html 返回好像被浏览器给破坏了

.. _timeout:

**timeout : float : optional**
    渲染的超时值，以秒为单位(默认为30s)

    默认情况下，允许的最大超时值为90s，您也可以在启动时通过 ``--wait-timeout`` 参数来修改这个值，比如使用这个命令来使默认最大超时时间为5分钟
    ::

        $ docker run -it -p 8050:8050 scrapinghub/splash --max-timeout 300

.. _resource-timeout:

**resource_timeout : float : optional**
    单个网络请求的超时时间

    更多信息请查看：splash:on_request 的 ``request:set_timeout(timeout)`` 方法 和 splash.resource_timeout属性

.. _wait:

**wait : float : optional**
    当收到响应包后等待的时长，单位为s默认为0，如果您所请求的页面中包含一些异步与延时加载的JavaScript脚本时请添加上这个值，
    当等待时间为0时这些JavaScript代码不会执行。当你想要获取整个页面的PNG和JPEG图片的话，最好也加上此值（请查看 :ref:`render_all <render-all>` ）

    这个等待值必须小于timeout这个超时值 [#4]_

.. _proxy:

**proxy : string : optional**
    指定代理配置文件的名称，或者代理url。参阅代理配置

    代理url的格式为: ``[protocol://][user:password@]proxyhost[:port])``

    其中protocol 为 http或者socks5,如果未指定端口，将会默认采用1080 端口

.. _js:

**js : string : optional**
    JavaScript配置文件名称，请参阅JavaScript配置

.. _js-source:

**js_source : string : optional**
    可被页面环境执行的JavaScript代码。请参阅：使用页面环境执行JavaScript代码

.. _filters:

**filters : string : optional**
    使用分号分隔的请求过滤的名称列表，请参阅请求包过滤

.. _allowed-domains:

**allowed_domains : string : optional**
    使用分号分隔的允许访问的域名列表。如果该值存在，Splash将不会加载任何来自不在此列表中的域以及不在此列表中的域的子域的任何内容。

.. _allowed-comtent-types:

**allowed_content_types : string : optional**
    使用分号分隔的允许内容类型列表。当该值存在时，如果请求包对应的响应包类型不在列表中那么该请求包将会被拒绝。允许内容的通配符使用 `fnmatch <https://docs.python.org/3/library/fnmatch.html>`_ 语法

.. _forbidden_content_types:

**forbidden_content_types**
    使用分号分隔的拒绝内容类型列表。当该值存在时，如果请求包对应的响应包类型在列表中那么该请求包将会被拒绝。允许内容的通配符使用 `fnmatch <https://docs.python.org/3/library/fnmatch.html>`_ 语法

**viewport : string : optional**
    用来渲染js的浏览器视口的大小，主要是宽和高,该值单位为像素,格式为"<宽>x<高>",比如 800x600，默认值为1024x768.

    这个值在生成PNG和JPEG的情况下十分重要，所有渲染端点都支持这个参数，因为JavaScript代码的执行以视口大小为依据

    出于向后兼容的考虑，它允许使用full为值 ``viewport=full`` 它的效果与使用 ``render_all=1`` 相同(请参阅: render_all)
**images : integer : optional**
    是否加载图片，当值为1时表示允许加载图片，为0时表示禁止加载

    在某些情况下即使设置了值为0，也会加载图片，你也可以使用请求包过滤的方式根据url来屏蔽不想看见的内容

**headers : JSON array or object : optional**
    为首个发出去的http请求包设置请求头

    这个参数仅仅在 ``application/json`` 类型的POST包中使用，它可以是使用(header_name, header_value)这种格式的数据组成的json对象，其中header_name表示请求头某项的键，header_value表示请求头某项的值

    其中“User-Agent”这个头比较特殊，它作用在所有请求包上而不仅仅是首个包

**body : string : optional**
    如果HTTP请求方式为POST，那么该值将作为请求体，此时默认的content-type请求头为 ``application/x-www-form-urlencoded``

**http_method : string : optional**
    传出的Splash包的请求方法 [#5]_ ，默认的方法是GET，当然Splash也支持POST。

**save_args : JSON 数据或者是一个以分号为分隔符的字符串 : optional**
    这是一个放入缓存中的参数名称列表，Splash将会把列表中对应的参数值放入到内部缓冲中，并通过Splash响应头的 ``X-Splash-Saved-Arguments`` 参数
    中进行返回，该参数会将对应值以SHA1列表的方式返回。这个返回值是一个以分号分隔的字符串，每个部分以键 = 哈希值 这种方式展现
    ::

        name1=9a6747fc6259aa374ab4e1bb03074b6ec672cf99;name2=ba001160ef96fe2a3f938fea9e6762e204a562b3

    在客户端中可以使用 `load_args <#arg-load-args>`_  参数将响应包头部的对应哈希值转化为真实的参数值。当参数值较长而且不变的情况下使用这种方式将会是一个很好的选择
    （特别是在表示js_source和lua_source的时候）

load_args : JSON 对象或者是一个字符串 : optional
    将参数值从缓存中加载出来，load_args 参数值必须是 ``{"name": "<SHA1 hash>", ...}``格式的json对象或者是响应包头
    的 ``X-Splash-Saved-Arguments`` 参数所对应的原始字符(以分号分隔的 name=hash 格式的字符串)

    针对每个存在在load_args 中的参数，Splash在取出对应的值的时候会使用hash值作为键值，从缓存中查找出对应的真实数据，如果对应的hash值在缓存中
    能够找到相应的值，那么会将这个找到的值作为参数的真实值，然后向往常一样处理请求

    如果在缓存中没有找到对应的值，那么Splash会返回一个 HTTP 498 状态码。在这种情况下客户端需要再次使用save_args 传入完整的参数值并 提交HTTP请求

    Splash通过load_args 和 save_args 参数的方式，在请求中不发送每个请求的大参数，以便达到节约网络流量的目的（通常在带有js_source和lua_source的参数中使用将会是一个很好的选择）

    splash使用LUR缓存来存储这些值, 在存储时限定了参数的条目数量，并且在每次重启Splash之后都会清理缓存，换句话说，Splash中的缓存不是持久性的
    客户端应该要有重发这些参数的操作

**html5_media : integer : optional**
    是否支持H5中的多媒体（比如<video> 标签）。使用1表示支持，0表示不支持，默认为0

    Splash默认是不支持H5 多媒体的，它可能会造成程序的不稳定。在未来的版本中可能会默认支持H5，所以在那以后如果不需要使用H5，
    那么请将参数设置为0 ``html5_media = 0``

    更多信息请参阅 splash.html5_media_enabled.

示例
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
curl 示例 ::

    curl 'http://localhost:8050/render.html?url=http://domain.com/page-with-javascript.html&timeout=10&wait=0.5'

返回的数据包都被编码为UTF-8，render.html端点会将返回的HTML也编码为UTF-8，哪怕是在HTML的标签想下面这样中明确指定了编码方式
::

    <meta http-equiv="Content-Type" content="text/html; charset=iso-8859-1">

render.png
------------------------------------
将页面渲染的结果以图片的方式返回（格式为png）

参数：

它的许多参数都与render.html的相同, 相比较于前者它多出来下面几个参数

**width : integer : optional**
    将生成图片宽度调整为指定宽度，以保持宽高比

**height : integer : optional**
    将生成的图片裁剪到指定的高度，通常与width参数一起使用以生成固定大小的图片

.. _render-all:

**render_all : int : optional**
    它可能的值有0和1，表示在渲染前扩展视口以容纳整个Web 页面(即使整个页面很长)，默认值为 ``render_all=0``

    .. note::
        render_all = 1 时需要一个不为0 的 wait值，这是一个不幸的限制，但是目前来看只能通过这种方式使得在 ``render_all = 1``
        这种情况下整个渲染变得可靠

**scale_method : string : optional**
    可能的值有 ``raster``(默认值) 和 ``vector``, 如果值为 raster, 通过宽度执行的缩放操作是逐像素的，如果值为vector, 在缩放是是按照
    元素在进行的 [#6]_

    .. note::
        基于矢量的重新缩放更加高效，并且会产生更清晰的字体和更锐利的元素边界，但是可能存在渲染问题，请谨慎使用

示例
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
curl 示例
::

    # 使用超时值进行渲染
    curl 'http://localhost:8050/render.png?url=http://domain.com/page-with-javascript.html&timeout=10'

    # 将生成图片尺寸设置为:320x240
    curl 'http://localhost:8050/render.png?url=http://domain.com/page-with-javascript.html&width=320&height=240'

render.jpeg
----------------------------------------------
将页面渲染的结果以图片的方式返回（格式为jpeg）

参数:

它的参数与render.png大致相同，相比于前者，它多出一个参数

**quality : integer : optional**
    该参数表示生成图片的质量，大小在0~100之前，默认值为 75

    .. note::
        该值应该尽量避免高于95，当 ``quality=100`` 时，会禁用JPEG的相关压缩算法，导致大量的图片实际上得不到质量的提升

示例
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
curl 示例
::

    # 生成默认质量的图片
    curl 'http://localhost:8050/render.jpeg?url=http://domain.com/'

    # 生成高质量的图片
    curl 'http://localhost:8050/render.jpeg?url=http://domain.com/&quality=30'

render.har
---------------------------------------
以HAR格式返回Splash与目标站点的交互信息，里面包含了请求信息、响应信息、时间信息和头信息等等

您可以使用在线的 `HAR查看工具 <http://www.softwareishard.com/har/viewer/>`_ 来查看该端点返回的具体信息。
这些信息与我们使用Chrome和FirFox等浏览器的Network工具得到的信息十分相似

目前这个端点不会公开原始的请求信息，目前只有一些元数据信息比如包头信息和时间信息是可用的，只有当‘response_body’参数被设置为1的时候才会包含响应体的信息

它的参数与render.html相似，多出来的参数如下：

**response_body : int : optional**
    可选的值有0和1，当值为1时，响应体的信息会被包含在返回的HAR数据中，默认情况下 ``response_body = 0``

render.json
----------------------------------------
将经过JavaScript渲染的页面信息以json格式返回，它可以返回HTML，PNG等其他信息。返回何种信息由相关参数指定

参数:

参数与 render.jpeg的参数相似，多余的参数如下:

**html : integer : optional**
    返回值中是否包含HTML，1为包含，0表示不包含，默认为0

**png : integer : optional**
    返回值中是否包含PNG图片，1为包含，0表示不包含，默认为0

**jpeg : integer : optional**
    返回值中是否包含JPEG图片，1为包含，0表示不包含，默认为0

**iframes : integer : optional**
    返回值中是否包含子frame的信息，1为包含，0表示不包含，默认为0

**script : integer : optional**
    是否在返回中包含执行的javascript final语句的结果（请参阅：在页面上下文中执行用户自定义的JavaScript代码），可选择的值有1（包含）
    0（不包含），默认是0

**history : integer : optional**
    返回值中是否包含主页面的历史请求/响应数据，可选择的值有1（包含）0（不包含），默认是0

    使用该参数来获取HTTP响应码和对应的头信息，它只会返回最主要的请求/响应信息（也就是说页面加载的资源信息和对应请求的AJAX信息是不会返回的）
    要获取请求和响应的更详细信息请使用 har参数
**har : integer : optional**
    是否在返回中包含 HAR信息，可选择的值有1（包含）0（不包含），默认是0，如果这个选项被打开，那么它将会在har键中返回与render.har 一样的数据

    默认情况下响应体未包含在返回中，如果要返回响应体，可以使用参数 response_body

**response_body : int : optional**
    可选择的值有1（包含）0（不包含），如果值为1，那么将会在返回的HAR信息中包含响应体的内容。在参数har 和 history为0的情况下该参数无效

示例
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
默认情况下，返回当前页面的url，请求url，页面标题，主frame的尺寸 [#7]_
::

    {
        "url": "http://crawlera.com/",
        "geometry": [0, 0, 640, 480],
        "requestedUrl": "http://crawlera.com/",
        "title": "Crawlera"
    }

设置参数 html=1 ，以便让HTML能够加入到返回值中
::

    {
        "url": "http://crawlera.com/",
        "geometry": [0, 0, 640, 480],
        "requestedUrl": "http://crawlera.com/",
        "html": "<!DOCTYPE html><!--[if IE 8]>....",
        "title": "Crawlera"
    }

设置参数 png=1 以便使渲染后的截图数据以base64的编码方式加入到返回值中
::

    {
        "url": "http://crawlera.com/",
        "geometry": [0, 0, 640, 480],
        "requestedUrl": "http://crawlera.com/",
        "png": "iVBORw0KGgoAAAAN...",
        "title": "Crawlera"
    }

同时设置html=1和png=1，能同时获取到截图和HTML代码。这样就保证了截图与HTML相匹配

通过添加 iframes=1，能够在返回中得到对应的frame的信息
::

    {
        "geometry": [0, 0, 640, 480],
        "frameName": "",
        "title": "Scrapinghub | Autoscraping",
        "url": "http://scrapinghub.com/autoscraping.html",
        "childFrames": [
            {
                "title": "Tutorial: Scrapinghub's autoscraping tool - YouTube",
                "url": "",
                "geometry": [235, 502, 497, 310],
                "frameName": "<!--framePath //<!--frame0-->-->",
                "requestedUrl": "http://www.youtube.com/embed/lSJvVqDLOOs?version=3&rel=1&fs=1&showsearch=0&showinfo=1&iv_load_policy=1&wmode=transparent",
                "childFrames": []
          }
      ],
      "requestedUrl": "http://scrapinghub.com/autoscraping.html"
    }

请注意，iframe可以嵌套

同时设置iframe=1和html=1,以获取所有iframe和HTML（包括iframe的HTML代码）
::

    {
        "geometry": [0, 0, 640, 480],
        "frameName": "",
        "html": "<!DOCTYPE html...",
        "title": "Scrapinghub | Autoscraping",
        "url": "http://scrapinghub.com/autoscraping.html",
        "childFrames": [
            {
                "title": "Tutorial: Scrapinghub's autoscraping tool - YouTube",
                "url": "",
                "html": "<!DOCTYPE html>...",
                "geometry": [235, 502, 497, 310],
                "frameName": "<!--framePath //<!--frame0-->-->",
                "requestedUrl": "http://www.youtube.com/embed/lSJvVqDLOOs?version=3&rel=1&fs=1&showsearch=0&showinfo=1&iv_load_policy=1&wmode=transparent",
                "childFrames": []
            }
        ],
        "requestedUrl": "http://scrapinghub.com/autoscraping.html"
    }

与'html = 1'不同，'png = 1'不会影响childFrame中的数据。

当需要执行JavaScript代码的时候（请参阅：在页面上下文中执行用户自定义的JavaScript代码），设置 ‘script=1’ 以便在结果中返回代码执行的结果
::

    {
        "url": "http://crawlera.com/",
        "geometry": [0, 0, 640, 480],
        "requestedUrl": "http://crawlera.com/",
        "title": "Crawlera",
        "script": "result of script..."
    }

可以在JavaScript代码中使用函数 console.log() 来记录相关信息，设置参数 console=1 以便在返回结果中包含控制台输出
::

    {
        "url": "http://crawlera.com/",
        "geometry": [0, 0, 640, 480],
        "requestedUrl": "http://crawlera.com/",
        "title": "Crawlera",
        "script": "result of script...",
        "console": ["first log message", "second log message", ...]
    }

curl实例
::

    # 返回完整的信息
    curl 'http://localhost:8050/render.json?url=http://domain.com/page-with-iframes.html&png=1&html=1&iframes=1'

    # 页面自身的HTML代码，元数据信息以及所有的iframe信息
    curl 'http://localhost:8050/render.json?url=http://domain.com/page-with-iframes.html&html=1&iframes=1'

    # 只返回元数据信息 (例如 页面/iframes 标题和url)
    curl 'http://localhost:8050/render.json?url=http://domain.com/page-with-iframes.html&iframes=1'

    # 渲染页面并将页面裁剪为 320x240的同时, 不返回iframe的相关信息
    curl 'http://localhost:8050/render.json?url=http://domain.com/page-with-iframes.html&html=1&png=1&width=320&height=240'

    # Render page and execute simple Javascript function, display the js output
    curl -X POST -H 'content-type: application/javascript' \
    -d 'function getAd(x){ return x; } getAd("abc");' \
    'http://localhost:8050/render.json?url=http://domain.com&script=1'

    # 渲染页面并执行简单的JavaScript代码, 显示js的执行结果和在控制台的输出
    curl -X POST -H 'content-type: application/javascript' \
        -d 'function getAd(x){ return x; }; console.log("some log"); console.log("another log"); getAd("abc");' \
        'http://localhost:8050/render.json?url=http://domain.com&script=1&console=1'

execute
------------------------------------------
执行自定义的渲染脚本并返回对应的结果

render.html, render.png, render.jpeg, render.har 和 render.json已经涵盖了许多常见的情形，但是在某些时候这些仍然不够，
这个端口允许用户编写自定义的脚本

参数:

**lua_source : string : required**
    需要浏览器执行的脚本代码，请查看 Splash脚本教程 以获取更多信息

**timeout : float : optional**
    与render.html中的timeout参数含义相同

**allowed_domains : string : optional**
    与render.html中的allowed_domains参数含义相同

**proxy : string : optional**
    与render.html中的proxy参数含义相同

**filters : string : optional**
    与render.html中的filters参数含义相同

**save_args : json对象或者是以分号分隔的字符串 : optional**
    与render.html中的save_args参数相同，请注意你不仅能保存Splash中的默认参数，也可以保存其他任何参数

**load_args : JSON object or a string : optional**
    与render.html中的load_args参数相同，请注意你不仅能加载Splash中的默认参数，也可以加载其他任何参数

您可以传入任何类型的参数，所有在端点execute中传入的参数在脚本中都可以通过splash.args这个table对象 来访问

run
------------------------------------------
这个端点与execute具有相同的功能，但是它会自动将 ``lua_source`` 包装在 ``function main(splash, args) ... end`` 结构中
比如您在execute 端点中传入脚本::

    function main(splash, args)
        assert(splash:go(args.url))
        assert(splash:wait(1.0))
        return splash:html()
    end

在使用run端点时只需要传入
::

    assert(splash:go(args.url))
    assert(splash:wait(1.0))
    return splash:html()

在页面上下文中执行用户自定义的JavaScript代码
----------------------------------------------------------
.. note::
      您也可以参考: 在Splash中执行JavaScript脚本

Splsh支持在页面上下文中执行JavaScript代码，这些JavaScript代码在页面加载完成之后执行（包括由'wait'参数定义的等待时间）。但是它允许在页面
渲染之前通过Javascript代码来修改渲染的结果。

可以通过js_source 这个参数来执行js代码。参数中保存的是需要执行的JavaScript代码

请注意，浏览器和代理限制了可以使用GET发送的数据量，所以采用POST发送 ``content-type: application/json`` 类型的请求包将会是一个不错的选择

Curl example:
::

    # 渲染页面并动态修改标题
    curl -X POST -H 'content-type: application/json' \
        -d '{"js_source": "document.title=\"My Title\";", "url": "http://example.com"}' \
        'http://localhost:8050/render.html'

另一个发送POST请求的方式是设置请求包的 ``Content-Type`` 为 ‘application/javascript’，并在请求体中包含需要执行的js代码
curl:
::

    # 渲染页面并动态修改标题
    curl -X POST -H 'content-type: application/javascript' \
        -d 'document.title="My Title";' \
        'http://localhost:8050/render.html?url=http://domain.com'

可以通过使用 render.json这个端点，并设置参数 script = 1 来获取js函数在页面上下文执行的结果

JavaScript配置
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Splash允许使用“JavaScript 配置”的方式来预加载JavaScript文件，配置文件中的JavaScript代码将会在页面加载之后执行，但是会在请求中定义的js代码被执行之前

预加载的文件可以被包含在用户发送的POST请求中

为了开启splash对JavaScript文件的支持，在启动splash服务的时候可以使用参数 ``--js-profiles-path=<path to a folder with js profiles>``
::

    python3 -m splash.server --js-profiles-path=/etc/splash/js-profiles

.. note::
    请参阅 splash 版本

然后根据上面参数中给定的名称创建文件夹，在文件夹中创建需要加载的js文件（请注意，文件编码格式必须为utf-8）这些文件都会在适当的时候被加载
比如这样的一个目录结构
::

    /etc/splash/js-profiles/
        mywebsite/
                  lib1.js

为了应用这些JavaScript的配置，请在请求中添加参数 ``js=mywebsite``
::

    curl -X POST -H 'content-type: application/javascript' \
        -d 'myfunc("Hello");' \
        'http://localhost:8050/render.html?js=mywebsite&url=http://domain.com'

请注意，这个例子中假设myfunc是在lib1.js中定义的一个JavaScript函数

Javascript安全
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
如果splash是通过 ``--js-cross-domain-access`` 的方式启动
::

    $ docker run -it -p 8050:8050 scrapinghub/splash --js-cross-domain-access

此时将允许JavaScript代码访问非原始安全页面中的iframe中的内容（一般在浏览器中是不允许这个做） [#8]_ 。
这个特性在爬取的时候非常有用，比如提取iframe中的HTML代码，它的一个使用的例子如下:
::

    curl -X POST -H 'content-type: application/javascript' \
        -d 'function getContents(){ var f = document.getElementById("external"); return f.contentDocument.getElementsByTagName("body")[0].innerHTML; }; getContents();' \
        'http://localhost:8050/render.html?url=http://domain.com'

这段JavaScript代码会查找一个id为"external" 的iframe，然后加载它的HTML代码

请注意：允许跨源调用JavaScript代码可能会造成一些安全问题，因为启用这些特性可能会泄漏一些敏感信息（例如cookie），
当禁用跨域安全时某些网站不会被加载，因此这个特性默认是关闭的

请求过滤
---------------------------------------------
splash允许通过 `Adblock Plus <https://adblockplus.org/>`_ 规则来对请求包进行过滤。您可以使用 `EasyList <https://easylist.adblockplus.org/en/>`_
规则来过滤广告和跟踪代码（从而提高页面的渲染速度）。或者您也可以自己书写规则来过滤一些请求（例如书写规则来避免渲染iframe，MP3，自定义字体等等）

要开启对请求的过滤，需要在启动splash的时候加上参数 ``--filters-path``
::

    python3 -m splash.server --filters-path=/etc/splash/filters

.. note::
    可以参阅 splash 版本

``filters-path`` 所指向的目录中必须包含以 Adblock Plus 格式编写的规则的``.txt``文件
您可以从 `EasyList <https://easylist.adblockplus.org/en/>`_ 的网站下载文件 ``easylist.txt`` 文件放到对应目录中，或者创建一个 ``.txt`` 文件编写自己的规则
例如，让我们创建一个过滤器，以阻止加载的ttf和woff格式的自定义字体（在Mac OS 中，可能会由于qt的bug导致splash产生一个段错误）
::

    ! put this to a /etc/splash/filters/nofonts.txt file
    ! comments start with an exclamation mark

    .ttf|
    .woff|

要使用这个规则您可以在请求包中添加参数 ``filters=nofonts``
::

    curl 'http://localhost:8050/render.png?url=http://domain.com/page-with-fonts.html&filters=nofonts'

您可以添加多个规则并用逗号隔开它们
::

    curl 'http://localhost:8050/render.png?url=http://domain.com/page-with-fonts.html&filters=nofonts,easylist'

如果对应目录中存在一个 ``default.txt`` 那么文件里面的规则将会在默认情况下执行，即使您没有使用参数 ``filters``
如果您不想使用默认的规则，您可以设置 ``filters=none``

只有与情求相关的资源才会被过滤掉，加载主页的请求不会被过滤 [#9]_
如果您确实想要这么做，请考虑在将URL发送到Splash之前使用Adblock Plus过滤器对URL进行检查（对python来说可以使用库 `adblockparser <https://github.com/scrapinghub/adblockparser>`_ ）

您可以点击下面的链接来学习Adblock Plus过滤的语法

- `https://adblockplus.org/en/filter-cheatsheet <https://adblockplus.org/en/filter-cheatsheet>`_

- `https://adblockplus.org/en/filters <https://adblockplus.org/en/filters>`_

splash不能支持所有的Adblock Plus过滤规则，它有一些对应的限制

- 元素隐藏规则不受支持；过滤器可以过滤掉某些网络请求，但是并不能隐藏已加载页面的内容

- 只支持 ``domain`` 选项

splash不支持的规则会被默默的丢弃

.. note::
    如果您想停止下载图片，请选择 'images' 参数,它不需要使用基于url的过滤器来进行过滤，它可以过滤掉那些使用基于url的过滤器很难过滤掉的图片

.. warning::
    如果您的过滤器中含有大量的规则，您就得需要安装 `pyre2 <https://github.com/axiak/pyre2>`_ 这个库。（这主要是针对从 `EasyList <https://easylist.adblockplus.org/en/>`_ 中下载下来的文件）

    在splash中传统的re库会比re2慢上 1000x+ 的时间，当有大量的规则而未使用re2 时下载文件会比过滤文件更快，但是使用re2会使规则的匹配更加迅速

    您需要确认您未通过PyPI来下载re2的0.2.20（这个版本已经被放弃了）;您应该使用最新版本

代理配置
------------------------------------------
splash 支持代理配置，它允许通过 ``proxy`` 参数来设置每个请求的代理处理规则

要支持代理配置，可以在启动splash的时候使用参数 ``--proxy-profiles-path=<path to a folder with proxy profiles>`` :
::

    python3 -m splash.server --proxy-profiles-path=/etc/splash/proxy-profiles

.. note::
    如果您通过docker启动，请参数 文件共享

然后在指定文件夹中创建一个以代理配置的规则编写的INI文件，例如在文件 ``/etc/splash/proxy-profiles/mywebsite.ini`` 中写下这些内容
::

    [proxy]

    ; required
    host=proxy.crawlera.com
    port=8010

    ; optional, default is no auth
    username=username
    password=password

    ; optional, default is HTTP. Allowed values are HTTP and SOCKS5
    type=HTTP

    [rules]
    ; optional, default ".*"
    whitelist=
        .*mywebsite\.com.*

    ; optional, default is no blacklist
    blacklist=
      .*\.js.*
      .*\.css.*
      .*\.png

whitelist 和 blacklist是以换行符分隔的正则表达式。如果url命中了白名单中的某项并且未命中黑名单中的任何一项，
此时就使用在 ``[proxy]`` 节中定义的代理，否则就不使用代理

要使用对应的规则，可以在请求中添加 ``proxy=mywebsite`` 参数
::

    curl 'http://localhost:8050/render.html?url=http://mywebsite.com/page-with-javascript.html&proxy=mywebsite'

如果存在一个 ``default.ini`` 文件，那么会默认使用这个，即使你没有指定 ``proxy`` 参数，如果您有 ``default.ini`` 但是不想使用它，
可以将 ``proxy`` 参数的值设置为 ``none``

其他端点
----------------------------------------

_gc
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
可以向 ``/_gc`` 端点发送一个POST请求来回收一些内存
::

    curl -X POST http://localhost:8050/_gc

它主要运行python的垃圾回收器，并清理webkit的缓存

_debug
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
可以向端点 ``/_debug`` 发送一个GET请求来获取splash历程的调试信息（RSS的最大使用量、使用的文件描述符的数量、存活的请求、请求队列的长度、
存活对象的个数）
::

    curl http://localhost:8050/_debug


_ping
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
向_ping端点发送一个GET请求可以ping splash的历程
::

    curl http://localhost:8050/_ping

如果splash历程存活，那么会返回 “ok”状态 和 RSS的最大使用数

.. [#1] 从后面对它的介绍可以看到，它会将响应的相关内容转化为JSON格式，这样我们就能很方便的进行解析了
.. [#2] 这里的格式为：参数名: 数据类型: 参数类型,参数类型有两种required表示必须提供，optional表示可选
.. [#3] 这里它针对的是相对路径的URI，它会以baseurl作为基础最终拼接成一个完整的路径
.. [#4] 超时值是指发送请求到接受请求并返回的所有时间之和，也就是说它包含了wait值在内
.. [#5] 注意，这里的请求方法是指由Splash发出去的请求包的请求方法，使用Splash进行渲染的过程实际上是分两步走的，第一步是向Splash发送请求包，然后由Splash向对应目标发送请求包，接着由Splash接收响应包，并渲染最后返给程序。不要理解成了向Splash发包的请求方式
.. [#6] 这里我的理解是一个进行的是位图的变换，一个是进行矢量图的变换
.. [#7] 原文这里是 geometry 在这里根据给出的例子我感觉还是用尺寸更为合适一些
.. [#8] 这里的原文是 then javascript code is allowed to access the content of iframes loaded from a security origin different to the original page (browsers usually disallow that)，翻译出来总感觉很别扭，我觉得这里的意思应该是跨站访问某些iframe并对它进行js渲染
.. [#9] 这里的意思是会过滤后续异步加载的请求而利用url针对主页面的请求不能被过滤，这样本省嵌入在主页中，与主页一起加载的内容不会被过滤掉
