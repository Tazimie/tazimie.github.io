## markdown写论文

- 图表的交叉引用、脚注等；
- 表格（尤其是论文里需要使用的三线表）；
- 按章节自动编号；
- 数学公式；
- 目录；
- 分节、分页；
- 参考文献的引用（这个应该是最重要的，但也是最麻烦的，我们以后再提） 暂时我只想到了这么多（没按顺序来），要是还有没提到的欢迎大家进行补充。





1. 文献引用（Citation & Bibliography）；
2. 图表交叉引用（cross references）；
3. 章节编号。

### 目录

pandoc用于文件格式的转换

`pandoc --toc  --number-sections input.md -o output.docx`

不显示标题序号`#前言 {-}`

### 交叉应用

> 所谓交叉引用就是对其他位置内容的引用，例如标题、图、表等等。举个例子，在长长的教科书中，我们经常会见到如下内容

`如第三章所述，xxx研究xxx； 现以图5-1说明生物种群密度效应的类型； 以上几种动物的差异见表1-4.`

`pandoc --filter pandoc-crossref input.md -o output.docx`

#### 引用图片
![nature上偷来的图片](https://cos-1304139510.file.myqcloud.com//2021/%2009/%2015/41586_2021_3852_Fig1_HTML.png){#fid:id}



`[@fid:id]`

```

![nature上偷来的图片](https://cos-1304139510.file.myqcloud.com//2021/%2009/%2015/41586_2021_3852_Fig1_HTML.png){#fig:nature上偷来的图片}

巴拉巴拉巴拉巴拉，见[@fig:nature上偷来的图片]
```

#### 引用表格

```
| a    | b    | c    |
| ---- | ---- | ---- |
| 1    | 2    | 3    |
| 3    | 2    | 1    |

: 我是表 {#tbl:我是表}

吧啦吧啦吧啦，见[@tbl:我是表]
```
#### 引用其他
- 公式：使用{#eq:id}和[@eq:id]；
- 章节：使用{#sec:id}和[@sec:id]； 还有一些代码块、表样式等都是可以被引用的，不仅如此，图片这种可能涉及到子图的引用也可以实现，不过需要稍微懂一丢丢前端的知识，我不会就不介绍了，具体可参考pandoc-crossref的帮助文档。
https://link.zhihu.com/?target=https%3A//lierdakil.github.io/pandoc-crossref/


#### 图表按章节引用
> 默认情况下pandoc在过滤器的帮助下也只能形成fig.1 ... fig.12这种，可是长篇论文或者书中都是使用的fig.2-3这种按照章节引用的。这里我们需要在命令中添加参数-M chapters：
> `pandoc --filter pandoc-crossref -M chapters input.md -o output.docx`

#### 修改图标中的默认前缀

默认情况下，图片的名称前面会加上Figure，表格则是加上Table，正文中进行引用的时候会加上fig.和tbl.，要是不符合我们心意怎么办。 我们可以指定参数-M figureTitle="图"来修改图片中的前缀，-M figPrefix="图"来修改正文中引用的前缀；表格则是-M tableTitle="表"和-M tblPrefix="表"。 注意，这些参数搭配使用时每一个都需要加上-M，不能省略。

```
pandoc --filter pandoc-crossref \
-M chapters \
-M figureTitle="图" -M figPrefix="图" \
-M tableTitle="表" -M tblPrefix="表" \
input.md -o output.docx
```

#### 修改编号的格式
除了使用阿拉伯数字进行图表编号以外，pandoc-crossref过滤器还支持别的几种语言哦，只需要使用参数-M figLabels=arabic。 这里可操作修改的对象有：

figLabels：图片的前缀编号方案，默认为arabic；
eqLabels：公式的前缀编号方案，默认为arabic；
tblLabels：表格的前缀编号方案，默认为arabic；
secLabels：章节的前缀编号方案，默认为arabic； 可供修改的类型有：
arabic：阿拉伯数字
roman：罗马数字
lowercase roman：小写罗马数字 当然官方还提供了另外两种自定义的方法，但容易在不熟练的时候出现奇怪的结果，故此不推荐使用。

### YAML元信息

在pandoc中也采用了YAML用来配置markdown的一些元信息（metadata），对于大部分markdown编辑器来说可能不包含这一功能，我常用的Typora和Obsidian是有的，其实有没有这一项功能也无所谓，它就是利用---包裹并放在文章的开头就相当于声明了YAML信息。

我们来看markdown的#其实表示的是header，严格意义上来说是一级标题而不是文章标题title，放在论文里来说就是header 1可以是前言、方法、结果、讨论等，但不是论文的题目。如此一来我们就需要一个用来存放title的地方，这就是metadata的存在的意义，当然它不仅仅能容纳title，还可以加上author、keywords、abstract等信息。
```
---
title:  "This is the title: it contains a colon"
author:
- Author One
- Author Two
keywords: [nothing, nothingness]
abstract: |
  This is the abstract.

  It consists of two paragraphs.
---

# 前言
# 材料与方法
## 材料 
```

#### 高效使用

YAML是用来设置metadata的，不知道大家有没有注意到，上一节里我们频繁使用的-M也是用来设置metadata的，这俩有联系吗？有的，其实YAML里的内容等价于-M或者--metadata这两个参数，因此，上一节中使用了多次的-M chapters等信息都可以放在YAML里。

```
pandoc --filter pandoc-crossref \
-M chapters \
-M figureTitle="图" -M figPrefix="图" \
-M tableTitle="表" -M tblPrefix="表" \
input.md -o output.docx

---
title:  "This is the title: it contains a colon"
author:
- Author One
- Author Two
keywords: [nothing, nothingness]
abstract: |
  This is the abstract.

  It consists of two paragraphs.
chapters: true
figureTitle: "图"
figPrefix: "图"
figLabels: roman
---

# 前言
# 材料与方法
## 材料 

![nature上偷来的图片](https://cos-1304139510.file.myqcloud.com//2021/%2009/%2015/41586_2021_3852_Fig1_HTML.png){#fig:nature上偷来的图片}

巴拉巴拉巴拉巴拉，见[@fig:nature上偷来的图片]
```

`pandoc -F pandoc-crossref input.md -o output.docx`

```
---
chapters: true
figureTitle: "图"
figPrefix: "图"
figLabels: roman
---
```

`pandoc -F pandoc-crossref --metadata-file metadata.yaml input.md -o output.docx`



### 引用文献
`pandoc --citeproc --bibliography=test.bib -M reference-section-title="参考文献" input.md -o output.docx`



- `--citeproc`：表示此时pandoc会开始对文档中的引文进行渲染了，也可以使用`-C`代替；
- `--bibliography`：该参数指定`.bib`文件的位置，因为这里的`test.bib`就在工作路径下，所以使用了相对路径；
- `-M reference-section-title`：为文末的引文书目添加的标题名字，如果不设置的话默认为`reference`

### **句末引用**

```text
1分钟等于60秒[@elton2001]。  
1分钟等于60秒[@cordovez2019;@cordovez2021]。  
1分钟等于60秒[@hanchangzhi2014;@hanchangzhi2019]。  
```

这里是最基础的句末引用，也是我们使用得最多的引用方式，多个引用之间用分号隔开。产生的结果如下：

```text
1分钟等于60秒(Elton 2001)。
1分钟等于60秒(Cordovez et al. 2019; Cordovez et al. 2021)。
1分钟等于60秒(韩长志 2014, 2019)。
```

大家应该注意到第二句和第三句结果上的差异了吧，**目前Pandoc只认为作者完全一致的情况下才会把把作者简化**，第二句中虽然第一作者是同一个人，但后续的作者不同，因此它无法简化作者。

### **句中引用**

```text
@elton2001 认为，1分钟等于60秒。
@hanchangzhi2014[@hanchangzhi2019] 认为1分钟等于60秒。
```

句中引用不需要加上`[]`方括号，特别需要注意的是，引用同一作者的不同文章时，使用的方式为：`@hanchangzhi2014[@hanchangzhi2019]`，而且在书写中文时，记得一个引用完成的ciation key和中文之间一定要有一个空格。 结果如下：

```text
Elton (2001) 认为，1分钟等于60秒。
韩长志 (2014, 2019) 认为1分钟等于60秒。
```

```text
pandoc --citeproc --bibliography=test.bib -M reference-section-title="参考文献" input.md -o output.docx
```

emm，还搞掉了一种引用方式，就是略去作者的方式，比如张三（2021）认为。。。这种方式，可以使用`[-@zhangsan2021]`的方式让结果只显示(2021)，具体效果大家自己试试。



### 引文格式



本篇接着上一期，针对输出的`docx`文件进行引文样式的调整。

总所周知，在论文里面最难调整的就是引文的参考格式了，所以我们才会借助各种个样的文献管理软件，因为它们往往都拥有调整引文样式的功能。而它们之所以能任意切换不同的引文样式是因为一种文件格式`.csl`。

这个`.csl`是citation-style-language/styles的缩写，它定义了引文的样式。在pandoc中，如果你只传入了`--citeproc`参数的话，默认的引文样式为`chicago-author-date`，也就是芝加哥著者年代制的样式。而不同的期刊，往往有着不同的要求。



在pandoc中使用`csl`样式很简单，使用参数`--csl`来指定你的`.csl`文件的路径，但先决条件是你必须先指定`--citeproc`参数来使得pandoc渲染引文。

为了方便，你可以把`.csl`格式的文件放在和`.md`等文件同一目录下，这样就可以直接用相对路径来指定，使得内容更加简洁。像我这里就专门创建了一个叫markdown教程的文件夹，把涉及到的内容都放在这里，这样比较直观。



```text
.
├── input.md
├── metadata.yaml
├── nature.csl
└── test.bib
```

- `input.md`：就是我们写内容的markdown本体；
- `metadata.yaml`：还记得metadata元信息吧，这个文件就是配置这些的；
- `test.bib`：存放我们所有参考文献信息的文件；
- `nature.csl`：这个就是一个配置引文样式的文件，它是nature的样式 所以我们只需要运行下面的命令，就可以得到一个`output.docx`：

`pandoc --citeproc --bibliography=test.bib --csl=nature.csl input.md -o output.docx`

这里我忘记加上参数`-M reference-section-title="参考文献"`了，所以在结尾的引文书目Bibliography处没有出现对应的标题。



## **如何获取csl文件？**

其实Zotero中内置了一些常用的`.csl`文件，我们可以通过在Zotero中的偏好设置里，找到「引用」，就可以看到内置的一些引文样式，而这些文件所在的位置为： `~/Zotero/styles`。
但是这里会有一个缺点，就是一旦涉及到中文和英文的双语，引文中的`et al.`和`等`，`and`和`和`只能出现一种，这是不管你用markdown还是直接在Word里使用文献管理工具都会遇到的问题，这个问题貌似没有比较好的解决办法，但我之前有一期提到过使用正则的思想来解决**[ 一日一技｜Word篇｜批量修改「et al.」与「等」 ](https://link.zhihu.com/?target=https%3A//mp.weixin.qq.com/s/cFabd2e3pad7tQu3NM7lzg)**。
文件内部的**样式**进行调整，包括段落之间的间距，各级标题和正文的字体字号、表标题、图标题的设置，还有表格的三线表样式等

```bash
pandoc test.md -o result.docx
```
```bash
pandoc --citeproc --bibliography=test.bib test.md -o result.docx
```
```bash
pandoc --citeproc --bibliography=test.bib -M reference-section-title="参考文献" test.md -o result.docx
```
```bash
pandoc --citeproc --bibliography=test.bib --pdf-engine=wkhtmltopdf test.md -o result.pdf
```
```bash
pandoc --print-default-data-file reference.docx > custom-reference.docx
```

使用 Markdown[^1]可以效率的书写文档, 直接转换成 HTML[^2], 你可以使用 Typora[^T] 编辑器进行书写。
[^1]:Markdown是一种纯文本标记语言
[^2]:HyperText Markup Language 超文本标记语言
[^T]:NEW WAY TO READ & WRITE MARKDOWN.