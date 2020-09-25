[TOC]: # ""

-[摘要](#summary)
-[模块重组](#module-reorganization)
-[迁移到0.60](#migrating-060)
  -[LineFormattingAppendable](#lineformattingappendable)
  -[BasedSequence](#basedsequence)
  -[SegmentedSequence](#segmentedsequence)

:warning:0.60.0版本由于重组,重命名,清理和更新而发生重大变化。
优化实现类。

在许多情况下,更改和优化的程度使得无法使用慢速
`@Deprecated`注释并最终删除。需要手动更改代码才能
迁移代码。

:information_source:如果您在迁移某些代码结构时遇到困难,请打开
一个[issue](https://github.com/vsch/flexmark-java/issues)<!-@IGNORE PREVIOUS:链接->与
要迁移的构造的详细信息。

主要改进包括:

*使用具有有效访问权限的二进制偏移树来实现##109##的新实现,
  存储和实例化。

*新的`SequenceBuilder`类,用于创建任意内容的分段序列,而无需
  关注段排序或它们是否共享公共碱基序列。

  不能转换为基本序列偏移范围的片段将是
  转换为基本字符之外的字符,以保留预期的字符序列结果。

  构建器将在与相应的基本序列匹配时优化文字字符
  特殊处理空格的字符和EOL字符。这意味着添加文字
  空格和EOL字符而不是使用子序列将导致它们有效
  替换为原始基本序列中的片段。

  为了方便起见,可以从任何基础序列中获取`SequenceBuilder`的实例
  通过`BasedSequence.getBuilder()`方法。

*新的`LineAppendable`实现用于呈现文本。在内部,该类建立一个
  行列表,并跟踪每行的前缀部分,从而允许高效访问和
  渲染结果中的行和前缀的操作。

  生成的##102##结果将导致`SegmentedSequence`的偏移量为
  保留源序列,允许将结果中的偏移量映射到原始序列。

  结果行存储为单独的`BasedSequences`,以最大程度地保留原始内容
  渲染重新排列源的行时的序列偏移信息,如
  使用参考定义排序或`MarkdownTable`排序进行格式化的情况。

*格式化模块现在是核心库的一部分,具有其他功能:
  *段落文字换行以适合设置的右边距。
  *从格式到原始减价的源偏移量跟踪
  *表格式包括按列排序和转置表方法

## Summary

*重大重组和实施代码清理
  *格式化程序实现现在是`flexmark`模块中核心实现的一部分
  * `Formatter`改进了更多选项,包括将文本包装到页边距。
    *增加了以格式化的顺序跟踪和将源偏移量映射到其索引的功能。这个
      此功能允许编辑者在格式化操作中插入符号的位置。
    *使用`TrackedOffset`统一进行偏移跟踪。`MarkdownParagraph`用于文本
      包装和`MarkdownTable`用于表格格式化,并能够在处理期间插入符号
      键入和退格键编辑操作,然后立即进行格式化或
      编辑的源。
  *清理测试以消除重复和黑客攻击
  * `flexmark-test-util`可重用于其他项目。将markdown作为源代码
    用于测试太方便了,因此仅在`flexmark-java`测试中使用。
  *优化的`SegmentedSequence`实现,使用二叉树搜索细分和
    字节有效的段压缩。解析器性能是否有所改善
    受影响,但允许使用`SegmentedSequences`收集`Formatter`和`HtmlRenderer`
    输出以最小的开销跟踪所有文本的源位置,并使性能提高了一倍
    旧的实现。
  * `LineAppendable`的新实现替换了用于文本的##78##
    渲染中的生成:
    *使用`SequenceBuilder`生成具有原始源偏移量的`BasedSequence`结果
      这些字符段来自源。这允许往返源
      在整个库中从Source-> AST-> Formatted Source-> Source进行跟踪。

      另外,使用可附加功能使格式化速度比以前快40%**
      实施,内存使用效率提高了160倍。对于以下测试,旧
      实施分配了价值6GB的分段序列,新实施分配了37MB。%
      新实施的开销是之前的四倍,但是是
      总开销字节减少了43倍,旧的实现需要342MB的开销,
      新实施8MB。

      由于提高了效率,因此可以创建两个大约600kB的附加文件。
      包含在测试运行中,并且仅使格式化程序运行时间增加0.6秒。

    测试在GitHub项目和其他一些用户示例的1141个markdown文件上运行。最大的
    是256k字节。

    |说明| Old SegmentedSequence | New Segmented Sequence | New Line Appendable |
    |:--------------------------- | -------------------- -:| -----------------------:| -------------------:|
    |墙上时钟总时间| 13.896秒| 9.672秒| 8.344秒|
    |解析时间| 2.402秒| 2.335秒| 2.297秒|
    |可附加的格式化程序| 0.603秒| 0.602秒| 0.831秒|
    |格式化程序序列生成器| 7.264秒| 3.109秒| 1.772秒|

    开销差异很大。总计是针对创建的所有分段序列
    在1141个文件的测试运行期间。解析器统计信息显示了解析和
    格式化。

    |说明|旧解析器|旧格式器|新解析器|新格式器|新行附加|
    |:------------------------------------------------ | -----------:| ---------------:| -----------:| ------ --------:| -------------------:|
    |个字节用于所有分段序列的字符| 917,016 | 6,029,774,526 | 917,016 | 6,029,774,526 | 37,663,196 |
    |个字节用于所有分段序列的开销| 1,845,048 | 12,060,276,408 | 93,628 | 342,351,155 | 8,204,796 |
    |开销%| 201.2%| 200.0%| 10.2%| 5.7%| 21.8%|

## Module Reorganization

*突破:从`flexmark-util`模块中将通用AST实用程序拆分为多个较小的独立实用程序
  模块。`com.vladsch.flexmark.util`不再包含任何文件,仅包含单独的实用程序
  `flexmark-utils`模块是所有实用程序模块的集合,类似于
  `flexmark-all`
  * `ast/`类到`flexmark-util-ast`
  * `builder/`类到`flexmark-util-builder`
  * `collection/`类到`flexmark-util-collection`
  * `data/`类到`flexmark-util-data`
  * `dependency/`类到`flexmark-util-dependency`
  * `format/`类到`flexmark-util-format`
  * `html/`类到`flexmark-util-html`
  * `mappers/`类到`flexmark-util-sequence`
  * `options/`类到`flexmark-util-options`
  * `sequence/`类到`flexmark-util-sequence`
  * `visitor/`类到`flexmark-util-visitor`
* Break:删除不推荐使用的属性,方法和类
*添加:`org.jetbrains:annotations:15.0`依赖项具有`@Nullable`/`@NotNull`批注
  为所有参数添加。在使用IntelliJ IDEA进行开发时,有助于这些
  注释,用于分析潜在问题,并简化了将库与
  科特林。
*中断:重构和清理测试,以消除重复的代码,并使测试更容易重用
  带有规格示例数据的案例。
*中断:将格式化程序测试移至`flexmark-core-test`模块以允许共享格式化程序库
  扩展中的类,而不会导致格式化程序模块中的依赖关系循环。
*中断:将格式化程序模块移入`flexmark`内核。这个模块几乎总是包含在内
  无论如何,因为大多数扩展都依赖于格式化程序进行自定义格式设置
  实现。将其作为核心的一部分可以完全依靠其功能
  模块。
*中断:将`com.vladsch.flexmark.spec`和`com.vladsch.flexmark.util`移入
  `flexmark-test-util`至`com.vladsch.flexmark.test.spec`和`com.vladsch.flexmark.test.util`
  分别尊重模块及其包之间的命名约定。
*中断:`NodeVisitor`实现细节已更改。如果您覆盖
  `NodeVisitor.visit(Node)`在以前的版本中现在是`final`以确保编译时间
  错误产生。您将需要更改您的实现。请参阅中的javadoc注释
  `NodeVisitor`类以获取说明。

  :information_source:`com.vladsch.flexmark.util.ast.Visitor`仅在实现时需要
  `NodeVisitor`和`VisitHandler`。如果`VisitHandler`的所有匿名实现都是
  转换为lambda,然后可以取消`Visitor`的导入。
  *修复:删除旧的访问者(如适配器),并基于未链接的通用类实施访问者
    标记AST节点。
  *删除旧的基类:
    * `com.vladsch.flexmark.util.ast.NodeAdaptedVisitor`参见javadoc中的类
    * `com.vladsch.flexmark.util.ast.NodeAdaptingVisitHandler`
    * `com.vladsch.flexmark.util.ast.NodeAdaptingVisitor`

## Migrating to 0.60

IntelliJ-IDEA迁移[migrate flexmark-java 0_50_x to 0_60_0.xml]可用于协助
从0.50.40迁移到0.60版本的库。它将迁移类名和包
仅更改。

对参数的更改和方法的更改必须手动解决。

### ## 78 ##

此类已重命名为`LineAppendable`。实现和子类类似地重命名
删除类名称中的`Formatting`。

现在,所有格式标记都以`F_`为前缀,如果存在,请选择给定的修改
附加文本。以前,`ALLOW_LEADING_WHITESPACE`和`ALLOW_LEADING_EOL`是倒置的
并设置它们会禁用文本修改。

* `ALLOW_LEADING_WHITESPACE`现在为`F_TRIM_LEADING_WHITESPACE`,并且**含义相反**。
* `ALLOW_LEADING_EOL`现在为`F_TRIM_LEADING_EOL`,并且**含义相反**。
* `CONVERT_TABS`现在是`F_CONVERT_TABS`
* `COLLAPSE_WHITESPACE`现在是`F_COLLAPSE_WHITESPACE`
* `TRIM_TRAILING_WHITESPACE`现在是`F_TRIM_TRAILING_WHITESPACE`
* `PASS_THROUGH`现在是`F_PASS_THROUGH`
* `TRIM_LEADING_WHITESPACE`现在为`F_TRIM_LEADING_WHITESPACE`
* `PREFIX_PRE_FORMATTED`现在是`F_PREFIX_PRE_FORMATTED`
* `FORMAT_ALL`现在为`F_FORMAT_ALL`

### ## 102 ##

对该接口和实现类进行了重构,并对其进行了重新设计以提高效率
与`SequenceBuilder`一起使用。

* `CharPredicate`类现在用于提供字符集,而不是`CharSequence`
  提供一致且有效的字符测试。具有`CharSequence`参数的方法
  用于选择字符集,现在为`CharPredicate`。

  更改方法调用的最简单方法是使用`CharPredicate.anyOf(CharSequence)`
  将字符序列转换为谓词。

*重命名了某些方法以更好地反映其操作。在这些情况下,旧名称
  不赞成使用方法,默认实现将调用新方法。

### ## 109 ##

此类已重命名为`SegmentedSequenceFull`,其中包含旧的,效率低下的
实施。不建议使用旧类,因为它效率低下并且
在某些情况下,错误的实现。

新的`SegmentedSequence`是抽象类,具体实现如下
`SegmentedSequenceFull`和`SegmentedSequenceTree`。后者是有效的实现
使用二进制搜索树。

创建`SegmentedSequence`实例的正确方法是使用的实例
`SequenceBuilder`构建序列,然后使用`SequenceBuilder.toSequence()`返回
`SegmentedSequenceTree`的实例,如果结果需要分段序列或子序列
基础`BasedSequence`的值(如果是单个段)。

[migrate flexmark-java 0_50_x to 0_60_0.xml]: https://github.com/vsch/flexmark-java/blob/master/assets/migrations/migrate%20flexmark-java%200_50_x%20to%200_60_0.xml