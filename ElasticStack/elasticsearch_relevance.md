# Elasticsearch의 Relevance

> 관련 자료 링크
>
> - [elasticsearch relevance 공식 레퍼런스](https://www.elastic.co/guide/en/elasticsearch/guide/current/relevance-intro.html#relevance-intro)
>
> - [BM25  - elasticsearch에서 검색하는 새로운 방법](https://www.popit.kr/bm25-elasticsearch-5-0%EC%97%90%EC%84%9C-%EA%B2%80%EC%83%89%ED%95%98%EB%8A%94-%EC%83%88%EB%A1%9C%EC%9A%B4-%EB%B0%A9%EB%B2%95/)
> - [elasticsearch로 알아보는 BM알고리즘1](https://inyl.github.io/search_engine/2017/04/01/bm25.html)
> - [elasticsearch로 알아보는 BM알고리즘2](https://inyl.github.io/search_engine/2017/04/18/bm25_2.html)

elasticsearch에서 score 값을 계산하는 것을 relevance라고 하고, 정확도라고 번역할 수 있다. 즉, 검색의 정확도를 뜻하며 기본적으로 relevance 알고리즘을 사용한다.

검색엔진에서 가장 일반적으로 사용되는 score 알고리즘에는 TF-IDF와 BM25가 있다. 가장 널리 사용되는 오픈소스 검색엔진인 lucene은 기존에 TF-IDF를 조금 변형한 형태의 스코어 알고리즘을 사용하고 있었으나, 최근 버전에서는 default를 BM25 알고리즘으로 변경했다. 따리서 lucene을 코어로 사용중인 apache solr와 elasticsearch도 최근 버전에서는 모두 BM25를 기본 score 알고리즘으로 사용하고 있다.

## The standard similarity algorithm used in Elasticsearch

- TF(Term Frequency)
  - 문서 내 발생한 term 빈도수가 클수록 weight가 높다. 문서내에서 같은 단어가 여러번 등장한다면 그 단어에 높은 가중치를 주는 알고리즘이다.
- IDF(Inverse Document Frequency)
  - 전체 문서에서 발생한 term 빈도수가 작을수록 weight가 높다. 문서에 자주 등장하는 단어일수록 낮은 가중치를 주는 알고리즘이다.
  - 똑같이 1번 검색이 되었다 하더라도 문서에 자주 등장한 단어가 매칭된 키워드일수록 낮은 가중치를 가지게 된다.
  - 문서에 많이 나오는게 좋은게 아닌가? 라고 생각할 수 있겠지만 문서에 공통적으로 많이 등장하는 단어는 실제 우리가 쓰는 단어로 살펴본다면 "은", "는", "다"처럼 형용사, 부사등이 되며 이는 실제로 큰 의미를 가지지 않을 확률이 높다.
- Field-length norm
  - 짧은 필드에 나타나는 term은 긴 필드에 나타나는 동일한 term보다 더 많은 weight를 가진다.

## Elasticsearch explain

- search쿼리에 `"explain": true` 옵션을 사용하면 내부에서 스코어를 어떤식으로 처리하고 있는지 확인 가능하다.

```json
GET library/_search
{
  "query": {
    "match": {
      "title": "fox"
    }
  }
  , "explain": true
}
```

```json
{
  "took": 48,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 4,
    "max_score": 0.2876821,
    "hits": [
      {
        "_shard": "[library][3]",
        "_node": "TvAwCthZQOqm2LAWqsXCuA",
        "_index": "library",
        "_type": "books",
        "_id": "1",
        "_score": 0.2876821,
        "_source": {
          "title": "The quick brow fox",
          "price": 5,
          "colors": [
            "red",
            "green",
            "blue"
          ]
        },
        "_explanation": {
          "value": 0.2876821,
          "description": "weight(title:fox in 0) [PerFieldSimilarity], result of:",
          "details": [
            {
              "value": 0.2876821,
              "description": "score(doc=0,freq=1.0 = termFreq=1.0\n), product of:",
              "details": [
                {
                  "value": 0.2876821,
                  "description": "idf, computed as log(1 + (docCount - docFreq + 0.5) / (docFreq + 0.5)) from:",
                  "details": [
                    {
                      "value": 1,
                      "description": "docFreq",
                      "details": []
                    },
                    {
                      "value": 1,
                      "description": "docCount",
                      "details": []
                    }
                  ]
                },
                {
                  "value": 1,
                  "description": "tfNorm, computed as (freq * (k1 + 1)) / (freq + k1 * (1 - b + b * fieldLength / avgFieldLength)) from:",
                  "details": [
                    {
                      "value": 1,
                      "description": "termFreq=1.0",
                      "details": []
                    },
                    {
                      "value": 1.2,
                      "description": "parameter k1",
                      "details": []
                    },
                    {
                      "value": 0.75,
                      "description": "parameter b",
                      "details": []
                    },
                    {
                      "value": 4,
                      "description": "avgFieldLength",
                      "details": []
                    },
                    {
                      "value": 4,
                      "description": "fieldLength",
                      "details": []
                    }
                  ]
                }
              ]
            }
          ]
        }
      },
      
        ...
```

#### IDF(Inverse Document Frequency)

- 전체 문서에서 발생한 term 빈도수가 작을수록 weight가 높다. 문서에 자주 등장하는 단어일수록 낮은 가중치를 주는 방법이다. 똑같이 1번 검색이 되었다 하더라도 문서에 자주 등장한 단어가 매칭된 키워드 일수록 낮은 가중치를 가지게 된다.

- `_explanation` > `details` > `details`
- `description`에 있는 공식이 IDF 점수를 구하는 공식이다.
- docCount가 전체 문서의 수이고,
- docFreq가 문서에 나타난 키워드 수이다.

```json
...
{
    "value": 0.2876821,
    "description": "idf, computed as log(1 + (docCount - docFreq + 0.5) / (docFreq + 0.5)) from:",
    "details": [
        {
            "value": 1,
            "description": "docFreq",
            "details": []
        },
        {
            "value": 1,
            "description": "docCount",
            "details": []
        }
    ]
},
...
```

#### TF(Term Frequency)

- 문서 내 발생한 term 빈도수가 클수록 weight가 높다. 문서내에서 같은 단어가 여러번 등장한다면 그 단어에 높은 가중치를 주는 알고리즘이다.
- `_explanation` > `details` > `details`
  - termFreq = 문서에 매칭된 키워드 수
  - k1 = es에서는 default 1.2 혹은 2.0을 사용
  - b = es default는 0.75
  - avgFieldLength = 평균 필드의 길이를 의미
  - fieldLength = 실제 문서가 검색된 문서의 길이

```json
...
{
    "value": 1,
    "description": "tfNorm, computed as (freq * (k1 + 1)) / (freq + k1 * (1 - b + b * fieldLength / avgFieldLength)) from:",
    "details": [
        {
            "value": 1,
            "description": "termFreq=1.0",
            "details": []
        },
        {
            "value": 1.2,
            "description": "parameter k1",
            "details": []
        },
        {
            "value": 0.75,
            "description": "parameter b",
            "details": []
        },
        {
            "value": 4,
            "description": "avgFieldLength",
            "details": []
        },
        {
            "value": 4,
            "description": "fieldLength",
            "details": []
        }
    ]
}
...
```

- IDF와 TF의 스코어 값을 곱하면 최종스코어를 얻을 수 있다.
  - `0.2876821 * 1`

```json
	...

"_explanation": {
    "value": 0.2876821,
    "description": "weight(title:fox in 0) [PerFieldSimilarity], result of:",
    
    ...
```

