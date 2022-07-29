`pandoc -F pandoc-plot -F pandoc-crossref  --citeproc --metadata-file metadata.yaml -i Introduction.md Methods.md Results.md Discussion.md -o output.docx`



`pandoc --print-default-data-file reference.docx > default-reference.docx`





这样会在当前工作路径下生成一个名叫`default-reference.docx`的文件。然后你针对这个模版文件内部的**样式**进行调整，包括段落之间的间距，各级标题和正文的字体字号、表标题、图标题的设置，还有表格的三线表样式等，一一修改成你想要的格式之后，保存，然后把pandoc命令修改如下：



`pandoc -F pandoc-plot -F pandoc-crossref  --citeproc --metadata-file metadata.yaml -i Introduction.md Methods.md Results.md Discussion.md -o output.docx --reference-doc=default-reference.docx`