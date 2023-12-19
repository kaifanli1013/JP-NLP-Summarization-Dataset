# JP-NLP-Summarization-Dataset

**This Dataset is constructed from 自然言語処理学会 paper**

> Original Latex Corpus: [自然言語処理学会](https://www.anlp.jp/resource/journal_latex/index.html)

## Format of JP-NLP-Summarization

Each line is a **json format item** with

* file_name
* language
* title: japanese title
* etitle: english title
* jabstract
* eabstract
* section_name
* sections:
  * sec_intro
  * sec_method
  * sec_result
  * sec_conclusion
* absctract:
  * abs_intro
  * abs_method
  * abs_result
  * abs_conclusion

## 预处理和后处理

**基于原始TEX文件的字段提取:**

* file_name
* language
* title: japanese title
* etitle: english title
* jabstract
* eabstract
* section_name

**Preprocessing** **: 对section内部的文本进行了一些删除**

* 删除 `\begin{document}`之前的所有内容
* 删除了所有的图片：`\begin{figure} ... \end{figure}`之间的内容
  * 对图片引用进行了替换 `\ref{fig:algorithm}` 替换为図 `[fig:algorithm]`
* 删除了所有的表格：`\begin{table} ... \end{table}` 和 `\begin{tabular} ... \end{tabular}`
  * 对表格的引用进行了替换: 表 `\ref{tab:fail}` 替换为表 `[tab:fail]`
* 对文献引用进行了替换：
  * 统一替换为@xcite
* 对公式进行了替换：
  * 单行公式$xxx$内的内容替换为@xmath0, @xmath1, @xmath2, ...
  * 多行公式

    $$
    (xxx)
    $$

    内的内容替换为@xlmath

**基于pandoc把所有tex文件转换成了txt**

**Postprocessing: 对没有删除干净的特殊符号进行删除**

* 首先删除换行符 \n
* 删除::: 开头的一些文本
  * ::: sent
  * ::: center
  * ::: table*
  * ::: flashleft
  * ::: flashright
  * ::: exe
  * ::: algorithm(ic)
  * ::: indent
  * ::: xlist
  * ::: lingexample
  * ::: figure*
  * ::: ex
  * ::: exB
  * ::: description
  * ::: itembox
  * ::: breakbox
  * ::: df
  * ::: enumerate
  * ::: savenotes
  * ::: dependency
  * ::: algorithm
  * ::: algorithmic
  * ::: 簡体中文
* 删除{.underline}和\underline
* 删除\begin{enumerate} \end{enumerate}
  * 删除\item
* 更换网址为@url

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

1. 新建一个列来存储所有的sections
2. 新建一个列来存储abs_incremental
