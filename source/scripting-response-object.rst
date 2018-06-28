.. _response-objects:

响应对象
=============================================

Respons Object 作为某些 Splash 方法执行的结果被返回(像 `splash:http_get <./scripting-ref.html#splash-http-get>`_
或者 `splash:http_post <./scripting-ref.html#splash-http-post>`_ );它也被传递给某些回调函数
(例如 `splash:on_response <./scripting-ref.html#splash-on-response>`_ 和 `splash:on_response_headers <./scripting-ref.html#splash-on-response-headers>`_ 回调)
这个对象包含响应的相关内容

.. _response-url:

response.url
---------------------------------
响应的URL，在发生重定向的情况下，这个值是请求中的最后一个url

这个字段为只读的字段

.. _response-status:

response.status
------------------------------------
响应的HTTP状态码

这个字段是一个只读字段

.. _response-ok:

response.ok
----------------------------------
成功获取到响应时为 ``true`` 否则为 ``false``

例如:
::

    local reply = splash:http_get("some-bad-url")
    -- reply.ok == false

这个字段是一个只读字段

.. _response-headers:

response.headers
-------------------------------------
一个用Lua中table结构表示的HTTP的响应头(一个由名称到键值的映射);键值为响应头对应的名称, 值为响应头对应的值.

查询时是不区分大小写的，因此 ``response.headers['content-type']`` 与 ``response.headers['Content-Type']`` 是相同的.

这个字段是一个只读字段

.. _response-info:

response.info
--------------------------------------
它是一个lua中的table结构，以 `HAR response <http://www.softwareishard.com/blog/har-12-spec/#response>`_ 的格式来表示响应数据

该字段是一个只读字段

.. _response-body:

response.body
----------------------------------------
原始响应信息(为一个 `二进制对象 <./scripting-binary-data.html#binary-objects>`_ )

如果您想通过Lua来处理这个响应体，可以使用 `treat.as_string <./scripting-libs.html#treat-as-string>`_ 来将二进制数据转化为字符串

默认情况下 :ref:`response.body <response-body>` 属性在回调 `splash:on_response <./scripting-ref.html#splash-on-response>`_
您可以使用 `splash.response_body_enabled <./scripting-ref.html#splash-response-body-enabled>`_ 或者
`request:enable_response_body <./scripting-request-object.html#splash-request-enable-response-body>`_ 来启用它

.. _response-request:

response.request
--------------------------------
一个对应的请求对象

这个字段是一个只读字段

.. _response-abort:

response:abort
--------------------------------
**原型:** ``response:abort()``

**返回值:** nil

**是否异步:** 否

放弃接收响应体, 这个函数仅仅在还没有开始下载响应体时起作用，您只能在 `splash:on_response_headers <./scripting-ref.html#splash-on-response-headers>`_ 定义的回调中使用
