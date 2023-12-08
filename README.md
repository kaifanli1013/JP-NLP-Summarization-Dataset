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

Preprocessing : 对section内部的文本进行了一些删除

1. 删除了 `\label`{sec:...} 内的内容
2. 删除了 `\size{}`内的内容
3. 

## Format of JP-NLP-Summarization

To fit our task: Incremental summarization
We did some sligt modification about abstract part 



## 一些关于latex语法的记录
