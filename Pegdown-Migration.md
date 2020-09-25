
`PegdownOptionsAdapter`类将pegdown `Extensions.*`标志转换为flexmark选项,并
扩展列表。为方便起见,还包括了挂钉`Extensions.java`。这些位于
`flexmark-profile-pegdown`模块已添加到Maven,但是您可以从中获取源代码
回购:[PegdownOptionsAdapter.java],如果需要,请[Extensions.java]并自己制作
版本,根据您的项目需求进行修改。

您可以将扩展标志传递给静态`PegdownOptionsAdapter.flexmarkOptions(int)`或
可以实例化`PegdownOptionsAdapter`并使用便捷方法设置,添加和删除
扩展标志。`PegdownOptionsAdapter.getFlexmarkOptions()`将返回的新副本
每次`DataHolder`带有反映pegdown扩展标志的选项。

```java
import com.vladsch.flexmark.html.HtmlRenderer;
import com.vladsch.flexmark.parser.Parser;
import com.vladsch.flexmark.profile.pegdown.Extensions;
import com.vladsch.flexmark.profile.pegdown.PegdownOptionsAdapter;
import com.vladsch.flexmark.util.data.DataHolder;

public class PegdownOptions {
     final private static DataHolder OPTIONS = PegdownOptionsAdapter.flexmarkOptions(
            Extensions.ALL
    );

    static final Parser PARSER = Parser.builder(OPTIONS).build();
    static final HtmlRenderer RENDERER = HtmlRenderer.builder(OPTIONS).build();

    //use the PARSER to parse and RENDERER to render with pegdown compatibility
}
```

## Strict pegdown HTML parsing Mode

默认的flexmark-java pegdown仿真使用不太严格的HTML块解析,这会中断
空白行上的HTML块。如果所有标签都在其中,则Pegdown仅在空白行上中断HTML块
HTML块已关闭。

为了更接近原始的钉住HTML区块解析行为,请使用需要
`boolean strictHtml`参数:

```java
import com.vladsch.flexmark.html.HtmlRenderer;
import com.vladsch.flexmark.parser.Parser;
import com.vladsch.flexmark.profile.pegdown.Extensions;
import com.vladsch.flexmark.profile.pegdown.PegdownOptionsAdapter;
import com.vladsch.flexmark.util.data.DataHolder;

public class PegdownOptions {
     final private static DataHolder OPTIONS = PegdownOptionsAdapter.flexmarkOptions(true,
            Extensions.ALL
    );

    static final Parser PARSER = Parser.builder(OPTIONS).build();
    static final HtmlRenderer RENDERER = HtmlRenderer.builder(OPTIONS).build();

    //use the PARSER to parse and RENDERER to render with pegdown compatibility
}
```

您可以找到更多的[Java Samples]可用。

### 通过PegdownOptionsAdapter可用的扩展

:information_source:[flexmark-java]的扩展名和配置选项比
[pegdown]除了pegdown 1.6.0中可用的扩展之外,以下扩展是
可用的:

* `SMARTS`:将`...` `. . .`,`--`和`---`美化为`…`,`…`,`–`和# #25 ##
* `QUOTES`:将单引号`'`,`"`,`<<`和`>>`修饰为`‘` `’` `‛`, `"` `"` `‟`,`«`
  和`»`
* `SMARTYPANTS`:便捷扩展功能一次启用`SMARTS`和`QUOTES`。
* `ABBREVIATIONS`:[PHP Markdown Extra]的缩写。
* `ANCHORLINKS`:通过获取字母数字的第一个范围来生成标题的锚链接
  和空格。
* `HARDWRAPS`:换行符的替代处理,请参阅[Github-flavoured-Markdown]
* `AUTOLINKS`:普通的,无限制的自动链接实现[Github-flavoured-Markdown]的方式。
* `TABLES`:与[MultiMarkdown]类似的表格(依次类似于
  [PHP Markdown Extra: tables]表,但具有colspan支持)。
* `DEFINITIONS`:定义以[PHP Markdown Extra: definition list]的方式列出。
* `FENCED_CODE_BLOCKS`:以[PHP Markdown Extra: fenced code]的方式围栏的代码块或
  [Github-flavoured-Markdown]。
* `SUPPRESS_HTML_BLOCKS`:禁止HTML块的输出。
* `SUPPRESS_INLINE_HTML`:禁止内联HTML元素的输出。
* `WIKILINKS`:通过可自定义的URL呈现逻辑支持`[[Wiki-style links]]`。
* `STRIKETHROUGH`:支持[Pandoc]中支持的`~~strikethroughs~~`和
  [Github-flavoured-Markdown]。
* `ATXHEADERSPACE`:根据需要,在`#`和标题标题文本之间需要有一个空格
  [Github-flavoured-Markdown]。释放`#`,而没有空格,只能是纯文本。
* `FORCELISTITEMPARA`:如果列表项或定义项包含更多内容,则将其包装在`<p>`标签中
  而不是简单的段落。
* `RELAXEDHRULES`:允许水平规则后面没有空白行。
* `TASKLISTITEMS`:解析格式为`* [ ]`,`* [x]`和`* [X]`的项目符号清单,以创建
  [Github-flavoured-Markdown]任务列表项。
* `EXTANCHORLINKS`:使用标题的完整内容为标题生成锚链接。
  *用`-`代替空格和非字母数字,将多个短划线修剪为一个。
  *锚链接添加为标头中具有空内容的第一个元素:`<h1><a
    name="header"></a>header</h1>`
* `EXTANCHORLINKS_WRAP`:与以上内容结合使用,以创建用于包装标头的锚点
  内容:`<h1><a name="header">header</a></h1>`
* `TOC`:用于启用目录扩展`[TOC]` TOC标签具有以下内容
  格式:`[TOC style]`。`style`由空格分隔的选项列表组成:
  * `levels=levelList`其中,级别列表是用逗号分隔的级别或范围列表。默认
    将包含标题级别2和3。示例:
    * `levels=4`包括级别2、3和4
    * `levels=2-4`包括级别2、3和4。与`levels=4`相同
    * `levels=2-4,5`包括级别2、3、4和5
    * `levels=1,3`包括1级和3级
  * `text`仅包含标题文本
  * `formatted`包括文本和内联格式
  * `bullet`将项目列表用于项目目录
  * `numbered`对目录项使用编号列表
  * `hierarchy`:标题的分层列表
  * `flat`:标题的固定列表
  * `reversed`:标题的倒排列表
  * `increasing`:扁平化,通过标题文本按字母顺序递增
  * `decreasing`:平,按标题文本按字母顺序减少
* `MULTI_LINE_IMAGE_URLS`:启用对跨越格式的一行以上的图片网址的解析
  严格`![alt text](urladdress?`必须是一行上的最后一个非空白段。的
  终止`)`或`"title")`必须是该行上的第一个非缩进段。一切
  除空白行外,它们之间的内容作为URL的一部分被吸收。
* `RELAXED_STRONG_EMPHASIS_RULES`:允许在没有开头的情况下以强/强调标记开头
  `_`的字母数字,并且只要不被`*`的空格包围,而不是仅当
  前面加空格。
* `SUBSCRIPT`:下标扩展名`~subscript~`
* `SUPERSCRIPT`:上标扩展名`^superscript^`
* `INSERTED`:插入或带下划线的扩展名`++inserted++`
* `FOOTNOTES`:支持MultiMarkdown样式脚注:`[^n] for footnote reference`和`[^n]:
  Footnote text`为脚注。其中`n`是一个或多个数字,字母,`-`,`_`或`.`。
  脚注将放在页面底部,并按出现顺序顺序编号
  脚注参考。未引用的脚注将不会包含在HTML中
  输出。

  ```markdown
  This paragraph has a footnote[^1] and another footnote[^two].

  This one has more but out of sequence[^4] and[^eight].

  [^two]: Footnote 2 with a bit more text
      and another continuation line

  [^1]: Footnote 1

  [^3]: Unused footnote, it will not be added to the end of the page.

  [^4]: Out of sequence footnote

  [^eight]: Have one that is used.
  ```

  将产生:

  ```html
  <div>
      <hr/>
      <p>This paragraph has a footnote<sup id="fnref-1"><a href="#fn-1" class="footnote-ref">1</a></sup> and another footnote<sup id="fnref-2"><a href="#fn-2" class="footnote-ref">2</a></sup>.</p>
      <p>This one has more but out of sequence<sup id="fnref-3"><a href="#fn-3" class="footnote-ref">3</a></sup> and<sup id="fnref-4"><a href="#fn-4" class="footnote-ref">4</a></sup>. </p>
      <hr/>
      <div class="footnotes">
         <ol style="list-style-type: decimal;">
             <li id="fn-1"><p>Footnote 1<a href="#fnref-1" class="footnote-backref">&#8617;</a></p></li>
             <li id="fn-2"><p>Footnote 2 with a bit more text  and another continuation line<a href="#fnref-2" class="footnote-backref">&#8617;</a></p></li>
             <li id="fn-3"><p>Out of sequence footnote<a href="#fnref-3" class="footnote-backref">&#8617;</a></p></li>
             <li id="fn-4"><p>Have one that is used.<a href="#fnref-4" class="footnote-backref">&#8617;</a></p></li>
         </ol>
      </div>
  </div>
  ```

  看起来像这样:

<div>
    <hr />
    <p>此段有一个脚注<sup id ="fnref-1"> <a href="#fn-1" class="footnote-ref"> 1 </a> </sup>和另一个脚注<sup id ="fnref-2"> <a href="#fn-2" class="footnote-ref"> 2 </a> </sup>。</p>
    <p>这个有更多但不完整的<sup id ="fnref-3"> <a href="#fn-3" class="footnote-ref"> 3 </a> </sup>和< sup id ="fnref-4"> <a href="#fn-4" class="footnote-ref"> 4 </a> </sup>。</p>
    <hr />
    <div class ="footnotes">
       <ol style ="list-style-type:decimal;">
           <li id ="fn-1"> <p>脚注1 <a href="#fnref-1" class="footnote-backref">&#8617; </a> </p> </li>
           <li id ="fn-2"> <p>脚注2,其中包含更多文本和另一行续行<a href="#fnref-2" class="footnote-backref">&#8617; </a> </p> </li>
           <li id ="fn-3"> <p>脚注乱序<a href="#fnref-3" class="footnote-backref">&#8617; </a> </p> </li >
           <li id ="fn-4"> <p>已使用一个。<a href="#fnref-4" class="footnote-backref">&#8617; </a> </p> </li>
       </ol>
    </div>
</div>

[Extensions.java]: https://github.com/vsch/flexmark-java/blob/master/flexmark-profile-pegdown/src/main/java/com/vladsch/flexmark/profile/pegdown/Extensions.java
[flexmark-java]: https://github.com/vsch/flexmark-java
[Github-flavoured-Markdown]: https://github.github.com/github-flavored-markdown
[MultiMarkdown]: https://fletcherpenney.net/multimarkdown
[Pandoc]: http://pandoc.org/MANUAL.html#pandocs-markdown
[pegdown]: http://pegdown.org
[PegdownOptionsAdapter.java]: https://github.com/vsch/flexmark-java/blob/master/flexmark-profile-pegdown/src/main/java/com/vladsch/flexmark/profile/pegdown/PegdownOptionsAdapter.java
[PHP Markdown Extra]: https://michelf.ca/projects/php-markdown/extra/#abbr
[PHP Markdown Extra: definition list]: https://michelf.ca/projects/php-markdown/extra/#def-list
[PHP Markdown Extra: fenced code]: https://michelf.ca/projects/php-markdown/extra/#fenced-code-blocks
[PHP Markdown Extra: tables]: https://michelf.ca/projects/php-markdown/extra/#table
[Java Samples]: https://github.com/vsch/flexmark-java/blob/master/flexmark-java-samples/src/com/vladsch/flexmark/java/samples