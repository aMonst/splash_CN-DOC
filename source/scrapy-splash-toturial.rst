===========================================
scrapy-splash 教程
===========================================

scrapy-splash 是为了方便scrapy框架使用splash而进行的封装。它能与scrapy框架更好的结合，相比较于在python中
使用requests库或者使用scrapy 的Request对象来说，更为方便，而且能更好的支持异步。

.. _install:

安装
================================
针对python来说可以直接使用:
::

    $ pip install scrapy-splash

.. configuration:

配置
=================================
1. 首先需要启动Splash，启动命令如下
::

    $ docker run -p 8050:8050 scrapinghub/splash

关于Splash安装等更多信息请参考 `splash 安装文档 <./Installation.html>`_

#. 然后在对应scrapy项目的settings里面配置Splash服务的地址,例如:
::

    SPLASH_URL = 'http://192.168.59.103:8050'

#. 在settings中的DOWNLOADER_MIDDLEWARES 加上splash的中间件，并设置 HttpCompressionMiddleware 对象的优先级
::

    DOWNLOADER_MIDDLEWARES = {
        'scrapy_splash.SplashCookiesMiddleware': 723,
        'scrapy_splash.SplashMiddleware': 725,
        'scrapy.downloadermiddlewares.httpcompression.HttpCompressionMiddleware': 810,
    }

#. 在SPIDER_MIDDLEWARES 中安装splash的 SplashDeduplicateArgsMiddleware 中间件
::

    SPIDER_MIDDLEWARES = {
        'scrapy_splash.SplashDeduplicateArgsMiddleware': 100,
    }

#. 您还可以设置对应的过滤中间件——DUPEFILTER_CLASS
::

    DUPEFILTER_CLASS = 'scrapy_splash.SplashAwareDupeFilter'

#. 您可以设置scrapy.contrib.httpcache.FilesystemCacheStorage 来使用Splash的HTTP缓存
::

    HTTPCACHE_STORAGE = 'scrapy_splash.SplashAwareFSCacheStorage'

.. _usage:

用法
==========================================
如果要使用Splash来对页面进行渲染，您可以使用SplashRequest来代替原始scrapy中的Request， 示例如下:
::

    yield SplashRequest(url, self.parse_result, callback #任务完成之后对应的回调函数
        #args设置的是端点API的参数，关于API参数问题，请参考: `Splash HTTP API <./api.html>`_
        args={
            # 可选参数，表示spalsh在执行完成之后会等待一段时间后返回
            'wait': 0.5,
            #url是一个必须的参数，表明将要对哪个url进行请求
            'url' : "http://www.example.com",
            #http_method:表示Splash将向目标url发送何种请求
            'http_method': 'GET'
            # 'body' 用于POST请求，作为请求的请求体
            # 'lua_source' 如果需要执行lua脚本，那么这个参数表示对应lua脚本的字符串
        },
        endpoint='render.json', # optional; default is render.html
        splash_url='<url>',     # optional; overrides SPLASH_URL
        slot_policy=scrapy_splash.SlotPolicy.PER_DOMAIN,  # optional,
        # "meta" 是一个用来向回调函数传入参数的方式，在回调函数中的response.meta中可以取到这个地方传入的参数
    )

如果在splash中使用lua脚本，那么args中的内容会通过main函数的 ``splash.args`` 参数传入，其余的内容会通过第二个
参数 ``args`` 传入。

比如下面有一个简单的用户登录的例子:
::
    lua_script= '''
    function main(splash, args)

    local ok, reason = splash:go(args.url)
    user_name = args.user_name
    user_passwd = args.user_passwd
    user_text = splash:select("#email")
    pass_text = splash:select("#pass")
    login_btn = splash:select("#loginbutton")
    if (user_text and pass_text and login_btn) then
        user_text:send_text(user_name)
        pass_text:send_text(user_passwd)
        login_btn:mouse_click({})
    end

    splash:wait(math.random(5, 10))
    return {
        url = splash:url(),
        cookies = splash:get_cookies(),
        headers = splash.args.headers,
     }
    end'''

    yield SplashRequest(
        url=self.login_url,
        endpoint="execute",
        args={
            "wait": 30,
            "lua_source": lua_script,
            "user_name": "xxxx",  # 在Lua脚本中这个参数可用通过args.user_name取得
            "user_passwd": "xxxx",
        },
        meta = {"user_name" : "xxxx"},
        callback=self.after_login,
        errback=self.error_parse,
    )

上述代码提交了一个Splash的请求，在脚本中首先获取用户名和密码的输入框元素和对应的提交按钮元素，接着填入用户名和
密码，最后点击提交并返回对应的cookie。
回调函数 after_login 的代码如下:
::

    def after_login(self, response):
        #首先根据一定条件判断登录是否成功
        self.login_user = response.meta["user_name"] # 保存当前登录用户
        self.cookie = response.data["cookies"]  # 保存cookie

在回调函数中，可以通过response.data来获取lua脚本中返回的内容，而对应的HTML代码的获取方式与使用传统的Request方式
相同。

另外在回调函数中可以通过response.meta来获取Request中meta传入的参数。

上述示例演示了如何使用SplashRequest来像Splash发送渲染请求，以及如何在回调函数中获取lua脚本中的返回、
以及如何在回调函数中获取lua脚本中的返回、如何向回调函数传递参数。

当然您也可以使用常规的scrapy.Request来向Splash发送请求，发送的示例如下:
::

    yield scrapy.Request(url, self.parse_result, meta={
        'splash': {
            'args': {
                # 在此处设置端点API的参数
                'html': 1,
                'png': 1,

                # 'url' is prefilled from request url
                # 'http_method' is set to 'POST' for POST requests
                # 'body' is set to request body for POST requests
            },

            # optional parameters
            'endpoint': 'render.json',  # optional; default is render.json
            'splash_url': '<url>',      # optional; overrides SPLASH_URL
            'slot_policy': scrapy_splash.SlotPolicy.PER_DOMAIN,
            'splash_headers': {},       # optional; a dict with headers sent to Splash
            'dont_process_response': True, # optional, default is False
            'dont_send_headers': True,  # optional, default is False
            'magic_response': False,    # optional, default is True
        }
    })

splash 参数中的内容是用于splash的，使用这个参数表明我们希望向splash发送渲染请求。

最终它们会被组织成 ``request.meta['splash']`` 。在scrapy处理这些请求的时候根据这个来确定是否创建spalsh的
中间件，最终请求会被中间件以HTTP API的方式转发到splash中。

splash中各个参数的作用如下:

- meta['splash']['args'] 是最终发送到splash HTTP API的参数
    - ``url`` 表示目标站点的url
    - ``http_method`` 表示向url发送的HTTP的请求方式
    - ``body`` 是采用POST方式发送请求时，请求体的内容

- meta['splash']['cache_args'] 表示将要被作为缓冲的参数的列表字符串，以分号分隔
- meta['splash']['endpoint'] 表示对应的端点
- meta['splash']['splash_url'] 与settings文件中的 SPLASH_URL 作用相同，但是会优先采用这里的设置
- meta['splash']['splash_headers'] 即将发送到splash服务器上的请求头信息，注意，这里它不是最终发送到对应站点的请求头信息

由于本人水平有限以及当时项目需求并没有对它的用法做很深入的了解，更为详细的用法请参见: https://github.com/scrapy-plugins/scrapy-splash
