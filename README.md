# JP-NLP-Summarization-Dataset

**This Dataset is constructed from 自然言語処理学会 paper**

> Original Latex Corpus: [自然言語処理学会](https://www.anlp.jp/resource/journal_latex/index.html)

## Format of JP-NLP-Summarization

Each line is a **json format item** with

* title: japanese title
* etitle: english title
* jabstract: japanese abstract
* eabstract: english abstract
* section_name
* sections

**Preprocessing** : 对section内部的文本进行了一些删除

1. 删除了 `\titel{}` 前的所有内容
2. 删除了 `\bibliographystyle `后的所有内容
3. 删除了所有的图片：`\begin{figure} ... \end{figure}`之间的内容

* 此外，对于 `\ref`引用了本figure的部分
* 进行如下操作：図 `\ref{fig:algorithm}` 替换为 図 `[fig:algorithm]`

4. 删除了所有的表格：`\begin{table} ... \end{table}` 和 `\begin{tabular} ... \end{tabular}`

* 此外，对于 `\ref`引用了本表格的部分
* 进行如下操作：表 `\ref{tab:fail}` 替换为 表 `[tab:fail]`

5. 替换掉所有的 `\ref{...}`引用为 `[...]`
6. 删除了所有的标签 `\label`

**Tex_to_csv** ：提取了除section部分之外的信息到csv文件内

## Format of JP-NLP-Summarization-Incremental

To fit our task: Incremental summarization

We did some sligt modification about abstract part

## 虽然我们采用四段式进行分割，但实际上很多文章的行文风格不适合用四段式来进行划分

intro的part包括了intro和先行研究

method方法主要是本paper的提案

result：如果考察在experiment章节就放在这里

conclusion：如果考察单独是一个章节就放在这里

删除了独立的案例分析和appendix章节

删除了只有标注的paper

例如关联研究往往会带来干扰，我们通常吧相关研究放在intro部分

# TODO

1. 删除每一列里的\n换行符
2. 如果考察单独是一个section那么就放到conclusion，否则下属于experiment的部分就不同
3. 删除双$$的公式
4. 删除::: center :::   ：：：table*
5. ::: flashleft flashright
6. 
7. 删除\
8. ：：：sent
9. ：：：cond
10. ：：：exe
11. ：：：algorithm（ic）
12. ::: iindent1zw
13. ：：：xlist
14. ::: lingexample
15. {.underline}
16. ：：：figure*
17. ::: 簡体中文
18. ：：：ex
19. ：：：exB
20. ：：：description
21. ：：：itembox
22. ：：：breakbox
23. 删除**强调
24. ：：：df
25. ：：：enumerate
26. ：：：screen
27. 删除空行
28. 删除{.underline} \underline
29. 处理 这样的数据\begin{itemize}
30. 删除了没有实验部分的文章和英文文章
31. 更换网站网址@url

::: savenotes

::: center

::: dependency

::: deptext

::: algorithm

::: algorithmic

:::

::: algorithm

::: algorithmic
