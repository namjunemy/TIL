# Elasticsearch APIs

> 김종민 - Elastic 테크 에반젤리스트
>
> **Reference**
>
> - https://www.elastic.co/kr/webinars/getting-started-elasticsearch
> - http://kimjmin.net

## Index 생성

- library 인덱스 생성
  - shard 수 : 5
  - replica 수 : 1

```json
PUT library
{
  "settings" : {
    "number_of_shards": 5,
    "number_of_replicas": 1
  }
}
```

## Bulk 색인

- 다량의 도큐먼트를 한꺼번에 색인 할 때는 bulk API를 사용

```json
POST library/books/_bulk
{"index":{"_id":1}}
{"title":"The quick brow fox", "price":5, "colors":["red", "green", "blue"]}
{"index":{"_id":2}}
{"title":"The quick brow fox jumps over the lazy dog", "price":15, "colors":["blue", "yellow"]}
{"index":{"_id":3}}
{"title":"The quick brow fox jumps over the quick dog", "price":8, "colors":["red", "blue"]}
{"index":{"_id":4}}
{"title":"brow fox brown dog", "price":2, "colors":["black", "yellow", "red", "blue"]}
{"index":{"_id":5}}
{"title":"Lazy dog", "price":9, "colors":["red", "blue", "green"]}
```

## 전체 도큐먼트 검색

- 기본적으로 _search의 옵션을 주지 않으면 기본적으로 인덱스의 전체 도큐먼트를 검색한다.
- 이때는 score 검색을 하지 않아서, 모든 score는 1로 동등하다.

```json
GET library/_search
{
  "query": {
    "match_all": {}
  }
}
```

```json
GET library/_search
```

## 맵핑 정보 검색

- RDB의 스키마에 해당하는 Elasticsearch의 맵핑정보를 검색한다.
- 리턴되는 값을 참조하면
- library라는 인덱스 밑에 books라는 도큐먼트가 있고, 도큐먼트에는 3개의 필드 colors, price, title이 존재한다.
- 필드안에는 타입과 하위 필드인 keyword 타입이 있다. keyword 타입은 aggregation에 쓰이는 필드를 저장하고, 이 내용들은 분석을 하지 않는다. 즉, 검색어로 쪼개지 않고 저장한다.

```json
GET library/_mapping
```

```json
{
  "library": {
    "mappings": {
      "books": {
        "properties": {
          "colors": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "price": {
            "type": "long"
          },
          "title": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          }
        }
      }
    }
  }
}
```

## 키워드가 포함된 도큐먼트 검색

- match query 사용해서 title필드에 fox가 포함된 도큐먼트들을  리턴

```json
GET library/_search
{
  "query": {
    "match": {
      "title": "fox"
    }
  }
}
```

## 복수의 키워드가 포함된 도큐먼트 검색

- match query 사용, " "(공백)으로 분리

```json
GET library/_search
{
  "query": {
    "match": {
      "title": "quick dog"
    }
  }
}
```

- 위의 검색 결과에서 중요한 점은, 검색의 정확도를 표현하는 **score 값이 리턴** 된다.
- 검색의 타겟인 quick과 dog이 많이 매칭 될수록 score 값이 높아진다.
  - [Elasticsearch에서 검색의 score(정확도)를 계산하는 방법]()

```json
{
  "took": 64,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 5,
    "max_score": 0.7564473,
    "hits": [
      {
        "_index": "library",
        "_type": "books",
        "_id": "2",
        "_score": 0.7564473,
        "_source": {
          "title": "The quick brow fox jumps over the lazy dog",
          "price": 15,
          "colors": [
            "blue",
            "yellow"
          ]
        }
      },
        
      ...
```

## 구문이 포함된 도큐먼트 검색

- match_phrase query 사용, 해당 문자열이 포함된 도큐먼트 리턴

```json
GET library/_search
{
  "query": {
    "match_phrase": {
      "title": "quick dog"
    }
  }
}
```

## 복합 쿼리 - bool 쿼리를 이용한 서브쿼리 조합

### must

* "quick" 과 "lazy dog" 이 포함된 모든 문서 검색

```json
GET /library/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "title": "quick"
          }
        },
        {
          "match_phrase": {
            "title": {
              "query": "lazy dog"
            }
          }
        }
      ]
    }
  }
}
```

### must_not

* "quick" 또는 "lazy dog" 이 포함되지 않은 문서 검색

```json
GET /library/_search
{
  "query": {
    "bool": {
      "must_not": [
        {
          "match": {
            "title": "quick"
          }
        },
        {
          "match_phrase": {
            "title": {
              "query": "lazy dog"
            }
          }
        }
      ]
    }
  }
}
```

## 특정 쿼리에 대한 가중치 조절(boost)

### should

* OR(||)랑 비슷하지만 같지는 않다.

* 반드시 매칭 될 필요는 없지만, 매칭 되는 경우 더 높은 스코어를 가짐
* 기본 boost값은 1이다. 아래의 경우 `"lazy dog"` 에 가중치를 3으로 설정했으므로 `quick dog`
*  를 포함하고 있는 도큐먼트보다

```json
GET /library/_search
{
  "query": {
    "bool": {
      "should": [
        {
          "match_phrase": {
            "title": "quick dog"
          }
        },
        {
          "match_phrase": {
            "title": {
              "query": "lazy dog",
              "boost": 3
            }
          }
        }
      ]
    }
  }
}
```

### must + should

* must와 should를 같이 사용하게 되면, must를 우선적으로 매칭하고 must 조건을 만족하는 도큐먼트 중에서 should 쿼리의 매칭 여부에 따라 score가 결정 된다.

```json
GET /library/_search
{
  "query": {
    "bool": {
      "should": [
        {
          "match_phrase": {
            "title": "lazy"
          }
        }
      ],
      "must": [  
        {
          "match_phrase": {
            "title": {
              "query": "dog"
            }
          }
        }
      ]
    }
  }
}
```

## Highlight - 검색어와 매칭 된 부분을 하이라이트로 표시

* 검색 결과값이 크고 여러 필드를 사용하는 경우 유용하다

### highlight

* 검색결과 해당 단어와 매칭되는 단어를 highlight 표시

```json
GET /library/_search
{
  "query": {
    "bool": {
      "should": [
        {
          "match_phrase": {
            "title": "lazy"
          }
        },
        {
          "match_phrase": {
            "title": {
              "query": "dog"
            }
          }
        }
      ]
    }
  },
  "highlight" : {
    "fields" : {
      "title" : {}
    }
  }
}
```

* 검색 후 리턴 값의 highlight속성에는 em태그로 해당 키워드가 표시 된다.

```json
{
  "took": 110,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 4,
    "max_score": 0.7564473,
    "hits": [
      {
        "_index": "library",
        "_type": "books",
        "_id": "2",
        "_score": 0.7564473,
        "_source": {
          "title": "The quick brow fox jumps over the lazy dog",
          "price": 15,
          "colors": [
            "blue",
            "yellow"
          ]
        },
        "highlight": {
          "title": [
            "The quick brow fox jumps over the <em>lazy</em> <em>dog</em>"
          ]
        }
      },
        
        ...
```

 ## Filter - 검색 결과의 sub-set 도출

* 스코어를 계산하지 않고 캐싱되어 쿼리보다 대부분 빠르다

### (bool) must + filter

* filter는 bool조건의 쿼리에서 사용할 수 있고, 아래의 쿼리는 `"dog"` 을 포함하고 있는 도큐먼트 중에서 price의 range가 5이상 10이하의 도큐먼트 들만 리턴한다.
* 알아둬야 할 것은 filter 구문을 제거하고 search만 수행했을 경우의 score와 filter를 설정했을 때의 score가 같다는 것이다. 스코어를 다시 계산하지 않고 검색 결과가 나온것 중에서 sub-set만 도출하기 때문에 쿼리보다 빠르다.

```json
GET /library/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "title": "dog"
          }
        }
      ],
      "filter": {
        "range": {
          "price": {
            "gte": 5,
            "lte": 10
          }
        }
      }
    }
  }
}
```

### score가 필요 없는 경우 filter만 사용

* score가 필요없이 특정 필드의 숫자 범위를 판단하는 상황에서는 filter만 사용할 수 있다. 이때는 true, false로 판단만 하게 된다.
* 결과는 price가 5보다 큰(>5) 도큐먼트들만 리턴된다

```json
GET /library/_search
{
  "query": {
    "bool": {
      "filter": {
        "range": {
          "price": {
            "gt": 5
          }
        }
      }
    }
  }
}
```

## 분석 - Analysis(_analyze)

### Tokenizer를 통해 문장을 검색어 term으로 분리

* text로부터 분리된 offset과 type, position 정보 표시

```json
GET library/_analyze
{
  "tokenizer": "standard",
  "text": "Brown fox brown dog"
}
```

```json
{
  "tokens": [
    {
      "token": "Brown",
      "start_offset": 0,
      "end_offset": 5,
      "type": "<ALPHANUM>",
      "position": 0
    },
    {
      "token": "fox",
      "start_offset": 6,
      "end_offset": 9,
      "type": "<ALPHANUM>",
      "position": 1
    },
    {
      "token": "brown",
      "start_offset": 10,
      "end_offset": 15,
      "type": "<ALPHANUM>",
      "position": 2
    },
    {
      "token": "dog",
      "start_offset": 16,
      "end_offset": 19,
      "type": "<ALPHANUM>",
      "position": 3
    }
  ]
}
```

### Filter(토큰필터)를 통해 쪼개진 term들을 가공

* lowercase - 소문자로 변경

```json
GET library/_analyze
{
  "tokenizer": "standard",
  "filter": [
    "lowercase"
  ],
  "text": "Brown fox brown dog"
}
```

```json
{
  "tokens": [
    {
      "token": "brown",
      "start_offset": 0,
      "end_offset": 5,
      "type": "<ALPHANUM>",
      "position": 0
    },
      ...
```

* unique - 중복 term 제거
  * 아래의 필터에서 lowercase를 제거하면, 대소문자를 구분하게 된다.

```json
GET library/_analyze
{
  "tokenizer": "standard",
  "filter": [
    "lowercase",
    "unique"
  ],
  "text": "Brown brown brown fox brown dog"
}
```

```json
{
  "tokens": [
    {
      "token": "brown",
      "start_offset": 0,
      "end_offset": 5,
      "type": "<ALPHANUM>",
      "position": 0
    },
    {
      "token": "fox",
      "start_offset": 18,
      "end_offset": 21,
      "type": "<ALPHANUM>",
      "position": 1
    },
    {
      "token": "dog",
      "start_offset": 28,
      "end_offset": 31,
      "type": "<ALPHANUM>",
      "position": 2
    }
  ]
}
```

### (Tokenizer + Filter) 대신 Analyzer 사용

* 기본적인 standard analyzer는 lowercase와 standard tokenizer가 적용된다.

```json
GET library/_analyze
{
  "analyzer": "standard",
  "text": "Brown fox brown dog"
}
```

## 복합적인 문장 분석

### T: standard, F: lowercase

* 아래의 결과를 통해서 중요하게 알고 넘어가야 할 것은 standard tokenizer와 lowercase filter로 텍스트를 저장했을 경우, 결과 값으로 리턴되는 분리된 token으로 검색해야만 원래의 문장을 검색할 수 있다.

```json
GET library/_analyze
{
  "tokenizer": "standard",
  "filter": [
    "lowercase"  
  ],
  "text": "THE quick.brown_FOx jumped! $19.95 @ 3.0"
}
```

```json
{
  "tokens": [
    {
      "token": "the",
      "start_offset": 0,
      "end_offset": 3,
      "type": "<ALPHANUM>",
      "position": 0
    },
    {
      "token": "quick.brown_fox",
      "start_offset": 4,
      "end_offset": 19,
      "type": "<ALPHANUM>",
      "position": 1
    },
    {
      "token": "jumped",
      "start_offset": 20,
      "end_offset": 26,
      "type": "<ALPHANUM>",
      "position": 2
    },
    {
      "token": "19.95",
      "start_offset": 29,
      "end_offset": 34,
      "type": "<NUM>",
      "position": 3
    },
    {
      "token": "3.0",
      "start_offset": 37,
      "end_offset": 40,
      "type": "<NUM>",
      "position": 4
    }
  ]
}
```

### T:letter, F:lowercase

* letter tokenizer는 실제로 알파벳 문자가 아닌 모든것을 구분자로 판단한다. 따라서, lowercase의 분리된 문자열만 리턴된다.

```json
GET library/_analyze
{
  "tokenizer": "letter",
  "filter": [
    "lowercase"  
  ],
  "text": "THE quick.brown_FOx jumped! $19.95 @ 3.0"
}
```

```json
{
  "tokens": [
    {
      "token": "the",
      "start_offset": 0,
      "end_offset": 3,
      "type": "word",
      "position": 0
    },
    {
      "token": "quick",
      "start_offset": 4,
      "end_offset": 9,
      "type": "word",
      "position": 1
    },
    {
      "token": "brown",
      "start_offset": 10,
      "end_offset": 15,
      "type": "word",
      "position": 2
    },
    {
      "token": "fox",
      "start_offset": 16,
      "end_offset": 19,
      "type": "word",
      "position": 3
    },
    {
      "token": "jumped",
      "start_offset": 20,
      "end_offset": 26,
      "type": "word",
      "position": 4
    }
  ]
}
```

### Email, URL  분석

* T: standard

```json
GET library/_analyze
{
  "tokenizer": "standard",
  "text": "elastic@example.com website: https://elastic.co"
}
```

```json
{
  "tokens": [
    {
      "token": "elastic",
      "start_offset": 0,
      "end_offset": 7,
      "type": "<ALPHANUM>",
      "position": 0
    },
    {
      "token": "example.com",
      "start_offset": 8,
      "end_offset": 19,
      "type": "<ALPHANUM>",
      "position": 1
    },
    {
      "token": "website",
      "start_offset": 20,
      "end_offset": 27,
      "type": "<ALPHANUM>",
      "position": 2
    },
    {
      "token": "https",
      "start_offset": 29,
      "end_offset": 34,
      "type": "<ALPHANUM>",
      "position": 3
    },
    {
      "token": "elastic.co",
      "start_offset": 37,
      "end_offset": 47,
      "type": "<ALPHANUM>",
      "position": 4
    }
  ]
}
```

* T: uax_url_email

```json
{
  "tokens": [
    {
      "token": "elastic@example.com",
      "start_offset": 0,
      "end_offset": 19,
      "type": "<EMAIL>",
      "position": 0
    },
    {
      "token": "website",
      "start_offset": 20,
      "end_offset": 27,
      "type": "<ALPHANUM>",
      "position": 1
    },
    {
      "token": "https://elastic.co",
      "start_offset": 29,
      "end_offset": 47,
      "type": "<URL>",
      "position": 2
    }
  ]
}
```













