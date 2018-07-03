.. _element-objects:

Element 对象
========================================
Element 对象是对JavaScript DOM对象的封装,它们在一些函数返回任意类型的DOM节点时被创建(节点、元素、HTML元素等等)。

`splash:select <./scripting-ref.html#splash-select>`_ 和 `splash:select_all <./scripting-ref.html#splash-select-all>`_
返回Element对象， `splash:evaljs <./scripting-ref.html#splash-evaljs>`_ 也可能返回Element对象,但是目前它们不能被放到数组或其他对象
中一起返回。它们必须作为顶层节点或者节点的列表被返回。

.. _methods:

方法
-----------------------------------
要修改或者检索有关元素的信息，您可以使用下列的方法

.. _element-mouse-click:

element:mouse_click
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
在元素上触发鼠标点击消息

**原型:** ``ok, reason = element:mouse_click{x=nil, y=nil}``

**参数:**

- x:可选值,相对于元素左上角的x的坐标
- y:可选值,相对于元素左上角的y的坐标

**返回值:** ``ok, reason`` 元组,当函数在执行过程中发生错误的时候 ``ok`` 的值为nil. ``reason`` 包含了一个表示错误类型的字符串

**是否异步:** 是

如果没有指定x和y的值，默认为元素宽的一半和高的一半，点击事件将在元素的中间被触发。

这个坐标可以是负值，这意味着点击事件将在元素之外触发。

例1：在靠近元素左上角的位置触发点击事件
::

    function main(splash)
        -- ...
        local element = splash:select('.element')
        local bounds = element:bounds()
        assert(element:mouse_click{x=bounds.width/3, y=bounds.height/3})
        -- ...
    end

例2：在元素上方10个像素的位置触发点击事件
::

    function main(splash)
        -- ...
        splash:set_viewport_full()
        local element = splash:select('.element')
        assert(element:mouse_click{y=-10})
        -- ...
    end

与 `splash:mouse_click <./scripting-ref.html#splash-mouse-click>`_ 不同，:ref:`element:mouse_click <element-mouse-click>`
会一直等待直到点击事件执行。所以如果要看点击事件执行之后在页面上产生的效果，您不需要调用
`splash:wait <./scripting-ref.html#splash-wait>`_ 进行等待

如果当前元素不在视口范围之内，视口会被自动滚动到合适的位置，以便让元素可见，如果需要滚动页面，在点击事件被触发之后，页面不会自动滚回原来的位置.

您可以在 `splash:mouse_click <./scripting-ref.html#splash-mouse-click>`_ 中查阅更多与鼠标事件相关的内容

.. _element-mouse-hover:

element:mouse_hover
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
在对应元素上触发鼠标悬停事件(JavaScript mouseover 事件)

**原型:** ``ok, reason = element:mouse_hover{x=0, y=0}``

**参数:**

- x:可选值,相对于元素左上角的x的坐标
- y:可选值,相对于元素左上角的y的坐标

**返回值:** ``ok, reason`` 元组,当函数在执行过程中发生错误的时候 ``ok`` 的值为nil. ``reason`` 包含了一个表示错误类型的字符串

**是否异步:** 否

如果没有指定x和y的值，默认为元素宽的一半和高的一半，鼠标悬停事件将在元素的中间被触发。

这个坐标可以是负值，这意味着鼠标悬停事件将在元素之外触发。

例1：在靠近元素左上角的位置触鼠标悬停事件
::

    function main(splash)
        -- ...
        local element = splash:select('.element')
        assert(element:mouse_hover{x=0, y=0})
        -- ...
    end

例2：在元素上方10个像素的位置触发鼠标悬停事件
::

    function main(splash)
        -- ...
        splash:set_viewport_full()
        local element = splash:select('.element')
        assert(element:mouse_hover{y=-10})
        -- ...
    end

与 `splash:mouse_hover <./scripting-ref.html#splash-mouse-hover>`_ 不同，:ref:`element:mouse_hover <element-mouse-hover>`
会一直等待直到点击事件执行。所以如果要看点击事件执行之后在页面上产生的效果，您不需要调用
`splash:wait <./scripting-ref.html#splash-wait>`_ 进行等待

如果当前元素不在视口范围之内，视口会被自动滚动到合适的位置，以便让元素可见，如果需要滚动页面，在点击事件被触发之后，页面不会自动滚回原来的位置.

您可以在 `splash:mouse_hover <./scripting-ref.html#splash-mouse-hover>`_ 中查阅更多与鼠标事件相关的内容

.. _element-styles:

element:styles
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
获取元素的所有样式信息

**原型:** ``styles = element:styles()``

**返回值:** ``style`` 是一个描述获取到的样式的table结构

**是否异步:** 否

这个方法返回在元素上调用 `window.getComputedStyle() <https://developer.mozilla.org/en-US/docs/Web/API/Window/getComputedStyle>`_ 获取到的结果

例子:获取元素所有的样式，并返回 ``font-size`` 属性
::

    function main(splash)
        -- ...
        local element = splash:select('.element')
        return element:styles()['font-size']
    end

.. _element-bounds:

element:bounds
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
返回元素bounding框的矩形

**原型:** ``bounds = element:bounds()``

**返回值:** ``bounds`` 是一个table结构，里面包含了bounding框的 ``top`` , ``right`` , ``bottom`` 和 ``left`` .
同时它也包含了bounding 框的 ``width`` 和 ``height`` 值

**是否异步:** 否

例子:获取元素的边界
::

    function main(splash)
        -- ..
        local element = splash:select('.element')
        return element:bounds()
        -- e.g. bounds is { top = 10, right = 20, bottom = 20, left = 10, height = 10, width = 10 }
    end

.. _element-png:

element:png
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
以png格式返回元素的截图

**原型:** ``shot = element:png{width=nil, scale_method='raster', pad=0}``

**参数:**

- width:可选值,截图的宽，以像素为单位
- scale_method:可选值，缩放图片的方式, 可选的值有 ``'raster'`` 和 ``'vector'``
- pad: 可选值, 整型或者是 ``{left, top, right, bottom}`` 结构的padding 值

**返回值:** 以 `二进制对象 <scripting-binary-data.html#binary-objects>`_ 表示的PNG 截图的数据, 如果结果为空
(例如元素在DOM中不存在,或者不可见)会返回nil

**是否异步:** 否

pad 参数设置返回图片的 padding  值,如果是一个整数值，那么padding中 所有的边距都相同,如果值为正数，那么截图会在原来元素的
基础上扩充指定像素大小,如果未负值，截图会在原来元素的基础上压缩指定像素大小.

例子:返回一个填充指定大小的元素截图
::

    function main(splash)
        -- ..
        local element = splash:select('.element')
        return element:png{pad=10}
    end

如果元素不在视口中, 视口会暂时滚动以便让元素可见，然后再滚回到原来的位置

您可以参阅 `splash:png <scripting-ref.html#splash-png>`_

.. _element-jpeg:

element:jpeg
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
以jpeg格式返回元素的截图

**原型:** ``shot = element:jpeg{width=nil, scale_method='raster', quality=75, region=nil, pad=0}``

**参数:**

- width:可选值,截图的宽，以像素为单位
- scale_method:可选值，缩放图片的方式, 可选的值有 ``'raster'`` 和 ``'vector'``
- quality: 可选值,生成jpeg图片的质量,为 ``0`` 到 ``100`` 的整数值
- pad: 可选值, 整型或者是 ``{left, top, right, bottom}`` 结构的padding 值

**返回值:** 以 `二进制对象 <scripting-binary-data.html#binary-objects>`_ 表示的jpeg 截图的数据, 如果结果为空
(例如元素在DOM中不存在,或者不可见)会返回nil

**是否异步:** 否

pad 参数设置返回图片的 padding  值,如果是一个整数值，那么padding中 所有的边距都相同,如果值为正数，那么截图会在原来元素的
基础上扩充指定像素大小,如果未负值，截图会在原来元素的基础上压缩指定像素大小.

如果元素不在视口中, 视口会暂时滚动以便让元素可见，然后再滚回到原来的位置

您可以参阅 `splash:jpeg <scripting-ref.html#splash-jpeg>`_

.. _element-visible:

element:visible
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
判断元素是否可见

**原型:** ``visible = element:visible()``

**返回值:** ``visible`` 表示元素是否可见

**是否异步:** 否

.. _element-focused:

element:focused
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
检测元素是否获取到焦点

**原型:** ``focused = element:focused()``

**返回值:** ``focused`` 表示元素是否获取到焦点

**是否异步:** 否

.. _element-text:

element:text
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
从元素中获取文本信息

**原型:** ``text = element:text()``

**返回值:** ``text`` 表示元素的文本内容

**是否异步:** 否

它会尝试返回以下JavaScript节点的属性值:

- textContent
- innerText
- value

如果这些值都为空，则会返回空值

.. _element-info:

element-info
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
获取元素的有用信息

**原型:** ``info = element:info()``

**返回值:** ``info`` 表示元素的相关信息

**是否异步:** 否

info 是一个包含以下字段的table结构

- nodeName - 以小写字母表示的节点的名称(比如 h1)
- attributes - 以属性名和属性值为键值对的table结构
- tag - 表示该元素的HTML的字符串
- html - 元素内部的HTML代码
- text - 元素内部的文本值
- x - 元素的x坐标
- y - 元素的y坐标
- width - 元素的宽
- height - 元素的高
- visible - 元素是否可见的一个标志

.. _element-field-value:

element:field_value
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
获取元素字段的值(针对 input, select, textarea, button)

**原型:** ``ok, value = element:field_value()``

**返回值:** ``ok, value`` 的元组,如果ok为 ``nil`` 表明在执行函数的过程中有错误发生,
此时 ``value`` 是一个包含错误类型信息的字符串。如果没有错误发生, ``ok`` 为 ``true`` ``value`` 为对应元素字段的值

**是否异步:** 否

这些方法按照以下方式工作:

    - **如果当前元素是** ``select`` :
        - 如果元素的 ``multiple`` 属性为 ``true`` 它会以table结构返回所有被选中的值
        - 否则返回被选中的一个值
    - **如果元素存在属性** ``type="radio"`` :
        - 如果它被选中，则返回选中的值
        - 否则返回 ``nil``
    - 如果元素存在属性 ``type="checkbox"`` 则返回bool值
    - 否则会返回元素 ``value`` 属性的值，或者当 ``value`` 属性不存在的时候返回空字符串

.. _element-form-values:

element:form_values
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
如果元素类型为表单，它会以table结构的方式返回表单中每项的值

**原型:** ``form_values, reason = element:form_values{values='auto'}``

**参数:**

- values: 返回值的类型，可以是 ``'auto'`` 、 ``'list'`` 或者 ``'first'`` 中的一个

**返回值:** ``form_values, reason`` 元组，如果 ``form_values`` 为nil，则表明当前执行函数时发生错误，或者当前元素不是表单类型。
``reason`` 返回错误类型信息, ``form_values`` 返回以表单元素名为键值，元素值为键值的table结构

**是否异步:** 否

函数的返回值取决于参数 ``values`` 的值，它的取值有下列三种

``'auto'``
    返回值将是table或者是一个单一的值，这将取决于表单的类型
        - 如果元素类型为 ``<select multiple>`` 返回值将会是一个包含所有选中项的table结构, 如果属性值不存在则返回一个文本内容
        - 如果表单中多个元素都有相同的 ``name`` 属性, 则返回一个包含该元素所有值的table结构
        - 否则将会是一个字符串(针对text框或者单选按钮框)或者是一个bool类型(多选框), 或者返回 ``nil``

    如果您需要在lua中使用这个返回值，这种返回值类型将会很方便

``'list'``
    返回值将一直是 table 结构(针对lists),即使在表单元素是一个单值的情况下。这一特性也可用于具有未知结构的表单。下面给出些许提示:
        - 如果元素是一个checkbox ，并且有value属性，那么table中将使用这个属性作为键值
        - 如果元素是一个 ``<select multiple>`` ,并且有一些还具有相同的名称,那么他们的值将于之前的值连接到一起
        - 在编写通用的表单处理代码时这个类型的返回值将会十分有用,与 ``'auto'`` 不同，您不需要考虑多种数据类型

``'first'``
    返回的值都是单一值,即使表单元素可以选择多个值,如果表单元素可以选择多个值，它总会返回被选中的第一个值。

例1：返回下列登录表单的值
::

    <form id="login">
        <input type="text" name="username" value="admin" />
        <input type="password" name="password" value="pass" />
        <input type="checkbox" name="remember" value="yes" checked />
    </form>

::

    function main(splash)
        -- ...
        local form = splash:select('#login')
        return assert(form:form_values())
    end

    -- returned values are
    { username = 'admin', password = 'pass', remember = true }

例2：当 ``values`` 的值为 ``'list'``
::

    function main(splash)
        -- ...
        local form = splash:select('#login')
        return assert(form:form_values{values='list'}))
    end

    -- returned values are
    { username = ['admin'], password = ['pass'], remember = ['checked'] }

例3：当 ``values`` 的值为 ``'first'`` 的时候返回下列表单的值
::

    <form>
        <input type="text" name="foo[]" value="coffee"/>
        <input type="text" name="foo[]" value="milk"/>
        <input type="text" name="foo[]" value="eggs"/>
        <input type="text" name="baz" value="foo"/>
        <input type="radio" name="choice" value="yes"/>
        <input type="radio" name="choice" value="no" checked/>
        <input type="checkbox" name="check" checked/>

        <select multiple name="selection">
            <option value="1" selected>1</option>
            <option value="2">2</option>
            <option value="3" selected>2</option>
        </select>
    </form>

::

    function main(splash)
        -- ...
        local form = splash:select('form')
        return assert(form:form_values(false))
    end

    -- returned values are
    {
        ['foo[]'] = 'coffee',
        baz = 'foo',
        choice = 'no',
        check = false,
        selection = '1'
    }

.. _element-fill:

element:fill
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
利用提供的值来填充表单

**原型:** ``ok, reason = element:fill(values)``

**参数:**

- values: 以表单项名称为键值，表单项的值作为键值的table结构

**返回值:** ``ok, reason`` 元组,如果 ``ok`` 为nil表明在函数执行过程中有错误发生, ``reason`` 返回错误类型的字符串信息

**是否异步:** 否

为了填充表单，您的输入框需要有 ``name`` 属性,而且需要事先选中表单

例1：获取表单的值，并修改password项
::

    function main(splash)
        -- ...
        local form = splash:select('#login')
        local values = assert(form:form_values())
        values.password = "l33t"
        assert(form:fill(values))
    end


例2：填充复杂的表单:
::

    <form id="signup" action="/signup">
        <input type="text" name="name"/>
        <input type="radio" name="gender" value="male"/>
        <input type="radio" name="gender" value="female"/>

        <select multiple name="hobbies">
            <option value="sport">Sport</option>
            <option value="cars">Cars</option>
            <option value="games">Video Games</option>
        </select>

        <button type="submit">Sign Up</button>
    </form>

::

    function main(splash)
      assert(splash:go(splash.args.url))
      assert(splash:wait(0.1))

      local form = splash:select('#signup')
      local values = {
        name = 'user',
        gender = 'female',
        hobbies = {'sport', 'games'},
      }

      assert(form:fill(values))
      assert(form:submit())
      -- ...
    end

.. _element-send-keys:

element:send_keys
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
向元素发送键盘消息

**原型:** ``ok, reason = element:send_keys(keys)``

**参数:**

- keys:要作为键盘事件发送的多个字符组成的字符串

**返回值:** ``ok, reason`` 元组,如果 ``ok`` 为nil 则表明在调用函数期间发生错误, ``reason`` 提供错误类型信息

**是否异步:** 否

这个方法主要进行这样几个操作

- 点击元素
- 向元素发送对应的键盘事件

更多信息，您可以参阅:  `splash:send_keys <./scripting-ref.html#splash-send-keys>`_

.. _element-send-text:

element:send_text
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

向元素发送键盘消息

**原型:** ``ok, reason = element:send_text(text)``

**参数:**

- text:将要被发送并作为元素输入的字符串

**返回值:** ``ok, reason`` 元组,如果 ``ok`` 为nil 则表明在调用函数期间发生错误, ``reason`` 提供错误类型信息

**是否异步:** 否

这个方法主要进行这样几个操作

- 点击元素
- 向元素发送指定的字符串,并作为元素的输入值

更多信息，您可以参阅:  `splash:send_text <./scripting-ref.html#splash-send-text>`_

.. _element-submit:

element:submit
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
提交表单

**原型:** ``ok, reason = element:submit()``

**返回值:** ``ok, reason`` 元组,如果 ``ok`` 为nil 则表明在调用函数期间发生错误(例如您尝试提交的元素不是表单),
``reason`` 提供错误类型信息

**是否异步:** 否

例：先获取表单的值，然后设置对应的值，最后提交
::

    <form id="login" action="/login">
        <input type="text" name="username" />
        <input type="password" name="password" />
        <input type="checkbox" name="remember" />
        <button type="submit">Submit</button>
    </form>

::

    function main(splash)
        -- ...
        local form = splash:select('#login')
        assert(form:fill({ username='admin', password='pass', remember=true }))
        assert(form:submit())
        -- ...
    end

.. _element-exists:

element:exists
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
在DOM中检查对应元素是否存在。如果不存在，某些方法可能会执行失败，并返回对应的标志

**原型:** ``exists = element:exists()``

**返回值:** ``exists`` 表明当前元素是否存在

**是否异步:** 否

.. note::

    不要使用 ``splash:select(..):exists()`` 来检查元素是否存在,如果选择器没有选择任何值那么
    `splash:select <./scripting-ref.html#splash-select>`_ 会返回 ``nil`` , 此时应该检查返回值是否为 ``nil``

    ``element:exists()`` 应该用于这样的情况：您之前确定有这么一个元素，但是不清楚后续它是否被从当前的DOM中移出。

下面将列举几种会将元素移出DOM的原因，其中一个理由是它可能被某些JavaScript代码给移除了

例1：元素被js代码移除
::

    function main(splash)
        -- ...
        local element = splash:select('.element')
        assert(splash:runjs('document.write("<body></body>")'))
        assert(splash:wait(0.1))
        local exists = element:exists() -- exists will be `false`
        -- ...
    end

另一个原因是元素是通过JavaScript代码创建的但是并没有加入到DOM中

例2：元素未被插入到DOM中
::

    function main(splash)
        -- ...
        local element = splash:select('.element')
        local cloned = element.node:cloneNode() -- the cloned element isn't in DOM
        local exists = cloned:exists() -- exists will be `false`
        -- ...
    end

.. _dom-methods:

DOM 方法
-------------------------------
除了特定的Splash自定义的方法，DOM元素也支持许多常见的DOM HTMLElement类的方法

.. _usage:

用法
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
您只需要在 ``element`` 对象上调用, 例如为了确定某个DOM元素是否具有某个特定的属性，
您可以调用 `hasAttribute <https://developer.mozilla.org/en-US/docs/Web/API/Element/hasAttribute>`_
::

    function main(splash)
        -- ...
        if splash:select('.element'):hasAttribute('foo') then
            -- ...
        end
        -- ...
    end

另一个例子：为了确保元素在视口中，您可以使用 ``scrollIntoViewIfNeeded`` 方法
::

    function main(splash)
        -- ...
        splash:select('.element'):scrollIntoViewIfNeeded()
        -- ...
    end

.. _supported-dom-methods:

被支持的DOM方法
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
**继承自** `EventTarget <https://developer.mozilla.org/en-US/docs/Web/API/EventTarget>`_ **的方法:**

- addEventListener
- removeEventListener

**继承自** `HTMLElement <https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement>`_ **的方法**

- blur
- click
- focus

**继承自** `Element <https://developer.mozilla.org/en-US/docs/Web/API/Element>`_ **的方法:**

- getAttribute
- getAttributeNS
- getBoundingClientRect
- getClientRects
- getElementsByClassName
- getElementsByTagName
- getElementsByTagNameNS
- hasAttribute
- hasAttributeNS
- hasAttributes
- querySelector
- querySelectorAll
- releasePointerCapture
- remove
- removeAttribute
- removeAttributeNS
- requestFullscreen
- requestPointerLock
- scrollIntoView
- scrollIntoViewIfNeeded
- setAttribute
- setAttributeNS
- setPointerCapture

**继承自** `Node <https://developer.mozilla.org/en-US/docs/Web/API/Node>`_ **的方法**

- appendChild
- cloneNode
- compareDocumentPosition
- contains
- hasChildNodes
- insertBefore
- isDefaultNamespace
- isEqualNode
- isSameNode
- lookupPrefix
- lookupNamespaceURI
- normalize
- removeChild
- replaceChild

这些方法应该作为JS的对应物,在Lua中使用

例如，您可以通过 ``element:addEventListener(event, listener)`` 方法来为元素添加对应事件的响应程序
::

    function main(splash)
        -- ...
        local element = splash:select('.element')
        local x, y = 0, 0

        local store_coordinates = function(event)
            x = event.clientX
            y = event.clientY
        end

        element:addEventListener('click', store_coordinates)
        assert(splash:wait(10))
        return x, y
    end

.. _attributes:

属性
--------------------------------

.. _element-node:

element.node
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
``element.node`` 可以对外暴露DOM元素所有可公开的属性和方法，但是不包括Splash自定义的属性和方法。如果您想要更准确的使用它，请使用
只读的方式。在将来他可能会允许避免可能出现的命名冲突

例如，您需要获取元素的 innerHTML 值，您可以使用 ``.node.innerHTML``
::

    function main(splash)
        -- ...
        return {html=splash:select('.element').node.innerHTML}
    end

.. _element-inner-id:

element.inner_id
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
元素内部表示的ID。它对于相等的元素实例进行比较可能很有用

例如:
::

    function main(splash)
        -- ...
        local same = element2.inner_id == element2.inner_id
        -- ...
    end

.. _dom-attributes:

DOM 属性
---------------------------------------

.. _id1:

用法
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Element 对象也提供了对几乎所有DOM元素属性的支持，例如您可以获取DOM元素的节点名称(p, div, a等等)
::

    function main(splash)
        -- ...
        local tag_name = splash:select('.foo').nodeName
        -- ...
    end

大部分元素属性不光是可读，它们也是可写的，例如您可以设置元素 innerHTML 属性的值
::

    function main(splash)
        -- ...
        splash:select('.foo').innerHTML = "hello"
        -- ...
    end

.. _supported-dom-attributes:

被支持的DOM属性
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
下面将列举被支持的DOM属性(某些是可读写的，某些是只读的)

**继承自** `HTMLElement <https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement>`_ **的属性**

- accessKey
- accessKeyLabel (只读)
- contentEditable
- isContentEditable (只读)
- dataset (只读)
- dir
- draggable
- hidden
- lang
- offsetHeight (只读)
- offsetLeft (只读)
- offsetParent (只读)
- offsetTop (只读)
- spellcheck
- style - 用一个table表示，table的值可以被修改
- tabIndex
- title
- translate

**继承自** `Element <https://developer.mozilla.org/en-US/docs/Web/API/Element>`_ **的属性**

- attributes (只读) - 一个 table 结构表示的元素的属性
- classList (只读) - 一个 table 结构表示的元素类名
- className
- clientHeight (只读)
- clientLeft (只读)
- clientTop (只读)
- clientWidth (只读)
- id
- innerHTML
- localeName (只读)
- namespaceURI (只读)
- nextElementSibling (只读)
- outerHTML
- prefix (只读)
- previousElementSibling (只读)
- scrollHeight (只读)
- scrollLeft
- scrollTop
- scrollWidth (只读)
- tabStop
- tagName (只读)

**继承自** `Node <https://developer.mozilla.org/en-US/docs/Web/API/Node>`_ **的属性**

- baseURI (只读)
- childNodes (只读)
- firstChild (只读)
- lastChild (只读)
- nextSibling (只读)
- nodeName (只读)
- nodeType (只读)
- nodeValue
- ownerDocument (只读)
- parentNode (只读)
- parentElement (只读)
- previousSibling (只读)
- rootNode (只读)
- textContent

当然您也可以通过属性在特殊的事件上添加指定的事件处理函数.当这个处理函数被调用时，他将会接收到一个 ``event`` 的table结构,
这个结构包含所有可用的方法和属性
::

    function main(splash)
        -- ...
        local element = splash:select('.element')

        local x, y = 0, 0

        element.onclick = function(event)
            event:preventDefault()
            x = event.clientX
            y = event.clientY
        end

        assert(splash:wait(10))

        return x, y
    end

如果您希望在一个事件上添加多个事件处理程序，请使用 ``element:addEventListener()``
