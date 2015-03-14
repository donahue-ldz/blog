title: "lucene基础"
date: 2015-03-13 20:29:02
categories: lucene
tags: [lucene，搜索]
---

##简介
Lucene 是一个基于 Java 的全文信息检索工具包，它不是一个完整的搜索应用程序，而是为你的应用程序提供索引和搜索功能。Lucene 目前是 Apache Jakarta 家族中的一个开源项目。也是目前最为流行的基于 Java 开源全文检索工具包。

![lucene结构](https://www.ibm.com/developerworks/cn/java/j-lo-lucene1/fig001.jpg)

##索引和搜索

索引和搜索
索引是现代搜索引擎的核心，建立索引的过程就是把源数据处理成非常方便查询的索引文件的过程。为什么索引这么重要呢，试想你现在要在大量的文档中搜索含有某个关键词的文档，那么如果不建立索引的话你就需要把这些文档顺序的读入内存，然后检查这个文章中是不是含有要查找的关键词，这样的话就会耗费非常多的时间，想想搜索引擎可是在毫秒级的时间内查找出要搜索的结果的。这就是由于建立了索引的原因，你可以把索引想象成这样一种数据结构，他能够使你快速的随机访问存储在索引中的关键词，进而找到该关键词所关联的文档。Lucene采用的是一种称为`反向索引（inverted index）的机制`。反向索引就是说我们维护了一个词 / 短语表，对于这个表中的每个词 / 短语，都有一个链表描述了有哪些文档包含了这个词 / 短语。这样在用户输入查询条件的时候，就能非常快的得到搜索结果
对文档建立好索引后，就可以在这些索引上面进行搜索了。搜索引擎首先会对搜索的关键词进行解析，然后再在建立好的索引上面进行查找，最终返回和用户输入的关键词相关联的文档。
##建立索引
### Document
Document 是用来描述文档的，这里的文档可以指一个 HTML 页面，一封电子邮件，或者是一个文本文件。一个 Document 对象由多个 Field 对象组成的。可以把一个 Document 对象想象成数据库中的一个记录，而每个 Field 对象就是记录的一个字段。
### Field
Field 对象是用来描述一个文档的某个属性的，比如一封电子邮件的标题和内容可以用两个 Field 对象分别描述。
### Analyzer
在一个文档被索引之前，首先需要对文档内容进行分词处理，这部分工作就是由 Analyzer 来做的。Analyzer类是一个抽象类，它有多个实现。针对不同的语言和应用需要选择适合的 Analyzer。Analyzer 把分词后的内容交给 IndexWriter来建立索引。
###IndexWriter
IndexWriter 是 Lucene 用来创建索引的一个核心的类，他的作用是把一个个的 Document 对象加到索引中来。
`IndexWriter indexWriter = new IndexWriter(indexDir,luceneAnalyzer,true); `

true表示新建，false表示原来的索引更新

### Directory
这个类代表了 Lucene 的索引的存储的位置，这是一个抽象类，它目前有两个实现，第一个是 FSDirectory，它表示一个存储在文件系统中的索引的位置。第二个是 RAMDirectory，它表示一个存储在内存当中的索引的位置。


##搜索

###Query
这是一个抽象类，他有多个实现，比如 TermQuery, BooleanQuery, PrefixQuery. 这个类的目的是把用户输入的查询字符串封装成 Lucene 能够识别的 Query。
###Term
Term 是搜索的基本单位，一个 Term 对象有两个 String 类型的域组成。生成一个 Term 对象可以有如下一条语句来完成：

`Term term = new Term(“fieldName”,”queryWord”);`

其中第一个参数代表了要在文档的哪一个Field上进行查找，第二个参数代表了要查询的关键词。

###TermQuery
TermQuery 是抽象类 Query 的一个子类，它同时也是 Lucene 支持的最为基本的一个查询类。生成一个 TermQuery 对象由如下语句完成：

`TermQuery termQuery = new TermQuery(new Term(“fieldName”,”queryWord”)); `


它的构造函数只接受一个参数，那就是一个 Term 对象。
###IndexSearcher
IndexSearcher 是用来在建立好的索引上进行搜索的。它只能以只读的方式打开一个索引，所以可以有多个 IndexSearcher 的实例在一个索引上进行操作,如。
``` 
FSDirectory directory = FSDirectory.getDirectory(indexDir,false); 
IndexSearcher searcher = new IndexSearcher(directory);

```
###Hits
Hits 是用来保存搜索的结果的。`hits.doc()`获得文档
```
Hits hits = searcher.search(luceneQuery); 
for(int i = 0; i < hits.length(); i++){ 
    Document document = hits.doc(i); 
    System.out.println(document.get("")); 
} 
```