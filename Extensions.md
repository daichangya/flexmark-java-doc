### 目录
- [配置选项](#configuring-options)
- [核心](#core)
  - [解析器](#解析器)
  - [渲染器](#渲染器)
  - [格式化](#formatter)
- [PDF输出模块](#pdf-output-module)
- [可用扩展名](#available-extensions)
- [缩写](#abbreviation)
- [警告](#admonition)
- [AnchorLink](#anchorlink)
- [Aside](#aside)
- [属性](#attributes)
- [自动连结](#autolink)
- [定义列表](#definition-lists)
- [Docx转换器](#docx-converter)
- [表情符号](#emoji)
- [枚举参考](#enumerated-reference)
- [脚注](#footnotes)
- [Gfm问题](#gfm-issues)
- [Gfm-删除线/下标](#gfm-删除线下标)
- [Gfm-TaskList](#gfm-tasklist)
- [Gfm用户](#gfm用户)
- [GitLab风味Markdown](#gitlab-flavored-markdown)
- [Html To Markdown](#html-to-markdown)
- [Ins](#ins)
- [Jekyll标签](#jekyll-tags)
- [Jira-Converter](#jira-converter)
- [宏](#macros)
- [媒体标签](#media-tags)
- [规格示例](#spec-example)
- [上标](#上标)
- [表格](#tables)
- [目录](#table-of-contents)
- [印刷](#typographic)
- [WikiLinks](#wikilinks)
- [XWiki宏扩展](#xwiki-macro-extension)
- [YAML前线](#yaml-front-matter)
- [YouTrack-Converter](#youtrack-converter)
- [YouTube嵌入式链接变压器](#youtube-embedded-link-transformer)


扩展需要扩展解析器,HTML渲染器,格式化程序或这些的任何组合。至
使用扩展名,可以为构建器对象配置扩展名列表。因为
扩展是可选的,它们位于单独的工件中,需要附加的依赖关系
被添加到项目中。

让我们看看如何启用GitHub Flavored Markdown或MultiMarkdown中的表
你喜欢 首先,将所有模块添加为依赖项(有关具体信息,请参见[Maven Central]
模块):

```xml
<dependency>
    <groupId>com.vladsch.flexmark</groupId>
    <artifactId>flexmark-all</artifactId>
    <version>0.60.0</version>
</dependency>
```

在构建器上配置扩展名:

```java
import com.vladsch.flexmark.ext.tables.TablesExtension;
import com.vladsch.flexmark.ext.gfm.strikethrough.StrikethroughExtension;

class SomeClass {
    static final DataHolder OPTIONS = new MutableDataSet()
                .set(Parser.EXTENSIONS, Arrays.asList(TablesExtension.create(), StrikethroughExtension.create()))
                .toImmutable();

    Parser parser = Parser.builder(options).build();
    HtmlRenderer renderer = HtmlRenderer.builder(options).build();
}
```

## Configuring Options

通用选项API允许轻松配置解析器,渲染器和扩展。它
由各种组件定义的`DataKey<T>`实例组成。每个数据键定义类型
其值和默认值。

可通过`DataHolder`和`MutableDataHolder`接口访问这些值,其中前者
是一个只读容器。由于数据密钥为那里的数据提供了唯一的标识符
对于选项没有冲突。

要配置解析器或渲染器,请使用以下命令将数据保存器传递给`builder()`方法:
配置所需的选项,包括扩展名。

```java
import com.vladsch.flexmark.html.HtmlRenderer;
import com.vladsch.flexmark.parser.Parser;

public class SomeClass {
    static final DataHolder OPTIONS = new MutableDataSet()
            .set(Parser.REFERENCES_KEEP, KeepType.LAST)
            .set(HtmlRenderer.INDENT_SIZE, 2)
            .set(HtmlRenderer.PERCENT_ENCODE_URLS, true)

            //for full GFM table compatibility add the following table extension options:
            .set(TablesExtension.COLUMN_SPANS, false)
            .set(TablesExtension.APPEND_MISSING_COLUMNS, true)
            .set(TablesExtension.DISCARD_EXTRA_COLUMNS, true)
            .set(TablesExtension.HEADER_SEPARATOR_COLUMN_MATCH, true)
            .set(Parser.EXTENSIONS, Arrays.asList(TablesExtension.create()))
            .toImmutable();

    static final Parser PARSER = Parser.builder(OPTIONS).build();
    static final HtmlRenderer RENDERER = HtmlRenderer.builder(OPTIONS).build();
}
```

在上面的代码示例中,`Parser.REFERENCES_KEEP`定义了引用行为
源中定义了重复引用。在这种情况下,它被配置为保留最后一个
值,而默认行为是保留第一个值。

`HtmlRenderer.INDENT_SIZE`和`HtmlRenderer.PERCENT_ENCODE_URLS`定义了用于
渲染。同样,可以同时添加扩展选项。任何未设置的选项,将
默认为其各自的默认值(由其数据键定义)。

所有markdown元素引用类型都应使用`NodeRepository<T>`的子类存储为
参考,缩写和脚注都是这种情况。这提供了一致的机制
覆盖这些引用的默认行为(从保持优先到保持重复)
持续。

按照惯例,数据键是在扩展类中定义的,对于核心,则是在扩展类中定义的。
`Parser`或`HtmlRenderer`。

数据密钥在其各自的扩展类以及`Parser`和
`HtmlRenderer`。

## Core

Core为CommonMark markdown实现了解析器,HTML渲染器和格式化程序功能
元素。

### 解析器

添加了统一选项处理,还用于有选择地禁用内核加载
解析器和处理器。

`Parser.builder()`现在实现了`MutableDataHolder`,因此您可以使用`get`/`set`进行自定义
直接在其上设置属性,或将其传递给具有预定义选项的DataHolder。

在`Parser`类中定义:



- `Parser.ASTERISK_DELIMITER_PROCESSOR`:默认`true`启用星号定界符内联处理。
- `Parser.BLANK_LINES_IN_AST`默认值`false`,设置为`true`以在AST中包含空白行节点。
- `Parser.BLOCK_QUOTE_ALLOW_LEADING_SPACE`:默认`true`,如果允许`>`之前的`true`前导空格
- `Parser.BLOCK_QUOTE_EXTEND_TO_BLANK_LINE`:当`true`块引号扩展到下一个空白行时,默认为`false`。比commonmark严格标准支持更多的常规块引用解析
- `Parser.BLOCK_QUOTE_IGNORE_BLANK_LINE`:默认值`false`,当`true`块引号在块引号之间包含空白行并将其视为空白行也由块引号引起来时
- `Parser.BLOCK_QUOTE_INTERRUPTS_ITEM_PARAGRAPH`:默认`true`,当`true`块引号可以中断列表项文本时,否则需要空行才能包含在列表项中
- `Parser.BLOCK_QUOTE_INTERRUPTS_PARAGRAPH`:默认`true`,当`true`块引用可以中断一段时,否则需要在空白行之前
- `Parser.BLOCK_QUOTE_PARSER`:启用块引用的`true`解析时,默认为`true`
- `Parser.BLOCK_QUOTE_WITH_LEAD_SPACES_INTERRUPTS_ITEM_PARAGRAPH`:默认为`true`,当`true`带有引号的块引号可以中断列表项文本时,否则需要在空白行之前插入空格
- `Parser.CODE_BLOCK_INDENT`,默认为`Parser.LISTS_ITEM_INDENT`,该设置确定有多少前导空格会将以下文本视为缩进块。

  :information_source:在解析器中,使用`state.getParsing().CODE_BLOCK_INDENT`确保所有解析器具有相同的设置。`Parsing`从创建时的选项复制设置,因此在解析阶段开始后更改此选项将无效。
- `Parser.CODE_SOFT_LINE_BREAKS`:默认`false`,设置为`true`以在代码块中包含软断行节点
- `Parser.CODE_STYLE_HTML_CLOSE`:默认`(String)null`覆盖HTML以用于换行样式,如果为null,则不覆盖
- `Parser.CODE_STYLE_HTML_OPEN`:默认`(String)null`覆盖HTML以用于换行样式,如果为null,则不覆盖
- `Parser.EMPHASIS_STYLE_HTML_CLOSE`:默认`(String)null`覆盖用于包装样式的HTML,如果为null,则不覆盖
- `Parser.EMPHASIS_STYLE_HTML_OPEN`:默认`(String)null`覆盖HTML以用于包装样式,如果为null,则不覆盖
- `Parser.EXTENSIONS`:用于构建器的扩展名的默认空白列表。可以使用此选项,而不是将扩展传递给解析器构建器和渲染器构建器。
- `Parser.FENCED_CODE_BLOCK_PARSER`:默认`true`启用解析的代码块解析
- `Parser.FENCED_CODE_CONTENT_BLOCK`:默认`false`将`CodeBlock`作为子代添加,以包含受防护块的代码。否则,代码包含在`Text`节点中。
- `Parser.HARD_LINE_BREAK_LIMIT`:默认值`false`如果设置为true,则仅在HardLineBreak节点中处理硬断行的最后2个空格。否则,将使用所有空格。
- `Parser.HEADING_CAN_INTERRUPT_ITEM_PARAGRAPH`:默认`true`允许标题中断列表项的段落
- `Parser.HEADING_NO_ATX_SPACE`:默认值`false`允许标头在`#`和标头文本之间没有空格
- `Parser.HEADING_NO_EMPTY_HEADING_WITHOUT_SPACE`默认值`false`,设置为`true`,以识别不带`#`后面没有空格的空标题
- `Parser.HEADING_NO_LEAD_SPACE`:默认的`false`不允许在`#`之前的atx标头和文本使用非缩进空格,对于setext则不允许使用`- `/`=`标记(如果为true, pegdown和GFM)(如果使用错误的通用标记规则)。
- `Parser.HEADING_PARSER`:默认`true`启用标题解析
- `Parser.HEADING_SETEXT_MARKER_LENGTH`:默认值`1`设置在setext标题文本下需要识别的最小`- `或`=`数量。
- `Parser.HTML_BLOCK_COMMENT_ONLY_FULL_LINE`:默认为`false`,当`true`允许嵌入的HTML注释中断段落时。
- `Parser.HTML_BLOCK_DEEP_PARSE_BLANK_LINE_INTERRUPTS_PARTIAL_TAG`默认值`true`,当真正的空白行中断部分打开的标签即。`<TAG`没有对应的`>`,并且在`>`之前有空白行
- `Parser.HTML_BLOCK_DEEP_PARSE_BLANK_LINE_INTERRUPTS`默认值`true`,当`true`空白行不在原始代码中时中断HTML块,否则仅在关闭时中断
- `Parser.HTML_BLOCK_DEEP_PARSE_FIRST_OPEN_TAG_ON_ONE_LINE`默认值`false`,如果`true`不分析打开的标记,除非它们包含在一行中。
- `Parser.HTML_BLOCK_DEEP_PARSE_INDENTED_CODE_INTERRUPTS`默认值`false`,当`true`缩进的代码可以中断HTML块而没有前面的空白行时。
- `Parser.HTML_BLOCK_DEEP_PARSE_MARKDOWN_INTERRUPTS_CLOSED`默认值`false`,当`true`其他markdown元素可以中断封闭的HTML块而无需插入空白行时
- `Parser.HTML_BLOCK_DEEP_PARSE_NON_BLOCK`默认`true`,解析HTML块内的非块标签
- `Parser.HTML_BLOCK_DEEP_PARSER`默认`false`-启用深度HTML块解析
- `Parser.HTML_BLOCK_PARSER`:默认`true`启用html块解析
- `Parser.HTML_BLOCK_START_ONLY_ON_BLOCK_TAGS`,深层html解析器的默认值为`true`和常规解析器`false`,但是您可以设置所需的特定值并覆盖默认值。当`true`不会在非阻止标签上启动HTML块时,`false`则是任何标签都将启动HTML块。
- `Parser.HTML_BLOCK_TAGS`,默认列表:`address`,`article`,`aside`,`base`,`basefont`,`blockquote`,## 133# #,`caption`,`center`,`col`,`colgroup`,`dd`,`details`,`dialog`,`dir`, `div`,`dl`,`dt`,`fieldset`,`figcaption`,`figure`,`footer`,`form`,`frame`,`frameset`,`h1`,`h2`,`h3`,`h4`,`h5`,`h6`,## 158# #,`header`,`hr`,`html`,`iframe`,`legend`,`li`,`link`,`main`, `math`,`menu`,`menuitem`,`meta`,`nav`,`noframes`,`ol`,`optgroup`,`option`,`p`,`param`,`section`,`source`,`summary`,`table`,`tbody`,## 183# #,`tfoot`,`th`,`thead`,`title`,`tr`,`track`,`ul`设置HTML块标签
- `Parser.HTML_COMMENT_BLOCKS_INTERRUPT_PARAGRAPH`:默认`true`启用HTML注释来中断段落,否则注释将被解释为HTML块之前需要空白行。
- `Parser.HTML_FOR_TRANSLATOR`:默认`false`,当`true`时,解析器被修改为允许识别翻译器占位符
- `Parser.INDENTED_CODE_BLOCK_PARSER`:默认`true`启用缩进代码块的解析
- `Parser.INDENTED_CODE_NO_TRAILING_BLANK_LINES`:默认`true`启用从缩进的代码块中删除尾随的空白行
- `Parser.INLINE_DELIMITER_DIRECTIONAL_PUNCTUATIONS`默认值`false`,设置为`true`以使用考虑到括号打开/关闭的定界符解析规则
- `Parser.INTELLIJ_DUMMY_IDENTIFIER`:默认`false`在所有解析模式中添加`'\u001f'`作为允许的字符,插件用于允许IntelliJ完成位置标记
- `Parser.LINKS_ALLOW_MATCHED_PARENTHESES`:默认值`true`,当`false`对链接地址使用规范0.27规则并且不解析匹配的`()`
- `Parser.LIST_BLOCK_PARSER`:默认`true`启用列表解析
- `Parser.LIST_REMOVE_EMPTY_ITEMS`,默认为`false`。如果为`true`,则从输出中删除空列表项或仅包含`BlankLine`节点的列表项。
- `Parser.LISTS_END_ON_DOUBLE_BLANK`:默认值`false`,根据规范0.27规则,当`true`列表以双空行终止时。
- `Parser.LISTS_ITEM_PREFIX_CHARS`:默认`"*-+"`,指定标记列表项的字符
- `Parser.LISTS_LOOSE_WHEN_CONTAINS_BLANK_LINE`:默认为`false`,当`true`包含空白行的列表项被视为松散列表项时。
- `Parser.LISTS_LOOSE_WHEN_LAST_ITEM_PREV_HAS_TRAILING_BLANK_LINE`:默认为`false`,当`true`会将最后一个项目后跟空白行的列表视为松散列表时。
- `Parser.MATCH_CLOSING_FENCE_CHARACTERS`:默认值`true`是否关闭围栏字符必须与打开字符匹配,如果为false,则可以打开反back并关闭波浪号,反之亦然。开封器和关闭器中的字符数仍必须相同。
- `Parser.MATCH_NESTED_LINK_REFS_FIRST`:接受测试的`[]`的默认`true`自定义链接引用处理器优先于未接受测试的## 230 ##。即。`[[^f]][test]`是一个Wiki链接,其中`^f`作为页面引用,当此选项为true时,后面是引用链接`test`。如果为false,则引用链接为`test`,并带有脚注`^f`引用文本
- `Parser.PARSE_INNER_HTML_COMMENTS`:默认`false`当`true`将解析HTML块中的内部HTML注释时
- `Parser.PARSE_JEKYLL_MACROS_IN_URLS`默认值`false`,设置为`true`,以允许任何字符出现在网址中的`{{`和`}}`之间,包括空格,竖线和反斜杠
- `Parser.PARSE_MULTI_LINE_IMAGE_URLS`:默认`false`允许解析多行网址:
- `Parser.PARSER_EMULATION_PROFILE`默认`ParserEmulationProfile.COMMONMARK`,设置为所需的`ParserEmulationProfile`值之一
- `Parser.REFERENCE_PARAGRAPH_PRE_PROCESSOR`:默认`true`启用对引用定义的解析
- `Parser.REFERENCES_KEEP`:默认的`KeepType.FIRST`重复进行保留。
- `Parser.REFERENCES`:文档参考定义的默认新存储库
- `Parser.SPACE_IN_LINK_ELEMENTS`默认为`false`,设置为`true`以允许链接或图像的`![]`或`[]`与`()`之间有空格。
- `Parser.SPACE_IN_LINK_URLS`:默认`false`将接受链接地址中的空格,只要它们后面没有`"`
- `Parser.STRONG_EMPHASIS_STYLE_HTML_CLOSE`:默认`(String)null`覆盖HTML以用于包装样式,如果为null,则不覆盖
- `Parser.STRONG_EMPHASIS_STYLE_HTML_OPEN`:默认`(String)null`覆盖HTML以用于包装样式,如果为null,则不覆盖
- `Parser.STRONG_WRAPS_EMPHASIS`:默认`false`,当`true`使用规范0.27规则进行强调/强分辨率时
- `Parser.THEMATIC_BREAK_PARSER`:默认`true`启用主题中断解析
- `Parser.THEMATIC_BREAK_RELAXED_START`:默认`true`启用主题中断的解析,该主题中断之前没有空行
- `Parser.TRACK_DOCUMENT_LINES`默认`false`。在文档的`lineSegments`列表中跟踪`true`文档行时,可以使用offset to line方法获取给定偏移量的从0开始的行号。当`false`时,这些函数通过计算偏移量之前的EOL序列来计算行号。
- `Parser.UNDERSCORE_DELIMITER_PROCESSOR`:默认`true`是否处理下划线定界符
- `Parser.USE_HARDCODED_LINK_ADDRESS_PARSER`:默认`true`,当`false`使用正则表达式进行链接地址解析时,这会导致`StackOverflowError`出现长网址

- `Parser.WWW_AUTO_LINK_ELEMENT`:默认`false`,当`true` `www.`前缀被识别为自动链接前缀时



:information_source:`Parser.USE_HARDCODED_LINK_ADDRESS_PARSER`设置为`true`是默认设置
因为基于正则表达式的解析需要更多的堆栈空间,并且会导致
尝试解析大于约1.5k个字符的链接URL时,出现`StackOverflowError`错误。
此选项仅用于向后兼容,如果有人自定义
正则表达式进行解析。硬编码解析器的性能与regex相当
不需要用于解析的堆栈空间。

|测试| Regex解析器|硬编码解析器|选项|
| --------------------------------- | ------------------- :| -----------------:| --------------- |
|强调没有开启者的关闭器|205 ms | 221 ms |默认|
|重点打开者没有关闭者| 162 ms | 170 ms |默认|
| linkClosersWithNoOpeners | 61 ms | 65 ms |默认|
| linkOpenersAndEmphasisCloseers | 277毫秒| 286毫秒|默认|
| linkOpenersWithNoClosers | 87毫秒| 89毫秒|默认|
| StackOverflow long ImageLinkTest | 136毫秒| 738毫秒|默认值|
| longLinkTest | 77 ms | 63 ms |默认|
|不匹配的开启者和关闭者| 264 ms | 342 ms |默认|
| nestedBrackets | 85 ms | 72 ms |默认|
| nestedStrongEmphasis | 8 ms | 6 ms |默认|
|强调没有开启者的关闭器| 173 ms | 113 ms | URL中的空格|
|强调OpenersWithNoClosers | 163 ms | 123 ms | URL中的空格|
| linkClosersWithNoOpeners | 55 ms | 56 ms | URL中的空格|
| linkOpenersAndEmphasisCloseers | 216 ms | 229 ms | URL中的空间|
| linkOpenersWithNoClosers | 85 ms | 85 ms | URL中的空格|
| longImageLinkTest |堆栈溢出| 684 ms | URL中的空间|
| longLinkTest |堆栈溢出| 71 ms | URL中的空间|
|不匹配的打开器和关闭器| 214 ms | 327 ms | URL中的空间|
| nestedBrackets | 55 ms | 79 ms | URL中的空格|
| nestedStrongEmphasis | 5 ms | 6 ms | URL中的空格|

####列表解析选项

因为列表解析是Markdown解析器实现之间最大的差异。之前
CommonMark没有用于解析列表的硬规范,并且每个实现都花了
艺术许可,以及如何确定清单的外观。

`flexmark-java`根据它们的列表处理特性实现四个解析器系列。
除`ParserEmulationProfile`设置外,每个系列都有一个标准的
控制列表处理的常用选项,每个选项均设置默认值,但最后可以修改
用户。

有几种方法可以配置列表解析选项:

1.推荐的方法是通过将`ParserEmulationProfile`应用于选项
   `MutableDataHolder.setFrom(ParserEmulationProfile)`将所有选项配置为
   特定的解析器。
2.从`ParserEmulationProfile.getOptions()`开始,并修改系列和
   然后将其传递给`MutableDataHolder.setFrom(MutableDataSetter)`
3.通过配置`MutableListOptions`的实例,然后将其传递给
   `MutableDataHolder.setFrom(MutableDataSetter)`
4.首先通过单个键

##### 列出项目选项



- `Parser.LISTS_CODE_INDENT`:默认为`4`,可以与LISTS_ITEM_INDENT相同,也可以为解析器仿真系列的两倍,该系列从第一个列表项缩进开始计算缩进
- `Parser.LISTS_ITEM_INDENT`:默认`4`,也是INDENTED CODE INDENT。解析器仿真系列要么不使用它,要么期望它是下一个缩进项目的列数(在这种情况下,缩进代码是相同的)
- `Parser.LISTS_NEW_ITEM_CODE_INDENT`:默认值`4`,内容缩进> =的新项目,此值会导致代码缩进子级为空的项目,怪异的标准,到目前为止仅适用于CommonMark
-**空白**列表项在标记`Parser.LISTS_ITEM_MARKER_SPACE`,`ListOptions.itemMarkerSpace`后需要显式空格
- 不匹配项类型开始一个新列表:`Parser.LISTS_ITEM_TYPE_MISMATCH_TO_NEW_LIST`,`ListOptions.itemTypeMismatchToNewList`
- 不匹配的项目类型开始一个子列表:`Parser.LISTS_ITEM_TYPE_MISMATCH_TO_SUB_LIST`,`ListOptions.itemTypeMismatchToSubList`
- 项目符号或订购的项目定界符不匹配会启动一个新列表:`Parser.LISTS_DELIMITER_MISMATCH_TO_NEW_LIST`,`ListOptions.delimiterMismatchToNewList`
- 仅在数字后带有`.`的订购商品,否则允许`)`:`Parser.LISTS_ORDERED_ITEM_DOT_ONLY`,`ListOptions.orderedItemDotOnly`
- 第一个订购商品的前缀设置了列表的起始编号:`Parser.LISTS_ORDERED_LIST_MANUAL_START`,`ListOptions.orderedListManualStart`
- 如果项目文本后包含空白行,则该项目为松散的:`Parser.LISTS_LOOSE_WHEN_BLANK_LINE_FOLLOWS_ITEM_PARAGRAPH`,`ListOptions.looseWhenBlankLineFollowsItemParagraph`
- 如果项目是第一项并具有尾随空白行,或者如果前一项具有尾随空白行,则该项目是松散的:`Parser.LISTS_LOOSE_WHEN_PREV_HAS_TRAILING_BLANK_LINE`,`ListOptions.looseWhenPrevHasTrailingBlankLine`
- 如果项目具有松散的子项目,则该项目是松散的:`Parser.LISTS_LOOSE_WHEN_HAS_LOOSE_SUB_ITEM`,`ListOptions.looseWhenHasLooseSubItem`
- 如果项目或其最后一个子项中有尾随空白行,则该项目为松散的:`Parser.LISTS_LOOSE_WHEN_HAS_TRAILING_BLANK_LINE`,`ListOptions.looseWhenHasTrailingBlankLine`
- 如果项目具有非列表子项,则该项目是宽松的:`Parser.LISTS_LOOSE_WHEN_HAS_NON_LIST_CHILDREN`,`ListOptions.looseWhenHasNonListChildren`
- 如果列表中的任何项目都是宽松的,则所有项目都是宽松的:`Parser.LISTS_AUTO_LOOSE`,`ListOptions.autoLoose`
- 自动松散列表设置`Parser.LISTS_AUTO_LOOSE`仅适用于简单的1级列表:`Parser.LISTS_AUTO_LOOSE_ONE_LEVEL_LISTS`,`ListOptions.autoLooseOneLevelLists`
- 列表项标记后缀,后缀后计算的内容缩进:`Parser.LISTS_ITEM_MARKER_SUFFIXES`,`ListOptions.itemMarkerSuffixes`
- 列表项目标记后缀适用于编号项目:`Parser.LISTS_NUMBERED_ITEM_MARKER_SUFFIXED`,`ListOptions.numberedItemMarkerSuffixed`
- 列表项标记后缀不用于影响子项的缩进偏移:`Parser.LISTS_ITEM_CONTENT_AFTER_SUFFIX`,默认为`false`,设置为`true`,以将项目后缀视为列表项标记的一部分内容出于缩进目的而开始。Gfm任务列表使用默认设置。
- `Parser.LISTS_AUTO_LOOSE_ONE_LEVEL_LISTS`,默认为`false`,当`true`将根据其内容确定列表项的松散度时,否则将使用所包含列表的松散度
- `Parser.LISTS_LOOSE_WHEN_LAST_ITEM_PREV_HAS_TRAILING_BLANK_LINE`,默认为`false`,当`true`末尾列表项之前有空白行时,将被标记为松散项。
- `Parser.LISTS_LOOSE_WHEN_CONTAINS_BLANK_LINE`,默认为`false`,当`true`包含空白行的项目将被标记为宽松
- `Parser.LISTS_ITEM_PREFIX_CHARS`,默认为`"*-+"`,设置为允许的列表项前缀


:warning:如果`LISTS_ITEM_TYPE_MISMATCH_TO_NEW_LIST`和
`LISTS_ITEM_TYPE_MISMATCH_TO_SUB_LIST`设置为true,然后如果
项目的行为空,否则将创建子列表。

##### 列表项段落中断选项



- 项目符号可以中断一段:`Parser.LISTS_BULLET_ITEM_INTERRUPTS_PARAGRAPH`,`ListOptions.itemInterrupt.bulletItemInterruptsParagraph`
- 订购商品可以中断一段:`Parser.LISTS_ORDERED_ITEM_INTERRUPTS_PARAGRAPH`,`ListOptions.itemInterrupt.orderedItemInterruptsParagraph`
- 排序的非1项可以中断段落:`Parser.LISTS_ORDERED_NON_ONE_ITEM_INTERRUPTS_PARAGRAPH`,`ListOptions.itemInterrupt.orderedNonOneItemInterruptsParagraph`
- 空的项目符号可以中断一个段落:`Parser.LISTS_EMPTY_BULLET_ITEM_INTERRUPTS_PARAGRAPH`,`ListOptions.itemInterrupt.emptyBulletItemInterruptsParagraph`
- 空的订购商品可能会中断一个段落:`Parser.LISTS_EMPTY_ORDERED_ITEM_INTERRUPTS_PARAGRAPH`,`ListOptions.itemInterrupt.emptyOrderedItemInterruptsParagraph`
- 空的有序非1项可以中断一个段落:`Parser.LISTS_EMPTY_ORDERED_NON_ONE_ITEM_INTERRUPTS_PARAGRAPH`,`ListOptions.itemInterrupt.emptyOrderedNonOneItemInterruptsParagraph`
- 项目符号可以打断列表项目的段落:`Parser.LISTS_BULLET_ITEM_INTERRUPTS_ITEM_PARAGRAPH`,`ListOptions.itemInterrupt.bulletItemInterruptsItemParagraph`
- 订购商品可以中断列表商品的段落:`Parser.LISTS_ORDERED_ITEM_INTERRUPTS_ITEM_PARAGRAPH`,`ListOptions.itemInterrupt.orderedItemInterruptsItemParagraph`
- 有序的非1项目可以中断列表项目的段落:`Parser.LISTS_ORDERED_NON_ONE_ITEM_INTERRUPTS_ITEM_PARAGRAPH`,`ListOptions.itemInterrupt.orderedNonOneItemInterruptsItemParagraph`
- 空的项目符号项目可以打断列表项目的段落:`Parser.LISTS_EMPTY_BULLET_ITEM_INTERRUPTS_ITEM_PARAGRAPH`,`ListOptions.itemInterrupt.emptyBulletItemInterruptsItemParagraph`
- 空的有序非1项可能会中断列表项的段落:`Parser.LISTS_EMPTY_ORDERED_ITEM_INTERRUPTS_ITEM_PARAGRAPH`,`ListOptions.itemInterrupt.emptyOrderedItemInterruptsItemParagraph`
- 空的有序项目可能会中断列表项目的段落:`Parser.LISTS_EMPTY_ORDERED_NON_ONE_ITEM_INTERRUPTS_ITEM_PARAGRAPH`,`ListOptions.itemInterrupt.emptyOrderedNonOneItemInterruptsItemParagraph`
- 空的项目符号子项目可以打断列表项目的段落:`Parser.LISTS_EMPTY_BULLET_SUB_ITEM_INTERRUPTS_ITEM_PARAGRAPH`,`ListOptions.itemInterrupt.emptyBulletSubItemInterruptsItemParagraph`
- 空的有序非1子项目可能会中断列表项目的段落:`Parser.LISTS_EMPTY_ORDERED_SUB_ITEM_INTERRUPTS_ITEM_PARAGRAPH`,`ListOptions.itemInterrupt.emptyOrderedSubItemInterruptsItemParagraph`
- 空的有序子项目可能会中断列表项目的段落:`Parser.LISTS_EMPTY_ORDERED_NON_ONE_SUB_ITEM_INTERRUPTS_ITEM_PARAGRAPH`,`ListOptions.itemInterrupt.emptyOrderedNonOneSubItemInterruptsItemParagraph`

### 渲染器

添加了统一选项处理,保留了现有配置选项,但现在它们修改了
对应的统一选项。

渲染器`Builder()`现在具有`indentSize(int)`方法来设置缩进的大小
分层标签。与在选项中设置`HtmlRenderer.INDENT_SIZE`数据键相同。

在`HtmlRenderer`类中定义:


- `AUTOLINK_WWW_PREFIX`:如果默认前缀`"http://"`以`www.`开头,则添加到自动链接中
- `CODE_STYLE_HTML_CLOSE`默认``(String)null,自定义内联代码关闭HTML
- `CODE_STYLE_HTML_OPEN`默认`(String) null`,自定义内联代码打开HTML
- `DO_NOT_RENDER_LINKS`默认值`false`,在文档中禁用链接呈现。这将导致子上下文也禁用链接渲染。
- `EMPHASIS_STYLE_HTML_CLOSE`默认`(String) null`,自定义强调关闭HTML
- `EMPHASIS_STYLE_HTML_OPEN`默认`(String) null`,自定义强调打开HTML
- `ESCAPE_HTML_BLOCKS`的默认值`ESCAPE_HTML`,可在文档中转义html块
- `ESCAPE_HTML_COMMENT_BLOCKS`的默认值`ESCAPE_HTML_BLOCKS`,可转义文档中找到的html注释块。
- `ESCAPE_HTML`默认`false`,转义文档中找到的所有html
- `ESCAPE_INLINE_HTML_COMMENTS`的默认值`ESCAPE_HTML_BLOCKS`,可在文档中找到转义内联html
- `ESCAPE_INLINE_HTML`的默认值`ESCAPE_HTML`,可在文档中找到转义内联html
- `FENCED_CODE_LANGUAGE_CLASS_MAP`,默认为`new HashMap()`,为类映射字符串提供了单独的语言。如果语言字符串不在地图中,则将使用`FENCED_CODE_LANGUAGE_CLASS_PREFIX` +语言。
- `FENCED_CODE_LANGUAGE_CLASS_PREFIX`默认值`"language-"`,用于为受防护的代码块生成`<code>`类的前缀,仅当info不为空并且在`FENCED_CODE_LANGUAGE_CLASS_MAP`中未定义语言时使用
- `FENCED_CODE_NO_LANGUAGE_CLASS`默认值`""` `,<code>`缩进代码或带防护代码块的`<code>`标签,没有信息(语言)规范
- `FORMAT_FLAGS`默认值`0`,用于`FormattingAppendable`用于呈现HTML的标志
- `GENERATE_HEADER_ID`默认值`false`,使用配置的`HtmlIdGenerator`生成标头ID属性,但不呈现它。该id可以由其他元素(例如锚链接)使用。
- `HARD_BREAK`默认值`"<br />\n"`,用于呈现硬中断的字符串
- `HEADER_ID_ADD_EMOJI_SHORTCUT`,默认为`false`。当设置为`true`时,表情符号快捷方式节点会将快捷方式添加到用于生成标题ID的收集文本中。
- `HEADER_ID_GENERATOR_NO_DUPED_DASHES`默认`false`,如果ID中的`true`重复`- `将被替换为单个`- `
- `HEADER_ID_GENERATOR_NON_ASCII_TO_LOWERCASE`,默认为`true`。设置为`false`时,更改默认标题ID生成器,以不将非ascii字母字符转换为小写字母。##442 ## id兼容性需要。
- `HEADER_ID_GENERATOR_RESOLVE_DUPES`默认值`true`,当`true`将添加一个递增整数以重复ID以使其唯一
- `HEADER_ID_GENERATOR_TO_DASH_CHARS`默认`"_"`,用于生成ID的文本中要转换为`- `的字符集,未设置的非字母数字字符将被删除
- `HEADER_ID_REF_TEXT_TRIM_LEADING_SPACES`,默认为`true`。当设置为`false`时,标题中链接参考文本中的前导空格不会为用于生成ID的文本修剪。
- `HEADER_ID_REF_TEXT_TRIM_TRAILING_SPACES`,默认为`true`。当设置为`false`时,标题中链接参考文本中的尾部空格不会为用于生成ID的文本修剪。
- `HTML_BLOCK_CLOSE_TAG_EOL`默认值`true`,当`false`将禁止在HTML呈现期间自动生成的HTML块标记之后的EOL。
- `HTML_BLOCK_OPEN_TAG_EOL`默认值`true`,当`false`在HTML呈现期间自动生成的HTML块标记之前取消EOL时。
- `INDENT_SIZE`默认值`0`,每个缩进级别的嵌套标签要使用多少空间
- `INLINE_CODE_SPLICE_CLASS`默认值`(String) null`,用于拆分和拼接内联代码范围以进行源代码行跟踪
- `MAX_TRAILING_BLANK_LINES`默认`1`,已渲染的HTML允许的最大尾随空白行
- `OBFUSCATE_EMAIL_RANDOM`默认为`true`,当`false`不会使用随机数生成器进行混淆时。用于测试
- `OBFUSCATE_EMAIL`默认`false`,何时`true`会混淆`mailto`链接
- `PERCENT_ENCODE_URLS`默认`false`,百分比编码网址
- `RECHECK_UNDEFINED_REFERENCES`默认为`false`,对于标记为未定义的链接和图像引用,请重新检查`Parser.REFERENCES`中是否存在引用。解析后添加新引用时使用
- `RENDER_HEADER_ID`默认`false`,使用已配置的`HtmlIdGenerator`为标头呈现标头ID属性
- `SOFT_BREAK`默认值`"\n"`,用于呈现软中断的字符串
- `SOURCE_POSITION_ATTRIBUTE`默认值`""`,源位置HTML属性的名称,源位置分配为`startOffset + '-' + endOffset`
- `SOURCE_POSITION_PARAGRAPH_LINES`默认值`false`,如果为true,则在设置了源位置属性的情况下,将单个段落源行包装在`span`中
- `SOURCE_WRAP_HTML_BLOCKS` SOURCE_WRAP_HTML的默认值,如果生成源位置属性,则将HTML块包装在具有源位置的`div`中
- `SOURCE_WRAP_HTML`默认值`false`,如果生成源位置属性,则用源位置包装HTML
- `STRONG_EMPHASIS_STYLE_HTML_CLOSE`默认`(String) null`,自定义重点强调,关闭HTML
- `STRONG_EMPHASIS_STYLE_HTML_OPEN`默认`(String) null`,自定义强调打开HTML
- `SUPPRESS_HTML_BLOCKS`的默认值`SUPPRESS_HTML`,禁止html块的html输出
- `SUPPRESS_HTML_COMMENT_BLOCKS`的默认值`SUPPRESS_HTML_BLOCKS`,取消html注释块的html输出
- `SUPPRESS_HTML`默认`false`,取消所有html的html输出
- `SUPPRESS_INLINE_HTML_COMMENTS`的默认值`SUPPRESS_INLINE_HTML`,禁止将HTML输出用于内嵌html注释
- `SUPPRESS_INLINE_HTML`的默认值`SUPPRESS_HTML`,取消内联html的html输出
- `SUPPRESS_LINKS`的默认值`"javascript:.*"`,该正则表达式可禁止匹配的任何链接。测试在使用链接解析器解析链接之前进行。因此,任何链接匹配都将在解决之前被抑制。同样,返回被抑制链接的链接解析器也不会被抑制,因为这取决于自定义链接解析器要处理的内容。

  被抑制的链接将仅呈现子节点,实际上,`[Something
  New](javascript:alert(1))`将呈现为`Something New`。

  基于URL前缀的链接抑制不适用于HTML内联或块元素。采用
  HTML抑制选项。
- `TYPE`默认`"HTML"`,渲染器类型。渲染器类型扩展可以添加自己的扩展。`JiraConverterExtension`定义`JIRA`
- `UNESCAPE_HTML_ENTITIES`默认值`true`,当`false`会将HTML实体保留为呈现的HTML时。

:exclamation:所有的转义和抑制选项都具有动态默认值。这使您能够
设置`ESCAPE_HTML`并转义所有html。如果您设置特定键的值,则
设置值将用于该键。同样,影响键的注释取自它们的值
非注释对应项。如果要排除注释不受抑制的影响
或转义,则需要将相应的注释键设置为`false`并设置
`true`的非注释键。

### Formatter

Formatter使用各种格式设置选项将AST呈现为markdown,以清理并制作
源一致,并且可能从一种压痕规则集转换为另一种压痕规则集。格式器API
允许扩展为自定义节点提供和处理格式设置选项。

:information_source:在0.60.0之前的版本中,格式化程序功能已在
`flexmark-formatter`模块,并需要其他依赖项。

请参阅:[Markdown Formatter](Markdown-Formatter)

格式化程序还可用于通过以下方式帮助将markdown文档翻译成另一种语言:
提取可翻译字符串,用标识符替换不可翻译字符串,然后
最后用翻译版本替换可翻译文本范围。

请参阅:[Translation Helper API](Translation-Helper-API)

Formatter可用于将多个Markdown文档合并为一个文档,而
保留每个文档中参考的参考分辨率,即使参考ID
合并文档之间的冲突。

请参阅:[Markdown Merge API](Markdown-Merge-API)

## PDF Output Module

使用`PdfConverterExtension`中的[Open HTML To PDF]库将HTML转换为PDF
`flexmark-pdf-converter`模块。

请参阅:[PDF Renderer Converter](ext/PDF-Renderer-Converter)

[Usage PDF Output](Usage#pdf-output)

## Available Extensions

以下扩展是使用此库开发的,每个扩展都有自己的工件。

扩展选项在其扩展类中定义。

## Abbreviation

允许创建缩写,将其以纯文本形式替换为`<abbr></abbr>`标签或
(可选)带有缩写扩展名的`<a></a>`。

使用工件`flexmark-ext-abbreviation`中的类`AbbreviationExtension`。

提供以下选项:

在`AbbreviationExtension`类中定义:

|静态字段|默认值|说明|
|:-------------------------- |:-------------------- ----- |:----------------------------------------------- ---------- |
| `ABBREVIATIONS` |新存储库|用于文档缩写定义的存储库|
| `ABBREVIATIONS_KEEP` | `KeepType.FIRST` |可重复保存。|
| `USE_LINKS` | `false` |使用`<a>`代替`<abb>`标签来呈现html |
| `ABBREVIATIONS_PLACEMENT` | `ElementPlacement.AS_IS` |格式设置选项,请参见:[Markdown Formatter](Markdown-Formatter) |
| `ABBREVIATIONS_SORT` | `ElementPlacement.AS_IS` |格式设置选项,请参见:[Markdown Formatter](Markdown-Formatter) |

## Admonition

创建块样式的辅助内容。基于[Admonition Extension, Material for MkDocs]
(*个人观点:MkDocs的材料令人眼花。乱。如果您没有看过它,则表示
缺少视觉享受。*)。参见[Admonition Extension](ext/Admonition-Extension.md)

使用工件`flexmark-ext-admonition`中的类`AbbreviationExtension`。

#### CSS和JavaScript必须包含在您的页面中

默认的CSS和JavaScript作为资源包含在jar中:

* [admonition.css](https://github.com/vsch/flexmark-java/blob/master/flexmark-ext-admonition/src/main/resources/admonition.css)
* [admonition.js](https://github.com/vsch/flexmark-java/blob/master/flexmark-ext-admonition/src/main/resources/admonition.js)

也可以通过调用`AdmonitionExtension.getDefaultCSS()`和
`AdmonitionExtension.getDefaultScript()`静态方法。

该脚本应包含在文档正文的底部,并用于切换
可折叠警告元素的打开/关闭状态。

## AnchorLink

使用GitHub ID生成算法自动将锚链接添加到标题

:warning:此扩展将仅为具有id属性的标头呈现锚链接
与他们相关联。您需要将`HtmlRenderer.GENERATE_HEADER_ID`选项设置为
`true`以便生成标头ID。

使用工件`flexmark-ext-anchorlink`中的类`AnchorLinkExtension`。

提供以下选项:

在`AnchorLinkExtension`类中定义:

|静态字段|默认值|说明|
|:--------------------------- |:-------------- |:--- -------------------------------------------------- ------------ |
| `ANCHORLINKS_SET_ID` | `true` |是否将id属性设置为标题id,如果为true |
| `ANCHORLINKS_SET_NAME` | `false` |是否将name属性设置为标头ID
| `ANCHORLINKS_WRAP_TEXT` | `true` |是否将标题文本包装在锚点中,如果为true |
| `ANCHORLINKS_TEXT_PREFIX` | `""` |原始html前缀。在标题文本之前添加,已包装或展开|
| `ANCHORLINKS_TEXT_SUFFIX` | `""` |原始html后缀。在标题文本之前添加,已包装或展开|
| `ANCHORLINKS_ANCHOR_CLASS` | `""` |类别为`a`标签|

## Aside

与块引号相同,但使用`|`作为前缀并生成`<aside>`标签。为了使其兼容
使用表格扩展名时,除了块行,不能将`|`作为行的最后一个字符,
如果使用此扩展名,则表的行必须以`|`结尾,否则它们
将被视为备用块。

使用工件`flexmark-ext-aside`中的类`AsideExtension`。

在`AsideExtension`类中定义:

|静态字段|默认值|说明|
|:--------------------------------------------- |:- ------------- |:----------------------------------- -------------------------------------------------- -------------------------------------------------- ---- |
| `IGNORE_BLANK_LINE` | `false` | aside块将在aside块之间包含空白行,并将其视为空白行之前还带有aside块标记|
| `EXTEND_TO_BLANK_LINE` | `false` |当为true时,旁边的块会延伸到空白行。比commonmark严格标准|支持更多的la块引用解析
| `ALLOW_LEADING_SPACE` | `true` |(允许`>`之前的`true`前导空格)|
| `INTERRUPTS_ITEM_PARAGRAPH` | `true` |当`true`块引号可以中断列表项文本时,否则需要空行才能包含在列表项|中
| `INTERRUPTS_PARAGRAPH` | `true` |当`true`块引用可以中断一段时,否则需要在|之前使用空行
| `WITH_LEAD_SPACES_INTERRUPTS_ITEM_PARAGRAPH` | `true` |当带前导空格的`true`块引号可以中断列表项文本时,否则需要在空白行之前或没有前导空格|

`AsideExtension`选项键是动态数据键,取决于相应的Parser块引用
其默认值的选项。如果未明确设置它们,则将采用默认值
从相应的块引用值的值(前缀`BLOCK_QUOTE_`到
`AsideExtension`键名以获取`Parser`块引号键名)。

:warning:这可能会破坏依赖于`0.40.20`之前的扩展版本的代码
因为解析规则可以更改,具体取决于哪些块引用选项与其
默认值。

为了确保有独立的选项保留旁白和引用,请明确地保留旁白选项。
下面将把所有备用选项设置为默认值,与块引用无关
选项:

    .set(EXTEND_TO_BLANK_LINE, false)
    .set(IGNORE_BLANK_LINE, false)
    .set(ALLOW_LEADING_SPACE, true)
    .set(INTERRUPTS_PARAGRAPH, true)
    .set(INTERRUPTS_ITEM_PARAGRAPH, true)
    .set(WITH_LEAD_SPACES_INTERRUPTS_ITEM_PARAGRAPH, true)

## Attributes

将属性`{...}`语法转换为属性AST节点,并将属性提供程序添加到
设置HTML呈现期间紧邻的前一个兄弟元素的属性。看到
[Attributes Extension](ext/Attributes-Extension.md)

由工件`flexmark-ext-attributes`在`AttributeExtension`中定义

* `ASSIGN_TEXT_ATTRIBUTES`,默认为`true`。当`false`节点的属性分配规则是
  更改为不允许文本元素获取属性。

* `FENCED_CODE_INFO_ATTRIBUTES`,默认为`true`。当`false`属性分配在末尾时
  受防护的代码信息字符串将被忽略。

* `USE_EMPTY_IMPLICIT_AS_SPAN_DELIMITER`,默认为`false`。设置为`true`时将处理`{#}`
  或`{.}`,没有嵌入的空格,作为开始属性跨度分隔符来标记开始
  属性分配给`{.}`或`{#}`与匹配的attribute元素之间的文本。

使用工件`flexmark-ext-attributes`中的类`AttributesExtension`。

完整规格:
[ext_attributes_ast_spec](https://github.com/vsch/flexmark-java/blob/master/flexmark-ext-attributes/src/test/resources/ext_attributes_ast_spec.md)

## Autolink

将普通链接(例如URL和电子邮件地址)转换为链接(基于[autolink-java])。

警告:当前的实现对大文件的性能有重大影响。

使用工件`flexmark-ext-autolink`中的类`AutolinkExtension`。

在`AsideExtension`类中定义:

* `IGNORE_LINKS`,默认为`""`,是一个正则表达式,用于匹配不应为
  自动链接。这可以包含完整的网址(例如`www.google.com`)或包含通配符的部分
  匹配模式。任何与表达式匹配的已识别自动链接将呈现为
  文本。

## Definition Lists

将[Php Markdown Extra Definition List]的定义语法转换为`<dl></dl>` HTML和
相应的AST节点。

定义项的前面可以是`:`或`~`,具体取决于配置的选项。

使用工件`flexmark-ext-definition`中的类`DefinitionExtension`。

提供以下选项:

在`DefinitionExtension`类中定义:

|静态字段|默认值|说明|
|:-------------------------------- |:-------------- --------- |:--------------------------------------- -------------------------------------------------- ----------------- |
| `COLON_MARKER` | `true` |允许使用`:`作为定义项目标记|
| `MARKER_SPACES` | `1` |有效定义项目|的定义项目标记后的最小空格数
| `TILDE_MARKER` | `true` |允许使用`~`作为定义项目标记|
| `DOUBLE_BLANK_LINE_BREAKS_LIST` | `false` |当真定义项和下一个定义项之间的双空行将打破定义列表|
| `FORMAT_MARKER_SPACES` | `3` |格式设置选项,请参见:[Markdown Formatter](Markdown-Formatter) |
| `FORMAT_MARKER_TYPE` | `DefinitionMarker.ANY` |格式化选项,请参见:[Markdown Formatter](Markdown-Formatter) |

:information_source:此扩展使用列表解析和缩进规则,并将尽最大努力
根据所选选项对齐列表项和定义项的解析。对于非固定缩进
解析器族将使用定义项的内容缩进列作为子项,否则
将`Parser.LISTS_ITEM_INDENT`值用于子项。

Wiki:[[Definition List Extension]]

## Docx Converter

使用[docx4j]库将已解析的Markdown AST渲染为docx格式。

人工制品:`flexmark-docx-converter`

有关代码,请参见[DocxConverterCommonMark Sample],有关代码,请参见[[Customizing Docx Rendering]]
概述和有关自定义样式的信息。

Pegdown版本可以在[DocxConverterPegdown Sample]中找到

有关详细信息,请参见[[Docx Renderer Extension]]

## Emoji

允许使用[Emoji-Cheat-Sheet.com]从表情符号快捷方式创建指向表情符号图像的图像链接
并可以选择用Mark Wunsch映射的Unicode等效字符替换
在[mwunsch/rumoji]中找到(https://github.com/mwunsch/rumoji)

使用工件`flexmark-ext-emoji`中的类`EmojiExtension`。

提供以下选项:

在`EmojiExtension`类中定义:

* `ATTR_ALIGN`,默认为`"absmiddle"`,用于在没有
  属性提供者覆盖
* `ATTR_IMAGE_SIZE`,默认为`"20"`,在没有
  属性提供者覆盖
* `ATTR_IMAGE_CLASS`,默认为`""`,如果不为空,则会将图片标签类属性设置为此
  值。
* `ROOT_IMAGE_PATH`,默认为`"/img/"`,是表情符号图片文件的根路径。看到
  <https://github.com/arvida/emoji-cheat-sheet.com>
* `USE_SHORTCUT_TYPE`,默认为`EmojiShortcutType.EMOJI_CHEAT_SHEET`,选择快捷方式类型:
  * `EmojiShortcutType.EMOJI_CHEAT_SHEET`
  * `EmojiShortcutType.GITHUB`
  * `EmojiShortcutType.ANY_EMOJI_CHEAT_SHEET_PREFERRED`使用任何来源的任何快捷方式。如果
    图像类型选项不是`UNICODE_ONLY`,将生成指向Emoji备忘单文件的链接或
    GitHub URL,优先使用Emoji Cheat Sheet文件。
  * `EmojiShortcutType.ANY_GITHUB_PREFERRED`-使用任何来源的任何快捷方式。如果图像类型
    options不是`UNICODE_ONLY`,将生成指向Emoji备忘单文件或GitHub URL的链接,
    优先考虑GitHub URL。
* `USE_IMAGE_TYPE`,默认为`EmojiImageType.IMAGE_ONLY`,用于选择图像类型
  允许的。
  * `EmojiImageType.IMAGE_ONLY`,仅使用图片链接
  * `EmojiImageType.UNICODE_ONLY`转换为unicode,如果没有unicode,则视为无效
    表情符号快捷方式
  * `EmojiImageType.UNICODE_FALLBACK_TO_IMAGE`转换为unicode,如果没有unicode,则使用图片。
    使用Emoji-Cheat-Sheet.com的emoji快捷方式https://www.webfx.com/tools/emoji-cheat-sheet

## Enumerated Reference

用于为文档中的元素创建编号的引用和编号的文本标签。
[[Enumerated References Extension]]

使用工件`flexmark-ext-enumerated-reference`中的类`EnumeratedReferenceExtension`。

:exclamation:注意需要[Attributes](#attributes)扩展名才能使引用成为
正确解决了渲染问题。

## Footnotes

在文档中创建脚注引用。脚注位于渲染的底部
从脚注引用到脚注的链接,反之亦然。[[Footnotes Extension]]

将`[^footnote]`转换为脚注引用,将`[^footnote]: footnote definition`转换为
文件中的脚注。

## Gfm-Issues

启用格式为`#123`的Gfm问题参考解析

在工件`flexmark-ext-gfm-issues`中使用类`GfmIssuesExtension`。

提供以下选项:

在`GfmIssuesExtension`类中定义:



* `GIT_HUB_ISSUES_URL_ROOT`:默认`"issues"`覆盖用于生成URL的根
* `GIT_HUB_ISSUE_URL_PREFIX`:默认的`"/"`覆盖前缀,用于将发行编号附加到URL
* `GIT_HUB_ISSUE_URL_SUFFIX`:默认的`""`覆盖后缀,用于将问题编号附加到URL
* `GIT_HUB_ISSUE_HTML_PREFIX`:默认`""`覆盖HTML以用作问题文本的前缀
* `GIT_HUB_ISSUE_HTML_SUFFIX`:默认`""`覆盖HTML以用作问题文本的后缀


## Gfm-Strikethrough/Subscript

通过将文本括在`~~`中来启用文本删除线。例如,在`hey ~~you~~`中,`you`将
呈现为删除线文本。

在工件`flexmark-ext-gfm-strikethrough`中使用类`StrikethroughExtension`。

通过将文本下标括在`~`中来启用它。例如,在`hey ~you~`中,`you`将是
呈现为下标文本。

在工件`flexmark-ext-gfm-strikethrough`中使用类`SubscriptExtension`。

要同时启用下标和删除线:

在工件`flexmark-ext-gfm-strikethrough`中使用类`StrikethroughSubscriptExtension`。

:warning:这些扩展名中只有一个可以包含在扩展名列表中。如果你两个都想要
删除线和下标使用`StrikethroughSubscriptExtension`。

提供以下选项:

在`StrikethroughSubscriptExtension`类中定义:



* `STRIKETHROUGH_STYLE_HTML_OPEN`:默认`(String)null`覆盖用于包装样式的HTML,如果为null,则不覆盖
* `STRIKETHROUGH_STYLE_HTML_CLOSE`:默认`(String)null`覆盖HTML以用于包装样式,如果为null,则不覆盖
* `SUBSCRIPT_STYLE_HTML_OPEN`:默认`(String)null`覆盖HTML以用于包装样式,如果为null,则不覆盖
* `SUBSCRIPT_STYLE_HTML_CLOSE`:默认`(String)null`覆盖HTML以用于包装样式,如果为null,则不覆盖


## Gfm-TaskList

启用基于列表项的任务列表,这些任务列表的文本开头为:`[ ]`,`[x]`或`[X]`

在工件`flexmark-ext-gfm-tasklist`中使用类`TaskListExtension`。

提供以下选项:

在`TaskListExtension`类中定义:

|静态字段|默认值|说明|
|:----------------------------- |:----------------- -------------------------------------------------- -------------------------------------------------- -------------------------------------------------- ----------------------- |:------------------------- -------------------------------------------------- --------- |
| `ITEM_DONE_MARKER` | `<input` <br>&nbsp;&nbsp; `type="checkbox"` <br>&nbsp;&nbsp; `class="task-list-item-checkbox"` <br>&nbsp;&nbsp;# #753 ## <br>&nbsp;&nbsp; `disabled="disabled" />` |字符串,用于项目完成标记html。|
| `ITEM_NOT_DONE_MARKER` | `<input` <br>&nbsp;&nbsp; `type="checkbox"` <br>&nbsp;&nbsp; `class="task-list-item-checkbox"` <br>&nbsp;&nbsp;# #759 ## |字符串,用于未完成的标记html。|
| `ITEM_CLASS` | `"task-list-item"` |严格清单项目类别属性|
| `ITEM_ITEM_DONE_CLASS` | `""` |已完成任务列表项目|的列表项目类
| `ITEM_ITEM_NOT_DONE_CLASS` | `""` |未完成任务列表项|的列表项类
| `LOOSE_ITEM_CLASS` | `ITEM_CLASS` |的值松散清单项目类别属性,如果未设置,则使用严格项目类别|的值
| `PARAGRAPH_CLASS` | `""` | p标签类属性,仅适用于松散列表项|
| `FORMAT_LIST_ITEM_CASE` | `TaskListItemCase.AS_IS` |格式化选项,请参见:[Markdown Formatter](Markdown-Formatter) |
| `FORMAT_LIST_ITEM_PLACEMENT` | `TaskListItemPlacement.AS_IS` |格式化选项,请参见:[Markdown Formatter](Markdown-Formatter) |

* `TaskListItemCase`
  * `AS_IS`:不变
  * `LOWERCASE`:将`[X]`更改为`[x]`
  * `UPPERCASE`:将`[x]`更改为`[X]`

* `TaskListItemPlacement`
  * `AS_IS`:不变
  * `INCOMPLETE_FIRST`:对所有列表进行排序,以将不完整的任务项放在第一位
  * `INCOMPLETE_NESTED_FIRST`:对所有列表进行排序,以放置不完整的任务项或带有
    不完整的任务子项目优先
  * `COMPLETE_TO_NON_TASK`:对所有列表进行排序,以将不完整的任务项放在第一位并进行更改
    完成任务项到非任务项
  * `COMPLETE_NESTED_TO_NON_TASK`:对所有列表进行排序,以放置不完整的任务项目或带有以下内容的项目
    首先将不完整的任务子项并将完整的任务项更改为非任务项

## Gfm-Users

启用`#123`形式的Gfm用户引用解析

在工件`flexmark-ext-gfm-users`中使用类`GfmUsersExtension`。

提供以下选项:

在`GfmUsersExtension`类中定义:



* `GIT_HUB_USERS_URL_ROOT`:默认`"https://github.com"`覆盖用于生成URL的根
* `GIT_HUB_USER_URL_PREFIX`:默认的`"/"`覆盖前缀,用于将用户名附加到URL
* `GIT_HUB_USER_URL_SUFFIX`:默认的`""`覆盖后缀,用于将用户名附加到URL
* `GIT_HUB_USER_HTML_PREFIX`:默认`"<strong>"`覆盖HTML以用作用户名文本的前缀
* `GIT_HUB_USER_HTML_SUFFIX`:默认`"</strong>"`覆盖HTML以用作用户名文本的后缀


## GitLab Flavoured Markdown

解析并渲染
[GitLab风味Markdown](https://gitlab.com/gitlab-org/gitlab-ce/blob/master/doc/user/markdown.md)。

*添加:视频链接渲染器,可将视频文件的链接转换为嵌入的内容。有效的视频
  扩展名是`.mp4`,`.m4v`,`.mov`,`.webm`和`.ogv`。

  ```
  <div class="video-container">
  <video src="video.mp4" width="400" controls="true"></video>
  <p><a href="video.mp4" target="_blank" rel="noopener noreferrer" title="Download 'Sample Video'">Sample Video</a></p>
  </div>
  ```

*多行块引用分隔符`>>>`

*已删除的文字标记`{- -}`或`[- -]`

*插入文字标记`{+ +}`或`[+ +]`

*数学,通过```$``$```内联,或作为包含`math`信息字符串的受限制代码,要求包含
  呈现的HTML页面中的Katex。

*通过美人鱼作为带有`mermaid`信息字符串的受保护代码进行图形绘制,通过美人鱼包含
  上面的数学解决方案。

在工件`flexmark-ext-gitlab`中使用类`GitLabExtension`。

提供以下选项:

在`GitLabExtension`类中定义:



* `INS_PARSER`:默认`true`,启用ins解析器
* `DEL_PARSER`:默认`true`,启用del解析器
* `BLOCK_QUOTE_PARSER`:默认`true`,启用多行块引用解析器
* `INLINE_MATH_PARSER`:默认`true`,启用内联数学解析器
* `RENDER_BLOCK_MATH`:默认`true`,启用将数学围栏代码块呈现为数学元素
* `RENDER_BLOCK_MERMAID`:默认`true`,启用将美人鱼栅栏代码块渲染为美人鱼元素
* `RENDER_VIDEO_IMAGES`:默认`true`,启用将带有视频扩展名的图像链接渲染为视频元素
* `RENDER_VIDEO_LINK`:默认`true`,启用与视频图像的视频链接渲染

* `INLINE_MATH_CLASS`:默认`"katex"`,内联数学类属性
* `BLOCK_MATH_CLASS`:默认`"katex"`块数学类属性
* `BLOCK_MERMAID_CLASS`:默认`"mermaid"`阻止美人鱼类别属性
* `VIDEO_IMAGE_CLASS`:默认的`"video-container"`类属性
* `VIDEO_IMAGE_LINK_TEXT_FORMAT`:视频链接文字的默认`"Download '%s'"`格式,`%s`替换为图片替代文字
* `BLOCK_INFO_DELIMITERS`:默认`" "`,用于从受保护代码信息字符串中提取数学和美人鱼字符串的定界符
* `VIDEO_IMAGE_EXTENSIONS`:默认`"mp4,m4v,mov,webm,ogv"`,是要呈现为视频的视频文件的扩展名


:information_source:正确渲染Math和Mermaid需要包含
[Katex](https://github.com/Khan/KaTeX)和[Mermaid](https://github.com/knsv/mermaid)脚本
HTML页面。

如果文件与HTML页面位于同一目录中,则在`<head>`和
`</head>`标签,您需要包括:

```html
<link rel="stylesheet" href="katex.min.css">
<script src="katex.min.js"></script>
<script src="mermaid.min.js"></script>
```

除了Katex脚本外,您还需要在页面正文底部添加JavaScript,以
加载DOM时转换数学元素:

```html
<script>
    (function () {
      document.addEventListener("DOMContentLoaded", function () {
        var mathElems = document.getElementsByClassName("katex");
        var elems = [];
        for (const i in mathElems) {
            if (mathElems.hasOwnProperty(i)) elems.push(mathElems[i]);
        }

        elems.forEach(elem => {
            katex.render(elem.textContent, elem, { throwOnError: false, displayMode: elem.nodeName !== 'SPAN', });
        });
    });
})();
```

## Html To Markdown

并不是真正的扩展,但在需要将HTML转换为Markdown时很有用。

将HTML转换为Markdown,并假定将使用所有非特定于应用程序的扩展名:

*缩写
* 在旁边
*块引号
*粗体,斜体,内联代码
*项目符号和编号列表
*定义
*围栏代码
*通过
*下标
*上标
*桌子
* Gfm任务列表项
*还将处理多行URL图像的转换

该转换器具有扩展API,允许自定义HTML标签向Markdown转换,链接URL
更换和选项。

在工件`flexmark-html2md-converter`中使用类`FlexmarkHtmlConverter`。

使用`FlexmarkHtmlConverter.builder(options).build().convert(htmlString)`获得与
旧的`FlexmarkHtmlParser.parse(htmlString, options)`

提供以下选项:

在`FlexmarkHtmlConverter`类中定义:

* `BR_AS_EXTRA_BLANK_LINES`默认值`true`,当在空白行后遇到true `<br>`时
  已经在输出中或当前输出以`<br>`结尾,然后将内联HTML `<br>`插入
  输出以在渲染结果中创建额外的空白行。
* `BR_AS_PARA_BREAKS`默认值`true`,如果在新的开始时遇到了`<br>`
  行将被视为段落中断
* `CODE_INDENT`,默认为4个空格,用于缩进代码的缩进
* `DEFINITION_MARKER_SPACES`,默认`3`,`:`之后的最小空格用于定义
* `DIV_AS_PARAGRAPH`默认值`false`,当为true时,将`<div>`包装的文本视为
  `<p>`通过在文本后添加空白行来换行。
* `DOT_ONLY_NUMERIC_LISTS`,默认为`true`。设置为假的右括号时作为列表
  如果出现在MS-Word样式列表中,则分隔符将在Markdown中使用。否则括号
  分隔列表将转换为点`.`。
* `EOL_IN_TITLE_ATTRIBUTE`,默认为`" "`,用于在图片和链接标题中代替EOL的字符串
  属性。
* `LISTS_END_ON_DOUBLE_BLANK`默认为`false`,当设置为`true`时,连续列表是
  用双空白行分隔,否则用空白HTML注释行分隔。
* `LIST_CONTENT_INDENT`,默认`true`,列表项的连续行和定义缩进
  到内容列,否则为4个空格
* `MIN_SETEXT_HEADING_MARKER_LENGTH`,默认`3`,最小值3,最小setext标题标记长度
* `MIN_TABLE_SEPARATOR_COLUMN_WIDTH`,默认`1`,最小值1,分隔符中`- `的最小数量
  列,不包括对齐冒号`:`
* `MIN_TABLE_SEPARATOR_DASHES`,默认`3`,最小值3,最小分隔符列宽,包括
  对齐冒号`:`
* `NBSP_TEXT`,默认为`" "`,用于代替非中断空间的字符串
* `ORDERED_LIST_DELIMITER`,默认`'.'`,订购商品的分隔符
* `OUTPUT_UNKNOWN_TAGS`,默认为`false`,当输出真正的未处理标签时,否则
  他们被忽略
* `RENDER_COMMENTS`,默认为`false`。设为true时,HTML注释将在
  markdown。
* `SETEXT_HEADINGS`,默认为`true`,如果为true,则将Setext标题用于h1和h2
* `THEMATIC_BREAK`,默认`"*** ** * ** ***"`,`<hr>`替换
* `UNORDERED_LIST_DELIMITER`,默认`'*'`,无序列表项的分隔符
* `OUTPUT_ATTRIBUTES_ID`,默认为`true`,如果`true` id属性将在
  [属性](#attributes)语法
* `OUTPUT_ID_ATTRIBUTE_REGEX`,默认`"^user-content-(.*)"`,用于处理ID的正则表达式
  属性,如果匹配,则将连接所有非空的组(如果结果字符串)
  修剪后为空,则不会生成ID。如果值为空,则不进行任何处理
  完成和id将呈现为HTML中。

  :information_source:默认值将去除GitHub呈现的HTML标题ID前缀
  `user-content- `。
* `OUTPUT_ATTRIBUTES_NAMES_REGEX`默认值`""`,如果不为空,则将输出属性
  与正则表达式匹配的属性名称的扩展语法。
* `ADD_TRAILING_EOL`,默认为`false`。将尾随EOL添加到生成的markdown文字中。
*将某些markdown格式元素转换为其内部文本,所有的默认值为false:
  * `SKIP_HEADING_1`-标题1
  * `SKIP_HEADING_2`-标题2
  * `SKIP_HEADING_3`-标题3
  * `SKIP_HEADING_4`-标题4
  * `SKIP_HEADING_5`-标题5
  * `SKIP_HEADING_6`-标题6
  * `SKIP_ATTRIBUTES`-属性扩展转换
  * `SKIP_FENCED_CODE`-跳过受限制的代码扩展转换
  * `SKIP_LINKS`-将链接转换为纯文本
  * `SKIP_CHAR_ESCAPE`-无转义字符转换

*控制内联元素的转换,默认为`ExtensionConversion.MARKDOWN`
  * `EXT_INLINE_STRONG`-强
  * `EXT_INLINE_EMPHASIS`-重点
  * `EXT_INLINE_CODE`-代码
  * `EXT_INLINE_DEL`-删除
  * `EXT_INLINE_INS`-插件
  * `EXT_INLINE_SUB`-子
  * `EXT_INLINE_SUP`-sup

  可用设置:

  * `ExtensionConversion.MARKDOWN`-转换为减价
  * `ExtensionConversion.TEXT`-转换为内部文本
  * `ExtensionConversion.HTML`-保留HTML原样
* `UNWRAPPED_TAGS`,默认为`new String[] { "article", "address", "frameset", "section",
  "small", "iframe", }`,定义应呈现其内部html内容的标签
* `WRAPPED_TAGS`,默认为`new String[] { "kbd", "var" }`,定义应呈现为
  外层HTML。内部文字将转换为markdown。
* `TYPOGRAPHIC_REPLACEMENT_MAP`选项键带有`Map<String,String>`,如果不为空,则为
  用于代替捆绑地图。缺少的任何印刷字符或HTML实体
  地图将不经转换而输出。如果要取消印刷,则不
  输出将`""`添加到其映射值。

  捆绑的地图按以下方式填充:

  ```java
  class HtmlParser {
      static Map<String, String> specialCharsMap = new HashMap<>();
      static {
          specialCharsMap.put(""", "\"");
          specialCharsMap.put(""", "\"");
          specialCharsMap.put("&ldquo;", "\"");
          specialCharsMap.put("&rdquo;", "\"");
          specialCharsMap.put("‘", "'");
          specialCharsMap.put("’", "'");
          specialCharsMap.put("&lsquo;", "'");
          specialCharsMap.put("&rsquo;", "'");
          specialCharsMap.put("&apos;", "'");
          specialCharsMap.put("«", "<<");
          specialCharsMap.put("&laquo;", "<<");
          specialCharsMap.put("»", ">>");
          specialCharsMap.put("&raquo;", ">>");
          specialCharsMap.put("…", "...");
          specialCharsMap.put("&hellip;", "...");
          specialCharsMap.put("–", "--");
          specialCharsMap.put("&endash;", "--");
          specialCharsMap.put("—", "---");
          specialCharsMap.put("&emdash;", "---");
      }
  }
  ```
*要控制如何在markdown中呈现转换后的HTML表,您可以使用任何
  `Formatter`表选项,因为HTML解析器使用格式化程序来呈现表。

使用出色的[jsoup](https://jsoup.org/)HTML解析库和emoji快捷方式,使用
[Emoji-Cheat-Sheet.com](https://www.webfx.com/tools/emoji-cheat-sheet)

## Ins

通过将文本括在`++`中来启用ins标记文本。例如,在`hey ++you++`中,`you`将
呈现为插入的文本。

在工件`flexmark-ext-ins`中使用类`InsExtension`。

提供以下选项:

在`InsExtension`类中定义:



* `INS_STYLE_HTML_CLOSE`:默认`(String)null`覆盖HTML以用于包装样式,如果为null,则不覆盖
* `INS_STYLE_HTML_OPEN`:默认`(String)null`覆盖HTML以用于包装样式,如果为null,则不覆盖


## Jekyll Tags

允许渲染带有和不带有参数的Jekyll `{% tag %}`。

在工件`flexmark-ext-jekyll-tag`中使用类`JekyllTagExtension`。

:information_source:您可以处理include标记,并将这些文件的内容添加到
`INCLUDED_HTML`映射,以便呈现内容。如果包含的内容以
Markdown并可能包含包含文件的文档所使用的引用,然后
您可以使用以下方式将引用从包含的文档传输到包含的文档:
`Parser.transferReferences(Document, Document)`。看到:
[JekyllIncludeFileSample.java](https://github.com/vsch/flexmark-java/blob/master/flexmark-java-samples/src/com/vladsch/flexmark/java/samples/JekyllIncludeFileSample.java)

提供以下选项:

在`JekyllTagExtension`类中定义:

|静态字段|默认值|说明|
|:--------------------- |:------------------------- ---- |:-------------------------------------------- -------------------------------------------------- --------- |
| `ENABLE_INLINE_TAGS` | `true` |解析内联代码。|
| `ENABLE_BLOCK_TAGS` | `true` |解析块标签。|
| `ENABLE_RENDERING` | `false` |将标签呈现为文本|
| `INCLUDED_HTML` | `(Map<String, String>)null` |包含标记参数字符串到HTML内容的映射,用于替换呈现|中的标记元素
| `LIST_INCLUDES_ONLY` | `true` |如果为true,则仅将包含标签添加到TAG_LIST,否则添加所有标签|
| `TAG_LIST` | `new ArrayList<JekyllTag>()` |文档中所有jekyll标签的列表|

## Jira-Converter

允许将markdownAST渲染为JIRA格式的文本

在工件`flexmark-jira-converter`中使用类`JiraConverterExtension`。

没有定义选项。不支持JIRA格式的扩展将不会生成任何扩展
其相应节点的输出。

## Macros

宏定义是块元素,可以包含任何markdown元素,但可以是
在块或内联上下文中扩展,允许在仅内联的地方使用块元素
语法允许使用元素。[[Macros Extension]]

在工件`flexmark-ext-macros`中使用类`MacroExtension`。

## Media Tags

媒体链接转换器扩展由Cornelia Schultz(GitHub @CorneliaXaos)提供
使用自定义前缀链接到音频,嵌入,图片和视频HTML5标签。

* `!A[Text](link|orLinks)`-音频
* `!E[Text](link|orLinks)`-嵌入
* `!P[Text](link|orLinks)`-图片
* `!V[Text](link|orLinks)`-视频

在工件`flexmark-ext-media-tags`中使用类`MediaTagsExtension`。

没有定义选项。

## Spec Example

将flexmark扩展规范示例解析为示例节点,这些节点的详细信息已损坏并且许多
渲染选项。否则,这些解析为受限制的代码。[[Flexmark Spec Example Extension]]

[Markdown Navigator]插件将此扩展名用于JetBrains IDE,以简化与
flexmark-java测试规范文件。

## Superscript

通过将文本括在`^`中来启用ins标记文本。例如,在`hey ^you^`中,`you`将是
呈现为上标文本。

在工件`flexmark-ext-superscript`中使用类`SuperscriptExtension`。

提供以下选项:

在`SuperscriptExtension`类中定义:



* `SUPERSCRIPT_STYLE_HTML_CLOSE`:默认`(String)null`覆盖HTML以用于包装样式,如果为null,则不覆盖
* `SUPERSCRIPT_STYLE_HTML_OPEN`:默认`(String)null`覆盖HTML以用于包装样式,如果为null,则不覆盖


## Tables

像[GitHub Flavored Markdown][gfm-tables]中一样使用管道启用表。随着增加的选择
处理列跨度语法,具有多个标题行的能力,不同的列号
行之间等。[[Tables Extension]]

## Table of Contents

目录扩展实际上是两个扩展合而为一:`[TOC]`元素呈现一个
目录和模拟的`[TOC]:#`元素,该元素还呈现目录,但
也用于后处理程序,该后处理程序将在目录后附加目录。
生成带有目录元素的源文件,该目录将在任何减价时呈现
处理器。

在工件`flexmark-ext-toc`中使用类`TocExtension`或`SimTocExtension`。

有关详细信息,请参见[[Table of Contents Extension]]

## Typographic

将纯文本转换为印刷字符。[[Typographic Extension]]

* `'`到撇号`&rsquo;`&rsquo;
* `...`和`. . .`省略号`&hellip;`&hellip;
* `-- `破折号`&ndash;`&ndash;
* `--- ` em破折号`&mdash;`&mdash;
*单引号`'some text'`至`&lsquo;some text&rsquo;`&quot;某些文本&rsquo;
*双引号`"some text"`至`&ldquo;some text&rdquo;`"一些文字"
*双角引用`<<some text>>`至`&laquo;some text&raquo;`&laquo;一些文本&raquo;

## WikiLinks

启用Wiki链接`[[page reference]]`和可选的Wiki图像`![[image reference]]`

要正确地将Wiki链接解析为URL,您很可能需要实现自定义链接
解析器来处理项目的逻辑。请参阅:
[PegdownCustomLinkResolverOptions](https://github.com/vsch/flexmark-java/blob/master/flexmark-java-samples/src/com/vladsch/flexmark/java/samples/PegdownCustomLinkResolverOptions.java)

在工件`flexmark-ext-wikilink`中使用类`WikiLinkExtension`。

提供以下选项:

在`WikiLinkExtension`类中定义:

|静态字段|默认值|说明|
|:------------------------ |:---------------------- --|:---------------------------------------------- -------------------------------------------------- -------------------------------------------------- ------ |
| `ALLOW_INLINES` | `false` |允许在使用`|`或包含的文本中合并文本和页面引用时在文本部分中进行定界符处理。|
| `DISABLE_RENDERING` | `false` |如果真实的Wiki链接呈现节点的文本,则禁用Wiki链接呈现。用于将WikiLink解析为AST,但不以HTML |呈现它们
| `IMAGE_LINKS` | `false` |启用Wiki图像链接格式`![[]]`与Wiki链接相同,但前缀为`!`,替代文本与Wiki链接文本相同,并受##影响1054 ## |
| `IMAGE_PREFIX` | `""` |要添加到生成的链接URL |的前缀
| `IMAGE_PREFIX_ABSOLUTE` |的值`IMAGE_PREFIX` |前缀添加到绝对Wiki链接的生成的链接URL中(从`/`开始)|
| `IMAGE_FILE_EXTENSION` | `""` |扩展到附加到生成的链接URL |
| `LINK_FIRST_SYNTAX` | `false` |当使用两部分语法`[[first|second]]`时,如果链接部分位于最前面,则为true,则为Creole语法;或false(当文本优先出现时),GFM语法|
| `LINK_PREFIX` | `""` |要添加到生成的链接URL |的前缀
| `LINK_PREFIX_ABSOLUTE` |的值`LINK_PREFIX` |前缀添加到绝对Wiki链接的生成的链接URL中(从`/`开始)|
| `LINK_FILE_EXTENSION` | `""` |扩展到附加到生成的链接URL |
| `LINK_ESCAPE_CHARS` | `" +/<>"` |要在页面引用中替换的字符,方法是将它们替换为`LINK_REPLACE_CHARS` |中的相应字符
| `LINK_REPLACE_CHARS` | `"-----"` |个字符通过替换`LINK_ESCAPE_CHARS` |中的相应字符在页面引用中替换

## XWiki Macro Extension

应用程序提供的格式为`{{macroName}}content{{/macroName}}`的宏或块宏
形成:

```markdown
{{macroName}}
content
{{/macroName}}
```

在工件`flexmark-ext-xwiki-macros`中使用类`MacroExtension`。

提供以下选项:

在`MacroExtension`类中定义:

|静态字段|默认值|说明|
|:----------------------- |:-------------- |:------- --------------------------------- |
| `ENABLE_INLINE_MACROS` | `true` |启用内联宏处理|
| `ENABLE_BLOCK_MACROS` | `true` |启用块宏处理|
| `ENABLE_RENDERING` | `false` |启用将宏节点呈现为文本|

## YAML front matter

通过YAML前题块添加了对元数据的支持。此扩展程序仅支持
YAML语法的子集。这是受支持的示例:

```markdown
---

key: value list: - value 1 - value 2 literal: | this is literal value.

literal values 2
---

document starts here
```

在工件`flexmark-ext-yaml-front-matter`中使用类`YamlFrontMatterExtension`。获取
元数据,请使用`YamlFrontMatterVisitor`。

## YouTrack-Converter

允许将Markdown AST呈现为YouTrack格式的文本

在工件`flexmark-youtrack-converter`中使用类`YouTrackConverterExtension`。

没有定义选项。不支持YouTrack格式的扩展程序将不会生成任何扩展名
其相应节点的输出。

## YouTube Embedded Link Transformer

YouTube链接转换器扩展程序由Vyacheslav N.Boyko(GitHub @ bvn13)提供
youtube视频到嵌入式视频iframe的简单链接。

在工件`flexmark-ext-youtube-embedded`中使用类`YouTubeLinkExtension`。

没有定义选项。

将指向youtube视频的简单链接转换为嵌入式视频HTML代码。

[Admonition Extension, Material for MkDocs]: https://squidfunk.github.io/mkdocs-material/extensions/admonition/
[autolink-java]: https://github.com/robinst/autolink-java
[docx4j]: https://www.docx4java.org/trac/docx4j
[DocxConverterCommonMark Sample]: https://github.com/vsch/flexmark-java/blob/master/flexmark-java-samples/src/com/vladsch/flexmark/java/samples/DocxConverterCommonMark.java
[DocxConverterPegdown Sample]: https://github.com/vsch/flexmark-java/blob/master/flexmark-java-samples/src/com/vladsch/flexmark/java/samples/DocxConverterPegdown.java
[Emoji-Cheat-Sheet.com]: https://www.webfx.com/tools/emoji-cheat-sheet
[gfm-tables]: https://help.github.com/articles/organizing-information-with-tables/
[Markdown Navigator]: http://vladsch.com/product/markdown-navigator
[Maven Central]: https://search.maven.org/#search|ga|1|g%3A%22com.vladsch.flexmark%22
[Open HTML To PDF]: https://github.com/danfickle/openhtmltopdf
[Php Markdown Extra Definition List]: https://michelf.ca/projects/php-markdown/extra/#def-list
[CommonMark]: https://commonmark.org
[commonmark.js]: https://github.com/jgm/commonmark.js
[Markdown]: https://daringfireball.net/projects/markdown/
[Semantic Versioning]: https://semver.org