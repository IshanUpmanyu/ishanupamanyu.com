---
title: 'Apache Lucene 7 concepts that will help you get started'
date: '2021-11-30T13:06:00+05:30'
status: publish
url: /blog/apache-lucene-7-concepts-that-will-help-you-get-started
author: 'Ishan Upamanyu'
description: 'These 7 concepts will help you get started with Apache Lucene. This covers concepts like inverted index, analysis, documents, fields, IndexWriter, IndexSearcher and QueryParser'
type: post
id: 572
image: /uploads/2022/01/Apache-Lucene-1-e1648997525598.jpeg
tags:
    - 'Apache Lucene'
---
Apache Lucene is a popular information retrieval library written in JAVA. It helps you add full-text search capability to your application. This library is at the heart of search servers like Elasticsearch and Apache Solr.

It was created in 1999 by [Doug Cutting](https://en.wikipedia.org/wiki/Doug_Cutting), who also created Apache Hadoop, and is now [a top-level Apache Project.](https://lucene.apache.org/)

You can use Apache Lucene to create your own search application which can include features like term matching, phrase matching, fuzzy search, proximity search, and much more.

If you have been working with Elasticsearch or Apache Solr, then understanding Apache Lucene will help you to grasp how these technologies work.

To help you begin your learning journey, here are 7 concepts that will get you familiar with Apache Lucene.

Inverted Index – The reason Lucene is so fast
---------------------------------------------

Lucene enables super-fast text search by using a special data structure known as the [inverted index](/blog/inverted-index-data-structure?utm_source=lucene-basic-concepts&utm_medium=www.ishanupamanyu.com&utm_campaign=internal-linking). Instead of having to look for text in sequential order in all the documents, Lucene pre-creates an inverted index of the documents.

You can imagine this inverted index to be similar to the glossary you find at the end of a book. The glossary at end of a book contains references to the page numbers where a word occurred. Similarly, the inverted index contains a mapping of terms and a list of documents containing that term.

For example, consider the following documents:

- Document1: My name is Ishan Upamanyu
- Document2: Abhishek Upmanyu is a great comedian
- Document3: I would like to name my dog as Bruno
- Document4: Alexander the great.
- Document5: Abhishek is studying in Delhi.

If these documents are indexed in an inverted index, the index will look like this:

![](/uploads/2022/01/Apache-Lucene-Inverted-index.png "Apache Lucene creates an inverted index of the documents it indexes.") 

The inverted index has two components:

1. Dictionary: This contains all the terms in the index along with their frequency
2. Postings: This is the list of documents that contains the terms. In addition, the postings might also store information about where the word occurred in the document.

To understand how Lucene returns results using the inverted index, read the article on the inverted index [here](/blog/inverted-index-data-structure?utm_source=lucene-basic-concepts&utm_medium=www.ishanupamanyu.com&utm_campaign=internal-linking), followed by the article on how scoring works [here](/blog/how-search-scoring-algorithm-works?utm_source=lucene-basic-concepts&utm_medium=www.ishanupamanyu.com&utm_campaign=internal-linking).

Fields and Documents – The building blocks of your search system
----------------------------------------------------------------

When you start working with Apache Lucene, you will come across two terms: Field and Document. These are the ways in which you describe your data to Lucene.

#### Fields

Fields are the most basic unit in Apache Lucene. You can consider a field as a column. You can store information about your data using fields. For e.g. let us say you are indexing novels written by multiple authors. You can have separate fields for Author Name, Title of the Novel, the publishing date, and so on.

A field is made of 3 parts:

1. Name: This is used to identify the field
2. Type: This describes the various properties of the field. It is a subclass of [IndexableFieldType](https://lucene.apache.org/core/7_3_1/core/org/apache/lucene/index/IndexableFieldType.html)
3. Data: This is what the field holds. It can be a normal text, a number, or even a binary value.

To help you get started, Lucene has already created different types of fields. You can read about them in the [field documentation](https://lucene.apache.org/core/7_3_1/core/org/apache/lucene/document/Field.html).

Some of the available subclasses of Field are [BinaryDocValuesField](https://lucene.apache.org/core/8_1_1/core/org/apache/lucene/document/BinaryDocValuesField.html), [BinaryPoint](https://lucene.apache.org/core/8_1_1/core/org/apache/lucene/document/BinaryPoint.html), [DoubleDocValuesField](https://lucene.apache.org/core/8_1_1/core/org/apache/lucene/document/DoubleDocValuesField.html), [DoublePoint](https://lucene.apache.org/core/8_1_1/core/org/apache/lucene/document/DoublePoint.html), [DoubleRange](https://lucene.apache.org/core/8_1_1/core/org/apache/lucene/document/DoubleRange.html), [FeatureField](https://lucene.apache.org/core/8_1_1/core/org/apache/lucene/document/FeatureField.html), [Field](https://lucene.apache.org/core/8_1_1/core/org/apache/lucene/document/Field.html), [FloatDocValuesField](https://lucene.apache.org/core/8_1_1/core/org/apache/lucene/document/FloatDocValuesField.html), [FloatPoint](https://lucene.apache.org/core/8_1_1/core/org/apache/lucene/document/FloatPoint.html), [FloatRange](https://lucene.apache.org/core/8_1_1/core/org/apache/lucene/document/FloatRange.html), [IntPoint](https://lucene.apache.org/core/8_1_1/core/org/apache/lucene/document/IntPoint.html), [IntRange](https://lucene.apache.org/core/8_1_1/core/org/apache/lucene/document/IntRange.html), [LatLonDocValuesField](https://lucene.apache.org/core/8_1_1/core/org/apache/lucene/document/LatLonDocValuesField.html), [LatLonPoint](https://lucene.apache.org/core/8_1_1/core/org/apache/lucene/document/LatLonPoint.html), [LongPoint](https://lucene.apache.org/core/8_1_1/core/org/apache/lucene/document/LongPoint.html), [LongRange](https://lucene.apache.org/core/8_1_1/core/org/apache/lucene/document/LongRange.html), [NumericDocValuesField](https://lucene.apache.org/core/8_1_1/core/org/apache/lucene/document/NumericDocValuesField.html), [SortedDocValuesField](https://lucene.apache.org/core/8_1_1/core/org/apache/lucene/document/SortedDocValuesField.html), [SortedNumericDocValuesField](https://lucene.apache.org/core/8_1_1/core/org/apache/lucene/document/SortedNumericDocValuesField.html), [SortedSetDocValuesField](https://lucene.apache.org/core/8_1_1/core/org/apache/lucene/document/SortedSetDocValuesField.html), [StoredField](https://lucene.apache.org/core/8_1_1/core/org/apache/lucene/document/StoredField.html), [StoredFieldsWriter.MergeVisitor](https://lucene.apache.org/core/8_1_1/core/org/apache/lucene/codecs/StoredFieldsWriter.MergeVisitor.html), [StringField](https://lucene.apache.org/core/8_1_1/core/org/apache/lucene/document/StringField.html), and [TextField](https://lucene.apache.org/core/8_1_1/core/org/apache/lucene/document/TextField.html)

##### IndexableFieldTypes – the properties of fields

There are a few details you need to know when working with Fields. Unless explicitly specified, Lucene will not keep the original text of the field. If you want that it should return the original text of the field that matched your query, then you need to tell this explicitly to Lucene by using and implementable of `IndexableFieldType`. The commonly used subtype of this interface is `FieldType`  
The following methods of `FieldType` are noteworthy  
`public void setStored(boolean value)`: This method can be used to store the original text of the field.  
`public void setTokenized(boolean value)`: If tokenized is set to true only then the field will be analyzed by the `Analyzer`. Don’t worry we will cover the analysis concept shortly.  
`public void setIndexOptions(IndexOptions value)`: This method is used to control the information that should be stored in the inverted index. Check out the `IndexOptions` enum [here](https://lucene.apache.org/core/8_8_2/core/org/apache/lucene/index/IndexOptions.html).

Note that this can also be used to control advanced aspects like DocTypes and Term Vectors which are beyond the scope of this article.

You can check out other methods of `FieldType` class [here](https://lucene.apache.org/core/8_8_2/core/org/apache/lucene/document/FieldType.html).

#### Documents

A document is a collection of fields. You can consider it as a record in a database table. When you search for something, Lucene tries to find the document in which matching data is present. In our example when you indexed all novels, each novel record is a Document in Lucene.

So to index your data, you have to think about what fields each document will contain. You can describe this to Lucene using the following code:

To create a document use:

```
Document doc = new Document();
```

Now create a field. Here we will use an already created field class. The first constructor argument is the name of the field. The second argument is the content that we want to store. Ignore the last constructor argument, for now, you will understand this when we cover IndexableFieldType.

`Field pathField = new StringField("path", file.toString(), Field.Store.YES);`

Once you have a field, you need to add it to the document.

`doc.add(pathField);`

Analyzer – Preprocessing for indexed data and search queries
------------------------------------------------------------

##### What is Analysis?

We know that Lucene stores data in an inverted index. Remember that inverted index stores terms in its dictionary. But what we give to Lucene is plain text. So how does Lucene convert this text to terms of the inverted index? This is done by a process known as Analysis. The analysis is done both at indexing time for the fields of the document that is indexed, and also at the Query time for the query so that Lucene can find the terms it needs to look up in the inverted index.

**NOTE**: Lucene does not care about parsing your documents. Your document can be in any format be it PDF, DOC, EXCEL, etc. But Lucene does not understand these formats. Before you can index this data, you need to convert this into plain text format. This process is known as *parsing*. Lucene does not handle parsing. Parsing the data is the format of the application using Lucene.

##### Tokenization

The process of converting the text into smaller chunks ( known as Tokens) for adding in the inverted index is known as Tokenization. This can include various steps like removing extra spaces, removing stop words, converting the words to the root stems, lowercasing the tokens, etc. Some of these steps are done before converting the text to tokens, in which case these are PreTokenization steps. Others are done after we convert them to smaller tokens, in which case they are known as PostTokenization steps.

Pre-tokenization analysis can include, among other things, removing HTML markup, and converting or removing text matching arbitrary patterns or sets of fixed strings.

Many post-tokenization steps can also be done, including (but not limited to):

- [Stemming](http://en.wikipedia.org/wiki/Stemming) – Replacing words with their stems. For instance with English stemming “bikes” is replaced with “bike”; now query “bike” can find both documents containing “bike” and those containing “bikes”.
- [Stop Words Filtering](http://en.wikipedia.org/wiki/Stop_words) – Common words like “the”, “and” and “a” rarely add any value to a search. Removing them shrinks the index size and increases performance. It may also reduce some “noise” and actually improve search quality.
- [Text Normalization](http://en.wikipedia.org/wiki/Text_normalization) – Stripping accents and other character markings can make for better searching.
- [Synonym Expansion](http://en.wikipedia.org/wiki/Synonym) – Adding in synonyms at the same token position as the current word can mean better matching when users search with words in the synonym set.

##### Analyzer components

The analyzer performs its job, by chaining multiple components together. These components are as follows:

###### CharFilter

A char filter is used to perform PreTokenization steps like removing extra spaces or stripping HTML or any other conversion that you want to do to your text before converting it into tokens. Lucene provides some inbuilt CharFilter implementations like [CJKWidthCharFilter](https://javadoc.io/static/org.apache.lucene/lucene-analyzers-common/8.8.2/org/apache/lucene/analysis/cjk/CJKWidthCharFilter.html), [HTMLStripCharFilter](https://javadoc.io/static/org.apache.lucene/lucene-analyzers-common/8.8.2/org/apache/lucene/analysis/charfilter/HTMLStripCharFilter.html), [MappingCharFilter](https://javadoc.io/static/org.apache.lucene/lucene-analyzers-common/8.8.2/org/apache/lucene/analysis/charfilter/MappingCharFilter.html), and [PatternReplaceCharFilter](https://javadoc.io/static/org.apache.lucene/lucene-analyzers-common/8.8.2/org/apache/lucene/analysis/pattern/PatternReplaceCharFilter.html).

###### Tokenizer

A `Tokenizer` is what converts the text into individual tokens. It is a subclass of `TokenStream`. You can think of `TokenStream` as something which will help you get your tokens. In many cases, where `CharFilter` is not used, the `Tokenizer` is the first step of the analysis process.

###### TokenFilter

A `TokenFilter` is also a subclass of `TokenStream`. It is used to modify the tokens that are generated by the `Tokenizer`. Some common operations performed by a `TokenFilter` are deletion, stemming, synonym injection, and case folding.

##### Commonly used Lucene Analyzers

Here are some Analyzers that come built-in in Apache Lucene

[WhitespaceAnalyzer](https://javadoc.io/static/org.apache.lucene/lucene-analyzers-common/8.8.2/org/apache/lucene/analysis/core/WhitespaceAnalyzer.html): This breaks the text on WhiteSpaces. The logic for what is considered whitespace can be found [here](https://docs.oracle.com/javase/8/docs/api/java/lang/Character.html?is-external=true#isWhitespace-int-). This does not perform any case change.

[StopAnalyzer](https://javadoc.io/static/org.apache.lucene/lucene-analyzers-common/8.8.2/org/apache/lucene/analysis/core/StopAnalyzer.html): This analyzer can help you to remove stop words from the text. It breaks the text using `LetterTokenizer` which divides text at non-letters. Further, it lowercases the token using `LowerCaseFilter` and removes stop word provided using `StopFilter`

[StandardAnalyzer](https://lucene.apache.org/core/8_8_2/core/org/apache/lucene/analysis/standard/StandardAnalyzer.html): This analyzer tries to identify word boundaries as it uses `StandardTokenizer`. `StandardTokenizer` implements the Word Break rules from the Unicode Text Segmentation algorithm, as specified in [Unicode Standard Annex #29](https://unicode.org/reports/tr29/). It further uses `LowerCaseFilter` to convert tokens to lower case and removes stop words using `StopFilter`

There are many more analyzers that are already created by Lucene. They are part of the [analyzers-common API](https://javadoc.io/static/org.apache.lucene/lucene-analyzers-common/8.8.2/overview-summary.html).

Exploring the Lucene API
------------------------

Now that we know the basic concepts about Lucene, we are ready to write the code. In this section, we will explore few concepts that are related to the Lucene API.

### IndexWriter – Writing data to Lucene indexes

`IndexWriter` is the main class that is used to create an index in Lucene. It is used to add documents to the index. To create an `IndexWriter`, we need two components: a `Directory` and a `IndexWriterConfig`. Further, `IndexWriterConfig` is used to specify the analyzer that we want to use. Let’s use `StandardAnalyzer` for our example.

Here is a sample code. Here we create Document with a single Field. We want to return the complete value of this field when we return the results. So we create a stored field.

```java {class="my-class" id="my-codeblock" lineNos=inline tabWidth=2}
Analyzer analyzer = new StandardAnalyzer();

Path indexPath = Files.createTempDirectory("tempIndex");
Directory directory = FSDirectory.open(indexPath)
IndexWriterConfig config = new IndexWriterConfig(analyzer);
IndexWriter iwriter = new IndexWriter(directory, config);
Document doc = new Document();
String text = "This is the text to be indexed.";
doc.add(new Field("description", text, TextField.TYPE_STORED));
iwriter.addDocument(doc);
iwriter.close();

```

### QueryParser – Make Lucene understand what you want to search

Now that we have a document indexed, we will want to search it. But before we can do that, we need to make Lucene understand our query. This is done by `QueryParser`.

Let’s consider an example. Consider the simple query: “*path*“. This is a simple term query. This will try searching for the string “path” in the default field specified while creating the query. Lucene also supports searching by specifying a field explicitly. To do this you can issue a query in the format: &lt;fieldName&gt;:&lt;query term&gt;. eg: description: text. This will search for string “text” only in the description field. There are many other ways you can issue a query. The details of the Lucene query format can be seen [here](https://lucene.apache.org/core/8_0_0/queryparser/org/apache/lucene/queryparser/classic/package-summary.html#package.description).

The job of `QueryParser` is to understand this query and construct the `Query` class that will be needed to actually run this query. You can use the query parser as:

```java {class="my-class" id="my-codeblock" lineNos=inline tabWidth=2}
QueryParser parser = new QueryParser("description", analyzer);
Query query = parser.parse("text");

```

Note: In this example since we have not specified any field, it will search in the default field “description” only.

### IndexSearcher – Searching for documents in your index

Once you have a Query, you can use IndexSearcher to run the query. This is the class that actually runs the query against the Lucene indexes and it gives you results in a TopDocs object. This object contains the documents that matched your query in form of an array of ScoreDoc objects.

```java {class="my-class" id="my-codeblock" lineNos=inline tabWidth=2}
DirectoryReader ireader = DirectoryReader.open(directory);
IndexSearcher isearcher = new IndexSearcher(ireader);
ScoreDoc[] hits = isearcher.search(query, 10).scoreDocs;

```

The `IndexSearcher` needs a `DirectoryReader`. This should point to the same directory where you created the index.

To bring it all together, now our code looks like this:

```java {class="my-class" id="my-codeblock" lineNos=inline tabWidth=2}
 Analyzer analyzer = new StandardAnalyzer();

Path indexPath = Files.createTempDirectory("tempIndex");
Directory directory = FSDirectory.open(indexPath)
IndexWriterConfig config = new IndexWriterConfig(analyzer);
IndexWriter iwriter = new IndexWriter(directory, config);
Document doc = new Document();
String text = "This is the text to be indexed.";
doc.add(new Field("description", text, TextField.TYPE_STORED));
iwriter.addDocument(doc);
iwriter.close();


// Parse a simple query that searches for "text":
QueryParser parser = new QueryParser("description", analyzer);
Query query = parser.parse("text");

// Now search the index:
DirectoryReader ireader = DirectoryReader.open(directory);
IndexSearcher isearcher = new IndexSearcher(ireader);

ScoreDoc[] hits = isearcher.search(query, 10).scoreDocs;
assertEquals(1, hits.length);

// Iterate through the results:
for (int i = 0; i < hits.length; i++) {
  Document hitDoc = isearcher.doc(hits[i].doc);
  assertEquals("This is the text to be indexed.", hitDoc.get("fieldname"));
}
ireader.close();
directory.close();
IOUtils.rm(indexPath);

```

These were all the concepts that you needed to know to get started using Apache Lucene. You are now ready to start using Apache Lucene. If you want a fully functional java code example, try running a demo as explained [here](https://lucene.apache.org/core/8_3_0/demo/index.html). Now go get your hands dirty and write some code!