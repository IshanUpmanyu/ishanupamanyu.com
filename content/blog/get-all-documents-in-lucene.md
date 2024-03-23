---
title: 'How to get all documents in Lucene?'
date: '2022-04-01T23:34:14+05:30'
status: publish
url: /blog/get-all-documents-in-lucene
author: 'Ishan Upamanyu'
description: ''
type: post
id: 652
image: /uploads/2022/01/Apache-Lucene-1-e1648997525598.jpeg
tags:
    - 'Apache Lucene'
    - 'Information Retrieval'
---
If you are new to Lucene you may be wondering how to get all documents in the Lucene index. We can easily get documents matching a particular term or matching a query but how do we get all the documents in the Lucene index?

> We can get all documents in Lucene by using either the [MatchAllDocsQuery](https://lucene.apache.org/core/9_1_0/core/org/apache/lucene/search/MatchAllDocsQuery.html) query or by using \*:\* as the search string with QueryParser.

In this blog post, we will look at both ways with fully working code to get all documents from the Lucene index.

Get all documents in Lucene using MatchAllDocsQuery
---------------------------------------------------

Lucene provides a special query called [MatchAllDocsQuery](https://lucene.apache.org/core/9_1_0/core/org/apache/lucene/search/MatchAllDocsQuery.html). As the name suggests this query will match all the documents that are contained in the index. You can create this query as:

```java {class="my-class" id="my-codeblock" lineNos=inline tabWidth=2}
//This query will match with all documents in the index
Query query = new MatchAllDocsQuery();
```

Get all documents in Lucene using QueryParser
---------------------------------------------

The other way is to create the query using Lucene QueryParser with the following \*:\* as the search string. This tells Lucene to match all terms in all fields thereby matching all documents. Here is the code to get all documents using this method:

```java {class="my-class" id="my-codeblock" lineNos=inline tabWidth=2}
QueryParser queryParser = new QueryParser("title", analyzer);
//We pass in our special syntax to fetch all documents. This searches for all terms in all fields.
Query query = queryParser.parse("*:*");

```

Working example to get all documents in Lucene
----------------------------------------------

As promised, here is the fully functional code to get all documents present in the Lucene index.

In this example, we first index 100 documents into our index. Then we present the two ways to fetch all the documents.

```java {class="my-class" id="my-codeblock" lineNos=inline tabWidth=2}
package upmanyu.ishan.lucene.tutorial;

import org.apache.lucene.analysis.Analyzer;
import org.apache.lucene.analysis.standard.StandardAnalyzer;
import org.apache.lucene.document.Document;
import org.apache.lucene.document.Field;
import org.apache.lucene.document.TextField;
import org.apache.lucene.index.DirectoryReader;
import org.apache.lucene.index.IndexReader;
import org.apache.lucene.index.IndexWriter;
import org.apache.lucene.index.IndexWriterConfig;
import org.apache.lucene.queryparser.classic.ParseException;
import org.apache.lucene.queryparser.classic.QueryParser;
import org.apache.lucene.search.*;
import org.apache.lucene.store.Directory;
import org.apache.lucene.store.FSDirectory;

import java.io.IOException;
import java.nio.file.Paths;

/**
 * This class illustrates how to get all documents in Lucene index.
 */
public class GetAllDocs {

    /**
     * Get all documents in Lucene index using query parser syntax of *:*
     *
     * @param indexPath the index path where index is stored
     * @throws IOException    the io exception
     * @throws ParseException the parse exception
     */
    public void getAllDocsWithQueryParser(String indexPath) throws IOException, ParseException {

        //We need to open an IndexReader to read the lucene index stored at given indexPath
        IndexReader indexReader = DirectoryReader.open(FSDirectory.open(Paths.get(indexPath)));

        //IndexSearcher will help us query the index
        IndexSearcher indexSearcher = new IndexSearcher(indexReader);

        //We will use standard analyzer while we parse our query.
        Analyzer analyzer = new StandardAnalyzer();

        //This query parser will search in title field by default if no field is specified.
        //Also, this will use our Standard analyzer to create terms for the query.
        QueryParser queryParser = new QueryParser("title", analyzer);

        //We pass in our special syntax to fetch all documents. This searches for all terms in all fields.
        Query query = queryParser.parse("*:*");

        searchAndPrintResults(indexSearcher, query);

    }

    /**
     * Get all documents in Lucene index using MatchAllDocsQuery.
     *
     * @param indexPath the index path where index is stored
     * @throws IOException    the io exception
     * @throws ParseException the parse exception
     */
    public void getAllDocsWithMatchAllDocsQuery(String indexPath) throws IOException, ParseException {

        //We need to open an IndexReader to read the lucene index stored at given indexPath
        IndexReader indexReader = DirectoryReader.open(FSDirectory.open(Paths.get(indexPath)));

        //IndexSearcher will help us query the index
        IndexSearcher indexSearcher = new IndexSearcher(indexReader);

        //This query will match with all documents in the index
        Query query = new MatchAllDocsQuery();

        searchAndPrintResults(indexSearcher, query);

    }


    private void searchAndPrintResults(IndexSearcher indexSearcher, Query query) throws IOException {
        //We perform the search and get search results 10 at a time.
        TopDocs topDocs = indexSearcher.search(query, 10);

        //Print the count of matching documents.
        long totalHits = topDocs.totalHits.value;
        System.out.println(String.format("Found %d hits.", totalHits));

        //Print the title field of each matching document
        while(topDocs.scoreDocs.length != 0){
            ScoreDoc[] results  = topDocs.scoreDocs;
            for(ScoreDoc scoreDoc: results){

                //Returns the id of the document matching the query
                int docId = scoreDoc.doc;
                float score = scoreDoc.score;

                //We fetch the complete document from index via its id
                Document movie = indexSearcher.doc(docId);

                //Now we print the title of the document
                System.out.println(String.format("Found: %s", movie.get("title")));
            }

            //we fetch the last doc of this page. We will need to pass this to index searcher to get next page.
            ScoreDoc lastDoc = results[results.length -1];

            //Get next 10 documents after lastDoc. This gets us the next page of search results.
            topDocs = indexSearcher.searchAfter(lastDoc, query, 10);
        }
    }

    /**
     * This method adds 100 documents to our index.
     *
     * @param indexPath the index path where we want to store the index
     * @throws IOException the io exception
     */
    public void index(String indexPath) throws IOException {

        //We open a File System directory as we want to store the index on our local file system.
        Directory directory = FSDirectory.open(Paths.get(indexPath));

        //The analyzer is used to perform analysis on text of documents and create the terms that will be
        //added in the index.
        Analyzer analyzer = new StandardAnalyzer();
        IndexWriterConfig indexWriterConfig = new IndexWriterConfig(analyzer);

        //This will always overwrite the existing index. This way even if we run the program multiple times
        //we won't see duplicate documents.
        indexWriterConfig.setOpenMode(IndexWriterConfig.OpenMode.CREATE);
        IndexWriter indexWriter = new IndexWriter(directory, indexWriterConfig);

        System.out.println("Going to index 100 documents.");

        //Now we create 100 documents. We have only one Field called title in each document.
        for(int i = 1; i <= 100; i++){
            Document doc = new Document();
            doc.add(new TextField("title", "This is document "+ i, Field.Store.YES));
            indexWriter.addDocument(doc);
        }

        System.out.println("Documents Indexed Successfully!");

        indexWriter.close();
    }

    /**
     * The entry point of application.
     *
     * @param args the input arguments
     * @throws IOException    the io exception
     * @throws ParseException the parse exception
     */
    public static void main(String[] args) throws IOException, ParseException {
        String indexPath = "index";
        GetAllDocs getAllDocsExample = new GetAllDocs();
        getAllDocsExample.index(indexPath);

        System.out.println("Here are all documents fetched with QueryParser Syntax");
        getAllDocsExample.getAllDocsWithQueryParser(indexPath);

        System.out.println("\n");
        System.out.println("Here are all documents fetched with MatchAllDocsQuery");
        getAllDocsExample.getAllDocsWithMatchAllDocsQuery(indexPath);
    }
}
```


Conclusion
----------

In this post, we saw how to get all documents in Lucene using two methods. We first introduced the methods and then we gave a detailed example of how to use both methods.

What next?
----------

You now seem to be enjoying working with Lucene, but have you ever wondered how Lucene is able to give search result this fast? It is so because it is powered by an awesome data structure known as the inverted index. Read all about it [here](/blog/inverted-index-data-structure/).