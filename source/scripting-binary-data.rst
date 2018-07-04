.. _working-with-binary-data:

使用二进制数据
==========================================

.. _motivation:

动机
-----------------------------------
Splash假设脚本中所有字符串被以 UTF-8 格式进行编码。即使HTTP响应信息不是通过 UTF-8 编码的，它对其使用UTF-8的方式还原为字符串。
由于内部的浏览器使用UTF-8，所以通过 `splash:html <./scripting-ref.html#splash-html>`_ 返回的HTML代码也是UTF-8的

如果您在 ``main`` 函数中通过table返回一组数据,Splash会将它编码为一个json数据。json是一个无法处理二进制的文本协议。所以在通过json返回数据时，
Splash认为里面所有的字符串都是UTF-8格式的

但是在某些场合下使用二进制数据十分重要,例如这个二进制数据可以是通过 `splash:png <./scripting-ref.html#splash-png>`_
方法获取的图片的原始数据,或者是通过 `splash:http_get <scripting-ref.html#splash-http-get>`_ 获取到的未经
UTF-8编码的原始的HTTP响应信息

.. _binary-objects:

二进制对象
-------------------------------------
为了将非UTF-8数据传递给Splash(在 ``main`` 函数中返回数据,或者向Splash方法传递参数),在脚本中可以使用
`treat.as_binary <scripting-libs.html#treat-as-binary>`_ 来将数据作为二进制对象进行标记

一些函数本身就返回二进制对象,像: `splash:png <./scripting-ref.html#splash-png>` ,
`splash:jpeg <scripting-ref.html#splash-jpeg>`_ ; `response.body <./scripting-response-object.html#splash-response-body>`_

二进制对象可以直接通过 ``main`` 函数返回。这也是为什么下面的例子可以运行( `render.png <./api.html#render-png>`_ 的基础例子)
::

    -- 模拟基础的render.png的实现
    function main(splash)
        assert(splash:go(splash.args.url))
        return splash:png()
    end

所有的二进制对象都添加了 Content-Type 字段，例如 splash:png 会拥有的Content-Type值为 ``image/png``

如果直接返回,二进制对象会被作为响应体的值来使用, HTTP 响应体中 Content-Type 的值会被设置成二进制对象所代表的数据类型。所以在上面的例子中
响应的Content-Type 会被设置成PNG 图片的类型，也就是 ``image/png``

如果要构造您自己的二进制对象，请使用 `treat.as_binary <scripting-libs.html#treat-as-binary>`_ 函数。例如让我们将一个 1x1 像素
大小的黑色GIF图片作为响应内容
::

    treat = require("treat")
    base64 = require("base64")

    function main(splash)
        local gif_b64 = "AQABAIAAAAAAAAAAACH5BAAAAAAALAAAAAABAAEAAAICTAEAOw=="
        local gif_bytes = base64.decode(gif_b64)
        return treat.as_binary(gif_bytes, "image/gif")
    end

当 ``main`` 函数返回时，二进制对象自身的Content-Type 值优于通过
`splash:set_result_content_type <./scripting-ref.html#splash-set-result-content-type>`_ 设置的值。
为了覆盖二进制对象自身的 Content-Type 值，您可以新建一个 二进制对象并重新指定 Content-Type值
::

    lcoal treat = require("treat")
    function main(splash)
        -- ...
        local img = splash:png()
        return treat.as_binary(img, "image/x-png") -- default was "image/png"
    end

当一个二进制值需要被序列化为json对象时,在序列化之前会自动将二进制值采用 base64 进行编码，
例如当需要在 ``main`` 函数中返回一个 table
::

    function main(splash)
        assert(splash:go(splash.args.url))

        -- result is a JSON object {"png": "...base64-encoded image data"}
        return {png=splash:png()}
    end
