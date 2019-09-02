[DBPedia](https://wiki.dbpedia.org/) automatically extracts data from
Wikipedia and may contain links to the municipalities' websites.

The Portuguese language DBPedia extracts data from the Portuguese language
Wikipedia, while the English language DBPedia extracts data from the English
language Wikipedia. We are going to query them both.

## Portuguese language DBPedia

The Portuguese language DBPedia does not use the `dbo:country` property, so
getting Brazilian cities is a little tricky. Here we use having a link to
the wiki page "States of Brazil" as a filter for getting only cities located
in Brazil, instead.

The use of the `foaf:homepage` property is rare, so we have to resort to using
a `dbo:wikiPageExternalLink` property in addition to that. Keep in mind that
this will pollute the results to other pages which are not the official pages
of the municipality, so we need to filter them out somehow. The simplest way
of doing that is by using a SPARQL `FILTER` clause to get only containing
`.gov.br`. Unfortunately, some municipality websites do not conform to that
and will be missing in the query.

The following SPARQL query will extract links from the Portuguese language
DBPedia:

```sparql
PREFIX rdf:<http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX dbo:<http://dbpedia.org/ontology/>
PREFIX dbr:<http://dbpedia.org/resource/>

SELECT * WHERE {
    ?city a dbo:City ;
        dbo:wikiPageWikiLink dbr:States_of_Brazil ;
        dbo:wikiPageExternalLink|foaf:homepage ?link .
    FILTER REGEX(STR(?link), ".gov.br")
}
```

Results in
[HTML](http://pt.dbpedia.org/sparql?default-graph-uri=http%3A%2F%2Fdbpedia.org&query=PREFIX+rdf%3A+%3Chttp%3A%2F%2Fwww.w3.org%2F1999%2F02%2F22-rdf-syntax-ns%23%3E%0D%0APREFIX+dbo%3A%3Chttp%3A%2F%2Fdbpedia.org%2Fontology%2F%3E%0D%0APREFIX+dbr%3A%3Chttp%3A%2F%2Fdbpedia.org%2Fresource%2F%3E%0D%0A%0D%0ASELECT+*+WHERE+%7B%0D%0A++++%3Fcity+a+dbo%3ACity+%3B%0D%0A++++++++dbo%3AwikiPageWikiLink+dbr%3AStates_of_Brazil+%3B%0D%0A++++++++dbo%3AwikiPageExternalLink%7Cfoaf%3Ahomepage+%3Flink+.%0D%0A++++FILTER+REGEX%28STR%28%3Flink%29%2C+%22.gov.br%22%29%0D%0A%7D+LIMIT+100&format=text%2Fhtml&timeout=0&debug=on)
and
[CSV](http://pt.dbpedia.org/sparql?default-graph-uri=http%3A%2F%2Fdbpedia.org&query=PREFIX+rdf%3A+%3Chttp%3A%2F%2Fwww.w3.org%2F1999%2F02%2F22-rdf-syntax-ns%23%3E%0D%0APREFIX+dbo%3A%3Chttp%3A%2F%2Fdbpedia.org%2Fontology%2F%3E%0D%0APREFIX+dbr%3A%3Chttp%3A%2F%2Fdbpedia.org%2Fresource%2F%3E%0D%0A%0D%0ASELECT+*+WHERE+%7B%0D%0A++++%3Fcity+a+dbo%3ACity+%3B%0D%0A++++++++dbo%3AwikiPageWikiLink+dbr%3AStates_of_Brazil+%3B%0D%0A++++++++dbo%3AwikiPageExternalLink%7Cfoaf%3Ahomepage+%3Flink+.%0D%0A++++FILTER+REGEX%28STR%28%3Flink%29%2C+%22.gov.br%22%29%0D%0A%7D&format=text%2Fcsv&timeout=0&debug=on).

## English language DBPedia

This is query is a little bit simpler compared to the Portuguese language
DBPedia, because the data is more structured. We can simply use the
`dbo:country` property to determine that a city is located in Brazil.

```sparql
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX dbo:<http://dbpedia.org/ontology/>
PREFIX dbr:<http://dbpedia.org/resource/>
PREFIX foaf:<http://xmlns.com/foaf/0.1/>

SELECT * WHERE {
    ?city a dbo:City ;
        dbo:country dbr:Brazil .
        OPTIONAL {
            ?city foaf:homepage ?link .
        }
        OPTIONAL {
            FILTER REGEX(STR(?external_link), ".gov.br")
            ?city dbo:wikiPageExternalLink ?external_link .
        }
}
```

Results in
[HTML](http://dbpedia.org/sparql?default-graph-uri=http%3A%2F%2Fdbpedia.org&query=PREFIX+rdf%3A+%3Chttp%3A%2F%2Fwww.w3.org%2F1999%2F02%2F22-rdf-syntax-ns%23%3E%0D%0APREFIX+dbo%3A%3Chttp%3A%2F%2Fdbpedia.org%2Fontology%2F%3E%0D%0APREFIX+dbr%3A%3Chttp%3A%2F%2Fdbpedia.org%2Fresource%2F%3E%0D%0APREFIX+foaf%3A%3Chttp%3A%2F%2Fxmlns.com%2Ffoaf%2F0.1%2F%3E%0D%0A%0D%0ASELECT+*+WHERE+%7B%0D%0A++++%3Fcity+a+dbo%3ACity+%3B%0D%0A++++++++dbo%3Acountry+dbr%3ABrazil+.%0D%0A++++++++OPTIONAL+%7B%0D%0A++++++++++++%3Fcity+foaf%3Ahomepage+%3Flink+.%0D%0A++++++++%7D%0D%0A++++++++OPTIONAL+%7B%0D%0A++++++++++++FILTER+REGEX%28STR%28%3Fexternal_link%29%2C+%22.gov.br%22%29%0D%0A++++++++++++%3Fcity+dbo%3AwikiPageExternalLink+%3Fexternal_link+.%0D%0A++++++++%7D%0D%0A%7D+LIMIT+100&format=text%2Fhtml&CXML_redir_for_subjs=121&CXML_redir_for_hrefs=&timeout=30000&debug=on&run=+Run+Query+)
and
[CSV](http://dbpedia.org/sparql?default-graph-uri=http%3A%2F%2Fdbpedia.org&query=PREFIX+rdf%3A+%3Chttp%3A%2F%2Fwww.w3.org%2F1999%2F02%2F22-rdf-syntax-ns%23%3E%0D%0APREFIX+dbo%3A%3Chttp%3A%2F%2Fdbpedia.org%2Fontology%2F%3E%0D%0APREFIX+dbr%3A%3Chttp%3A%2F%2Fdbpedia.org%2Fresource%2F%3E%0D%0APREFIX+foaf%3A%3Chttp%3A%2F%2Fxmlns.com%2Ffoaf%2F0.1%2F%3E%0D%0A%0D%0ASELECT+*+WHERE+%7B%0D%0A++++%3Fcity+a+dbo%3ACity+%3B%0D%0A++++++++dbo%3Acountry+dbr%3ABrazil+.%0D%0A++++++++OPTIONAL+%7B%0D%0A++++++++++++%3Fcity+foaf%3Ahomepage+%3Flink+.%0D%0A++++++++%7D%0D%0A++++++++OPTIONAL+%7B%0D%0A++++++++++++FILTER+REGEX%28STR%28%3Fexternal_link%29%2C+%22.gov.br%22%29%0D%0A++++++++++++%3Fcity+dbo%3AwikiPageExternalLink+%3Fexternal_link+.%0D%0A++++++++%7D%0D%0A%7D&format=text%2Fcsv&CXML_redir_for_subjs=121&CXML_redir_for_hrefs=&timeout=30000&debug=on&run=+Run+Query+).