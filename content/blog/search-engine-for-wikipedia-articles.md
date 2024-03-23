---
title: 'How I created a search engine for Wikipedia articles'
date: '2022-03-16T21:02:48+05:30'
status: publish
url: /blog/search-engine-for-wikipedia-articles
author: 'Ishan Upamanyu'
description: ''
type: post
id: 623
image: /uploads/2022/03/Screenshot-2022-03-19-at-8.15.22-PM-1.png
tags:
    - projects
---
It is said that the best way to learn a new software technology is to get your hands dirty. Recently I followed this advice when I was learning to be a search engineer. I found myself getting more and more interested in learning how search engines like google and product search engines like Amazon search work. I started learning about information retrieval theory and gradually moved on to practical technologies like Apache Lucene and Apache Solr. While reading books and watching videos was all well and great, I was not sure if I am actually learning real-life skills. So to fix that I thought why not create a search engine of my own and apply the techniques that I am learning. And thus the idea for [wikisearchengine.com](http://wikisearchengine.com) was born.

The first step in creating the engine was in deciding what do I want to search on? While brainstorming for options I remembered I had read a while ago that complete Wikipedia data is available for download. When I realized this, I thought what better data set to create a search engine than Wikipedia’s. Wikipedia has millions of articles, and creating a search experience on top of this data will be a great learning experience.

If you want to check out the app before reading the article, it is deployed at [wikisearchengine.com](http://wikisearchengine.com) and the source is available on Github [here](https://github.com/IshanUpmanyu/wikisearchengine).

![](/uploads/2022/03/Screenshot-2022-03-19-at-8.15.22-PM-1.png "Wiki Search Engine allows you to search on English Wikipedia articles") 

"Technologies used at a glance
-----------------------------

**Backend:** Java 11, SpringBoot, Apache Solr 8

**Front End:** HTML, Javascript, JQuery, Bootstrap 5

**Infrastructure:** [DigitalOcean](https://www.digitalocean.com/) droplets and block storage.

**Tools:** Git, Github, Maven

**Some helpful Libraries:** [WikiClean](https://github.com/lintool/wikiclean), [IndexWikipedia](https://github.com/lemire/IndexWikipedia)

Getting the Wikipedia data
--------------------------

The first step to creating the search engine is to index that data. To do that we need to grab the data first. I googled how to download Wikipedia data. That’s when I found the wiki dumps website. I found on the website that data is available in database as well as XML format. I didn’t want to run a database so I went for the XML format data which can be downloaded from [here](https://dumps.wikimedia.org/enwiki/latest/).

The file I chose to download was a large one almost 20 GB in size. [Here](https://dumps.wikimedia.org/enwiki/latest/enwiki-latest-pages-articles.xml.bz2) is the link to the file. It is the English Wikipedia article pages data. You get the complete dataset. It has roughly 11.5 million articles and is in XML format.

Parsing the English Wikipedia XML dump
--------------------------------------

Once I had the data the next step was to extract indexable text from it. To do that I needed to be able to read the data. I tried going through the documentation on wiki dumps but did not find anything useful. So I did a google search looking for any library that can read data from Wikipedia XML dumps. I found a few tools but they were for python. Since I am more comfortable with Java, I was looking for something written in Java. Since I am lazy I tried googling if there is a library that directly indexes data to either Solr or Lucene. That’s when I found something useful.

I found a project called [IndexWikipedia](https://github.com/lemire/IndexWikipedia) on Github. This project parses the XML dump file I had downloaded and indexes that using Apache Lucene. I started reading through the source code. I quickly found that it is using code from Lucene benchmarks source library that performs benchmarking on Apache Lucene using Wikipedia data. I found that inside this project there was a parser that can be used to iteratively get articles from the data dump. I found that I can extract the article title, body, page id, and date from the dumps using this parser. Since this is based on the SAX XML parser, this does not load the complete dump in memory and thus can be used with low memory systems as well. I extracted the parser out of the project and that was the beginning of the first piece of the puzzle of WikiSearchEngine – the WikiIndexer.

Removing Wikipedia text markup
------------------------------

The text as extracted from the XML dump is marked up in Wikipedia markup language. This posed problems for me as it did not generate the tokens correctly. I needed to extract plain text from the data. So I looked for ways to clean this data. That is when I came across [WikiClean](https://github.com/lintool/wikiclean). [Wikiclean](https://github.com/lintool/wikiclean) is a very easy-to-use library that provides a simple API. All I had to do was create an object of [WikiClean](https://github.com/lintool/wikiclean) and pass my raw data to the clean method. The library automatically removed all the Wikipedia markup.

Choosing the search backend – Apache Lucene vs Apache Solr
----------------------------------------------------------

Now that I had the data, I needed to index it. I had the options of Apache Lucene, Apache Solr, and Elasticsearch. I work in [Algonomy](https://algonomy.com/) in the [Find](https://algonomy.com/omnichannel-personalization/personalized-ecommerce-search/) project which is the personalized search product of the company. There we use Apache Solr. So I thought of picking a technology that will help me in work too. Thus I never considered Elasticserach. Though I must admit, it could have been a good option.

I had been reading Lucene in Action and naturally, the idea came to try to use Lucene directly. It also had the advantage of being lighter on resources as compared to Solr. I discussed this with my colleague. He suggested not going with bare-bones Lucene, because Lucene would be much more low level for my use case. I would have to write a lot of code to do even the basic of tasks. Also since we were using Solr in my day job, it would help me learn and get better at my day job too. So I decided to go with Apache Solr.

Indexing data in Solr
---------------------

I read through the Solr documentation to find how to index and search data with Solr. I found I had two options:

1. Issue HTTP requests direclty to solr.
2. Use the SolrJ library that provides an API to interact with Apache Solr.

I quickly rejected the first option as that would involve a lot of boilerplate code to parse and work with JSON objects. Since I was already planning on using JAVA, SolrJ was the way to go for me.

I added SolrJ 8.11 to the pom file. (I was using Maven in my project). Then I went through SolrJ documentation and found how to index data. The steps in indexing are simple:

1. Create a collection if it does not exist.
2. Index data. It is recommended to do batch update rather than doing a document by document update. I quickly created a program to iterate and fetch documents from my wiki parser and index the data in Solr.

In Solr, I added each article I got as a document. Document in Solr can be considered as one record in a database. Just as in a database there are multiple columns Solr documents can have multiple fields. I added 4 fields to each document that I indexed:

1. id: the unique id used to indentify the article. This id is later used to create link to the wikipedia page.
2. title: This was the title of the article.
3. body: The contents of the article page
4. date: The date the article was added/modified.

### Selecting Field types for the document

In Solr, it is very important to select the right field type if you want your search to work correctly. For eg, if you select String instead of Text for a field, you will not be able to search for individual words of the text as Solr would index complete data as one token. I selected the following field types:

1. id: I indexed this as String Field sInce I did not want to break this down to tokens as I needed to do an exact match on id.
2. title: This was indexed as Text field because i wanted full text search to work on this. I wanted that if someone enters only few words from the title, then he should find the article.
3. body: This was also added as Text Field for the same reasons as for the title.
4. date: Solr has a DateField. Indexing it as Date allows me to do range queries on date field. This would enable the users to filter the serach results by date field.

In Solr, the field types are specified in the Solr schema. I did not want to modify the schema and hence I went with dynamic fields schema

#### Dynamic Fields

A [dynamic field](https://solr.apache.org/guide/8_11/dynamic-fields.html) is just like a regular field except it has a name with a wildcard in it. When you are indexing documents, a field that does not match any explicitly defined fields can be matched with a dynamic field. Solr has dynamic fields defined for different data types. For example, if you want to have a field with text type, you can add \_t suffix to your field name. Solr will automatically index this as a text field. For string, you can add the suffix \_s to the field and this will be indexed as a string field. I created the following field names: title\_t, body\_t, id, and date\_dt, and Solr automatically indexed the field in my desired field type.

Thus my indexer was completed. I created the indexer as a module called WikiIndexer.

Now that indexing was complete it was time to build a search API.

Creating a Search API
---------------------

Now that I had the data in Solr. I needed to create an API to fetch data. I needed an API for two main reasons.

1. I did not want to issue queries to solr from front end directly as that would mean openning access to the solr instance for the complete web. Anyone could modify or delete the solr data if I allowed open access to Solr from the web.
2. Having a service sit between Solr and the frontend, allows me to write logic for relevancy tunining in backed itself. This would be useful in future when I want to add more features or want to improve the search experience.

I used Spring Boot to create a rest service. The rest service does the following things:

1. It takes in the word searched and queries that word in the title and body fields.
2. A match in the title field is 40 times more important than a match in body field. I implemented this by boosting the title field by 20 and decreasing the boost of body field to 0.5.
3. The API provides paginated access to search results with two parameters: page and resultsPerPage. The backend has restriction that we cannot fetch more than 20 results per page. This is added to prevent a malicious actor from overloading the backend by trying to fetch all the data.

Other than this logic, the search service is really simple.

The Frontend – making it easier for anyone to use the app
---------------------------------------------------------

Now that I had the data indexed and an API to get search results, I decided to create a UI so that it becomes simpler to test the search engine. I did not want to spend much time on the front end, so I went with the simplest cleanest UI possible and did so without any heavy frontend framework.

I used simple Bootstrap and Jquery to create the front end. Some custom CSS and a logo image are all that were needed to make it work.

Deploying the app
-----------------

Now that the app was ready I had to think about deployment. Since both my indexer and searcher were made with maven, it was fairly simple to create the deployable jar with dependencies.

### Creating the deployment package

To avoid having to type java command every time I needed to restart I created simple startup scripts. Also added a config file to change Solr URL or wiki dump file path as per requirement. The only thing needed was to create a deployment package. The maven assembly plugin came to the rescue here and I could get zips for both searcher and indexer.

### Choosing the cloud to deploy the app

With the deployment package ready I had to plan how I wanted to deploy my app. I chose to do with DigitalOcean as my cloud. I did not go with AWS free tier as I wanted at least 2 GB of RAM and 100 GB of Disk space. DigitalOcean was the cheapest service for my requirements. Plus I have used this for my personal projects in the past and I liked the experience so far.

I created a droplet with 2 GB RAM and 50 GB SSD storage and added 50 GB as additional hard drive storage. With this my infrastructure was ready. I copied the deployment packages on the VM and installed Solr 8 as well. With that done, all that was needed was to set up configurations and start the Indexer and Searcher using start scripts.

### Choosing the domain name

Now my app was live and I could access it from the web! But I had to remember the IP of my cloud machine. It also did not look as professional to share an IP address with others. I wanted to share a domain name with anyone who wanted to give my app a go. That’s why I decided to go buy a domain.

I went to Godaddy and started searching for some domains. I wanted WikiSearch.com but it was already taken. I brainstormed for a while but could not come up with any other good memorable name. Any other name I liked was either too expensive or not available. So I went with wikisearchengine.com. It was available. I purchased the domain. Pointed it to my server and voila within 15 min my app was live at <http://wikisearchengine.com>!

Conclusion
----------

It was really fun to build this app. It gives a sense of pride when you see that something you made from scratch is available out there for the world to use.

This project got me started with Solr and I am sure it will help me immensely in becoming a better search engineer. I plan on adding features to it in future like improving relevancy by using tuning ranking or maybe using NLP and machine learning to better understand user intent.

If you have read it so far, I want to thank you! I also encourage you to go visit <http://wikisearchengine.com> and give the app a try. Those wanting to see under the hood, are welcome to check out the source code [here](https://github.com/IshanUpmanyu/wikisearchengine).

Any feedback or feature suggestions would be really appreciated.