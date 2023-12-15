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


# TODO

1. 删除每一列里的\n换行符
