########
安全
########

这篇文章将介绍一些基本的关于 Web 安全的 "最佳实践" ，并详细说明了 CodeIgniter
内部的安全特性。

URI 安全
============

CodeIgniter 严格限制 URI 中允许出现的字符，以此来减少恶意数据传到你的应用程序的可能性。
URI 中只允许包含一些字符：

-  字母和数字
-  波浪符：~
-  百分号：%
-  句号：.
-  分号：:
-  下划线：\_
-  连字号：-
-  空格

Register_globals
================

在系统初始化期间，如果发现任何 ``$_GET``、``$_POST``、``$_REQUEST`` 和 ``$_COOKIE`` 
数组中的键值变成了全局变量，则删除该变量。

这个过程和设置 *register_globals = off* 效果是一样的。
（译注：阅读这里了解 `register_globals 设置 <http://php.net/manual/zh/security.globals.php>`_ ）

display_errors
==============

在生产环境下，一般都是通过将 *display_errors* 标志设置为 0 来禁用 PHP 的错误报告。
这可以阻止原生的 PHP 错误被显示到页面上，错误中可能会包含潜在的敏感信息。

在 CodeIgniter 中，可以将 index.php 文件中的 **ENVIRONMENT** 常量设置为 **\'production\'** ，
这样也可以关闭这些错误信息。在开发模式下，建议将它设置为 'development' 。
关于不同环境之间的区别可以阅读 :doc:`处理多环境 <environments>` 页面了解更多。

magic_quotes_runtime
====================

在系统初始化期间， *magic_quotes_runtime* 指令会被禁用，
这样当你在从数据库中获取数据时就不用再去除反斜线了。

**************
最佳实践
**************

在你的应用程序处理任何数据之前，无论这些数据是来自于提交的表单 POST ，还是来自
COOKIE、URI、XML-RPC ，或者甚至是来自于 SERVER 数组，你都应该使用下面这三步
来处理：

#. 验证数据类型是否正确，以及长度、大小等等
#. 过滤不良数据
#. 在提交到数据库或者显示到浏览器之前对数据进行转义

CodeIgniter 提供了以下的方法和技巧来帮你处理该过程：

XSS 过滤
=============

CodeIgniter 自带有一个 XSS 过滤器，这个过滤器可以查找一些 XSS 的常用技术，
譬如向你的数据中嵌入恶意的 JavaScript 脚本，劫持 cookie 信息或其他一些技术。
XSS 过滤器在 :doc:`这里 <../libraries/security>` 有更详细的描述。

.. note:: XSS 过滤 *只应该在输出数据时使用* 。 对输入的数据进行过滤可能会
	在无意中对数据造成修改，譬如过滤密码中的特殊字符，这样会降低安全性，
	而不是提高安全性。

CSRF 保护
===============

CSRF（Cross-Site Request Forgery，跨站请求伪造）是攻击者骗取受害者
在不知情的情况下提交请求的攻击方式。

CodeIgniter 提供了对 CSRF 的保护，会在每个非 GET HTTP 请求时自动触发，
当然前提是你要使用某种方式来创建表单，这在 :doc:`安全类 <../libraries/security>` 
文档中有进一步的解释。

密码处理
=================

在你的应用程序中正确处理密码是非常关键的。

但是不幸的是，许多开发者并不知道怎么去做，而且网络上充斥着大量过时的
甚至错误的建议，提供不了任何帮助。

我们提供了一个清单来帮助你，告诉你什么该做，什么不该做。

-  绝不要以明文存储密码。

   永远使用 **哈希算法** 来处理密码。

-  绝不要使用 Base64 或其他编码方式来存储密码。

   这和以明文存储密码是一样的，使用 **哈希** ，而不要使用 **编码** 。

   编码以及加密，都是双向的过程，而密码是保密的，应该只被它的所有者知道，
   这个过程必须是单向的。哈希正是用于做这个的，从来没有解哈希这种说法，
   但是编码就存在解码，加密就存在解密。

-  绝不要使用弱哈希或已被破解的哈希算法，像 MD5 或 SHA1 。

   这些算法太老了，而且被证明存在缺陷，它们一开始就并不是为了保存密码而设计的。

   另外，绝不要自己发明算法。

   只使用强密码哈希算法，譬如 BCrypt ，在 PHP 自己的 `密码哈希 <http://php.net/password>`_ 
   函数中也是使用它。

   即使你的 PHP 版本不是 5.5+ ，也请使用它们，CodeIgniter 为你提供了这些算法，只要你的 PHP
   版本是 5.3.7 以上都可以使用。（如果不满足这点要求，那么请升级你的 PHP）

   如果你连升级 PHP 也无法做到，那么使用 `hash_pbkdf() <http://php.net/hash_pbkdf2>` 吧，
   为实现兼容性我们提供了这个函数。

-  绝不要以明文形式显示或发送密码。

   即使是对密码的所有者也应该这样。如果你需要 "忘记密码" 的功能，可以随机生成一个新的
   一次性的（这点很重要）密码，然后把这个密码发送给用户。

-  绝不要对用户的密码做一些没必要的限制。

   如果你使用除 BCrypt （它有最多 72 字符的限制）之外的其他哈希算法，你应该设置一个相对
   长一点的密码长度（例如 1024 字符），这样可以缓解 DoS 攻击 。（这样可以缓解 DoS 攻击？）

   但是除此之外，对密码的其他限制诸如密码中只允许使用某些字符，或者密码中不允许包含某些字符，
   就没有任何意义了。

   这样做不仅不会提高安全性，反而降低了安全性，而且真的没有任何理由需要这样做。
   只要你对密码进行哈希处理了，那么无论是技术上，还是在存储上都没有任何限制。

验证输入数据
===================

CodeIgniter 有一个 :doc:`表单验证类 <../libraries/form_validation>` 用于帮助你验证、
过滤以及预处理你的数据。

就算这个类不适用于你的使用场景，那么你也应该确保对输入数据进行验证过滤。
例如，你希望接受一个数字型的参数，你可以使用  ``is_numeric()`` 或 ``ctype_digit()``
函数来检查一下。永远将数据限制在你运行的范围内。

记住，不仅要验证 ``$_POST`` 和 ``$_GET`` 变量，而且也不要放过 cookie 、user-agent
以及 **其他所有的不是直接由你的代码生成的数据** 。

插入数据库之前对数据进行转义
=========================================

永远不要不做转义就将数据插入到数据库，更多信息，可以阅读 :doc:`数据库查询
<../database/queries>` 这一节。

隐藏你的文件
===============

另一个很好的安全实践是，在你的 *webroot* 目录（通常目录名为 "htdocs/"）下只保留
*index.php* 文件和 "assets" 目录（用于存放 js、css、图片等静态资源）。
只需要这些文件能从 Web 上访问就可以了。

允许你的访问者访问其他位置可能潜在的导致他们访问一些敏感数据或者执行脚本等等。

如果你不允许这样做，你可以使用 .htaccess 文件来限制对这些资源的访问。

CodeIgniter 在每个目录下放置了一个 index.html 文件，视图隐藏这些敏感数据，
但是要记住的是，这对于防止一个真正的攻击者来说并不够。
