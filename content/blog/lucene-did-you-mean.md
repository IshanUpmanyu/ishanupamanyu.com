---
title: 'How to create in Lucene DID YOU MEAN feature like google?'
date: '2022-04-03T21:00:13+05:30'
status: publish
url: /blog/lucene-did-you-mean
author: 'Ishan Upamanyu'
description: ''
type: post
id: 677
image: /uploads/2022/04/Screenshot-2022-04-03-at-8.46.39-PM.png
tags:
    - 'Apache Lucene'
---
In this blog post, we will create in Lucene `Did you mean` feature as seen in Google. Let’s begin by understanding what this feature is.

Whenever you misspell a word in google, google is able to figure out that you made a mistake. If the mistake is not significant, then google simply shows results for what it thinks is the correct spelling. It also gives you the option to search instead for the original query you entered.

![](/uploads/2022/04/Screenshot-2022-04-03-at-8.53.29-PM.png)
  But if it feels there is a significant difference in the spelling of the query and what it thinks the right query is, it will ask you what you meant by using the “Did you mean” feature as shown below:

![](/uploads/2022/04/Screenshot-2022-04-03-at-8.53.01-PM-1.png)


Lucene did you mean feature java example
----------------------------------------

If you are creating a search engine using Apache Lucene, you can create a similar feature.

> We can create in Lucene DID YOU MEAN feature by using the Hunspell dictionaries. These come prebuilt in Lucene in the analyzer common library.

To create a `Did you mean` feature in Lucene we need to have the following capabilities:

1. We need to know when a word is not spelled correctly. This will help us show the *did you mean* prompt to the user only when we need to.
2. Once we know the word is misspelled, then we need the ability to show the correct spelling.

We can achieve both these things using Lucene’s [Hunspell](https://lucene.apache.org/core/8_9_0/analyzers-common/org/apache/lucene/analysis/hunspell/Hunspell.html) class. Here is a working example of how to use the Hunspell class.

### Dependencies

We need Lucene’s core and analyzers-common library.

In case you are using MAVEN, copy the following dependencies in pom.xml

```xml {class="my-class" id="my-codeblock" lineNos=inline tabWidth=2}
<dependency>
   <groupId>org.apache.lucene</groupId>
   <artifactId>lucene-core</artifactId>
   <version>8.10.1</version>
</dependency>

<dependency>
   <groupId>org.apache.lucene</groupId>
   <artifactId>lucene-analyzers-common</artifactId>
   <version>8.10.1</version>
</dependency>

```


### Hunspell Dictionary files

For this feature to work you need the following two libraries:

1. Affix file: This can be found [here](https://github.com/ropensci/hunspell/blob/master/inst/dict/en_US.aff).
2. The dictionary file: Can be found [here](https://github.com/ropensci/hunspell/blob/master/inst/dict/en_US.dic).

You can get affix and dictionary files for other languages [here](https://cgit.freedesktop.org/libreoffice/dictionaries/tree/).

### JAVA code

Here is the java code. It does two things: 
1. Check the spelling of words and 
2. Get some suggestions for words.

```java {class="my-class" id="my-codeblock" lineNos=inline tabWidth=2}
package upmanyu.ishan.lucene.tutorial;

import org.apache.lucene.analysis.hunspell.Dictionary;
import org.apache.lucene.analysis.hunspell.Hunspell;
import org.apache.lucene.store.Directory;
import org.apache.lucene.store.FSDirectory;

import java.io.*;
import java.net.URL;
import java.nio.file.Files;
import java.text.ParseException;

public class DidYouMean {
    public static void main(String[] args) throws IOException, ParseException {
        Directory directory = FSDirectory.open(Files.createTempDirectory("temp"));
        InputStream affFileStream = new FileInputStream("src/main/resources/hunspell/en-US/en_US.aff");

        InputStream dicFileStream = new FileInputStream("src/main/resources/hunspell/en-US/en_US.dic");
        Dictionary dictionary = new Dictionary(directory, "spellCheck", affFileStream, dicFileStream);

        Hunspell spellChecker = new Hunspell(dictionary);

        String correctWord = "guava";
        String misspelledWord = "recieve";

        System.out.println(String.format("Is %s spelled correctly?: %b", correctWord, spellChecker.spell(correctWord)));
        System.out.println(String.format("Is %s spelled correctly?: %b", misspelledWord, spellChecker.spell(misspelledWord)));
        System.out.println(String.format("Did you mean: %s", spellChecker.suggest(misspelledWord)));
    }
}
```


**OUTPUT**:

![lucene did you mean example output](/uploads/2022/04/Screenshot-2022-04-03-at-9.19.06-PM.png)

Now that you have seen the java code in action, let us try and understand how this code works. The core part of this code is the Hunspell class in Lucene. According to Wikipedia:

> Hunspell is a spell checker and morphological analyser designed for languages with rich morphology and complex word compounding and character encoding, originally designed for the Hungarian language.
> 
> <cite>https://en.wikipedia.org/wiki/Hunspell</cite>

Hunspell is used by LibreOffice office suite, free browsers, like Mozilla Firefox and Google Chrome, and other tools and OSes, like Linux distributions and macOS. It is also a command-line tool for Linux, Unix-like, and other OSes.

How do Hunspell dictionaries work?
----------------------------------

A Hunspell dictionary needs two files to work:

1. Affix File: An affix file (\*.aff) contains a list of suffixes and prefixes and rules describing them. The rules describe where we can use a suffix or prefix. For example, you may have a rule which says the suffix *`ied`* can be applied only if the last character of the word is y, in other cases suffix *`ed`* should be used.
2. Dictionary file: A dictionary file (\*.dic) contains a list of words one per line. The first line of the dictionaries (except personal dictionaries) contains the approximate word count (for optimal hash memory size). Each word may optionally be followed by a slash (“/”) and one or more flags, which represents the word attributes, for example, affixes.

Using these two files, the algorithm tries to find if the word can be formed with the given affix and dictionary file rules.

As stated in Linux man-pages:

Consider the Dictionary file:

```
3
hello
try/B
work/AB

```

> The flags B and A specify the attributes of these words.
> 
> <cite><http://manpages.ubuntu.com/manpages/bionic/man5/hunspell.5.html></cite>

and the Affix file:

```
SET UTF-8
TRY esianrtolcdugmphbyfvkwzESIANRTOLCDUGMPHBYFVKWZ'

REP 2
REP f ph
REP ph f

PFX A Y 1
PFX A 0 re .

SFX B Y 2
SFX B 0 ed [^y]
SFX B y ied y

```

In the affix file, prefix A and suffix B have been defined. Flag A defines a `re-‘ prefix. Class B defines two `-ed’ suffixes. First B suffix can be added to a word if the last character of the word isn’t `y’. The second suffix can be added to the words terminated with an `y’.

All accepted words with this dictionary and affix combination are: “hello”, “try”, “tried”, “work”, “worked”, “rework”, “reworked”.

> <http://manpages.ubuntu.com/manpages/bionic/man5/hunspell.5.html>

Conclusion
----------

In this blog post, we understood what `did you mean` feature in google is. Then we saw some code to create in Lucene `Did you mean` feature by using Hunspell dictionary. We ended this post by learning how Hunspell dictionaries work.

What Next?
----------

If you enjoyed this article I am guessing you might be using Lucene to create a search application. I also created a search engine to search on English Wikipedia Articles. Do you want to know how I did it? Read all about it [here](/blog/how-i-created-a-search-engine-for-wikipedia-articles/).

References
----------

- Linux MAN pages for Hunspell: <http://manpages.ubuntu.com/manpages/bionic/man5/hunspell.5.html>
- Hunspell Project on Github: <https://github.com/hunspell/hunspell>
- Hunspell Wikipedia page: <https://en.wikipedia.org/wiki/Hunspell>