.. splash中文文档 documentation master file, created by
   sphinx-quickstart on Thu May 24 21:40:31 2018.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Splash - 一个JavaScript渲染服务
==========================================
spalsh 提供JavaScript渲染服务，它是一个使用Twisted和QT5在Python
3中实现的支持HTTP
API调用的轻量级的web浏览器。它使用Twisted和QT的反射机制以使服务完全异步并通过QT主循环以便利用webkit并发性

   这段话的原文是"The (twisted) QT reactor is used to make the service
   fully asynchronous allowing to take advantage of webkit concurrency
   via QT main
   loop."这里我实在是不知道怎么样把这个意思表达清楚，只能根据我的理解做适当的修改，可能表述不当

Splash 的部分特征:

- 并行处理多个页面
- 获取返回的HTML代码或者获取返回页面的截屏图片
- 通过禁止图片加载或者使用Adblock Plus插件来提高加载页面的速度
- 在页面的上下文中执行用户的JavaScript代码
- 编写lua脚本来操作浏览器
- 在Splash-Jupyter中支持lua脚本
- 在格式化的HAR 数据中获取渲染的相关细节

文档目录
----------------------------------------------
.. toctree::
   :maxdepth: 2

   Installation
   api
   scripting-tutorial
   scripting-overview
   scripting-ref
   scripting-response-object
