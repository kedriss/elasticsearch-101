## API de Recherche
Commençons par insérer quelques données ...

	curl -XPOST 'http://localhost:9200/heroes/person/ironman' -d '{"firstName":"Tony","lastName":"Stark","aka":"Iron Man","team":"Avengers","age":45}'
	curl -XPOST 'http://localhost:9200/heroes/person/thor' -d '{"firstName":"Thor","lastName":"Odinson","aka":"Thor","team":"Avengers","age":27}'
	curl -XPOST 'http://localhost:9200/heroes/person/antman' -d '{"firstName":"Hank","lastName":"Pym","aka":"Ant-Man","team":"Avengers","age":41}'
	curl -XPOST 'http://localhost:9200/heroes/person/wasp' -d '{"firstName":"Janet","lastName":"van Dyne","aka":"Wasp","team":"Avengers","age":32}'
	curl -XPOST 'http://localhost:9200/heroes/person/hulk' -d '{"firstName":"Bruce","lastName":"Banner","aka":"Hulk","team":"Avengers","age":41}'
	curl -XPOST 'http://localhost:9200/heroes/person/misterfantastic' -d '{"firstName":"Reed","lastName":"Richards","aka":"Mister Fantastic","team":"FantasticFour","age":47}'
	curl -XPOST 'http://localhost:9200/heroes/person/invisiblewoman' -d '{"firstName":"Susan","lastName":"Storm","aka":"Invisible Woman","team":"FantasticFour","age":29}'
	curl -XPOST 'http://localhost:9200/heroes/person/thehumantorch' -d '{"firstName":"Johnny","lastName":"Storm","aka":"The Human Torch","team":"FantasticFour","age":25}'
	curl -XPOST 'http://localhost:9200/heroes/person/thething' -d '{"firstName":"Ben","lastName":"Grimm","aka":"The Thing","team":"FantasticFour","age":42}'


L'API `_search` permet d'effectuer des [recherches](https://www.elastic.co/guide/en/elasticsearch/reference/current/search.html) dans ElasticSearch.

### Recherche
La requête suivante lance une recherche sur l'ensemble des documents de type `person` dans l'index `heroes` (par défault, une recherche remonte 10 résultats) :

	curl -XPOST 'http://localhost:9200/heroes/person/_search'

La requête suivante permet de rechercher tous ls documents qui ont un attribut `lastName` dont la valeur est `storm` :

	curl -XPOST 'http://localhost:9200/heroes/person/_search' -d '{
	    "query": {
	        "match": {
	           "lastName": "storm"
	        }
	    }
	}'


La requête suivante permet d'effectuer une recherche sur les documents dont le prénom commence par un `t` :

	curl -XPOST 'http://localhost:9200/heroes/person/_search' -d '{
	    "query": {
	        "wildcard": {
	           "firstName": {
	              "value": "t*"
	           }
	        }
	    }
	}

Il est possible de faire des recherches à plusieurs niveaux :

* sur un type et un index donnés : `localhost:9200/heroes/person/_search`
* sur l'ensemble des types d'un index donné : `localhost:9200/heroes/_search`
* sur l'ensemble des index d'un cluster : `localhost:9200/_search`

Effectuez maintenant quelques recherches à l'aide de requêtes de type [Query String Query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-query-string-query.html).

### Agrégations
Les [agrégations](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations.html) permettent de regrouper les données et d'effectuer des calculs sur les documents contenus dans les index.

Bien qu'il soit possible de combiner recherche et aggrégations, nous ne nous intéressons pas ici aux recherches (d'où l'attribut `"size": 0` ...).

Obtenir la répartition des valeurs du terme `team` dans les documents de type `person` de l'index `heroes` :

```
	curl -XPOST 'http://localhost:9200/heroes/person/_search' -d '{
	    "size": 0,
	    "aggs" : {
	        "teams" : {
	            "terms": {
	                "field": "team.keyword"
	            }
	        }
	    }
	}
```

Il est possible de faire des sous-agrégations. Obtenir la répartition des valeurs du terme `lastName` dans la répartion du terme `team` dans les documents de type `person` de l'index `heroes` :

	curl -XPOST 'http://localhost:9200/heroes/person/_search' -d '{
	    "size": 0,
	    "aggs" : {
	        "teams" : {
	            "terms": {
	                "field": "team.keyword"
	            },
				"aggs" : {
	                "names" : {
	                    "terms": {
	    	                "field": "lastName.keyword"
	    	            }
	                }
				}
	        }
	    }
	}

Il est possible de faire des calculs avec les aggrégations. L'âge moyen des membres de chaque équipe :

	curl -XPOST 'http://localhost:9200/heroes/person/_search' -d '{
	    "size": 0,
	     "aggs" : {
	         "teams" : {
	            "terms": {
	                "field": "team.keyword",
	                "order" : { "avgAge" : "desc" }
	            },
				"aggs" : {
	                "avgAge" : {
	                    "avg" : {
	                        "field" : "age"
	                    }
	                }
				}
	        }
	    }
	}

Quelques exercices complémentaires :

* Vous pouvez à présent écrire une agrégation qui calcule l'age maximum par équipe.
* Vous pouvez à présent écrire une agrégation qui calcule l'age maximum par équipe pour les personnages dont le nom commence par la lettre 't'.
* En utilisant les agrégations de type [Histogram](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-bucket-histogram-aggregation.html), créez une aggrégation permettant de regrouper les héros par tranche d'age (par dizaine).