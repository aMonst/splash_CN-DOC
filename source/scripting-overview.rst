.. _splash-lua-api-overview:

Splash lua API概览
================================
Splash 提供了许多方法、函数和属性。这些内容分别被写进了文档：`Splash 脚本参考 <>`_  `可用的lua库 <>`_ `元素对象 <>`_ `请求对象 <>`_
`响应对象 <>`_ 和 `使用二进制数据 <>`_ 。下面将对这些内容做一些简单的描述

.. _script-as-an-http-api-endpoint:

脚本作为 Splash HTTP API 端点
--------------------------------------------
每一个splash 的lua脚本都可以看做是一个 HTTP API 端点，具有输入参数和结构化的返回值。您可以使用lua脚本来模拟一个render.png端点，包括它
所使用的参数。

- `splash.args <>`_ 可以从脚本中获取数据 [#1]_
- `splash:set_result_status_code <>`_ 可以在返回值中修改HTTP的状态码
- `splash:set_result_content_type <>`_ 运许更改返回给客户端的Content-Type
- `splash:set_result_header <>`_ 可以在返回值中增加自定义的HTTP头
- `Working with Binary Data <>`_ 描述了如何在splash中使用非文本数据、比如如何给客户端返回二进制数
- `treat <>`_ 运许将在返回结果时将自定义数据序列化为json格式

.. _navigation:

导航
---------------------------------------
- `splash:go <>`_ 在浏览器中加载指定的url
- `splash:set_content <>`_ 在浏览器中加载特定的内容（通常是HTML）
- `splash:lock_navigation <>`_  与 `splash:unlock_navigation <>`_ 锁定与解锁导航
- `splash:set_user_agent <>`_ 修改请求中使用的 User-Agent
- `splash:set_custom_headers <>`_ 允许设置splash默认使用的HTTP请求头
- `splash:on_request <>`_ 允许过滤或者替换对相应资源的请求，它允许在请求前设置HTTP 或者 SOCKS5代理服务
- `splash:on_response_headers <>`_ 允许通过请求头来过滤请求，比如通过Content-Type
- `splash:init_cookies <>`_ , `splash:add_cookie <>`_ , `splash:get_cookies <>`_ , `splash:clear_cookies <>`_ 与 `splash:delete_cookies <>`_ 允许对cookie进行管理

.. _Delays:

延迟
--------------------------------
- `splash:wait <>`_ 允许等待特定的时间
- `splash:call_later <>`_ 后续执行一项任务
- `splash:wait_for_resume <>`_ 等待某些js事件发生
- `splash:with_timeout <>`_ 允许设置代码执行的超时时间

.. _extracting-information-from-a-page:

在页面中提取相关的信息
-------------------------------------
- `splash:html <>`_ 返回页面通过浏览器渲染后的HTML 内容
- `splash:url <>`_ 返回当前浏览器中加载的url
- `splash:evaljs <>`_ 和 `splash:jsfunc <>`_ 允许在页面中通过js来提取数据
- `splash:select <>`_ 和 `splash:select_all <>`_ 允许在页面中使用CSS选择器。它们会返回对应元素的对象，这些对象在许多方法和后续的处理上都很有用(请参阅 `元素对象 <>`_ )
- `element:text <>`_  返回DOM对象的文本内容
- `element:bounds <>`_ 返回元素的边界框
- `element:styles <>`_ 返回自定义的元素样式
- `element:form_values <>`_ 返回表单元素的值
- DOM中的 ` HTMLElement <https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement>`_ 对象的绝大多数属性和方法都被支持，您可以参阅 `DOM属性 <>`_ 和 `DOM 方法 <>`_

.. _screenshots:

截图
-----------------------------------
- `splash:png <>`_, `splash:jpeg <>`_ 得到一个PNG 和 JPEG的截图
- `splash:set_viewport_full <>`_ 通过修改视口的大小(通常在使用 `splash:png <>`_, `splash:jpeg <>` 之前调用)可以获得页面完整的截图
- `splash:set_viewport_size <>`_ 修改视口大小
- `element:png <>`_ 和 `element:jpeg <>`_ 截取单个DOM元素的截图

.. _interacting-with-a-page:

与页面交互
--------------------------------------
- `splash:runjs <>`_ , `splash:evaljs <>`_ 和 `splash:jsfunc <>`_ 允许在页面上下文中执行任意的js脚本
- `splash:autoload <>`_ 允许在页面开始渲染的时候预加载一些JavaScript库或者执行JavaScript代码
- `splash:mouse_click <>`_ , `splash:mouse_hover <>`_ , `splash:mouse_press <>`_ , `splash:mouse_release <>`_ 允许向页面指定的坐标发送鼠标消息
- `splash:send_keys <>`_ 和 `splash:send_text <>`_ 允许向页面发送键盘事件
- `element:send_keys <>`_ and  `element:send_text <>`_ 向指定的DOM元素发送键盘事件
- 您可以通过使用 `element:form_values <>`_ 函数来设置表单的初始值, 使用 `element:fill <>`_ 来更新表单值, 使用 ` element:submit <>`_ 来提交表单
- `splash.scroll_position`_ 允许您滚动页面
- DOM中的 ` HTMLElement <https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement>`_ 对象的绝大多数属性和方法都被支持，您可以参阅 `DOM属性 <>`_ 和 `DOM 方法 <>`_

.. _making-http-requests:

构造HTTP 请求
--------------------------------------
- `splash:http_get <>`_ 发送一个HTTP 的GET请求，并获取一个未经过渲染的原始的响应
- `splash:http_post <>`_  发送一个HTTP 的POST请求，并获取一个未经过渲染的原始的响应

.. _inspecting-network-traffic:

检查网络流量
-----------------------------------------
- `splash:har <>`_ 在 `HAR <http://www.softwareishard.com/blog/har-12-spec/>`_ 结构体中返回所有的HTTP请求和响应
- `splash:history <>`_ 返回有关重定向和加载到主浏览器窗口的页面的信息;
- `splash:on_request <>`_ 允许捕捉浏览器页面或者脚本中的错误
- `splash:on_response_headers <>`_ 允许在获取响应头后，进行检查（或者丢弃）
- `splash:on_response <>`_ 允许检查接受到的原始响应信息（包括相应的资源新）
- `splash.response_body_enabled`_ 允许接受 `splash:har <>`_ 和 `splash:on_response <>` 中全部的响应信息
- 您可以参阅 `Response Object <>`_ 和 `Request Object <>`_ 来获取关于请求和响应的更详细的信息

.. _browsing-options:

浏览选项
------------------------------
- `splash.js_enabled <>`_ 允许关闭JavaScript的支持
- `splash.private_mode_enabled <>`_ 允许关闭专有模式 (它在有些情况下是有用的，比如Webkit在专有模式是没有本地存储)
- `splash.images_enabled <>`_ 允许禁止下载图片
- `splash.plugins_enabled <>`_ 允许禁止加载插件 (默认情况下docker中的镜像允许加载Flash)
- `splash.resource_timeout <>`_ 允许在超时后减缓或者挂起相关请求
- `splash.indexeddb_enabled <>`_ 允许打开IndexedDB
- `splash.webgl_enabled <>`_ 允许关闭WebGL
- `splash.html5_media_enabled <>`_ 允许打开对HTML5的支持(比如 播放 ``<video>`` 标签)
- `splash.media_source_enabled <>`_ 允许关闭对多媒体扩展API的支持

.. [#1] 这里的脚本指的是Python脚本
