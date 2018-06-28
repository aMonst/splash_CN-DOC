.. _request-object:

请求对象
=======================================
请求对象通过 `splash:on_request <./scripting-ref.html#splash-on-request>`_ 定义的回调函数接收, 它们也可以作为 `response.request <./scripting-response-object.html#splash-response-request>`_

.. _attributes:

属性
--------------------------------------
请求对象中用许多属性来描述HTTP请求的相关信息.这些字段仅仅作为一种信息，即使对他们进行修改也不会对最终发送的HTTP请求产生任何影响

.. _request-url:

request.url
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
请求的url

.. _request-method:

request.method
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
使用大写字母表示的请求的方法,例如 "GET"

.. _request-headers:

request.headers
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
一个用Lua中table结构表示的HTTP的请求头(一个由名称到键值的映射);键值为响应头对应的名称, 值为响应头对应的值.

查询时是不区分大小写的，因此 ``request.headers['content-type']`` 与 ``request.headers['Content-Type']`` 是相同的.

.. _request-info:

request.info
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
它是一个lua中的table结构，以 `HAR request <http://www.softwareishard.com/blog/har-12-spec/#request>`_ 的格式来表示请求数据

.. _methods:

方法
----------------------------------------------

为了修改或者丢弃某个请求，您可以使用下面列举的 ``request`` 相关方法。请注意这些方法只有在请求发送之前有效
(如果一个请求已经被发送，那么修改它也就没有意义)。目前您只能在 ``splash:on_request <splash-on-request>`` 定义的回调函数中使用

.. _request-abort:

request:abort
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
丢弃一个请求

**原型:** ``request:abort()``

**返回值:** ``nil``

**是否异步:** 否

.. _request-enable-response-body:

request:enable_response_body
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
启用对响应内容的追踪(例如启用之后您可以使用 `response.body <./scripting-response-object.html#splash-response-body>`_ )

**原型:** ``request:enable_response_body()``

**返回值:** ``nil``

**是否异步:** 否

当 `splash.response_body_enabled <./scripting-ref.html#splash-response-body-enabled>`_ 被设置为false时,
这个函数将会开启每个请求的响应内容追踪, 您可以在 ``splash:on_request <splash-on-request>`` 定义的回调函数中使用

.. _request-set-url:

request:set_url
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
修改请求的URL为用户指定的url

**原型:** ``request:set_url(url)``

**参数:**

- url: 一个请求的新的url

**返回值:** ``nil``

**是否异步:** 否

.. _request-set-proxy:

request:set_proxy
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

在当前的请求中设置代理服务器的相关信息

**原型:** ``request:set_proxy{host, port, username=nil, password=nil, type='HTTP'}``

**参数:**

- host
- port
- username
- password
- type: 代理类型; 允许的类型有 ‘HTTP’ 和 ‘SOCKS5’.

**返回值:** nil

**是否异步:** 否

如果一个代理服务不需要进行认证，那么 ``username`` 和 ``password`` 可以忽略

如果代理类型设置为 "HTTP" HTTPS的代理也能正常工作, 它是使用 CONNECT命令还实现的

.. _request-set-timeout:

request:set_timeout
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

您可以针对这个请求设置一个超时时间

**原型:** ``request:set_timeout(timeout)``

**参数:**

- timeout : 超时时间，单位为秒

**返回值:** nil

**是否异步:** 否

如果发生超时后，响应仍然没有接收完成，此时会丢弃请求。您可以参考: `splash.resource_timeout <./scripting-ref.html#splash-resource-timeout>`_

.. _request-set-header:

request:set_header
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
设置请求的请求头

**原型:** ``request:set_header(name, value)``

**参数:**

- name: 请求头的名称
- value: 请求头的值

**返回值:** nil

**是否异步:** 否

您可以参考: `splash:set_custom_headers <splash-set-custom-headers>`_
