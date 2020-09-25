## ![flexmark icon logo] flexmark-java

Java库,用于根据[CommonMark]解析和呈现[Markdown]文本
具有许多扩展的规范。最初是用来替代[pegdown]解析器的
在[JetBrains]系列IDE的[Markdown Navigator]插件中使用。

提供用于将输入解析为节点的抽象语法树(AST),访问和
操作节点,并渲染为HTML。生成AST是[commonmark-java]的重做
表示markdown源元素,其中包含每个字符的完整源偏移量详细信息
元素的AST,允许重新创建原始源或将其用于语法
突出显示。

它具有与[commonmark-java]类似的API,并为核心增加了许多扩展和增强功能
具有以下功能:

*核心库较小(最小依赖性),扩展依赖性有所不同。
*快速,比固定速度快30倍,比[intellij-markdown]快10倍,仅约
  添加源跟踪和其他markdown后,比[commonmark-java]慢35%-50%
  仿真模式。
*灵活:自定义块解析器,定界符解析器,解析后操作AST,自定义
  HTML渲染,为所有内容滚动自己的选项。
*可扩展到表格,删除线,自动链接和其他扩展
  框。
*完整的AST中的源位置跟踪,包括所有markdown源元素,并
  源中每个非空格字符的源偏移量
*通用选项API,用于配置解析器,渲染器和扩展。
*几乎可以修改核心解析器的所有行为,包括禁用和替换
  核心块解析器和默认的内联处理。

### Wiki主题

* [Usage](Usage)
* [Pegdown Migration](Pegdown-Migration)
* [Extensions](Extensions)
* [Markdown Formatter](Markdown-Formatter)
* [Writing Extensions](Writing-Extensions)
* [Testing](Testing)
* [Basic Benchmarks](Basic-Benchmarks)

### 要求

* Java 7或以上
*要添加Android兼容性
*该项目在Maven上:`com.vladsch.flexmark`
*核心没有依赖关系;有关扩展,请参见下文

  API仍在不断发展以适应新的扩展和功能。

###功能比较

|功能| flexmark-java | commmonmark-java | pegdown |
| ------------------------------------------------- --------------------------------- |:--------------- -------------------------------------------------- |:------------------------------------------------ ------------------ |:------------------------------ --------------------------------------- |
|相对解析时间(越少越好)|:heavy_check_mark:1x |:heavy_check_mark:0.6x至0.7x |:x:25x平均,对于病理学输入[[2]] [#2)|
| AST中的所有源元素|:heavy_check_mark:|:heavy_check_mark:|:heavy_check_mark:|
|个具有源位置|的AST元素:heavy_check_mark:|:x:|:heavy_check_mark:有一些错误和特质|
| AST易于操作|:heavy_check_mark:AST后处理是扩展机制|:heavy_check_mark:AST后处理是扩展机制|:x:不是一种选择。没有节点的父级信息,子级为List <>。|
| AST元素具有所有零件的详细源位置|:heavy_check_mark:|:x:|:x:仅节点开始/结束|
|可以禁用核心解析功能|:heavy_check_mark:|:x:|:x:|
|通过扩展API实现的核心解析器|:heavy_check_mark:|:x:针对特定块解析器和节点类的`instanceOf`测试|:x:核心仅暴露了几个扩展点|
|易于理解和修改解析器实现|:heavy_check_mark:|:heavy_check_mark:|:x:一个具有复杂交互作用的大型PEG解析器[(2)](#2)|
|块元素的解析彼此独立|:heavy_check_mark:[(1)](#1)|:heavy_check_mark:[[1)](#1)|:x:PEG语法|中的所有内容
|跨以下配置的统一配置:解析器,渲染器和所有扩展|:heavy_check_mark:|:x:无扩展列表之外的选项|:x:`int`内核的位标志,扩展的|
|解析性能已优化,可与扩展一起使用|:heavy_check_mark:|:x:解析内核的性能,扩展将尽其所能|:x:性能不是一项功能|
|功能丰富,开箱即用,具有许多配置选项和扩展功能|:heavy_check_mark:|:x:受限扩展,无选项|:heavy_check_mark:|
|处理器的依赖关系定义,以确保正确的处理顺序|:heavy_check_mark:|:x:由扩展列表顺序指定的顺序,容易出错|:x:不适用,核心定义添加扩展处理的位置|


###### (1)

pathological input 10,000个`[`在11ms内解析,10,000个嵌套的`[` `]`在450ms内解析

###### (2)

650ms解析17 `[`的pathological input ,1300ms解析18个`[`

###历史

创建该项目的动机是将[pegdown]替换为
[JetBrains]系列IDE的[Markdown Navigator]插件。[pegdown]有许多性能问题
由于其实施而无法解决。此外,[pegdown]解析
markdown元素彼此交互,并且只能解析完整文件
标记一个不应发生回滚的点。对于某些来源,这导致[pegdown]
进入指数解析时间,在少数情况下进入无限解析循环
可以通过将语法更改为不再与Markdown兼容来解决。

因为我需要实现很多扩展才能使此解析器成为[pegdown]的超集,所以我
想要提高扩展功能以修改解析器的行为以允许
通过扩展机制实现任何markdown方言。我也想删除
样板代码和扩展名中的测试使用[commonmark] spec.txt格式,但使用
将AST作为测试的一部分,以便可以对每一个AST进行验证
构造和每个扩展。

最终目标是拥有一个可以轻松扩展以与主要版本兼容的解析器。
处理器系列和单个处理器版本:

* [CommonMark]为最新实施的规范,当前为[CommonMark (spec 0.28)]
  * [ ]&nbsp; [League/CommonMark]
  * [CommonMark (spec 0.27)]用于特定版本兼容性
  * [CommonMark (spec 0.28)]用于特定版本兼容性
  * [GitHub]评论
* [Kramdown]
  * [ ]&nbsp; [Jekyll]
* [Markdown.pl][Markdown]
  * [ ]&nbsp; [Php Markdown Extra]
  * [GitHub]文档
*固定缩进
  * [MultiMarkdown]
  * [Pegdown]

尽管其名称,commonmark既不是其他Markdown风味的超集也不是其子集。
相反,它为原始的"核心"提出了标准,明确的语法规范
Markdown,从而有效地引入了另一种风味。虽然flexmark是默认设置
符合commonmark,它的解析器可以通过各种方式进行调整。需要进行的一组调整
在flexmark中模拟最常用的markdown解析器,如下所示:
ParserEmulationProfiles。

顾名思义,`ParserEmulationProfile`只是将解析器调整为
特定的markdown方言。应用配置文件不会添加功能超出了
共同商标。如果您希望flexmark更好地模拟另一个处理器的行为,则必须
调整解析器并配置扩展以支持您所需要的解析器功能
想效仿。

[CommonMark]: https://commonmark.org/
[CommonMark (spec 0.27)]: https://spec.commonmark.org/0.27
[CommonMark (spec 0.28)]: https://spec.commonmark.org/0.28
[commonmark-java]: https://github.com/atlassian/commonmark-java
[flexmark icon logo]: https://github.com/vsch/flexmark-java/raw/master/assets/images/flexmark-icon-logo.png
[GitHub]: https://github.com/vsch/laravel-translation-manager
[intellij-markdown]: https://github.com/valich/intellij-markdown
[Jekyll]: https://jekyllrb.com
[JetBrains]: https://plugins.jetbrains.com
[Kramdown]: https://kramdown.gettalong.org/
[League/CommonMark]: https://github.com/thephpleague/commonmark
[Markdown]: https://daringfireball.net/projects/markdown/
[Markdown Navigator]: http://vladsch.com/product/markdown-navigator
[MultiMarkdown]: https://fletcherpenney.net/multimarkdown/
[pegdown]: http://pegdown.org
[PHP Markdown Extra]: https://michelf.ca/projects/php-markdown/extra/#abbr
[.gitignore]: http://hsz.mobi
[Android Studio]: https://developer.android.com/sdk/installing/studio.html
[AppCode]: http://www.jetbrains.com/objc
[autolink-java]: https://github.com/robinst/autolink-java
[CLion]: https://www.jetbrains.com/clion
[commonmark.js]: https://github.com/jgm/commonmark.js
[commonMarkSpec.md]: https://github.com/vsch/idea-multimarkdown/blob/master/test/data/performance/commonMarkSpec.md
[Craig's List]: https://montreal.craigslist.org
[DataGrip]: https://www.jetbrains.com/datagrip
[flexmark-java]: https://github.com/vsch/flexmark-java
[flexmark-java on Maven]: https://search.maven.org/search?q=g:com.vladsch.flexmark

[gfm-tables]: https://help.github.com/articles/organizing-information-with-tables/
[GitHub Flavoured Markdown]: https://help.github.com/articles/basic-writing-and-formatting-syntax/
[GitHub Issues page]: issues
[hang-pegdown.md]: https://github.com/vsch/idea-multimarkdown/blob/master/test/data/performance/hang-pegdown.md
[hang-pegdown2.md]: https://github.com/vsch/idea-multimarkdown/blob/master/test/data/performance/hang-pegdown2.md
[idea-markdown]: https://github.com/nicoulaj/idea-markdown
[IntelliJ IDEA]: http://www.jetbrains.com/idea
[JetBrains plugin page]: https://plugins.jetbrains.com/plugin/7896-markdown-navigator
[Kotlin]: http://kotlinlang.org
[Maven Central status]: https://img.shields.io/maven-central/v/com.vladsch.flexmark/flexmark.svg
[nicoulaj]: https://github.com/nicoulaj
[nicoulaj/idea-markdown plugin]: https://github.com/nicoulaj/idea-markdown
[Pandoc]: http://pandoc.org/MANUAL.html#pandocs-markdown
[PhpExtra]: https://michelf.ca/projects/php-markdown/extra/
[PhpStorm]: http://www.jetbrains.com/phpstorm
[Pipe Table Formatter]: https://github.com/anton-dev-ua/PipeTableFormatter
[PyCharm]: http://www.jetbrains.com/pycharm
[RubyMine]: http://www.jetbrains.com/ruby
[Semantic Versioning]: https://semver.org/
[sirthias]: https://github.com/sirthias
[spec.txt]: https://github.com/vsch/idea-multimarkdown/blob/master/test/data/performance/spec.md
[table.md]: https://github.com/vsch/idea-multimarkdown/blob/master/test/data/performance/table.md
[VERSION.md]: https://github.com/vsch/idea-multimarkdown/blob/master/test/data/performance/VERSION.md
[vsch/pegdown]: https://github.com/vsch/pegdown/tree/develop
[WebStorm]: http://www.jetbrains.com/webstorm
[wrap.md]: https://github.com/vsch/idea-multimarkdown/blob/master/test/data/performance/wrap.md