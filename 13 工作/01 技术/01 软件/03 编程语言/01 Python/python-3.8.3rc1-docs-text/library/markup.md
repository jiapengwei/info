结构化标记处理工具
******************

Python 支持各种模块，以处理各种形式的结构化数据标记。 这包括使用标准通
用标记语言（SGML）和超文本标记语言（HTML）的模块，以及使用可扩展标记语
言（XML）的几个接口。

* "html" --- 超文本标记语言支持

* "html.parser" --- 简单的 HTML 和 XHTML 解析器

  * HTML 解析器的示例程序

  * "HTMLParser" 方法

  * 示例

* "html.entities" --- HTML 一般实体的定义

* XML处理模块

  * XML 漏洞

  * "defusedxml" 和 "defusedexpat" 软件包

* "xml.etree.ElementTree" ---  ElementTree XML API

  * 教程

    * XML树和元素

    * 解析XML

    * Pull API进行非阻塞解析

    * 查找感兴趣的元素

    * 修改XML文件

    * 构建XML文档

    * 使用命名空间解析XML

    * 其他资源

  * XPath支持

    * 示例

    * 支持的XPath语法

  * 参考引用

    * 函数

  * XInclude 支持

    * 示例

  * 参考引用

    * 函数

    * 元素对象

    * ElementTree 对象

    * QName Objects

    * TreeBuilder Objects

    * XMLParser对象

    * XMLPullParser对象

    * 异常

* "xml.dom" --- The Document Object Model API

  * 模块内容

  * Objects in the DOM

    * DOMImplementation Objects

    * 节点对象

    * 节点列表对象

    * 文档类型对象

    * 文档对象

    * 元素对象

    * Attr 对象

    * NamedNodeMap 对象

    * 注释对象

    * Text 和 CDATASection 对象

    * ProcessingInstruction 对象

    * 异常

  * 一致性

    * 类型映射

    * Accessor Methods

* "xml.dom.minidom" --- Minimal DOM implementation

  * DOM Objects

  * DOM Example

  * minidom and the DOM standard

* "xml.dom.pulldom" --- Support for building partial DOM trees

  * DOMEventStream Objects

* "xml.sax" --- Support for SAX2 parsers

  * SAXException Objects

* "xml.sax.handler" --- Base classes for SAX handlers

  * ContentHandler 对象

  * DTDHandler 对象

  * EntityResolver 对象

  * ErrorHandler 对象

* "xml.sax.saxutils" --- SAX 工具集

* "xml.sax.xmlreader" --- Interface for XML parsers

  * XMLReader 对象

  * IncrementalParser 对象

  * Locator 对象

  * InputSource 对象

  * The "Attributes" Interface

  * The "AttributesNS" Interface

* "xml.parsers.expat" --- Fast XML parsing using Expat

  * XMLParser对象

  * ExpatError Exceptions

  * 示例

  * Content Model Descriptions

  * Expat error constants
