
:warning:这是一场头脑风暴,正在进行中。任何反馈和建议将是
非常感激。

YAML前件配置允许将markdown文件中的所有模块选项配置为
相对于对选项/扩展名进行硬编码。此功能始于
版本0.6.0,只需最少的用户代码协作即可启用此强大功能。

使用YAML前件配置器可以使Markdown内容的保存更加容易,因为该文件
定义文档期望的解析和渲染选项,以便正确解析
并渲染其内容。

将`Parser.FLEXMARK_YAML_CONFIGURATOR`设置为`true`以启用YAML配置。

所有支持此设置的核心模块都将使用其解析器,渲染器,
转换器基于从解析的实际文本中获取的设置或具有Yaml前置字符的设置
文件AST中包含弹性标记配置设置的节点。

这些设置将应用到用于创建核心模块解析器,渲染器的选项上
转换器或扩展程序。

:warning:强烈建议通过以下方式实现对YAML前端配置的过滤:
将`Parser.FLEXMARK_YAML_FILTER`设置为您的过滤器实现。在过滤器中
禁止/自定义请求的设置以控制您允许用户修改的内容
您配置的默认设置中的文档。

* [ ] 配置以及适用的扩展/模块
  *核心模块:
    * [ ] 解析器选项:`parser`
    * [ ] HtmlRender选项:`html`
    * [ ] Formatter渲染选项:`formatter`
  *转换器:
    * [ ] docx转换器选项:`docx`
    * [ ] PDF转换器选项:`pdf`
    * [ ] html2md转换器选项:`html2md`
    * [ ] 吉拉转换器选项:`jira`
    * [ ] YouTrack转换器选项:`youtrack`
  * [ ] 钉住配置文件选项:`pegdown`
  *不带`flexmark-ext-`前缀的扩展名
    * [ ] `abbreviation`
    * [ ] `admonition`
    * [ ] `anchorlink`
    * [ ] `aside`
    * [ ] `attributes`
    * [ ] `autolink`
    * [ ] `definition`
    * [ ] `emoji`
    * [ ] `enumerated-reference`
    * [ ] `escaped-character`
    * [ ] `footnotes`
    * [ ] `gfm-issues`
    * [ ] `gfm-strikethrough`
    * [ ] `gfm-tasklist`
    * [ ] `gfm-users`
    * [ ] `gitlab`
    * [ ] `ins`
    * [ ] `jekyll-tag`
    * [ ] `macros`
    * [ ] `media-tags`
    * [ ] `spec-example`
    * [ ] `superscript`
    * [ ] `tables`
    * [ ] `toc`
    * [ ] `typographic`
    * [ ] `wikilink`
    * [ ] `youtube-embedded`
  *每个人都可以拥有扩展yaml配置支持所提供的选项图
    功能`Map<String, YamlOption> getYamlOptionsMap()`
    * `getYamlOptionsMap()`仅返回可以通过以下方式设置的值:
      *布尔值
      * int
      *浮动
      *字符串
      *枚举-字符串标识符映射
      *以上所有数组
      *上面任何键的映射,但数组映射到包括映射的任何值的数组
      * `YamlOption`提供名称,类型信息和设置器以处理实际设置
        `MutableDataSet`中来自YAML值的选项用于处理文档。

  ```yaml
  flexmark:
    - module:
      - option: ...  
  ```
* [ ] 添加:YAML配置选项和核心模块以通用的可重用方式进行处理,因此
      模块可以轻松使用核心功能。
* [ ] 添加:用于流的快速YAML预解析器,可以快速确定是否需要新的字符
      基于配置的选项。
* [ ] 添加:yaml配置过滤
  * `DataSet YamlConfigFilter(String module, DataSet options)`模块是
    配置和选项包含yaml配置中设置的所有选项。返回
    您要使用的配置;如果不应为此模块配置任何mod,则为null
    用过的。
* [ ] 规格示例配置:
  * 部分显示语言和ast位置信息,以便可以在markdown中正确映射这些信息
    导航 如果未提供,则默认为1-Markdown,2-Html,3-Ast(代表1)。
    * 节类型是该节中包含的任何文本指定语言,默认为
      `text`
    * ast类型对Markdown Navigator具有特殊含义,并将查找位置信息
      在节点中,并突出显示由`for`键指定的部分中的选定范围,默认情况下
      `for: 1`表示第1节中文本的位置信息。
  * [ ] 可见字符映射:`spc`,`tab`,`cr`,`lf`,`intellij`

  例如,当前的实现默认设置将具有以下配置:

  ```yaml
  flexmark:
    - spec-example:
      sections:
        1: markdown
        2: html
        3: ast
           for: 1
      visible-chars:
        spc: ""
        tab: "→"
        cr: "⏎"
        lf: ""
        intellij: "⎮"
  ```