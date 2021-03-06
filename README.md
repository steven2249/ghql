ghql
====



[![Build Status](https://travis-ci.org/ropensci/ghql.svg?branch=master)](https://travis-ci.org/ropensci/ghql)

`ghql` - general purpose GraphQL client

GraphQL - <http://graphql.org>

Examples of GraphQL APIs:

* [Github GraphQL API](https://developer.github.com/early-access/graphql/)

## Github Authentication

See <https://developer.github.com/early-access/graphql/guides/accessing-graphql/> for getting an OAuth token.

Store the token in a env var called `GITHUB_GRAPHQL_TOKEN`
before trying this pkg.

## Install

Development version


```r
devtools::install_github("ropensci/ghql")
```


```r
library("ghql")
library("jsonlite")
```

## initialize client


```r
library("httr")
token <- Sys.getenv("GITHUB_GRAPHQL_TOKEN")
cli <- GraphqlClient$new(
  url = "https://api.github.com/graphql",
  headers = add_headers(Authorization = paste0("Bearer ", token))
)
```

## load schema

Since not every GraphQL server has a schema at the base URL, have to manually
load the schema in this case


```r
cli$load_schema()
```


## basic query

Make a `Query` class object


```r
qry <- Query$new()
```


```r
qry$query('myquery', 'query { }')
qry
#> <ghql: query>
#>   queries:
#>     myquery
qry$queries
#> $myquery
#>  
#>  query { }
qry$queries$myquery
#>  
#>  query { }
```


```r
cli$exec(qry$queries$myquery)
#> $data
#> named list()
```

Gives back no result, as we didn't ask for anything :)


## Get some actual data


```r
qry$query('getdozedata', '{
  repositoryOwner(login:"sckott") {
    repositories(first: 5, orderBy: {field:PUSHED_AT,direction:DESC}, isFork:false) {
      edges {
        node {
          name
          stargazers {
            totalCount
          }
        }
      }
    }
  }
}')
qry
#> <ghql: query>
#>   queries:
#>     myquery    
#>     getdozedata
qry$queries$getdozedata
#>  
#>  {
#>   repositoryOwner(login:"sckott") {
#>     repositories(first: 5, orderBy: {field:PUSHED_AT,direction:DESC}, isFork:false) {
#>       edges {
#>         node {
#>           name
#>           stargazers {
#>             totalCount
#>           }
#>         }
#>       }
#>     }
#>   }
#> }
```


```r
cli$exec(qry$queries$getdozedata)
#> $data
#> $data$repositoryOwner
#> $data$repositoryOwner$repositories
#> $data$repositoryOwner$repositories$edges
#>             node.name node.totalCount
#> 1   sckott.github.com               8
#> 2 usdaplantsapistatus               0
#> 3       usdaplantsapi              19
#> 4            rforcats              34
#> 5             soylocs               2
```

## run a local GraphQL server

* Copy the `server.js` file from this package located at `inst/server.js` somewhere on your machine. Can locate it on your machine like `system.file("js/server.js", package = "ghql")`. Or you can run the file from where it's at, up to you.
* Make sure node is installed. If not, see <https://nodejs.org>
* Run `node server.js`
* Navigate to your browser - go to http://localhost:4000/graphql
* Back in R, user that URL to connect


```r
(cli <- GraphqlClient$new("http://localhost:4000/graphql"))
#> <ghql client>
#>   url: http://localhost:4000/graphql
```


```r
xxx <- Query$new()
xxx$query('query', '{
  __schema {
    queryType {
      name, 
      fields {
        name,
        description
      }
    }
  }
}')
```



```r
cli$exec(xxx$queries$query)
#> $data
#> $data$`__schema`
#> $data$`__schema`$queryType
#> $data$`__schema`$queryType$name
#> [1] "Query"
#> 
#> $data$`__schema`$queryType$fields
#>    name description
#> 1 hello            
#> 2  name 
```

## Meta

* Please note that this project is released with a [Contributor Code of Conduct](CONDUCT.md). By participating in this project you agree to abide by its terms.
