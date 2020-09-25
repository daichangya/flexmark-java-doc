flexmark-java扩展,用于脚注处理

### 概述

在文档中创建脚注引用。脚注位于渲染的底部
从脚注引用到脚注的链接,反之亦然。

### 句法

将`[^footnote]`转换为脚注引用,将`[^footnote]: footnote definition`转换为脚注
文献。

样品:

```markdown
Paragraph with a footnote reference[^1]

[^1]: Footnote text added at the bottom of the document
```

呈现为:

```html
<p><span class="selection-highlight">Paragraph with a footnote reference</span><sup id="fnref-1"><a class="footnote-ref" href="#fn-1">1</a></sup></p>
<div class="footnotes">
  <hr />
  <ol>
    <li id="fn-1">
      <p><span class="selection-highlight">Footnote text added at the bottom of the document</span></p>
      <a href="#fnref-1" class="footnote-backref">&#8617;</a>
    </li>
  </ol>
</div>
```

带有脚注的段落[^1]

[^1]:在文档底部添加了脚注文本

### 解析详细信息 

使用工件`flexmark-ext-footnotes`中的类`FootnoteExtension`。

提供以下选项:

在`FootnoteExtension`类中定义:

|静态字段|默认值|说明|
|:------------------------------- |:--------------- ---------- |:-------------------------------------- -------------------------------------------------- ------- |
| `FOOTNOTES` |新存储库|用于文档脚注定义的存储库|
| `FOOTNOTES_KEEP` | `KeepType.FIRST` |重复保存。|
| `FOOTNOTE_REF_PREFIX` | `""` |呈现脚注参考时使用的前缀。|
| `FOOTNOTE_REF_SUFFIX` | `""` |呈现脚注参考时使用的后缀。|
| `FOOTNOTE_BACK_REF_STRING` | `"&#8617;"` |用于在脚注上的链接文本的字符串返回到文本的链接,默认值:&#8617; |
| `FOOTNOTE_LINK_REF_CLASS` | `"footnote-ref"` |用于表示脚注参考的链接的class属性的字符串|
| `FOOTNOTE_BACK_LINK_REF_CLASS` | `"footnote-backref"` |用于链接的类属性的字符串,用于将链接呈现回脚注|
| `FOOTNOTE_PLACEMENT` | `ElementPlacement.AS_IS` |格式化选项,请参见:[Markdown Formatter](Markdown-Formatter) |
| `FOOTNOTE_SORT` | `ElementPlacement.AS_IS` |格式化选项,请参见:[Markdown Formatter](Markdown-Formatter) |