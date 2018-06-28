.. _splash-lua-api-overview:

Splash lua API概览
================================
Splash 提供了许多方法、函数和属性。这些内容分别被写进了文档：`Splash 脚本参考 <./scripting-ref.html#scripting-reference>`_ ,
`可用的lua库 <./scripting-libs.html#scripting-libs>`_ , `元素对象 <./scripting-element-object.html#splash-element>`_ ,
`请求对象 <./scripting-request-object.html#splash-request>`_ , `响应对象 <./scripting-response-object.html#splash-response>`_
和 `使用二进制数据 <./scripting-binary-data.html#binary-data>`_ 。下面将对这些内容做一些简单的描述

.. _script-as-an-http-api-endpoint:

脚本作为 Splash HTTP API 端点
--------------------------------------------
每一个splash 的lua脚本都可以看做是一个 HTTP API 端点，具有输入参数和结构化的返回值。您可以使用lua脚本来模拟一个render.png端点，包括它
所使用的参数。

- `splash.args <./scripting-ref.html#splash-args>`_ 可以从脚本中获取数据 [#1]_
- `splash:set_result_status_code <./scripting-ref.html#splash-set-result-status-code>`_ 可以在返回值中修改HTTP的状态码
- `splash:set_result_content_type <./scripting-ref.html#splash-set-result-content-type>`_ 运许更改返回给客户端的Content-Type
- `splash:set_result_header <./scripting-ref.html#splash-set-result-header>`_ 可以在返回值中增加自定义的HTTP头
- `Working with Binary Data <./scripting-binary-data.html#binary-data>`_ 描述了如何在splash中使用非文本数据、比如如何给客户端返回二进制数
- `treat <./scripting-libs.html#lib-treat>`_ 运许将在返回结果时将自定义数据序列化为json格式

.. _navigation:

导航
---------------------------------------
- `splash:go <./scripting-ref.html#splash-go>`_ 在浏览器中加载指定的url
- `splash:set_content <./scripting-ref.html#splash-set-content>`_ 在浏览器中加载特定的内容（通常是HTML）
- `splash:lock_navigation <./scripting-ref.html#splash-lock-navigation>`_  与 `splash:unlock_navigation <./scripting-ref.html#splash-unlock-navigation>`_ 锁定与解锁导航
- `splash:set_user_agent <./scripting-ref.html#splash-set-user-agent>`_ 修改请求中使用的 User-Agent
- `splash:set_custom_headers <./scripting-ref.html#splash-set-custom-headers>`_ 允许设置splash默认使用的HTTP请求头
- `splash:on_request <./scripting-ref.html#splash-on-request>`_ 允许过滤或者替换对相应资源的请求，它允许在请求前设置HTTP 或者 SOCKS5代理服务
- `splash:on_response_headers <./scripting-ref.html#splash-on-response-headers>`_ 允许通过请求头来过滤请求，比如通过Content-Type
- `splash:init_cookies <./scripting-ref.html#splash-init-cookies>`_ , `splash:add_cookie <./scripting-ref.html#splash-add-cookie>`_ , `splash:get_cookies <./scripting-ref.html#splash-get-cookies>`_ , `splash:clear_cookies <./scripting-ref.html#splash-clear-cookies>`_ 与 `splash:delete_cookies <./scripting-ref.html#splash-delete-cookies>`_ 允许对cookie进行管理

.. _Delays:

延迟
--------------------------------
- `splash:wait <./scripting-ref.html#splash-wait>`_ 允许等待特定的时间
- `splash:call_later <./scripting-ref.html#splash-call-later>`_ 后续执行一项任务
- `splash:wait_for_resume <./scripting-ref.html#splash-wait-for-resume>`_ 等待某些js事件发生
- `splash:with_timeout <./scripting-ref.html#splash-with-timeout>`_ 允许设置代码执行的超时时间

.. _extracting-information-from-a-page:

在页面中提取相关的信息
-------------------------------------
- `splash:html <./scripting-ref.html#splash-html>`_ 返回页面通过浏览器渲染后的HTML 内容
- `splash:url <./scripting-ref.html#splash-url>`_ 返回当前浏览器中加载的url
- `splash:evaljs <./scripting-ref.html#splash-evaljs>`_ 和 `splash:jsfunc <./scripting-ref.html#splash-jsfunc>`_ 允许在页面中通过js来提取数据
- `splash:select <./scripting-ref.html#splash-select>`_ 和 `splash:select_all <./scripting-ref.html#splash-select-all>`_ 允许在页面中使用CSS选择器。它们会返回对应元素的对象，这些对象在许多方法和后续的处理上都很有用(请参阅 `元素对象 <./scripting-element-object.html#splash-element>`_ )
- `element:text <./scripting-element-object.html#splash-element-text>`_  返回DOM对象的文本内容
- `element:bounds <./scripting-element-object.html#splash-element-bounds>`_ 返回元素的边界框
- `element:styles <./scripting-element-object.html#splash-element-styles>`_ 返回自定义的元素样式
- `element:form_values <./scripting-element-object.html#splash-element-form-values>`_ 返回表单元素的值
- DOM中的 ` HTMLElement <https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement>`_ 对象的绝大多数属性和方法都被支持，您可以参阅 `DOM属性 <./scripting-element-object.html#splash-element-dom-methods>`_ 和 `DOM 方法 <./scripting-element-object.html#splash-element-dom-attributes>`_

.. _screenshots:

截图
-----------------------------------
- `splash:png <./scripting-ref.html#splash-png>`_, `splash:jpeg <./scripting-ref.html#splash-jpeg>`_ 得到一个PNG 和 JPEG的截图
- `splash:set_viewport_full <./scripting-ref.html#splash-set-viewport-full>`_ 通过修改视口的大小(通常在使用 `splash:png <./scripting-ref.html#splash-png>`_, `splash:jpeg <./scripting-ref.html#splash-jpeg>` 之前调用)可以获得页面完整的截图
- `splash:set_viewport_size <./scripting-ref.html#splash-set-viewport-size>`_ 修改视口大小
- `element:png <./scripting-element-object.html#splash-element-png>`_ 和 `element:jpeg <./scripting-element-object.html#splash-element-jpeg>`_ 截取单个DOM元素的截图

.. _interacting-with-a-page:

与页面交互
--------------------------------------
- `splash:runjs <./scripting-ref.html#splash-runjs>`_ , `splash:evaljs <./scripting-ref.html#splash-evaljs>`_ 和 `splash:jsfunc <./scripting-ref.html#splash-jsfunc>`_ 允许在页面上下文中执行任意的js脚本
- `splash:autoload <./scripting-ref.html#splash-autoload>`_ 允许在页面开始渲染的时候预加载一些JavaScript库或者执行JavaScript代码
- `splash:mouse_click <./scripting-ref.html#splash-mouse-click>`_ , `splash:mouse_hover <./scripting-ref.html#splash-mouse-hover>`_ , `splash:mouse_press <./scripting-ref.html#splash-mouse-press>`_ , `splash:mouse_release <scripting-ref.html#splash-mouse-release>`_ 允许向页面指定的坐标发送鼠标消息
- `splash:send_keys <./scripting-ref.html#splash-send-keys>`_ 和 `splash:send_text <./scripting-ref.html#splash-send-text>`_ 允许向页面发送键盘事件
- `element:send_keys <./scripting-element-object.html#splash-element-send-keys>`_ and  `element:send_text <./scripting-element-object.html#splash-element-send-text>`_ 向指定的DOM元素发送键盘事件
- 您可以通过使用 `element:form_values <./scripting-element-object.html#splash-element-form-values>`_ 函数来设置表单的初始值, 使用 `element:fill <./scripting-element-object.html#splash-element-fill>`_ 来更新表单值, 使用 ` element:submit <./scripting-element-object.html#splash-element-submit>`_ 来提交表单
- `splash.scroll_position <./scripting-ref.html#splash-scroll-position>`_ 允许您滚动页面
- DOM中的 ` HTMLElement <https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement>`_ 对象的绝大多数属性和方法都被支持，您可以参阅 `DOM属性 <./scripting-element-object.html#splash-element-dom-methods>`_ 和 `DOM 方法 <./scripting-element-object.html#splash-element-dom-attributes>`_

.. _making-http-requests:

构造HTTP 请求
--------------------------------------
- `splash:http_get <./scripting-ref.html#splash-http-get>`_ 发送一个HTTP 的GET请求，并获取一个未经过渲染的原始的响应
- `splash:http_post <./scripting-ref.html#splash-http-post>`_  发送一个HTTP 的POST请求，并获取一个未经过渲染的原始的响应

.. _inspecting-network-traffic:

检查网络流量
-----------------------------------------
- `splash:har <./scripting-ref.html#splash-har>`_ 在 `HAR <http://www.softwareishard.com/blog/har-12-spec/>`_ 结构体中返回所有的HTTP请求和响应
- `splash:history <./scripting-ref.html#splash-history>`_ 返回有关重定向和加载到主浏览器窗口的页面的信息;
- `splash:on_request <./scripting-ref.html#splash-on-request>`_ 允许捕捉浏览器页面或者脚本中的错误
- `splash:on_response_headers <./scripting-ref.html#splash-on-response-headers>`_ 允许在获取响应头后，进行检查（或者丢弃）
- `splash:on_response <./scripting-ref.html#splash-on-response>`_ 允许检查接受到的原始响应信息（包括相应的资源新）
- `splash.response_body_enabled <splash-response-body-enabled>`_ 允许接受 `splash:har <./scripting-ref.html#splash-har>`_ 和 `splash:on_response <./scripting-ref.html#splash-on-response>` 中全部的响应信息
- 您可以参阅 `Response Object <./scripting-response-object.html#splash-response>`_ 和 `Request Object <./scripting-request-object.html#splash-request>`_ 来获取关于请求和响应的更详细的信息

.. _browsing-options:

浏览选项
------------------------------
- `splash.js_enabled <./scripting-ref.html#splash-js-enabled>`_ 允许关闭JavaScript的支持
- `splash.private_mode_enabled <./scripting-ref.html#splash-private-mode-enabled>`_ 允许关闭专有模式 (它在有些情况下是有用的，比如Webkit在专有模式是没有本地存储)
- `splash.images_enabled <./scripting-ref.html#splash-images-enabled>`_ 允许禁止下载图片
- `splash.plugins_enabled <./scripting-ref.html#splash-plugins-enabled>`_ 允许禁止加载插件 (默认情况下docker中的镜像允许加载Flash)
- `splash.resource_timeout <./scripting-ref.html#splash-resource-timeout>`_ 允许在超时后减缓或者挂起相关请求
- `splash.indexeddb_enabled <./scripting-ref.html#splash-indexeddb-enabled>`_ 允许打开IndexedDB
- `splash.webgl_enabled <./scripting-ref.html#splash-webgl-enabled>`_ 允许关闭WebGL
- `splash.html5_media_enabled <./scripting-ref.html#splash-html5-media-enabled>`_ 允许打开对HTML5的支持(比如 播放 ``<video>`` 标签)
- `splash.media_source_enabled <./scripting-ref.html#splash-media-source-enabled>`_ 允许关闭对多媒体扩展API的支持

.. [#1] 这里的脚本指的是Python脚本
