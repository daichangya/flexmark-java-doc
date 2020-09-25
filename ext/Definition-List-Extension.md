flexmark-java扩展,用于定义列表处理

### 概述

定义列表是具有一个或多个定义的一个或多个定义术语。

将[Php Markdown Extra Definition List]的定义语法转换为`<dl></dl>` HTML和
相应的AST节点。

定义项的前面可以是`:`或`~`,具体取决于配置的选项。

### 句法

```markdown
Definition Term
: Definition of above term
: Another definition of above term
```

如果定义术语不是定义之前的最后一个术语,则它们将没有定义:

```markdown
Definition Term without definition  
Definition Term  
: Definition of above term  
: Another definition of above term
```

定义术语和定义之间,或定义术语和上一个之间的空白行
定义术语会通过将其文本包装在`<p></p>`中来创建"松散"定义。

```markdown
Definition Term

: Loose Definition of above term  
: Tight definition of above term

: Another loose definition term
```

将创建:

```html
<dl>
  <dt>Definition Term</dt>
  <dd><p>Loose Definition of above term</p></dd>
  <dd>Tight definition of above term</dd>
  <dd><p>Another loose definition term</p></dd>
</dl>
```

支持定义的延迟延续,并且遵循定义的定义术语
必须用空白行与其隔开。

```markdown
Definition Term  
: Definition for Definition Term

Another Definition Term  
: Definition for Another Definition Term
```

术语和定义均允许内联减价。定义可以包含任何其他
markdown元素,只要这些元素按照列表缩进规则缩进即可。

多个定义术语不能用空行分隔,因为除最后一项以外的所有术语都是
视为段落文字:

```markdown
Not a definition term.

Neither is this.

Definition Term
Another Definition Term

: Definition
```

术语和定义均允许内联减价。定义可以包含任何其他
markdown元素,只要这些元素按照列表缩进规则缩进即可。

在可以进行内联处理之前,将段块行划分为定义项
内联标记从一行开始并在另一行结束。如果本段被转换
定义术语,则标记将被忽略,其开头标记归因于
定义术语及其结束标记。

这是跨定义术语拆分markdown内联的示例:

```markdown
Definition **Term
Another** Definition Term
: definition `item`
```

将产生以下HTML:

```html
<dl>
   <dt>Definition **Term</dt>
   <dt>Another** Definition Term</dt>
   <dd>definition <code>item</code></dd>
</dl>
```

### 解析详细信息 

使用工件`flexmark-ext-definition`中的类`DefinitionExtension`。

提供以下选项:

在`DefinitionExtension`类中定义:

|静态字段|默认值|说明|
| --------------- | --------------- | ----------------- -------------------------------------------------- -------------- |
| COLON_MARKER | `true` |启用将`:`用作定义项目标记|
| MARKER_SPACES | `1` |有效定义项目|的定义项目标记后的最小空格数
| TILDE_MARKER | `true` |允许使用`~`作为定义项目标记|

:information_source:此扩展使用列表解析和缩进规则,并将尽最大努力
根据所选选项对齐列表项和定义项的解析。



[Php Markdown Extra Definition List]: https://michelf.ca/projects/php-markdown/extra/#def-list