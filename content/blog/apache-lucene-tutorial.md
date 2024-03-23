---
title: 'Apache Lucene Tutorial - A simple guide to getting started in 2022'
date: '2022-04-01T20:38:55+05:30'
status: publish
url: /blog/apache-lucene-tutorial
author: 'Ishan Upamanyu'
description: ''
type: post
id: 607
image: /uploads/2022/01/Apache-Lucene-1-e1648997525598.jpeg
tags:
    - 'Apache Lucene'
    - 'Information Retrieval'

---
Apache Lucene is a full-text search library written in Java. Lucene is a [top-level Apache Project.](https://lucene.apache.org/) and was written in 1999 by [Doug Cutting](https://en.wikipedia.org/wiki/Doug_Cutting).

This is the only Apache Lucene tutorial you will need to get started with Lucene in 2022. We will first go over the basic concepts of Apache Lucene. Then we will try and explore the Lucene API. At last, we will end this tutorial by building a search application that indexes and searches files on our local file system. We will be using Lucene 8 in this tutorial which is the current version of Lucene as of early 2022.

Introduction
------------

### What is Lucene?

Lucene is a full-text search library written in Java. It is a powerful library that forms the core of many Search based technologies like Elasticsearch and Solr. Lucene is very fast. It can help you find a document containing a given word among millions of documents in milliseconds. From searching in milliseconds to indexing speeds of as high as [700GB per hour](http://people.apache.org/~mikemccand/lucenebench/indexing.html), Lucene is a library with very high performance.

### History of Apache Lucene

Lucene was first published in 1999 by Doug Cutting. Cutting originally published Lucene on SourceForge. In September 2001 it joined the Jakarta family of software of Apache Software Foundation. In January 2005, it became its own top-level project.

Initially, various other projects like Mahout, Tika, and Nutch were also a part of Lucene. Later they got separated into their own top-level Apache Products.

### Lucene vs Solr and Elasticsearch

While Lucene is just a java library, Elasticsearch and Solr are full-fledged search servers. You can imagine the difference between Lucene being the engine and Elasticsearch being a Ferrari and Solr being a Lamborgini.

With Lucene, you get the core capabilities of performing the search but Elasticsearch and Solr add features to it like providing an HTTP interface to Lucene-based search, automatically managing the performance by using distributed computing on top of Lucene, and many other features.

Now that you understand what Lucene is we will begin this Lucene tutorial by building a simple java application.

Lucene Tutorial – A simple java application
-------------------------------------------

### Adding dependencies

We need `lucene-core `dependency to start working with Lucene.

If we want to use query parser we can also include the `lucene-queryparser` dependency.

In case you want to use any other analyzer than StandardAnalyzer, you will need to add the `lucene-analyzers-common` dependency too.

In case of using MAVEN, you can paste the following dependencies in `pom.xml`

```xml {class="my-class" id="my-codeblock" lineNos=inline tabWidth=2}
<dependency>
  <groupId>org.apache.lucene</groupId>
  <artifactId>lucene-core</artifactId>
  <version>8.10.1</version>
</dependency>

<dependency>
  <groupId>org.apache.lucene</groupId>
  <artifactId>lucene-queryparser</artifactId>
  <version>8.10.1</version>
</dependency>

<dependency>
  <groupId>org.apache.lucene</groupId>
  <artifactId>lucene-analyzers-common</artifactId>
  <version>8.10.1</version>
</dependency>
```

### A simple Indexer

We will create a simple indexer which will add three movies in our lucene index:

```java {class="my-class" id="my-codeblock" lineNos=inline tabWidth=2}
import org.apache.lucene.analysis.Analyzer;
import org.apache.lucene.analysis.standard.StandardAnalyzer;
import org.apache.lucene.document.Document;
import org.apache.lucene.document.Field;
import org.apache.lucene.document.TextField;
import org.apache.lucene.index.IndexWriter;
import org.apache.lucene.index.IndexWriterConfig;
import org.apache.lucene.store.Directory;
import org.apache.lucene.store.FSDirectory;

import java.io.IOException;
import java.nio.file.Paths;

/**
 * This class creates documents for three movie titles and indexes them into lucene.
 */
public class SimpleIndexer {
    public void index(String indexPath) throws IOException {

        //We open a File System directory as we want to store the index on our local file system.
        Directory directory = FSDirectory.open(Paths.get(indexPath));

        //The analyzer is used to perform analysis on text of documents and create the terms that will be
        //added in the index.
        Analyzer analyzer = new StandardAnalyzer();
        IndexWriterConfig indexWriterConfig = new IndexWriterConfig(analyzer);

        IndexWriter indexWriter = new IndexWriter(directory, indexWriterConfig);


        //Now we create three documents for 3 movies. We have only one Field called title in each document.
        Document movie1 = new Document();
        movie1.add(new TextField("title", "Harry Potter and the Prisoner of Azkaban", Field.Store.YES));

        Document movie2 = new Document();
        movie2.add(new TextField("title", "Lord of the Rings: The fellowship of the ring.", Field.Store.YES));

        Document movie3 = new Document();
        movie3.add(new TextField("title", "Toy Story 3", Field.Store.YES));

        System.out.println("Going to index 3 movies.");

        //Now we add the three documents to our index.
        indexWriter.addDocument(movie1);
        indexWriter.addDocument(movie2);
        indexWriter.addDocument(movie3);
        System.out.println("Movies Indexed Successfully!");

        indexWriter.close();
    }

    public static void main(String[] args) throws IOException {
        String path = "moviesIndex";
        SimpleIndexer indexer = new SimpleIndexer();
        indexer.index(path);
    }
}

```

**OUTPUT**:

```
Going to index 3 movies.
Movies Indexed Successfully!
```

### A simple Searcher

Then we create a simple searcher which will try to find a movie by title. It will return how many documents it found with matching titles and then print the title of the movie found.

```java {class="my-class" id="my-codeblock" lineNos=inline tabWidth=2}
import org.apache.lucene.analysis.Analyzer;
import org.apache.lucene.analysis.standard.StandardAnalyzer;
import org.apache.lucene.document.Document;
import org.apache.lucene.index.DirectoryReader;
import org.apache.lucene.index.IndexReader;
import org.apache.lucene.queryparser.classic.ParseException;
import org.apache.lucene.queryparser.classic.QueryParser;
import org.apache.lucene.search.IndexSearcher;
import org.apache.lucene.search.Query;
import org.apache.lucene.search.ScoreDoc;
import org.apache.lucene.search.TopDocs;
import org.apache.lucene.store.FSDirectory;

import java.io.IOException;
import java.nio.file.Paths;

/**
 * The class looks up a movie in the movies index created by {@SimpleIndexer}
 */
public class SimpleSearcher {
    public void search(String indexPath, String title) throws IOException, ParseException {

        //We need to open an IndexReader to read the lucene index stored at given indexPath
        IndexReader indexReader = DirectoryReader.open(FSDirectory.open(Paths.get(indexPath)));

        //IndexSearcher will help us query the index
        IndexSearcher indexSearcher = new IndexSearcher(indexReader);

        //We will use standard analyzer while we parse our query.
        Analyzer analyzer = new StandardAnalyzer();

        //This query parser will search in title field by default if no field is specified.
        //Also, this will use our Standard analyzer to create terms for the query.
        QueryParser queryParser = new QueryParser("title", analyzer);

        //We pass in the title as it is to the query parser.
        // This will create a term query for us on the terms in our title.
        Query query = queryParser.parse(title);

        //We perform the search and get top 10 search results.
        TopDocs topDocs = indexSearcher.search(query, 10);

        //Print the count of matching documents.
        long totalHits = topDocs.totalHits.value;
        System.out.println(String.format("Found %d hits.", totalHits));

        //Print the title field of each matching movie
        ScoreDoc[] results  = topDocs.scoreDocs;
        for(ScoreDoc scoreDoc: results){

            //Returns the id of the document matching the query
            int docId = scoreDoc.doc;
            float score = scoreDoc.score;

            //We fetch the complete document from index via its id
            Document movie = indexSearcher.doc(docId);

            //Now we print the title of the movie
            System.out.println(String.format("Found: %s", movie.get("title")));
        }
    }

    public static void main(String[] args) throws IOException, ParseException {
        String path = "moviesIndex";
        SimpleSearcher searcher = new SimpleSearcher();
        searcher.search(path, "Harry Potter");
    }
}

```

**OUTPUT**:

```
Found 1 hits.
Found: Harry Potter and the Prisoner of Azkaban
```

Before we try and understand our code we need to understand some core terminologies.

Core terminologies
------------------

We will start this Lucene tutorial with a few core terminologies. These will help you understand how to work with Lucene.

### Index

All the documents of similar types are stored in an index. An index can be thought of as a collection. Let’s say you want to search on all articles on Wikipedia, so you will create an index of Wikipedia articles.

Those coming from relation databases backgrounds can try to think of Lucene like a table. Just as you keep the records of one type in one table, similarly you keep documents of one type in one index.

Internally Lucene uses a data structure known as an inverted index. An inverted index can be thought of as the Glossary section at the end of the book. A Glossary contains words with page numbers at which the word occurs. Similarly, an inverted index is made up of a mapping of words to the list of documents that contains that word. To understand how an inverted index works, I suggest you check out [this article.](/blog/inverted-index-data-structure/)

### Documents

A document is the basic unit of search in Lucene. When you do a search in Lucene, the response is the id of the document.

Database users can think of a document as a record in a table.

A document is a collection of fields.

### Fields

The field is the subunit of a document. You can think of it as an attribute of a document.

Let us say you have a blog and you want to implement a search interface for your blog using Lucene. Each of your blog posts will be a document in the blog-articles index. On each document, you can have multiple fields like a field for title, a field for the body, a field for publishing date, and a field for the author.

Lucene supports searching for documents by fields. For eg in the blogging example, you can search for all articles written by Ishan. Or you can find all articles that contain the word *Apple* in the title.

### Analysis

The analysis is the process of splitting up text into tokens. These tokens can be single words or they can be a group of words. Each token gets added to the inverted index. When you search in Lucene, Lucene analyses your text and tries to create tokens for it. Then it searches for those tokens in the inverted index.

The analysis is a very important part of Lucene as it can make or break your search experience. The tokens generated at the analysis step while indexing should match the ones generated in the search time analysis of the query. If they do not, then you won’t see correct search results. To understand the analysis process in-depth, check out the article on the analysis here.

Lucene Indexing API
-------------------

It’s time to understand the Lucene APIs. We begin by looking at some of the core classes that make up the indexing API

### Directory

A `Directory` interface abstracts the details of how the index is stored. We have the option of storing the index in local file system, virtual memory or on network drive and even on the heap space on RAM . Given the different places where an index can live, Lucene needs to abstract that detail. This is what is achieved with the `Directory` interface.

Some of the concrete implementations of the `Directory` interface are:

- `BytesBufferDirectory`: A ByteBuffer-based Directory implementation that can be used to store index files on the heap.
- `NMapDirectory`: uses memory-mapped IO when reading. This is a good choice if you have plenty of virtual memory relative to your index size, eg if you are running on a 64 bit JRE, or you are running on a 32 bit JRE but your index sizes are small enough to fit into the virtual memory space.
- `NIOFSDirectory`: uses java.nio’s FileChannel’s positional io when reading to avoid synchronization when reading from the same file.

It is suggested to use a File System based directory if you want your index to persist even after your application shuts down. Lucene provides a very helpful `FSDirectory.open` method that automatically selects the best `FSDirectory` implementation for your operating system.

The following code ilustrates how to create a directory that is stored on local file system in a temporary directory called “tempIndex”.

```java {class="my-class" id="my-codeblock" lineNos=inline tabWidth=2}
Path indexPath = Files.createTempDirectory("tempIndex");
Directory directory = FSDirectory.open(indexPath);
```

### Field

A Field is an attribute of a document. Think of this like column value in databases.

A field can be of multiple type depending upon the data type of data we want to store. Lucene comes with many prebuilt field classes. Some of the noteworthy classes are:

- TextField: Use this field type if you want to store some text on which you want full text search capabilites
- StringField: Use this when you want to do exact search on the string.
- IntPoint: Use this when you want to add an integer data

Some of the available subclasses of Field are [BinaryDocValuesField](https://lucene.apache.org/core/8_1_1/core/org/apache/lucene/document/BinaryDocValuesField.html), [BinaryPoint](https://lucene.apache.org/core/8_1_1/core/org/apache/lucene/document/BinaryPoint.html), [DoubleDocValuesField](https://lucene.apache.org/core/8_1_1/core/org/apache/lucene/document/DoubleDocValuesField.html), [DoublePoint](https://lucene.apache.org/core/8_1_1/core/org/apache/lucene/document/DoublePoint.html), [DoubleRange](https://lucene.apache.org/core/8_1_1/core/org/apache/lucene/document/DoubleRange.html), [FeatureField](https://lucene.apache.org/core/8_1_1/core/org/apache/lucene/document/FeatureField.html), [Field](https://lucene.apache.org/core/8_1_1/core/org/apache/lucene/document/Field.html), [FloatDocValuesField](https://lucene.apache.org/core/8_1_1/core/org/apache/lucene/document/FloatDocValuesField.html), [FloatPoint](https://lucene.apache.org/core/8_1_1/core/org/apache/lucene/document/FloatPoint.html), [FloatRange](https://lucene.apache.org/core/8_1_1/core/org/apache/lucene/document/FloatRange.html), [IntPoint](https://lucene.apache.org/core/8_1_1/core/org/apache/lucene/document/IntPoint.html), [IntRange](https://lucene.apache.org/core/8_1_1/core/org/apache/lucene/document/IntRange.html), [LatLonDocValuesField](https://lucene.apache.org/core/8_1_1/core/org/apache/lucene/document/LatLonDocValuesField.html), [LatLonPoint](https://lucene.apache.org/core/8_1_1/core/org/apache/lucene/document/LatLonPoint.html), [LongPoint](https://lucene.apache.org/core/8_1_1/core/org/apache/lucene/document/LongPoint.html), [LongRange](https://lucene.apache.org/core/8_1_1/core/org/apache/lucene/document/LongRange.html), [NumericDocValuesField](https://lucene.apache.org/core/8_1_1/core/org/apache/lucene/document/NumericDocValuesField.html), [SortedDocValuesField](https://lucene.apache.org/core/8_1_1/core/org/apache/lucene/document/SortedDocValuesField.html), [SortedNumericDocValuesField](https://lucene.apache.org/core/8_1_1/core/org/apache/lucene/document/SortedNumericDocValuesField.html), [SortedSetDocValuesField](https://lucene.apache.org/core/8_1_1/core/org/apache/lucene/document/SortedSetDocValuesField.html), [StoredField](https://lucene.apache.org/core/8_1_1/core/org/apache/lucene/document/StoredField.html), [StoredFieldsWriter.MergeVisitor](https://lucene.apache.org/core/8_1_1/core/org/apache/lucene/codecs/StoredFieldsWriter.MergeVisitor.html), [StringField](https://lucene.apache.org/core/8_1_1/core/org/apache/lucene/document/StringField.html), and [TextField](https://lucene.apache.org/core/8_1_1/core/org/apache/lucene/document/TextField.html)

You can create a Field as:

```java {class="my-class" id="my-codeblock" lineNos=inline tabWidth=2}
Field f = new TextField("title", "As you like It", Field.Store.YES);
```

### Document

A document is a collection of Fields. You can have a document for one entity. Every attribute of that entity will be the fields on the document. For eg, if you are creating an index of blog posts, you can create a document for each blog post and there will be seperate fields in that document for title, body, author and creation date.

Here is a sample code which creates a document and adds some fields to it.

```java {class="my-class" id="my-codeblock" lineNos=inline tabWidth=2}
Document doc = new Document();
doc.add(new TextField("title", "Timon and Pumba", Field.Store.YES));
doc.add(new TextField("slogan", "Hakuna Matata", Field.Store.YES));
```

### IndexWriter

Once we have our document created we need to add it to the Lucene index. `IndexWriter` helps us add our document to the actual Lucene index.

An IndexWriter need and `IndexWriterConfig` and a `Directory` instance. The Directory tells the IndexWriter how to store the index. The IndexWriterConfig can be used to set various configurations on the IndexWriter like whether you want to append to the index or rewrite it completely, or it can be used to specify the standard analyzer that you want to use and much more.

You can add a document to the index as:

```java {class="my-class" id="my-codeblock" lineNos=inline tabWidth=2}
Directory dir = FSDirectory.open(Paths.get("index"));
IndexWriterConfig config = new IndexWriterConfig();
IndexWriter writer = new IndexWriter(dir, config);

writer.addDocument(doc);
```

### Analyzer

An `Analyzer` is responsible for converting the text added in document to tokens that can be added to our index. Lucene comes with some prebuilt analyzers that you can use.

The most commonly used Analyzer for English language is `StandardAnalyzer`. This analyzer does the following:

1. splits the text using Word Break rules from the Unicode Text Segmentation algorithm, as specified in [Unicode Standard Annex #29](http://unicode.org/reports/tr29/).
2. converts the tokens to lowercase and removes common words like a, an, the etc knwon as stop words.
3. convert the words to base form. For eg *drinking* will be converted to *drink*. This enables searches with term *drink* to match the document containing *drinking* as well.

You can set an analyzer on the IndexWriter as:

```java {class="my-class" id="my-codeblock" lineNos=inline tabWidth=2}
Directory dir = FSDirectory.open(Paths.get("index"));
Analyzer analyzer = new StandardAnalyzer();
IndexWriterConfig config = new IndexWriterConfig(analyzer);
IndexWriter writer = new IndexWriter(dir, config);
```

Lucene Search API
-----------------

Now that we are familiar with the indexig API, let’s dive right in to the juicy stuff – the search API.

### Query

The `Query` class is the base class for all the types of queries in Lucene. Lucene supports searching for documents in the index based on the types of queries you use. Some of the common query subclasses are:

1. `TermQuery`: When you know the token you want to find in the index, you need to use the term query. This will return all the documents that contain the term. Please note that the term must be similar to the one produced after analysis.
2. `PhraseQuery`: A term query can only lookup based on individual terms. Sometimes you want to search for words that occur together. Such words are called as a prhase and you can use a `PhraseQuery` to search for a phrase.

Some of the other query types are:

[BlendedTermQuery](https://lucene.apache.org/core/8_1_1/core/org/apache/lucene/search/BlendedTermQuery.html), [BooleanQuery](https://lucene.apache.org/core/8_1_1/core/org/apache/lucene/search/BooleanQuery.html), [BoostQuery](https://lucene.apache.org/core/8_1_1/core/org/apache/lucene/search/BoostQuery.html), [ConstantScoreQuery](https://lucene.apache.org/core/8_1_1/core/org/apache/lucene/search/ConstantScoreQuery.html), [DisjunctionMaxQuery](https://lucene.apache.org/core/8_1_1/core/org/apache/lucene/search/DisjunctionMaxQuery.html), [DocValuesFieldExistsQuery](https://lucene.apache.org/core/8_1_1/core/org/apache/lucene/search/DocValuesFieldExistsQuery.html), [IndexOrDocValuesQuery](https://lucene.apache.org/core/8_1_1/core/org/apache/lucene/search/IndexOrDocValuesQuery.html), [MatchAllDocsQuery](https://lucene.apache.org/core/8_1_1/core/org/apache/lucene/search/MatchAllDocsQuery.html), [MatchNoDocsQuery](https://lucene.apache.org/core/8_1_1/core/org/apache/lucene/search/MatchNoDocsQuery.html), [MultiPhraseQuery](https://lucene.apache.org/core/8_1_1/core/org/apache/lucene/search/MultiPhraseQuery.html), [MultiTermQuery](https://lucene.apache.org/core/8_1_1/core/org/apache/lucene/search/MultiTermQuery.html), [NGramPhraseQuery](https://lucene.apache.org/core/8_1_1/core/org/apache/lucene/search/NGramPhraseQuery.html), [NormsFieldExistsQuery](https://lucene.apache.org/core/8_1_1/core/org/apache/lucene/search/NormsFieldExistsQuery.html), [PointInSetQuery](https://lucene.apache.org/core/8_1_1/core/org/apache/lucene/search/PointInSetQuery.html), [PointRangeQuery](https://lucene.apache.org/core/8_1_1/core/org/apache/lucene/search/PointRangeQuery.html), [SpanQuery](https://lucene.apache.org/core/8_1_1/core/org/apache/lucene/search/spans/SpanQuery.html), [SynonymQuery](https://lucene.apache.org/core/8_1_1/core/org/apache/lucene/search/SynonymQuery.html), [TermInSetQuery](https://lucene.apache.org/core/8_1_1/core/org/apache/lucene/search/TermInSetQuery.html)

The following code creates a query to lookup the word *panda* in the *body* field.

```java {class="my-class" id="my-codeblock" lineNos=inline tabWidth=2}
Query query = new TermQuery(new Term("body", "panda"));
```

### Query Parser

Creating `Query` objects is powerful was to do search, but it is quiet verbose. You need to write all the java code. There is an easier ways to create `Query` object. Lucene comes with a very good query parser. Using a query parse you can write queries as normal text and use the `QueryParser` to create a Query object for you.

For e.g. To create a query to lookup the word *panda* in the *body* field using QueryParser use the following code:

```java {class="my-class" id="my-codeblock" lineNos=inline tabWidth=2}
Analyzer analyzer = new StandardAnalyzer();
QueryParser parser = new QueryParser("body", analyzer);
Query query = parser.parse("body:panda");
```

Note here that we needed to pass an `Analyzer` to `QueryParser`. Lucene will use the passed analyzer to create the terms it needs to lookup. This query time analysis is critcal in the sense that it should produce the same terms that were added to the index at index time. If not done properly you might not see results correctly.

[Here](https://lucene.apache.org/core/2_9_4/queryparsersyntax.html) is the detailed query parser syntax available. I encourage you to read it and give it a try.

### IndexReader

Just as we used an `IndexWriter` to add documents to the index, we need to use a `IndexReader` to read our index. An `IndexReader` provides low level API that can be used to interact directly with the Lucene index.

An `IndexReader` provides an interface for accessing a point-in-time view of an index. That is it will only show changes as were available at the time the reader was opened. Any other changes made after opening the `IndexReader` won’t be visible.

You can create an `IndexReader` as:

```java {class="my-class" id="my-codeblock" lineNos=inline tabWidth=2}
String indexPath = "/tmp/myIndex";
IndexReader reader = DirectoryReader.open(FSDirectory.open(Paths.get(indexPath)));

```

### IndexSearcher

While an IndexReader provides a low level API to read the internals of the index, IndexSearcher exposes the high level search API. An IndexSearcher can be created by passing in an IndexReader as:

```java {class="my-class" id="my-codeblock" lineNos=inline tabWidth=2}
IndexSearcher searcher = new IndexSearcher(reader);
```

You can then use the IndexSearcher to get top 10 documents in the index that match a query as:

```java {class="my-class" id="my-codeblock" lineNos=inline tabWidth=2}
TopDocs topDocs = indexSearcher.search(query, 10);
```

### TopDocs

The output of seach via IndexSearcer is a TopDocs instance. TopDocs contains the total number of documents that matched the query (TopDocs) and an array of search results ( Called ScoreDoc\[\]). The results are by default sorted by the score with the best matching result coming first and least matching result coming last. You can get these as:

```java {class="my-class" id="my-codeblock" lineNos=inline tabWidth=2}
long totalHits = topDocs.totalHits.value;
ScoreDoc[] results  = topDocs.scoreDocs;
```

### ScoreDoc

A ScoreDoc contains the id of the document along with its score. Note that this only contains the id of the document and not the complete document. So if you want the complete document you can get that as:

```java {class="my-class" id="my-codeblock" lineNos=inline tabWidth=2}
int docId = scoreDoc.doc;
float score = scoreDoc.score;
Document doc = indexSearcher.doc(docId);
```

Once you have a document you can get its individual field values by using the get method on document. However note that you will get a value only if you specified Field.Store.YES while adding the document to the index.

Conclusion
----------

In this blog post we covered an in-depth Lucene tutorial. We understood what Lucene is, how it is different from Solr and Elasticsearch. We then made a simple Lucene application in JAVA and then understood some core concepts of lucene. We ended the tutorial by going over the Indexing and Searching API of Lucene. With this knowledge you are ready to create your own Lucene application. If you have any queries let me know in the comments.

What next?
----------

Now that you know the basics of Lucene, you might be interested in understanding the concepts in depth. [Here](/blog/apache-lucene-7-concepts-that-will-help-you-get-started/) is a great post explaining the 7 concepts you need to understand to start using Lucene.