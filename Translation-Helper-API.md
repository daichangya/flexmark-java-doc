Formatter兼作翻译助手,可帮助提取可翻译文本范围
从文档中删除,并将非翻译文本替换为标识符,以使它们不会
由翻译更改。

:information_source:在0.60.0之前的版本中,格式化程序功能已在
`flexmark-formatter`模块,并需要其他依赖项。

提取文本的过程和格式的假设是翻译过程
不会更改由`*~()[]{}<>#`个字符组成的标记元素。非翻译文字
被占位符文本`_#_`代替,其中`#`是用于标识原始字符的整数
占位符的文本。

使用[Yandex.Translate](http://translate.yandex.com/)测试了翻译过程
在翻译过程中保留markdown标记方面做得非常好。

## Translation process

处理翻译的过程分为几个步骤:

1.解析文档以获取markdownAST,这是正常的`flexmark-java`markdown解析。

2.格式化文档以获取markdown字符串以进行翻译,步骤1中的文档节点为
   用于设置为`RenderPurpose.TRANSLATED_SPANS`的目的

3.从翻译处理程序中获取要翻译的字符串。

4.通过首选翻译服务翻译字符串。

5.在翻译处理程序中设置翻译后的字符串。

6.使用占位符为非翻译字符串和上下文生成标记
   翻译,使用步骤1中的文档节点,目的是将
   `RenderPurpose.TRANSLATED_SPANS`

7.使用占位符解析文档。这是正常的`flexmark-java`markdown解析
   从步骤6返回的文档文本。

8.生成最终翻译后的markdown,所有非翻译占位符均替换为
   原始文本和翻译占位符,按其翻译后的文本,步骤中的文档节点
   7的用途是设置为`RenderPurpose.TRANSLATED`。

提取的文本运行分为三种不同类型:

1.翻译跨度-这些是段落,标题文本,表格单元格和
   可以包含嵌入式文本元素的文本:粗体,斜体和其他自定义元素,例如
   删除线,插入线,删除线等。内联代码元素被排除在外并被视为
   非翻译,按原样保留其文本。
2.非翻译摘录-这些都是不应翻译的文本,例如:链接
   URI,标识符,html块,内联html标签等。
3.翻译片段-这些是其他元素的文本部分,例如链接文本,图片替换,
   参考链接ID,参考定义ID。此可翻译文本翻译为
   容器文本上下文之外的单独元素。

例如:

```markdown
Paragraph text with embedded link [Example Link](http://example.com) in it.
```

尽管链接文本显示在翻译文本范围内,但不应将其翻译为
部分原因是因为翻译人员可能会错误地使用其上下文来更改翻译。的
出现在不同文本上下文中的相同元素将导致不同的翻译。

为了消除这种影响,将在以下段落中替换文字`Example Link`
由其占位符`_1_`进行翻译,其文本作为单独的可翻译字符串提供。
非翻译网址将被`_2_`占位符替换,并从翻译中排除
文字清单。

在此示例中,提取的翻译文本字符串将为:

1. `Example Link`
2. `Paragraph text with embedded link [_1_](_2_) in it.`

如果以下示例翻译提供给翻译处理程序:

1. `eXaAmpLeE liINK`
2. `paARaAGRaAph teEXt WiIth eEmBeEDDeED LiINK [_1_](_2_) iIN iIt.`

在翻译过程的第6步生成文档将导致:

```markdown
paARaAGRaAph teEXt WiIth eEmBeEDDeED LiINK [_1_](_2_) iIN iIt.
```

在步骤7中解析此markdown文本并使用占位符生成最终文档
在步骤8中进行替换将产生翻译后的文档:

```markdown
paARaAGRaAph teEXt WiIth eEmBeEDDeED LiINK [eXaAmpLeE liINK](http://example.com) iIN iIt.
```

`flexmark-java-samples`模块中包含翻译器用法示例
[TranslationSample.java]

## Implementation Details

`Formatter.translationRender()`方法提供翻译帮助,这些方法采用
与`Formatter.render()`相同的参数,还有两个附加参数:`TranslationHandler,
RenderPurpose`。

`RenderPurpose`设置翻译渲染的目的:

* `RenderPurpose.FORMAT`-常规格式,与`Formatter.render`方法相同
* `RenderPurpose.TRANSLATION_SPANS`-从文档中提取翻译文本范围,并
  识别非翻译文本范围
* `RenderPurpose.TRANSLATED_SPANS`-将翻译文本范围替换为已翻译
  相应的文本。
* `RenderPurpose.TRANSLATED`-将占位符文本替换为翻译后的文本或原始文本
  取决于占位符。

`TranslationHandler`提供用于跟踪翻译范围和非翻译范围的功能,
渲染器调用之间的信息存储。默认实现可以是
定制或完全替换。

翻译过程中的困难是要确保带有占位符的中间文本
产生的文字将被识别为原始markdown元素,从而产生了
占位符。为此,将解析器修改为将占位符识别为有效元素。

例如,HTML块元素被替换为单个`<___#_>`,其中`#`是整数
占位符序数位置。通常,这不是有效的HTML块标记,但出于以下目的
翻译解析器将识别出它本身。同样,内联HTML元素被替换
`<__#_>`和自动链接URL与`<____#_>`。

其他注意事项,包括引用块元素ID及其适当的引用
markdown解析需要具有匹配的占位符,否则它们将无法正确解析
并不会为替换占位符产生所需的AST。

一个这样的警告涉及锚定引用,这些引用是指标题元素,并且由
标题文字。通过格式化程序进行的翻译过程将替换所有锚点
指向具有相同锚定引用的同一文档中标题的链接中的引用,生成了
翻译的标题文字。

参考一致性最复杂的处理方式是
[EnumeratedReferenceNodeFormatter.java]和[AttributesNodeFormatter.java]分别列举
引用包含两个部分`category:id`,这两个部分需要保持一致,因为
引用的`category`部分也可以不带`id`部分使用。

为了帮助自定义占位符格式,解析器和
排除非翻译文字摘录,以下选项可在
`Formatter`

* `TRANSLATION_ID_FORMAT`,默认`"_%d_"`,用于String.format(format,
  placeholderId)将整数ID转换为文本占位符。
* `TRANSLATION_HTML_BLOCK_PREFIX`,默认为`"__"`,以占位符文本开头的字符为
  将HTML块标记与HTML内联块标记和其他非翻译文本区分开来
  在翻译格式化期间。
* `TRANSLATION_HTML_INLINE_PREFIX`,默认为`"_"`,以占位符文本开头的字符为
  将HTML内联标签与HTML块标签和其他非翻译文本区分开来
  在翻译格式化期间。
* `TRANSLATION_AUTOLINK_PREFIX`,默认`"_"`,以占位符文本开头的字符
  区分自动链接占位符标记和HTML块标记以及其他
  翻译格式设置期间非翻译文本。
* `TRANSLATION_EXCLUDE_PATTERN`,默认`"^[^\\p{IsAlphabetic}]*$"`,模式以排除任何
  翻译与模式匹配的字符串。默认值将排除任何不包含的内容
  任何unicode字母字符组。
* `TRANSLATION_HTML_BLOCK_TAG_PATTERN`,默认为`"___(?:\\d+)_"`,解析器模式用于
  识别包含翻译占位符的HTML块标记。
* `TRANSLATION_HTML_INLINE_TAG_PATTERN`,默认为`"__(?:\\d+)_"`,解析器模式用于
  识别包含翻译占位符的HTML块标记。

### 自定义元素翻译

不包含标识符或非翻译文本的自定义元素无需更改,因为
默认情况下,所有文本节点均视为翻译范围。

自定义元素中的所有文本元素和引用标识符都需要实现
`NodeFormatter`通过使用翻译API方法进行渲染来处理节点渲染
在`MarkdownWriter`中实现的文本,用于附加格式化的markdown促销。

* `MarkdownWriter.appendNonTranslating(CharSequence)`-将呈现非翻译文本
  片段,取决于渲染目的,它要么将文本替换为
  占位符,采用一个占位符并按原样传递它,或将占位符替换为
  原文。
* `MarkdownWriter.appendTranslating(CharSequence)`-将呈现翻译文本片段,
  根据呈现的目的,要么使用占位符替换文本,
  占位符并按原样传递,或将其替换为已翻译
  文本。

翻译和非翻译文本范围的处理通过
`NodeFormatterContext`:

* `NodeFormatterContext.translatingSpan(TranslatingSpanRender)`-将处理所有渲染的文本
  由`TranslatingSpanRender`翻译。根据渲染目的将收集
  用于翻译的文本,将其替换为翻译后的文本,或直接将其传递给
  `MarkdownWriter`照原样。

* `NodeFormatterContext.nonTranslatingSpan(TranslatingSpanRender)`-将处理所有文本
  `TranslatingSpanRender`呈现为非翻译。取决于渲染目的
  将用占位符文本替换文本,将其替换为原始文本,或者简单地将其传递
  直达`MarkdownWriter`。

有关如何处理引用的示例,最好参考内核的实现
[CoreNodeFormatter.java]或扩展名中的元素:

* [AbbreviationNodeFormatter.java]
* [AdmonitionNodeFormatter.java]
* [AttributesNodeFormatter.java]
* [EmojiNodeFormatter.java]
* [EnumeratedReferenceNodeFormatter.java]
* [FootnoteNodeFormatter.java]
* [TableNodeFormatter.java]

[AbbreviationNodeFormatter.java]: https://github.com/vsch/flexmark-java/blob/master/flexmark-ext-abbreviation/src/main/java/com/vladsch/flexmark/ext/abbreviation/internal/AbbreviationNodeFormatter.java
[AdmonitionNodeFormatter.java]: https://github.com/vsch/flexmark-java/blob/master/flexmark-ext-admonition/src/main/java/com/vladsch/flexmark/ext/admonition/internal/AdmonitionNodeFormatter.java
[AttributesNodeFormatter.java]: https://github.com/vsch/flexmark-java/blob/master/flexmark-ext-attributes/src/main/java/com/vladsch/flexmark/ext/attributes/internal/AttributesNodeFormatter.java
[CoreNodeFormatter.java]: https://github.com/vsch/flexmark-java/blob/master/flexmark/src/main/java/com/vladsch/flexmark/formatter/internal/CoreNodeFormatter.java
[EmojiNodeFormatter.java]: https://github.com/vsch/flexmark-java/blob/master/flexmark-ext-emoji/src/main/java/com/vladsch/flexmark/ext/emoji/internal/EmojiNodeFormatter.java
[EnumeratedReferenceNodeFormatter.java]: https://github.com/vsch/flexmark-java/blob/master/flexmark-ext-enumerated-reference/src/main/java/com/vladsch/flexmark/ext/enumerated/reference/internal/EnumeratedReferenceNodeFormatter.java
[FootnoteNodeFormatter.java]: https://github.com/vsch/flexmark-java/blob/master/flexmark-ext-footnotes/src/main/java/com/vladsch/flexmark/ext/footnotes/internal/FootnoteNodeFormatter.java
[TableNodeFormatter.java]: https://github.com/vsch/flexmark-java/blob/master/flexmark-ext-tables/src/main/java/com/vladsch/flexmark/ext/tables/internal/TableNodeFormatter.java
[TranslationSample.java]: https://github.com/vsch/flexmark-java/blob/master/flexmark-java-samples/src/com/vladsch/flexmark/java/samples/TranslationSample.java