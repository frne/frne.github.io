+++
date = "2015-07-23T18:05:12+02:00"
draft = false
title = "Understanding Stemmers (Natural Language Processing)"
tags = [ "java", "nlp", "solr" ]
categories = ["Programming", "Data Science"]
author = "Frank Neff"
+++

I am interested in NLP and have already some experience with Apache Solr. It's time to dig a little in-deep regarding 
stemmers. First of all, I was looking for a general definition of what a stemmer is, and I found this one, which IMHO 
is quite good:

> stemmer &mdash; an algorithm for removing inflectional and derivational endings in order to reduce word forms to a common stem

<!--more-->

So what a stemmer does is nothing more, than converting words to their word stem. For example, the three 
words ```developing, developer, development``` will be converted to ```develop```. Therefore, stemmers are often used 
as filters.

This can be very handy in different situaltions e.g. when writing to a fulltext search index like 
[SOLR](http://lucene.apache.org/solr/) or [Elasticsearch](https://www.elastic.co/products/elasticsearch). 
There are plenty of different stemming algorithms out there. To use them in a project, some different things have to 
be concerned:

## Language

Stemming is based on common patterns, so it works a bit different in English than e.g. in German. Some stemmers support 
a variety of languages, some are only available in English. I would not recommend to use a stemmer in another language
than your content is, because it maybe works, but only maybe.

[Here](https://wiki.apache.org/solr/AnalyzersTokenizersTokenFilters#Stemming) is a list of different stemmers which can 
be used in Apache SOLR, including supported languages.

## Time consumed

As with every algorithm, there are faster and slower ones. Depending on where they are used, performance could have a huge 
impact on success. Generally said: Slower does not necessarily mean more precise.

## Agressiveness

Because a stemmer does not really "understand" the language of the content which it's processing, it all depends on the 
patterns which are applied. The PorterStemmer for example removes word endings like "e", "er" and "ing". 
Therefore, the following conversion will be made:

> horse, horses &gt; **hors**
>
> develoment, developer, developing &gt; **develop**

Depending on the task to be done, this can be great or totally disfunctional. I would recommend to try out different 
stemming algorithms whith some edge-cases of specific domain and evaluating the result. For many stemmers, there is 
good documentation describing the agressiveness of the algorithm.

## Different stemmers

* [Porter](http://tartarus.org/~martin/PorterStemmer/) (English)
* [Porter 2](http://snowball.tartarus.org/algorithms/english/stemmer.html) (English)
* [Snowball](http://snowball.tartarus.org/) (Multi Language)
* [KStem](http://lexicalresearch.com/kstem-doc.txt) (English, less agressive then porter)

## Usage in Java

For a recent project, I used some of the SOLR / Lucene filters in Java. Just a short example how they can be used:

```java
public class EnglishTokenizer implements TokenizerInterface {

    @Override
    public Collection<String> tokenize(String content) throws TokenizerException {
        try {
            // read content
            StringReader inputText = new StringReader(content);
            Map<String, String> tkargs = new HashMap<String, String>();
            tkargs.put("luceneMatchVersion", "LUCENE_51");

            // char filter (html)
            CharFilterFactory hcff = new HTMLStripCharFilterFactory(tkargs);
            Reader strippedInput = hcff.create(inputText);

            // tokenizer
            TokenizerFactory tkf = new StandardTokenizerFactory(tkargs);
            Tokenizer tkz = tkf.create();
            tkz.setReader(inputText);

            // stopwords filter
            Map<String, String> stfargs = new HashMap<String, String>();
            stfargs.put("luceneMatchVersion", "LUCENE_51");
            stfargs.put("words", "lucene/en/stopwords.txt");
            stfargs.put("ignoreCase", "true");
            StopFilterFactory stff = new StopFilterFactory(stfargs);
            stff.inform(new ClasspathResourceLoader());
            TokenStream stfts = stff.create(tkz);

            // K stem filter
            Map<String, String> ksffparam = new HashMap<String, String>();
            KStemFilterFactory ksff = new KStemFilterFactory(ksffparam);
            TokenStream ksts = ksff.create(stfts);

            // synonyms filter
            Map<String, String> syffargs = new HashMap<String, String>();
            syffargs.put("luceneMatchVersion", "LUCENE_51");
            syffargs.put("synonyms", "lucene/en/synonyms.txt");
            syffargs.put("ignoreCase", "true");
            syffargs.put("expand", "false");
            SynonymFilterFactory syff = new SynonymFilterFactory(syffargs);
            syff.inform(new ClasspathResourceLoader());
            TokenStream syfts = syff.create(ksts);

            // lower case filter
            LowerCaseFilterFactory lcf = new LowerCaseFilterFactory(tkargs);
            TokenStream ts = lcf.create(syfts);

            // process token stream
            ts.reset();

            CharTermAttribute termAttrib = (CharTermAttribute) ts.getAttribute(CharTermAttribute.class);

            Collection<String> tokens = new ArrayList<String>();
            while (ts.incrementToken()) {
                tokens.add(termAttrib.toString());
            }

            ts.end();
            ts.close();

            return tokens;
        } catch (IOException e) {
            throw new TokenizerException(e);
        }
    }
}
```

In the above example, a set of filters is applied to a string to tokenize, filter (and stem) words. For stemming, the 
KStem filter is used. The filters are all from the package ```org.apache.solr:solr-core:5.2.1``` which is available on 
the [maven repository](http://mvnrepository.com/artifact/org.apache.solr/solr-core/5.2.1). 

### Tokeinzer Example

**Content**

> Java Developer - Zurich - financial markets - Salary negotiable depending on experience. A number of exciting Java 
> opportunities exist to join a highly successful and growing provider...

**Result (Tokens)**

> java, development, zurich, financial, market, salary, negotiable, depend, experience, number, exciting, java, 
> opportunity, exist, join, highly, successful, grow, provider

## Sources / More Information

* [lexicalresearch.com](http://lexicalresearch.com/papers.html)
* [Apache SOLR Wiki](https://wiki.apache.org/solr/LanguageAnalysis)
* [Snowball Project](http://snowball.tartarus.org/)
