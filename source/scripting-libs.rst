.. _available-lua-libraries:

可使用的lua库
==================================
当Splash的 `沙盒模式 <./scripting-tutorial.html#lua-sandbox>`_ 关闭时，可以支持所有的lua标准库，而当沙盒模式开启(默认开启)时，只有部分标准库可以调用。
您可以参阅 :ref:`Standard Library <standard-library>` 以获取更多信息

splash 默认提供多个非标准模块

- :ref:`json <json>` : 对json数据进行编码/解码
- :ref:`base64 <lib-base64>` : 对数据进行 Base64 编码/解码
- :ref:`treat <treat>` : 将lua变量进行相应的类型转化，以便适应Splash [#1]_

与标准模块不同，扩展模块在使用前需要引用,例如
::

    base64 = require("base64")
    function main(splash)
        return base64.encode('hello')
    end

您可以参阅 :ref:`自定义lua模块 <adding-your-own-modules>` 的内容，来为Splash添加更多lua模块

.. _standard-library:

lua 标准库
--------------------------------
当 splash 的沙盒模式开启时(默认),splash支持lua 5.2 版本中的下列标准库:

- `string <http://www.lua.org/manual/5.2/manual.html#6.4>`_
- `table <http://www.lua.org/manual/5.2/manual.html#6.5>`_
- `math <http://www.lua.org/manual/5.2/manual.html#6.6>`_
- `os <http://www.lua.org/manual/5.2/manual.html#6.9>`_

上述库是预先导入的，您不需要显示的调用 ``require``

.. note::
    目前当沙盒模式开启时，上述库中并不是的所有方法都可以使用。

.. _json:

json
----------------------------------------
json 库可以将一个数据序列化为json数据，或者将json数据反序列化为lua中的table结构。库中提供了两个方法
:ref:`json.encode <json-encode>` 和 :ref:`json.decode <json-decode>`

.. _json-encode:

json.encode
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
将数据编码为JSON格式

**原型:** ``result = json.encode(obj)``

**参数:**

- obj: 将被转化为json格式的对象

**返回值:** 转化完成后的一个json字符串

目前不支持针对二进制数据的json格式化, json.encode 函数在处理二进制对象的时候，先将其使用Base64方式的加密

.. _json-decode:

json.decode
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
将json对象反序列化为普通对象

**原型:** ``decoded = json.decode(s)``

**参数:**

- s: 一个json字符串

**返回值:** 反序列化完成之后产生的Lua对象

示例
::

    json = require("json")

    function main(splash)
        local resp = splash:http_get("http:/myapi.example.com/resource.json")
        local decoded = json.decode(resp.content.text)
        return {myfield=decoded.myfield}
    end

请注意，不同于 :ref:`json.encode <json-encode>` 函数,  :ref:`json.decode <json-decode>` 函数没有针对
`二进制对象 <./scripting-binary-data.html#binary-data>`_ 的支持。这意味着如果您希望将之前使用
:ref:`json.encode <json-encode>` 得到的json数据还原成最初的二进制对象，您需要自己使用base64的方式进行解码，您可以通过使用
lua中的 :ref:`base64 <lib-base64>` 模块来完成这个操作

.. _lib-base64:

base64
-------------------------------------------
使用base64方式对字符串进行编码/解码, 它提供了两个函数 :ref:`base64.encode <base64-encode>`
和 :ref:`base64.decode <base64-decode>` 。如果您需要在json请求或者响应中传递一些二进制数据，那么这些函数将会很有用

.. _base64-encode:

base64.encode
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
将一个字符串使用base64编码

**原型:** ``encoded = base64.encode(s)``

**参数:**

- s：需要进行编码的字符串或者 `二进制对象 <./scripting-binary-data.html#binary-objects>`_

**返回值:** ``s`` 通过base64编码后的字符串

.. _base64-decode:

base64.decode
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
将一个字符串通过base64解码

**原型:** ``data = base64.decode(s)``

**参数:**

- s: 需要进行解码的字符串

**返回值:** 解码后的lua字符串

请注意，base64.decode可能会返回一个非UTF-8格式的字符串,这个字符串在传入splash时可能不太安全
(作为 ``main`` 的返回值或者传入其他splash函数)。如果您知道原始数据是UTF-8格式或者ASCII格式会非常好，但是如果您不知道原始数据
的格式或者原始数据根本就不是UTF-8格式的，您可以在 :ref:`base64.encoded <base64-encode>` 函数 的返回结果上调用
:ref:`treat.as_binary <treat-as-binary>`

例子：返回一个黑色的 1x1 像素大小的gif图片
::

    treat = require("treat")
    base64 = require("base64")

    function main(splash)
        local gif_b64 = "AQABAIAAAAAAAAAAACH5BAAAAAAALAAAAAABAAEAAAICTAEAOw=="
        local gif_bytes = base64.decode(gif_b64)
        return treat.as_binary(gif_bytes, "image/gif")
    end

.. _treat:

treat
----------------------------------------

.. _treat-as-binary:

treat.as_binary
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
将字符串转化为 `二进制对象 <./scripting-binary-data.html#binary-objects>`_

**原型:** ``bytes = treat.as_binary(s, content_type="application/octet-stream")``

**参数:**

- s: 字符串
- content_type: ``s`` 的content_type值

**返回值:** 一个  `二进制对象 <./scripting-binary-data.html#binary-objects>`_

:ref:`treat.as_binary <treat-as-binary>` 返回一个二进制对象，
这个对象不能再在lua中做进一步处理，但是它可以作为main函数的返回值。

.. _treat-as-string:

treat.as_string
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
从 `二进制对象 <./scripting-binary-data.html#binary-objects>`_ 的原始数据中得到一个字符串

**原型:** ``s, content_type = treat.as_string(bytes)``

**参数:**

- bytes: 一个原始的 `二进制对象 <./scripting-binary-data.html#binary-objects>`_

**返回值:** ``(s, content_type)`` 元组,转化后得到的字符串以及它对应的content_type值

:ref:`treat.as_string <treat-as-string>` 会“解开”一个 `二进制对象 <./scripting-binary-data.html#binary-objects>`_
并得到一个普通的lua字符串，这个字符串可以被lua程序进一步处理。如果返回的字符串没有使用UTF-8进行解码，那么它仍然可以被lua
做进一步的处理,但是如果将其作为 ``main`` 函数的返回值或者作为其他splash函数的参数,将会很不安全,如果您想在Splash中
将其还原成二进制对象，请使用函数 :ref:`treat.as_binary <treat-as-binary>`

.. _treat-as-array:

treat.as_array
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
将lua中的table标记为数组(针对编码过后的json数据或者table 到js的转换)

**原型:** ``tbl = treat.as_array(tbl)``

**参数:**

- tbl: lua table

**返回值:** 与参数相同的table

json 可以表示数组和对象,但是在lua中没法对他们进行区分, 不管是键值对还是数组在lua中都是以table的格式存储。

默认情况下, 从Splash ``main`` 函数中返回结果，或者使用 :ref:`json.encode <json-encode>`
函数和 `splash.jsfunc <./scripting-ref.html#splash-jsfunc>`_ 函数时，Lua table将会转化为json
::

    function main(splash)
        -- client gets {"foo": "bar"} JSON object
        return {foo="bar"}
    end

使用类似于数组的lua table将会带来意想不到的结果
::

    function main(splash)
        -- client gets {"1": "foo", "2": "bar"} JSON object
        return {"foo", "bar"}
    end

:ref:`treat.as_array <treat-as-array>` 将table标记为 Json数组。
::

    treat = require("treat")

    function main(splash)
        local tbl = {"foo", "bar"}
        treat.as_array(tbl)

        -- client gets ["foo", "bar"] JSON object
        return tbl
    end

此函数在其中修改其参数, 但是为了方便，它仍然返回原始的table对象,它允许我们简化代码
::

    treat = require("treat")
    function main(splash)
        -- client gets ["foo", "bar"] JSON object
        return treat.as_array({"foo", "bar"})
    end

.. note::

    针对table，每有什么方法可以检测它具体的类型,因为 ``{}`` 符号本身可以表示多种含义,它既可以作为json数组，
    也可以作为json对象。当table的具体类型不确定的时候就容易产生一个错误的输出:即使某些数据总是一个数组，当数组为空
    时，它可能会被当做一个对象, 为了避免这种错误，Splash提供了一个工具函数 :ref:`treat.as_array <treat-as-array>`

.. _adding-your-own-modules:

添加您自己的模块
-------------------------------------
Splash 提供了一个通过HTTP API等方式在脚本中使用自定义Lua模块的功能(存储在服务端)。这个功能允许：

1. 重用代码，而不是通过网络一遍又一遍的发送
#. 使用第三方lua模块
#. 实现需要不安全代码的功能，并能够在沙盒中安全的使用它

.. note::

    您可以访问 http://lua-users.org/wiki/ModulesTutorial 来
    学习关于lua模块的知识, 请爱上最新的编写模块的方法，因为它会让您的模块在沙盒模式下更好的工作,这里有一个很好的lua
    模块样式指南: http://hisham.hm/2014/01/02/how-to-write-lua-modules-in-a-post-module-world/

.. _setting-up:

配置
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
请您进行如下配置，以便使用lua的自定义模块

1. 配置lua模块的路径，让后在对应目录下添加您自己的模块
#. 告诉Splash，在沙盒中可以使用哪些自定义模块
#. 在lua脚本中使用 ``require`` 来导入一个库

您可以在启动Splash的时候加上参数 ``--lua-package-path`` 来指定lua库的路径。 ``--lua-package-path`` 的值应该是
以分号分隔的多个路径的字符串，每一个路径应该有一个 ``?`` 它被用来替代模块名称
::

    $ python3 -m splash.server --lua-package-path "/etc/splash/lua_modules/?.lua;/home/myuser/splash-modules/?.lua"

.. note::

    如果您使用docker来安装splash，请参考 `文件共享 <Installation.html#docker-folder-sharing>`_
    这部分内容来获取与设置文件路径相关的内容

.. note::

    ``--lua-package-path`` 中的值会被添加到 Lua的 ``package.path``

当您使用 Lua `沙盒的时候 <./scripting-tutorial.html#lua-sandbox>`_ (默认情况下开启) .在脚本中 ``require`` 会
收到一定的限制, 它只能加载在白名单中的模块。白名单在默认情况下是空的，也就是说在默认情况下您不能加载任何东西,为了让
您的模块可用，您可以在启动Splash的时候设置 ``--lua-sandbox-allowed-modules`` 选项。它应该包含以分号为分隔符的字符串，
其中每个子串代表允许被加载的模块的名称
::

    $ python3 -m splash.server --lua-sandbox-allowed-modules "foo;bar" --lua-package-path "/etc/splash/lua_modules/?.lua"

设置完成之后，您就可以通过 ``require`` 来加载您的自定义模块
::

    local foo = require("foo")
    function main(splash)
        return {result=foo.myfunc()}
    end

.. _writing-modules:

模块的编写
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
一个基础的模块看起来像这样:
::

    -- mymodule.lua
    local mymodule = {}

    function mymodule.hello(name)
        return "Hello, " .. name
    end

    return mymodule

在脚本中可以这样来使用
::

    local mymodule = require("mymodule")

    function main(splash)
        return mymodule.hello("world!")
    end

在许多情况下，模块可能会使用 ``splash`` 对象,这里给出一些办法来编写这类模块。最简单的办法就是让
函数允许传入 ``splash`` 对象作为参数。
::

    -- utils.lua
    local utils = {}

    -- wait until `condition` function returns true
    function utils.wait_for(splash, condition)
        while not condition() do
            splash:wait(0.05)
        end
    end

    return utils

用法:
::

    local utils = require("utils")

    function main(splash)
        splash:go(splash.args.url)

        -- wait until <h1> element is loaded
        utils.wait_for(splash, function()
           return splash:evaljs("document.querySelector('h1') != null")
        end)

        return splash:html()
    end

另一个编写此类模块的方法是，往一个 ``splash`` 对象中添加对应的方法。您可以通过向 ``Splash`` 类来添加方法。
这种方法在Ruby中被称之为 “打开类”,而在python中被叫做补丁。
::

    -- wait_for.lua

    -- Sandbox is not enforced in custom modules, so we can import
    -- internal Splash class and change it - add a method.
    local Splash = require("splash")

    function Splash:wait_for(condition)
        while not condition() do
            self:wait(0.05)
        end
    end

    -- no need to return anything

用法:
::

    require("wait_for")

    function main(splash)
        splash:go(splash.args.url)

        -- wait until <h1> element is loaded
        splash:wait_for(function()
           return splash:evaljs("document.querySelector('h1') != null")
        end)

        return splash:html()
    end

具体使用哪种风格，取决于您的个人喜好,函数的方式更加明确，更加容易组合。使用补丁的方式可以使代码更加紧凑,无论您使用哪种
办法来向 ``splash`` 对象中添加函数，都可以直接使用 ``require``

如前面的例子所示。 标准的lua模块和函数的沙盒限制不适用于自定义的Lua模块。换句话说自定义的lua模块不受限制，您可以发挥
出lua的全部功效，这将令我们可以使用第三发模块，利用lua来写出更加高级的功能。但是在使用的时候需要特别小心。
例如让我们来使用 `os <http://www.lua.org/manual/5.2/manual.html#6.9>`_  模块
::

    -- evil.lua
    local os = require("os")
    local evil = {}

    function evil.sleep()
        -- Don't do this! It blocks the event loop and has a startup cost.
        -- splash:wait is there for a reason.
        os.execute("sleep 2")
    end

    function evil.touch(filename)
        -- another bad idea
        os.execute("touch " .. filename)
    end

    -- todo: rm -rf /

    return evil

.. [#1] 这段话的原文是 "fine-tune the way Splash works with your Lua varaibles and returns the result" 这里为了理解方便采用意译
