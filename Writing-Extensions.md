### 目录
- [解析过程](#parsing-process)
- [源跟踪](#source-tracking)
- [配置选项](#configuring-options)
- [从commonmark-java对扩展API的更改](#changes-to-extension-api-from-commonmark-java)
    - [解析器](#parser)
    - [渲染器](#renderer)
    - [格式化](#formatter)


扩展需要扩展解析器或HTML渲染器,或两者都扩展。要使用扩展名,
可以使用扩展列表配置构建器对象。由于扩展名是可选的,
它们位于单独的工件中,因此也需要添加其他依赖项。

创建扩展的最佳方法是从现有扩展的副本开始,然后修改
源和测试以适应新的扩展。

:information_source:感谢[Alex Karezin](mailto:javadiagrams@gmail.com)的设置
[Flexmark体系结构和依赖关系图](https://sourcespy.com/github/flexmark/)和
<https://sourcespy.com>。您现在可以大致了解模块依赖性
向下钻取从存储库源更新的包和类。:竖起大拇指:

### 解析过程

解析按照不同的步骤进行,并且可以在每个步骤中添加自定义处理。

![Flexmark解析](https://github.com/vsch/flexmark-java/raw/master/assets/images/flexmark-parsing.png)

1.源代码中的文本分为"块"节点。块解析器决定当前是否
   行开始于被他们识别为创建的一个块。某些区块未声明的任何行
   解析器由核心`ParagraphParser`声明。

   块解析器可以声明具有以下功能的段落解析器当前累积的行:
   将其替换为当前活动的解析器。

2.处理段落块以删除不应该被处理的前导**行
   文本。例如,处理链接引用并将其从段落中删除。**仅满
   行**应在此步骤中处理。文本的部分删除应在下一个完成
   步。

   内联解析器实例目前可用于阻止,以允许他们使用API
   帮助处理文本。需要对其内容进行内联处理的处理器应
   在核心引用链接段落处理器之后运行,否则某些链接引用将不会
   被认可,因为它们尚未定义。定制处理器工厂应退货
   `getAfterDependents()`中的`ReferencePreProcessorFactory.class`。同样,如果一个段落
   处理器需要在另一个预处理器之前运行,然后它应该返回该处理器的
   `getBeforeDependents()`中的工厂类。

   任何循环依赖性将导致在构建器期间抛出`IllegalStateException()`
   准备阶段。

   段落预处理分为段落预处理器的各个阶段,以允许
   依赖关系。

   段落预处理器还可以报告它们`affectsGlobalStat()`,这意味着
   一些文档属性受其处理结果的影响。例如,
   `ReferencePreProcessorFactory`这样做是因为引用链接定义会影响
   `REFERENCES`节点存储库。

   由于全局处理器将在文档中的所有段落之前运行,
   允许从属程序处理任何段落,全局处理器将是唯一的处理器
   在各自的预处理阶段。

   同一阶段中的非全局处理器将在每个段落块上顺序运行
   直到不再对该段落进行任何更改。这意味着非全局处理器
   同一阶段允许混合内容,而全局阶段只能运行一次
   对于每个段落。依赖一个或多个全局处理器的非全局处理器将是
   在所有全局依赖项完成后,在第一个可用阶段运行
   处理。

   阶段中预处理器的顺序将基于处理器之间的依赖关系
   并没有对其相应扩展名的注册顺序加以限制的地方。

   :warning:最好通过使用块解析器来实现所需的自定义
   比段落预处理器 仅当无法正确解释时才使用后者
   而不使用内联解析器API。在块解析期间使用内联解析器API是一种
   严重的性能问题。

3.块预处理仅适用于自定义处理器。此时,块节点可以是
   添加,替换或删除。任何其他节点也可以添加到AST,但是没有内联
   此时已创建块。

   节点的创建和删除应通过其实例传达给`ParserState`实例
   `BlockParserTracker`和`BlockTracker`接口的实现。这是必要的
   允许更新内部解析器优化结构,以便进一步阻止
   预处理将正确进行。

4.在内联处理期间,每个块都有机会处理包含的内联元素
   在其一个或多个文本节点中。此步骤有两种类型的自定义项:链接
   ref处理和定界符处理。分隔符是具有开头和
   结束字符以确定其跨度。分隔符可以嵌套,并具有最小值和
   最大嵌套水平。

   链接引用处理器负责处理被识别为
   可能的链接参考,即 它们以`![`或`[`分隔,并以`]`终止。链接参考
   处理器确定是否可以嵌套方括号,是否应将`!`处理为
   文本或节点的一部分,并确定他们是否接受潜在的链接引用文本为
   他们的节点。给出全文`![]`或`[]`进行批准,以最大程度地灵活
   处理包含空白。

   脚注`[^footnote]`和Wiki链接`[[wiki link]]`扩展利用了ths扩展
   机制。

5.后处理步骤用于最终AST修改。后处理器有两种:
   节点和文档后处理器。尽管`PostProcessor`接口都用于
   类型,后处理器只能是一个。

   ##### 节点后处理器

   节点后处理器使用祖先指定要后处理的节点的类
   排除标准。处理器的`process(NodeTracker, Node)`函数将被调用
   对于每个与给定条件匹配的AST节点。

   对AST的任何修改都必须传达给`NodeTracker`实例,该实例是
   负责更新用于优化节点选择的内部结构
   处理器。

   具体来说,在AST层次结构中添加或移动的每个新节点都需要具有其
   已更新祖先列表,以进行进一步的节点后处理。这些通知功能应
   在完成特定更改的层次结构更改以消除不必要的操作后调用
   中间AST更改的更新。

   包含新的子节点或已从其先前父节点移出的子节点的节点
   需要通过`nodeAddedWithChildren(Node)`进行通知,而不是使用
   每个单独节点的`nodeAdded(Node)`回调。同样,更大的深度变化应该
   通过`nodeAddedWithDescendants(Node)`通知进行通信。

   删除所有节点后,应通过`nodeRemoved()`函数进行通信
   需要移动到其他位置的子节点已被删除。

   :warning:所有节点删除功能都将对节点执行节点删除,并对其所有节点_ **
   后代** _,因为未链接节点的所有子节点都从AST中删除。

   ##### 文件后处理器

   使用`processDocument(Document)`成员函数调用文档后处理器
   返回的文档将用作进一步处理的文档。

   文档处理器负责通过递归遍历查找感兴趣的节点
   AST。因此,仅在处理时才使用文档后处理器
   无法在单个节点上完成。

   虽然,遍历AST的扩展要比创建,维护和
   访问解析器中的优化结构,只需对一个扩展进行两个
   大文档是一个慢得多的过程。

   对于基于节点排除节点的扩展,这种性能提升尤其如此
   AST中的祖先类型。对于节点后处理器,此层次结构是在
   一次遍历AST即可构建所有节点跟踪结构。如果扩展
   通过回顾节点的`getParent()`函数来确定继承
   在大型文档上效率很低。

6. HTML渲染步骤。渲染最终的AST。扩展程序为其提供默认渲染器
   自定义节点。可以通过替换默认渲染器或自定义任何节点的渲染。
   通过属性提供程序,它将覆盖HTML元素的默认属性
   渲染器。LinkResolvers负责将链接网址文本从
   呈现网址的markdown元素。

7.包含文件支持允许扩展从以下位置复制其自定义引用定义元素
   包含文档到包含文档,因此任何包含的自定义元素都需要
   这些引用的定义将在渲染之前正确解析。这是个
   可选步骤,由应用程序在解析文档和
   渲染之前。参见[Include Markdown and HTML File Content]

### 源跟踪

若要跟踪AST中的源位置,请使用`BasedSequence`类执行所有解析,
扩展`CharSequence`并使用以下命令包装文档的原始源字符序列
起始和结束偏移量以表示其自身的内容。`subSequence()`返回另一个
`BasedSequence`具有原始基本序列和新的开始/结束偏移量的实例。

这样,可以使用
`BasedSequence.getStartOffset()`和`BasedSequence.getEndOffset()`。同时解析是
与使用`CharSequence`一样复杂。AST中存储的任何字符串都必须是
`subSequence()`原始来源。由于AST代表
资源。

美中不足的是,从AST解析未转义的文本会涉及更多的工作,因为
它是转义的原始文件,必须添加到AST中。为此,
`Escaping`实用程序类添加了`BasedSequence`和`ReplacedTextMapper`
类。返回的结果是修改后的序列,其内容可以映射到原始序列
源代码使用`ReplacedTextMapper`对象的方法。允许解析已压缩的文本
能够提取未按摩的对应对象以放置在AST中。见实施
`flexmark-ext-autolink` [AutolinkNodePostProcessor]举例说明如何实现
在有效的扩展中。

同样,在使用正则表达式匹配时,您不能简单地采用`group()`返回的字符串,而是
必须使用组的开始/结束偏移量从输入中提取`subSequence`。最好的
方法是从用于创建的原始序列中提取`subSequence()`
`matcher()`,这消除了偏移量计算中的错误。这方面的例子很多
核心解析器实现。

开销是在AST中拥有完整的源代码参考且易于实现所需的一小笔费用
解析而不必携带单独的状态来表示源位置或使用专用
任务的语法工具。

领先的制表符扩展和前缀删除功能使核心中的源跟踪变得复杂
解析的行,然后将这些部分结果串联起来进行内联解析,这也是
必须跟踪原始源位置。此问题已通过其他`BasedSequence`解决
实现类:`PrefixedSubSequence`用于部分使用的选项卡,`SegmentedSequence`
用于连接的序列。结果几乎是源位置的透明传播
在整个解析过程中。

:information_source:`SegmentedSequence`的实现已在版本`0.60.0`中更改为
将二进制搜索树用于包含字符索引和每个线程缓存的段,以及
优化,消除了在几乎所有连续的`charAt`中搜索片段的需要
访问,包括从序列开头或结尾开始的正向和反向序列扫描。的
减少`charAt`的小代价可以减少分段序列的开销
旧实施中为200%,新实施中平均为5.7%。另外,
当这些段代表完整的行时(如它们在解析器中所做的那样),则不会感觉到或
`charAt`访问的可衡量的罚款。在包含1000多个文件的测试中,总解析时间为
所有文件为2.8秒,因此使用二叉树分段序列的代价小于
50ms,并且在进行此类简单时序测量的误差范围内。

`SegmentedSequence`的构造也更改为使用`BaseSequenceBuilder`实例,
消除了将片段按正确顺序放置和使用的麻烦
`PrefixedSubSequence`用于所有非基础文本。只要从任何一个构建器实例
`BasedSequence.getBuilder()`并调用其方法以附加`char`或`CharSequence`。它会
生成优化的段列表,转换任何重叠或无序的基本序列
范围到插入的文本。完成后,调用`BasedSequenceBuilder.toSequence()`以获取
基于优化的序列,等效于所有附加文本。

该生成器的效率足以用作`Appendable`的文本累积
`Formatter`和`HtmlRenderer`的性能损失因子小于8。听起来像
高昂的价格,但好处是能够获取格式化或呈现的HTML结果并提取
来自原始文件的任何文本的源位置信息。如果你需要这个
7.4倍的信息是可以接受的,尤其是在生成时不需要额外的信息
除了将`BasedSequenceBuilder`用作附件之外,还需要付出更多努力。

### 配置选项

添加了通用选项API,可以轻松配置解析器,渲染器和
扩展名。它由各种组件定义的`DataKey<T>`实例组成。每个数据键
定义其值的类型,默认值和可选的。


可通过`DataHolder`和`MutableDataHolder`接口访问这些值,其中前者
是一个只读容器。由于数据密钥为那里的数据提供了唯一的标识符
对于选项没有冲突。

`Parser.EXTENSIONS`选项包含要用于`Parser`和`HtmlWriter`的扩展名列表。
这允许使用一组优化器配置解析器和渲染器。

要配置解析器或渲染器,请将数据保持器传递给`builder()`方法。

```java
public class SomeClass {
    static final DataHolder OPTIONS = new MutableDataSet()
            .set(Parser.REFERENCES_KEEP, KeepType.LAST)
            .set(HtmlRenderer.INDENT_SIZE, 2)
            .set(HtmlRenderer.PERCENT_ENCODE_URLS, true)
            .set(Parser.EXTENSIONS, Arrays.asList(TablesExtension.create()))
            .toImmutable();

    static final Parser PARSER = Parser.builder(OPTIONS).build();
    static final HtmlRenderer RENDERER = HtmlRenderer.builder(OPTIONS).build();
}
```

在上面的代码示例中,`ReferenceRepository.KEEP`定义了引用行为
源中定义了重复引用。在这种情况下,它被配置为保留最后一个
值,而默认行为是保留第一个值。

`HtmlRenderer.INDENT_SIZE`和`HtmlRenderer.PERCENT_ENCODE_URLS`定义了用于
渲染。同样,可以同时添加其他扩展选项。未设置任何选项
将默认使用其数据键定义的默认值。

所有markdown元素引用类型都应使用`NodeRepository<T>`的子类存储为
参考,缩写和脚注都是这种情况。这提供了一致的机制
覆盖这些引用的默认行为(从保持优先到保持重复)
持续。

按照惯例,数据键是在扩展类中定义的,对于核心,则是在扩展类中定义的。
`Parser`或`HtmlRenderer`。

:warning:在版本`0.60`之前,将`DataHolder`参数传递给
创建只读默认值时,`DataValueFactory::create()`方法将为`null`
密钥使用的实例。类的构造函数应该能够处理这种情况
无缝地。为了方便实现此类,请使用
`DataKey.getFrom(DataHolder)`方法而不是`DataHolder.get(DataKey)`方法来访问
利息价值。如果数据持有人,前者将提供密钥的默认值
参数为`null`,后者将生成运行时`java.lang.ExceptionInInitializerError`
错误。

有关`Parser`的Option数据键,请参见[[Extensions: Parser|Extensions#parser]]

有关`HtmlRenderer`的Option数据键,请参见[[Extensions: Renderer|Extensions#renderer]]

## Changes to Extension API from commonmark-java

1.添加了`PhasedNodeRenderer`和`ParagraphPreProcessor`接口并关联了
   `Builder`扩展解析器的方法。

   `PhasedNodeRenderer`允许扩展为HTML的各个部分生成HTML
   文献。这些阶段按在文档渲染期间发生的顺序列出:

   * `HEAD_TOP`
   * `HEAD`
   * `HEAD_CSS`
   * `HEAD_SCRIPTS`
   * `HEAD_BOTTOM`
   * `BODY_TOP`
   * `BODY`
   * `BODY_BOTTOM`
   * `BODY_LOAD_SCRIPTS`
   * `BODY_SCRIPTS`

   `BODY`阶段是使用`NodeRenderer::render(Node ast)`的标准HTML生成阶段
   方法。文档中的每个节点都会调用它。

   其他阶段仅在`Document`根节点上调用,并且仅用于自定义渲染器
   实现`PhasedNodeRenderer`接口。`PhasedNodeRenderer::render(Node ast,
   RenderingPhase phase)`。

   该分机可以在任何时候呼叫`context.render(ast)`和`context.renderChildren(ast)`
   渲染阶段。函数将像在`BODY`渲染期间一样处理节点
   相。`FootnoteExtension`使用`BODY_BOTTOM`阶段渲染脚注
   在页面内引用。同样,目录扩展可以使用`BODY_TOP`
   阶段将目录插入文档顶部。

   `HEAD...`阶段不被任何扩展名使用,但可以用于生成完整的HTML
   文档以及样式表和脚本。

2. `CustomBlockParserFactory`,`BlockParserFactory`和`BlockParser`用于扩展
   解析将文档划分为多个块的块,然后对其进行解析
   用于内联和后期处理。

3. `ParagraphPreProcessor`和`ParagraphPreProcessorFactory`接口允许自定义
   在解析器关闭时对块元素进行预处理。这是通过
   `ParagraphParser`从段落中提取主要参考定义。特别
   `ParagraphParser`块的处理已从解析器中删除,而改为通用
   添加了机制,以允许任何`BlockParser`执行类似的功能并允许
   添加自定义预处理器以处理内置引用以外的元素
   定义。

4. `BlockPreProcessor`和`BlockPreProcessorFactory`接口允许对块进行预处理
   在`ParagraphPreProcessor`实例运行之后但在执行内联解析之前。
   如果您想根据上下文将标准节点替换为自定义节点,则非常有用
   子元素,但不包含内联元素信息。当前未使用此机制。也许
   如果事实证明没有用,请在将来删除。

5.文档级别,添加了可扩展属性以允许扩展具有文档级别
   渲染期间可用的属性。解析时可以从
   `ParserState::getProperties()`,`state`参数以及后期处理和渲染期间
   通过任何`Node`的`getDocument()`方法可到达的`Document`节点。

   `DocumentParser`和`Document`属性还将包含在上传递或定义的选项
   `Parser.builder()`对象,以及在解析过程中添加的任何对象
   文献。

   :warning:`HtmlRenderer`选项仅在渲染上下文对象上可用。
   NodeRenderer扩展应使用以下命令检查其选项:
   `NodeRendererContext.getOptions()`不是`getDocument()`方法。如果`HtmlRenderer`是
   使用未传递给`Parser.Builder`的选项进行自定义,则这些选项将不会
   通过文档属性可用。节点渲染器上下文选项将包含
   为`HtmlRenderer.builder()`定义的所有自定义选项以及所有文档属性,其中
   将包含传递给`Parser.builder()`的所有选项,以及解析期间定义的所有选项
   处理。如果在渲染器中自定义或定义了选项,则其值来自文档
   将无法访问。对于这些,您将需要使用可通过以下途径获得的文档:
   呈现上下文`getDocument()`方法。

   `DataKey`定义属性,其类型和默认值实例化。`DataHolder`和
   `MutableDataHolder`接口分别用于访问或设置属性。

   `NodeRepository`是用于为节点创建存储库的抽象类:引用,
   脚注和缩写。

6.由于AST现在代表的是文档的来源而不是要呈现的HTML,因此文本
   AST中存储的内容必须与源中的内容相同。这意味着所有逃跑和
   引用的解析必须在渲染阶段完成。例如脚注
   对未定义脚注的引用将呈现为好像是一个Text节点,包括任何
   强调嵌入在脚注ID中。如果定义了脚注参考,它将呈现
   两者都符合预期。

   处理源中使用的不同行尾。现在也必须在
   渲染阶段。这意味着包含行尾的文本必须在
   由于在解析过程中不再对其进行标准化,因此将其呈现。

   由于必要的成员方法是
   添加到`BasedSequence`类中,该类用于表示AST中的所有文本。

7.节点未定义`accept(Visitor)`方法。相反,访问者处理是通过委派的
    `VisitHandler`实例和`NodeVisitor`派生类。

    [[Usage: Use A Visitor To Process Parsed Nodes|Usage#use-a-visitor-to-process-parsed-nodes]]

### 解析器

添加了统一选项处理,还可用于有选择地禁用加载
核心处理器可实现更大的定制化。

`Parser.builder()`现在实现了`MutableDataHolder`,因此您可以使用`get`/`set`进行自定义
属性。

`Parser.builder()`现在实现了`MutableDataHolder`,因此您可以使用`get`/`set`自定义p

解析器的新扩展点:

* `ParagraphBlock`使用`ParagraphPreProcessor`从中提取引用定义
  该段的开头,但可用于同一目的的任何其他块。任何
  自定义块预处理器将按照其注册顺序被首先调用。多
  由于删除某些文本可能会为其他预处理程序公开文本,因此可能会导致调用。块
  调用预处理器,直到不对该块进行任何更改。

* `InlineParserFactory`用于覆盖默认的内联解析器。仅一个自定义内联
  可以设置解析器工厂。如果未设置,则将使用默认值。

* `LinkRefProcessor`用于创建从链接引用句法派生的自定义元素:
  `[]`或`![]`。这对于元素中的嵌套`[]`将正常工作,并允许处理
  如果自定义元素未使用前导`!`作为纯文本。脚注(`[^footnote
  ref]`)和Wiki链接(`[[]]`或`[[text|link]]`)是此类元素的示例。

#### AST中的空白行

在AST中的`BlankLine`节点上出现空行的选项要求任何自定义
阻止要包含空行的节点以实现`BlankLineContainer`接口。
目前没有任何方法,仅用于识别哪些块可以包含空白
线。

从`ListBlock`和`ListItem`继承自定义节点的类将自动继承
空行处理,但当将块末尾的空行移动到父行时
块已关闭。这可以通过在上调用`Node.moveTrailingBlankLines()`方法来完成
节点。

### 渲染器

添加了统一选项处理,保留了现有配置选项,但现在它们修改了
对应的统一属性。

渲染器`Builder()`现在具有`indentSize(int)`方法来设置缩进的大小
分层标签。与在选项中设置`HtmlRenderer.INDENT_SIZE`数据键相同。

现在,所有`HtmlWriter`方法都返回`this`,因此可以使用方法链接。另外,
使用`Runnable`的`tag()`和`indentedTag()`方法将自动关闭标记,并且
`run()`方法执行后取消缩进。这样可以更轻松地查看HTML层次结构
渲染的输出。

而不是单独写出所有的开始,结束标记和属性:

```java
class CustomNodeRenderer implements NodeRenderer {
    public void render(BlockQuote node, NodeRendererContext context, HtmlWriter html) {
        html.line();
        html.tag("blockquote", getAttrs(node));
        html.line();
        context.renderChildren(node);
        html.line();
        html.tag("/blockquote");
        html.line();
    }
}
```

您可以合并它们并使用lambda来渲染子代,这样缩进和关闭
标签会自动处理:

```java
class CustomNodeRenderer implements NodeRenderer {
    public void render(BlockQuote node, NodeRendererContext context, HtmlWriter html) {
        html.withAttr().tagIndent("blockquote", (Runnable)()-> context.renderChildren(node));
    }
}
```

对于使用增加堆栈的代价,增加的好处是:

*缩进子标签
*属性更易于处理,因为它们只需要使用`.attr()`设置属性
  并在`tag...()`方法之前使用`.withAttr()`调用
*标签会自动关闭使用显式属性参数的先前行为是
  仍然保存。

缩进对于测试很有用,因为它更容易在视觉上进行验证和关联:

```markdown
> - item 1
> - item 2
>     1. item 1
>     2. item 2
```

呈现的html:

```html
<blockquote>
  <ul>
    <li>item 1</li>
    <li>item 2
      <ol>
        <li>item 1</li>
        <li>item 2</li>
      </ol>
    </li>
  </ul>
</blockquote>
```

比这个:

```html
<blockquote>
<ul>
<li>item 1</li>
<li>item 2
<ol>
<li>item 1</li>
<li>item 2</li>
</ol>
</li>
</ul>
</blockquote>
```

您可以从当前渲染上下文中获取渲染器子上下文,并设置其他html
与其他作家不同的不呈现链接设置。使用`TextCollectingAppendable`并通过
当您需要从节点捕获html时,请使用`NodeRendererContext.getSubContext()`方法。
TocExtension使用它来获取标题文本html,但没有任何可能嵌入的链接
在标题中。

### 格式化程序

`flexmark-formatter`模块使用各种格式设置选项将AST呈现为markdown
清理并使源一致。它还带有一个API,允许扩展
提供并处理自定义节点的格式设置选项。

块节点的默认渲染将使用任何前缀/后缀字符来渲染子级
为节点定义。非块节点将仅按原样呈现其字符内容。

要提供自定义格式,请实施`FormatterExtension`并注册一个
`NodeFormatterFactory`用于节点格式化程序。行为和API类似于
`NodeRenderer`除了Markdown是可以预期的,并且`FormattingPhase`适用于Markdown
文档而不是HTML。

示例代码的最佳来源是实现自定义节点格式和
格式化程序模块本身:

* `flexmark-ext-abbreviation`
* `flexmark-ext-definition`
* `flexmark-ext-footnotes`
* `flexmark-ext-jekyll-front-matter`
* `flexmark-ext-tables`
* `flexmark-formatter`

[AutolinkNodePostProcessor]: https://github.com/vsch/flexmark-java/blob/master/flexmark-ext-autolink/src/main/java/com/vladsch/flexmark/ext/autolink/internal/AutolinkNodePostProcessor.java
[Include Markdown and HTML File Content]: Usage#include-markdown-and-html-file-content
[autolink-java]: https://github.com/robinst/autolink-java
[CommonMark]: https://commonmark.org
[commonmark.js]: https://github.com/jgm/commonmark.js
[Emoji-Cheat-Sheet.com]: https://www.webfx.com/tools/emoji-cheat-sheet
[gfm-tables]: https://help.github.com/articles/organizing-information-with-tables/
[Markdown]: https://daringfireball.net/projects/markdown/
[Semantic Versioning]: https://semver.org