
Formatter兼作合并渲染器,以将多个markdown文档合并为一个
单个文档,同时尝试保留每个文档中的参考分辨率。

:information_source:在0.60.0之前的版本中,格式化程序功能已在
`flexmark-formatter`模块,并需要其他依赖项。

这是通过向创建冲突定义的所有引用添加数字序列来完成的
文件之间。

:information_source:默认情况下,`flexmark-ext-abbreviation`扩展元素不会更改
缩写和缩写定义使其具有唯一性,因为期望使用缩写
在文档中定义为相同的值,并添加数字后缀会更改
显示的文字。但是,可以通过设置更改此行为
使用的格式化程序选项中的`AbbreviationExtension.MAKE_MERGED_ABBREVIATIONS_UNIQUE`至`true`
合并文档。

### 相对链接分辨率

如果将`Formatter.DEFAULT_LINK_RESOLVER`属性设置为`true`,则doc relative和doc root
相对(以`/`开头的相对)将在每个文档中更改为前缀为的链接
在每个合并文档上设置`Formatter.DOC_RELATIVE_URL`或`Formatter.DOC_ROOT_URL`属性。
这允许将所有链接和图像引用映射为一个链接,该链接将解析为正确的
目标在合并文档中。这些属性应设置为以下路径的完整路径前缀:
文档或某些前缀,这将导致合并的文档链接正确解析。的
该值应该是什么的详细信息并不取决于该库,而是取决于链接分辨率
用于呈现合并文档及其文档相对和文档根URL属性。

### API

* `Formatter.mergeRender(Document[] documents, Appendable out, int maxBlankLines,
  HtmlIdGeneratorFactory idGeneratorFactory)`-是最通用的api,具有:
  * `documents`是要合并的文档列表。
  * `out`合并文档输出的附件
  * `maxBlankLines`合并后的每个文档后允许的最大空行数
    结果
  * `idGeneratorFactory`是HTML ID生成器工厂,用于生成标题ID。如果
    `null`然后将使用默认的HTML渲染器标头ID生成器。
* `void Formatter.mergeRender(Document[], Appendable, HtmlIdGeneratorFactory)`,将对`maxBlankLines`使用`void
  Formatter.MAX_TRAILING_BLANK_LINES`属性,其余如上
* `String Formatter.mergeRender(Document[], int, HtmlIdGeneratorFactory)`,将返回
  生成的合并文档的字符串值。

示例代码:[FormatterMergeSample.java]

### 冲突的标题ID

合并文档中的标题冲突将导致添加唯一的标题ID作为显式
属性,如果使用`flexmark-ext-attributes`扩展名。这将映射本地标题
对唯一ID的引用会导致文件中正确的标头引用。如果
不使用属性扩展,则不会生成唯一的ID,并且所有本地标题链接
将参考第一个匹配的标题。

合并以下文件:

```markdown
# Heading

[](#heading) 
```

```markdown
# Heading

[](#heading) 
```

将产生合并的markdown文件:

```markdown
# Heading

[](#heading) 

# Heading {#heading1}

[](#heading1) 
```

### 自定义参考元素合并

合并功能是使用格式化程序转换API实现的,无需实际更改
任何文本,仅引用。这意味着任何实现
使用`NodeRepositoryFormatter<>`基类的翻译呈现将具有合并功能。

有关如何处理引用的示例,最好参考内核的实现
[CoreNodeFormatter.java]或扩展名中的元素:

* [AbbreviationNodeFormatter.java]
* [AdmonitionNodeFormatter.java]
* [AttributesNodeFormatter.java]
* [EmojiNodeFormatter.java]
* [EnumeratedReferenceNodeFormatter.java]
* [FootnoteNodeFormatter.java]

[AbbreviationNodeFormatter.java]: https://github.com/vsch/flexmark-java/blob/master/flexmark-ext-abbreviation/src/main/java/com/vladsch/flexmark/ext/abbreviation/internal/AbbreviationNodeFormatter.java
[AdmonitionNodeFormatter.java]: https://github.com/vsch/flexmark-java/blob/master/flexmark-ext-admonition/src/main/java/com/vladsch/flexmark/ext/admonition/internal/AdmonitionNodeFormatter.java
[AttributesNodeFormatter.java]: https://github.com/vsch/flexmark-java/blob/master/flexmark-ext-attributes/src/main/java/com/vladsch/flexmark/ext/attributes/internal/AttributesNodeFormatter.java
[CoreNodeFormatter.java]: https://github.com/vsch/flexmark-java/blob/master/flexmark/src/main/java/com/vladsch/flexmark/formatter/internal/CoreNodeFormatter.java
[EmojiNodeFormatter.java]: https://github.com/vsch/flexmark-java/blob/master/flexmark-ext-emoji/src/main/java/com/vladsch/flexmark/ext/emoji/internal/EmojiNodeFormatter.java
[EnumeratedReferenceNodeFormatter.java]: https://github.com/vsch/flexmark-java/blob/master/flexmark-ext-enumerated-reference/src/main/java/com/vladsch/flexmark/ext/enumerated/reference/internal/EnumeratedReferenceNodeFormatter.java
[FootnoteNodeFormatter.java]: https://github.com/vsch/flexmark-java/blob/master/flexmark-ext-footnotes/src/main/java/com/vladsch/flexmark/ext/footnotes/internal/FootnoteNodeFormatter.java
[FormatterMergeSample.java]: https://github.com/vsch/flexmark-java/blob/master/flexmark-java-samples/src/com/vladsch/flexmark/java/samples/FormatterMergeSample.java