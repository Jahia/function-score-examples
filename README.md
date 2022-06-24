Function Score Examples
==========================
This module provides some examples of Function Score Queries to use with Jahia - Augmented Search. Function Score makes it possible to
 update the score of the documents returned by a query, to adjust the desired relevance based on your own criteria. 

The examples of this module show a few use cases
 where one can use Function Score, but much more applications can be considered. Learn more about Function Score, and its capabilities
 , directly in the [Elasticsearch Function Score documentation](https://www.elastic.co/guide/en/elasticsearch/reference/7.17/query-dsl-function-score-query.html).

_The examples in this repository mostly use multiplication factors to increase the score of some search results over others, which is an
 easy way to
 demonstrate how content score work, but for production usage more advanced calculation methods, described in the Elasticsearch
  documentation page, shall be applied._
 
 ## Table of content
 
 - [Presentation](#presentation)
 - [Boost contents of a given type](#Boost contents of a given type)
 - [Boost a specific content](#Boost a specific content)
 - [Apply boosts based on the sections of a website](#Apply boosts based on the sections of a website)
 - [Take the publication date into account](#Take the publication date into account)
 - [Personalized search](#Personalized search)
 
 
 
 ## Presentation
Function Score queries can be declared in a `augmented-search-score-functions.json` file, located under the `src/main/resources/META-INF
/` folder of a module.

### List all the available function score queries

The following GraphQL query lists the ID of all the available Function Score queries of your Jahia platform:

        query{
          admin {
            search{
              functionScore{
                functions {
                  nodes {
                    id
                  }
                }
              }
            }
          }
        }
 
 ### Usage
 Simply passe the function score ID (retrieved with the above query) as a parameter of your search query:
 
         query{
           search(
             q: "searched terms"
             language: "en"
             searchIn: [CONTENT, FILES]
             workspace: LIVE
             functionScoreId: "function-score-examples--contentTypeBoost"
           ) {
             results {
               totalHits
               hits {
                 displayableName
                 nodeType
                 path
                 score
               }
             }
           }
         }

 ## Boost contents of a given type
The [contentTypeBoost](/src/main/resources/META-INF/augmented-search-score-functions.json#L2-L25) function score boosts files (jnt:file) and news(jnt:news) over the other contents returned by the search query. The file scores are
 multiplied by 10, while the scores of news are multiplied by 5.
 
 In this example you can see how to boost contents of a given type, and that in the same function you can apply different boosts by
  providing different functions and filters. The `score_mode` parameter defines the strategy to use when there are several functions. In
   the contentTypeBoost example, the value is `first` as results cannot be both of types file and news.

 ## Boost a specific content
The [contentBoost](/src/main/resources/META-INF/augmented-search-score-functions.json#L26-L44) function score is used to allow content editors to manually boost pages, main resource contents (contents that can be displayed as full
 page) and files.

To do so, this example module comes with the [jmix:boostContentInSearch mixin](/src/main/resources/META-INF/definitions.cnd) that can be
 activated by editors in Content Editor. Once activated, editors can provide a boost number. This number is retrieved by the contentBoost
  function score, and used to multiply the result score, in order to increase it (if the input boost is >1), or reduce it (if the input
   boost is between 0 and 1). It has to be a positive value.
   
Please note that you need to add `jmix:boostContentInSearch` to the indexing configurations ([how to do edit the configuration file
](https://academy.jahia.com/documentation/developer/augmented-search/3/overview-and-setup/administering-augmented-search)):
* `org.jahia.modules.augmentedsearch.content.mappedNodeTypes`
* `org.jahia.modules.augmentedsearch.file.mappedNodeTypes`

_Note: to improve and simplify the editor experience, it would probably be better to implement a drop down with a couple options (e.g
. "reduce", "min", "medium", "max") so the editor does not have to figure out which value to use._
 
 ## Apply boosts based on the sections of a website
The [sectionBoost](/src/main/resources/META-INF/augmented-search-score-functions.json#L61-L76) function score example boosts contents based on their path. This allows to apply different boosts on the different sections of a
 website. To do so, it relies on a regex done on the indexed `jgql:nodePath` property.

You will also note that it has `min_score` value of 50, meaning that results with a score below 50 (thus, which contain the search terms
) are not returned by the query. Setting a `min_score` value is a way to ensure that only relevant results are returned. The `min_score
` value to use depends on your data set, and the different boost and function score features put in place.

 ## Take the publication date into account
The [lastPublication](/src/main/resources/META-INF/augmented-search-score-functions.json#L45-L60) function score reduces the score of older contents (based on the last publication date). This allows to give more importance to the
 most recently published contents. For instance, if a search query returns two pages with the same relevance, the last one to be
  published will appear higher in the search results.

_Note that due to a bug in Augmented Search 3.3.0, to use this function score with main resources and files, you need to add `jmix
:lastPublished` to the indexing configurations ([how to do edit the configuration file](https://academy.jahia.com/documentation/developer
/augmented-search/3/overview-and-setup/administering-augmented-search)):_
* `org.jahia.modules.augmentedsearch.content.mappedNodeTypes`
* `org.jahia.modules.augmentedsearch.file.mappedNodeTypes`

 ## Personalized search
 It is possible to use function score queries to provide some level of personalized search. The idea here is to personalize the search
  form/query to use one function score or another, based on the visitor's interests.
  
 Let's say that in the [sportInterestBoost](/src/main/resources/META-INF/augmented-search-score-functions.json#L77-L88) example, a
  marketer set a `sport` interest on a page/news. Then, a visitor who already have a high interest in sports, search for something, then
   the search form applies the sportInterestBoost, to make the content associated with the `sport` interest appear higher in the search
    results.
  
Please note that you need to add `wemmix:wemInterests` to the indexing configuration ([how to do edit the configuration file
](https://academy.jahia.com/documentation/developer/augmented-search/3/overview-and-setup/administering-augmented-search)) `org.jahia
.modules.augmentedsearch.content.mappedNodeTypes`


[reference]: #Take the publication date or other dates into account