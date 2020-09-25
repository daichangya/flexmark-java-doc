flexmark-java目录扩展

### 概述

目录扩展实际上是两个扩展合而为一:`[TOC]`元素呈现一个
目录和模拟的`[TOC]:#`元素,该元素还呈现目录,但
也用于后处理程序,该后处理程序将在目录后附加目录。
生成带有目录元素的源文件,该目录将在任何减价时呈现
处理器。

### 句法

该扩展程序具有一个类,用于生成更新的文本,使其位于模拟的TOC元素下。

TOC标记具有以下格式:`[TOC style]`

模拟的TOC标签具有以下格式:`[TOC style]: # "Title"`

1. `style`由空格分隔的选项列表组成:

   * `levels=levelList`其中,级别列表是用逗号分隔的级别或范围列表。默认
     包含标题级别2和3。如果使用`-`作为分隔符指定范围
     在数字值之间,则包含的范围是从第一个数字到最后一个数字
     包括的。如果缺少第一个数字,则将在其位置使用1。如果最后一个数字是
     如果缺少,则将使用6。
   
     例子:
     * `levels=1`仅包含1级,与`levels=1,1`相同
     * `levels=2`仅包含2级,与`levels=2,2`相同
     * `levels=3`包括2级和3级
     * `levels=4`包括2,3和4级
     * `levels=2-4`包括级别2,3和4。与`levels=4`相同
     * `levels=2-4,5`包括2,3,4和5级
     * `levels=1,3`包括1级和3级
     * `levels=-3`包括1到3级
     * `levels=3-`包含3到6级

   * `html`仅模拟TOC,生成HTML版本的TOC内容

   * `markdown`仅模拟TOC,生成TOC内容的Markdown版本

   * `text`仅包含标题文本

   * `formatted`包括文本和内联格式

   * `bullet`将项目列表用于项目目录

   * `numbered`对目录项使用编号列表

   * `hierarchy`:标题的分层列表

   * `flat`:标题的固定列表

   * `reversed`:标题的倒排列表

   * `increasing`:扁平化,通过标题文本按字母顺序递增

   * `decreasing`:平,按标题文本按字母顺序减少

2. `"Title"`指定目录标题的文本。如果省略或为空白,则为否
   将为目录生成标题。标题中的`#`前缀将指定
   标头级别,用于目录列表上方的标题。如果没有`#`前缀
   则标题级别将从`SimTocExtension.TITLE_LEVEL`选项中获取。

`[TOC]: #`标记后的行旨在进行更新,以反映出
文献。子节点`SimTocContent`表示将被替换为
更新的目录。

### 解析详细信息

在工件`flexmark-ext-toc`中使用类`TocExtension`或`SimTocExtension`。

提供以下选项:

在`TocExtension`类中定义:

|静态字段|默认值|说明|
|:----------------------- |:-------------- |:------- -------------------------------------------------- -------------------------------------------------- --------------------------------------- |
| LEVEL | `0x00C0` |要包括在TOC中的标头级别的位掩码,1 << header_level,使用`TocOptions.getLevels(int...)`并传入标头级别列表|
| IS_TEXT_ONLY | `false` |呈现的目录仅使用标题元素的文本,强调和其他内联处理被忽略。|
| IS_NUMBERED | `false` |使用有序列表来生成标题|的列表
| IS_HTML | `false` |使用HTML版本的TOC生成内容文本,否则Markdown(仅适用于必须提供的文件的后处理)|
| TITLE_LEVEL | 1 |如果标题字符串中未指定带前缀`#`的|,则目录标题的默认标题级别
| TITLE | `""` |如果未使用元素|指定目录标题的默认文本
| DIV_CLASS | `""` |用于内容表`div`包装器|的类属性
| LIST_CLASS | `""` |用于目录`ul` |的类属性
| AST_INCLUDE_OPTIONS | `false` |包括在TOC元素中定义的样式选项,作为AST的一部分。默认样式不包含在ast中,样式可在TocBlock |中使用
| CASE_SENSITIVE_TOC_TAG | `true` |将其设置为false可以匹配`[TOC]`和`[TOC]: #`,而不会区分大小写。toc选项的语法仍然区分大小写。|

在`SimTocExtension`类中定义:

|静态字段|默认值|说明|
|:------------------------- |:--------------------- -|:--------------------------------------------------- -------------------------------------------------- -------------------------------------------------- ------ |
|级* | `0x00C0` |要包括在目录中的标头级别的位掩码,1 << header_level,使用`TocOptions.getLevels(int...)`并传入标头级别列表|
| IS_TEXT_ONLY * | `false` |呈现的目录仅使用标题元素的文本,强调和其他内联处理被忽略。|
| IS_NUMBERED * | `false` |使用有序列表生成标题列表|
| IS_HTML * | `false` |使用HTML版本的TOC生成内容文本,否则Markdown(仅适用于对必须提供的文件进行后期处理)|
| TITLE_LEVEL * | 1 |如果未在带有`#`前缀的标题字符串中指定一个标题,则目录标题的默认标题级别为|
| TITLE * | `"Table of Contents"` |如果未使用元素|指定目录标题的默认文本
| DIV_CLASS * | `""` |用于内容表`div`包装器|的类属性
| LIST_CLASS * | `""` |用于内容表的类属性`ul` |
| AST_INCLUDE_OPTIONS * | `false` |包含在sim TOC元素中定义的样式选项,作为AST的一部分。ast中未包含默认样式,SimTocBlock |中提供了样式
| CASE_SENSITIVE_TOC_TAG * | `true` |将其设置为false可以匹配`[TOC]`和`[TOC]: #`而不会区分大小写。toc选项的语法仍然区分大小写。|

\* :information_source:这些键是TocExtension中的键的引用。两组
引用数据集中的相同数据点。

##### 自定义HTML属性提供程序

自`Node`以来,AttributablePart区分用于呈现元素的各个标签。
不能用于区分复合HTML呈现中的标记。例如自定义
属性提供者参考
[AttributeProviderSample2.java](https://github.com/vsch/flexmark-java/blob/master/flexmark-java-samples/src/com/vladsch/flexmark/java/samples/AttributeProviderSample2.java)

这些定义在`TocExtension`中,为方便起见,在`SimTocExtension`中重复

|可归属的零件|说明|
|:----------------- |:----------------------------- ----------------------------- |
| TOC_CONTENT |标识wrap目录|的`div`标签
| TOC_LIST |标识用于TOC标题列表|的`ul`或`ol`标签