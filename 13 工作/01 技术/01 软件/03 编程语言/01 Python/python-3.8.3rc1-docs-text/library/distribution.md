软件打包和分发
**************

这些库可帮助你发布和安装 Python 软件。 虽然这些模块设计为与`Python 包
索引 <https://pypi.org>`__结合使用，但它们也可以与本地索引服务器一起使
用，或者根本不使用任何索引服务器。

* "distutils" --- 构建和安装 Python 模块

* "ensurepip" --- Bootstrapping the "pip" installer

  * Command line interface

  * Module API

* "venv" --- 创建虚拟环境

  * 创建虚拟环境

  * API

  * 一个扩展 "EnvBuilder" 的例子

* "zipapp" --- Manage executable Python zip archives

  * Basic Example

  * 命令行界面

  * Python API

  * 示例

  * Specifying the Interpreter

  * Creating Standalone Applications with zipapp

    * Making a Windows executable

    * Caveats

  * The Python Zip Application Archive Format
