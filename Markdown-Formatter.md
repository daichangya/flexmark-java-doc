
Formatter使用各种格式设置选项将AST呈现为markdown,以清理并制作
来源一致。它还附带了一个API，允许扩展提供格式化选项并处理自定义节点的标记呈现。
     
:information_source:在0.60.0之前的版本中,格式化程序功能已在
`flexmark-formatter`模块,并需要其他依赖项。

格式化程序模块还实现了用于将markdown文档翻译成另外一种语言的API:
[Translation Helper API](Translation-Helper-API)

`Formatter`类是一个渲染器,它输出markdown并将其格式化为指定的选项。
使用它代替`HtmlRenderer`获取格式化的markdown。它也可以用来转换
从一个`ParserEmulationProfile`到另一个的缩进:

```java
package com.vladsch.flexmark.samples;

import com.vladsch.flexmark.formatter.Formatter;
import com.vladsch.flexmark.parser.Parser;
import com.vladsch.flexmark.profile.pegdown.Extensions;
import com.vladsch.flexmark.profile.pegdown.PegdownOptionsAdapter;
import com.vladsch.flexmark.util.data.DataHolder;
import com.vladsch.flexmark.util.data.MutableDataSet;

public class PegdownToCommonMark {
     final private static DataHolder OPTIONS = PegdownOptionsAdapter.flexmarkOptions(
            Extensions.ALL
    );

    static final MutableDataSet FORMAT_OPTIONS = new MutableDataSet();
    static {
        //copy extensions from Pegdown compatible to Formatting, but leave the rest default
        FORMAT_OPTIONS.set(Parser.EXTENSIONS, Parser.EXTENSIONS.get(OPTIONS));
    }

    static final Parser PARSER = Parser.builder(OPTIONS).build();
    static final Formatter RENDERER = Formatter.builder(FORMAT_OPTIONS).build();

    //use the PARSER to parse pegdown indentation rules and RENDERER to render CommonMark

}
```

将pegdown 4空格缩进转换为CommonMark列表项文本列缩进。

##### Pegdown input

    #Heading
    -----
    paragraph text 
    lazy continuation
    
    * list item
        > block quote
        lazy continuation
    
    ~~~info
            with uneven indent
               with uneven indent
         indented code
    ~~~ 
    
            with uneven indent
               with uneven indent
         indented code
    1. numbered item 1   
    1. numbered item 2   
    1. numbered item 3   
        - bullet item 1   
        - bullet item 2   
        - bullet item 3   
            1. numbered sub-item 1   
            1. numbered sub-item 2   
            1. numbered sub-item 3   
        
        ~~~info
                with uneven indent
                   with uneven indent
             indented code
        ~~~ 
        
                with uneven indent
                   with uneven indent
             indented code

##### CommonMark output

转换为CommonMark缩进,添加了ATX标题空间,添加了空行,用了围栏和
缩进的代码缩进最小化:

    # Heading
    
    -----
    
    paragraph text
    lazy continuation
    
    * list item
    
      > block quote
      > lazy continuation
    
    ~~~info
       with uneven indent
          with uneven indent
    indented code
    ~~~
    
           with uneven indent
              with uneven indent
        indented code
    
    1. numbered item 1
    2. numbered item 2
    3. numbered item 3
       - bullet item 1
       - bullet item 2
       - bullet item 3
         1. numbered sub-item 1
         2. numbered sub-item 2
         3. numbered sub-item 3
    
       ~~~info
          with uneven indent
             with uneven indent
       indented code
       ~~~
    
              with uneven indent
                 with uneven indent
           indented code

获取完整的示例源[PegdownToCommonMark.java]。

## Options

这些是`Formatter`类中可用的选项。处理格式的扩展
他们的自定义节点可以并且确实提供了自己的格式设置选项。

这些在`Formatter`类中定义:

* `FORMATTER_EMULATION_PROFILE`:默认`Parser.PARSER_EMULATION_PROFILE`,仿真配置文件为
  用于格式化。可用于更改解析器使用的缩进规则。

* `MAX_BLANK_LINES`:默认`2`,文件中保留的最大空行数

* `MAX_TRAILING_BLANK_LINES`:默认`1`,文件中的最大尾随空白行

* `SPACE_AFTER_ATX_MARKER`:默认`DiscretionaryText.ADD`,在atx标记后处理空格

* `SETEXT_HEADER_EQUALIZE_MARKER`:默认值`true`,如果为true,则将setext标记等于
  标头文字长度

* `ATX_HEADER_TRAILING_MARKER`:默认`EqualizeTrailingMarker.AS_IS`,后跟atx `#`
  标记:
  * `AS_IS`:什么都不做
  * `ADD`:与开始标记添加相同数量的`#`
  * `EQUALIZE`:与开始标记添加相同数量的`#`
  * `REMOVE`:删除

* `THEMATIC_BREAK`:默认为`(String)null`,是用于主题中断的字符串。`null`意味着离开
  照原样。

* `BLOCK_QUOTE_BLANK_LINES`:默认`true`,在块引号周围添加空白行

* `BLOCK_QUOTE_MARKERS`:默认`BlockQuoteMarker.ADD_COMPACT_WITH_SPACE`
  * `AS_IS`:不变,第一行标记传播到完整的块引用内容
  * `ADD_COMPACT`:使用`>`和`>>..>>`来嵌套块引用
  * `ADD_COMPACT_WITH_SPACE`:使用`> `和`>>..>> `来嵌套块引用
  * `ADD_SPACED`:使用`> `和`> > ..> > `来嵌套块引用

* `INDENTED_CODE_MINIMIZE_INDENT`:默认`true`,如果为true,将删除常见的额外缩进
  所有内容行

* `FENCED_CODE_MINIMIZE_INDENT`:默认`true`,如果为true,将删除常见的额外缩进
  所有内容行

* `FENCED_CODE_MATCH_CLOSING_MARKER`:默认值`true`,当真正的开始标记用于
  结束标记

* `FENCED_CODE_SPACE_BEFORE_INFO`:默认值`false`,如果为true,则会在打开之间添加一个空格
  标记和信息字符串

* `FENCED_CODE_MARKER_LENGTH`:默认`3`,最小代码围栏标记长度

* `FENCED_CODE_MARKER_TYPE`:默认`CodeFenceMarker.ANY`,
  * `ANY`:不变,无论使用什么
  * `BACK_TICK`:更改为倒勾
  * `TILDE`:更改为`~`

* `LIST_ADD_BLANK_LINE_BEFORE`:默认值`false`,此时`true`将在
  第一个列表项(如果在段落之后)

* `LIST_RENUMBER_ITEMS`:默认值`true`,当`true`重新编号排序的列表项时

* `LIST_BULLET_MARKER`:默认`ListBulletMarker.ANY`,
  * `ANY`:不变
  * `DASH`:全部更改为`-`
  * `ASTERISK`:全部更改为`*`
  * `PLUS`:全部更改为`+`

* `LIST_NUMBERED_MARKER`:默认`ListNumberedMarker.ANY`,
  * `ANY`:不变
  * `DOT`:全部更改为`.`
  * `PAREN`:全部更改为`)`

* `LIST_SPACING`:默认`ListSpacing.AS_IS`,
  * `AS_IS`:不变
  * `LOOSEN`:如果有松散物品则松散
  * `TIGHTEN`:如果有紧的物品则紧
  * `LOOSE`:总是松动的
  * `TIGHT`:总是紧

* `REFERENCE_PLACEMENT`:默认`ElementPlacement.AS_IS`,
  * `AS_IS`:不变
  * `DOCUMENT_TOP`:将所有引用置于文档顶部
  * `GROUP_WITH_FIRST`:将所有内容与首次引用分组
  * `GROUP_WITH_LAST`:将所有与最后引用一起分组
  * `DOCUMENT_BOTTOM`:文档底部

* `REFERENCE_SORT`:默认`ElementPlacementSort.AS_IS`,仅在`REFERENCE_PLACEMENT`时适用
  不是`AS_IS`
  * `AS_IS`:不变
  * `SORT`:按参考文本按字母顺序排序
  * `SORT_UNUSED_LAST`:按参考文本按字母顺序排序,最后放置未参考的

* `KEEP_IMAGE_LINKS_AT_START`:默认值`false`,当`true`图片链接始终被包装时
  成为线上的第一个非空格

* `KEEP_EXPLICIT_LINKS_AT_START`:默认`false`,当`true`图片链接始终为
  包装成行上的第一个非空格

* `KEEP_HARD_LINE_BREAKS`:默认值`true`,当消除`false`硬线中断时,
  与他们的停工期。

* `KEEP_SOFT_LINE_BREAKS`:默认的`true`,消除了`false`软换行符后。允许
  为将软换行符视为硬换行符的处理器设置markdown格式。

* `APPEND_TRANSFERRED_REFERENCES`:默认`false`,当`true`追加传输时
  对要格式化的文档底部的引用。

* `SKIP_FENCED_CODE`,默认为`false`。当`true`将受防护的代码转换为缩进代码时
  产生的markdown。

* `SKIP_CHAR_ESCAPE`,默认为`false`。`true`不会转义特殊字符。

* `RIGHT_MARGIN`,默认为`0`,从0.60.0开始,如果> 0,则文本将换行到给定的边距。

* `APPLY_SPECIAL_LEAD_IN_HANDLERS`,默认为`true`,从0.60.0开始,如果为true,则将转义为特殊
  换行到行首的导入字符,转义从换行首的字符
  行。用于防止段落体内的特殊字符开始新的元素
  包装到行首时。

### 源代码示例

示例代码的最佳来源是实现自定义节点格式和
格式化程序模块本身:

* `flexmark-ext-abbreviation`
* `flexmark-ext-definition`
* `flexmark-ext-footnotes`
* `flexmark-ext-jekyll-front-matter`
* `flexmark-ext-tables`
* `flexmark-formatter`

[PegdownToCommonMark.java]: https://github.com/vsch/flexmark-java/blob/master/flexmark-java-samples/src/com/vladsch/flexmark/java/samples/PegdownToCommonMark.java