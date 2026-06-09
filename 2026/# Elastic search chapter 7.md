# Elastic search chapter 7 

---

## 7-2. 분석기 (Analyzer)

### 1. 분석기 모듈의 역할

- 토큰화: 특정 규칙에 따라 문장 → 개별 단어로 분할 (by 토크나이저)
- 정규화: 토큰의 어간 추출, 동의어/불용어(stop word) 처리 → 가공 변환 수정 강화
- 불용어: a, an, the, and, is, but

### 2. 분석기 구조

- **문자 필터** (optional)
  - 문자 수준에 적용
  - 원하지 않는 문자 제거
  - `<h1>`, `<href>`, `<src>` 등 html 태그 제거
  - 특정 텍스트를 다른 텍스트로 변환
  - 정규식의 텍스트를 일치
- **토크나이저** (essential)
  - 구분 기호(공백, 구두점, 단어 경계 등)을 활용해 텍스트 필드를 단어로 분할
  - 분석기의 필수 구성 요소
- **토큰 필터**
  - 토큰에 대한 추가 처리 (대소문자 변환 / 동의어 / 어근 / n-그램 / shingle 생성)

### 3. 분석기 테스트: `_analyze` 엔드포인트 활용

```javascript
GET _analyze
{
  "text": "James Bond 007",
  "analyzer": "simple"
}

// => 출력: "james" "bond"
```

```javascript
GET _analyze
{
  "tokenizer": "path_hierarchy",
  "filter": ["uppercase"],
  "text": "/Volumes/FILES/Dev"
}

// => 출력
{
  "tokens": [
    { "token": "/VOLUMES", "start_offset": 0, "end_offset": 20, "type": "word", "position": 0 },
    { "token": "/VOLUMES/FILES", "start_offset": 0, "end_offset": 20, "type": "word", "position": 0 },
    { "token": "/VOLUMES/FILES/DEV", "start_offset": 0, "end_offset": 20, "type": "word", "position": 0 }
  ]
}
```

---

## 7-7. 토크나이저 (Tokenizer)

- 특정 기준에 따라 토큰 생성
- 입력 필드 → (개별 단어인) 토큰으로 분할

### 1. standard 토크나이저

- 단어 경계(공백 등), 구두점 기준으로 분할
- `max_token_length`: 정의된 크기의 토큰 생성 (default: 255)

### 2. ngram과 edge_ngram 토크나이저

- 철자와 잘못된 단어를 수정하기 위해 활용
- default: 최소 크기 1, 최대 크기 2
- **n-gram**: 단어가 요청받은(셋팅된) 크기로 나눈 단어 시퀀스
  - bi-gram: "co", "of", "ff", "fe", "ee"
  - tri-gram: "cof", "off", "ffe", "fee"
- **edge n-gram**: 단어의 시작 부분에 문자가 고정된 단어
  - "c", "co", "cof", "coff", "coffe", "coffee"

```javascript
PUT index_with_ngram_tokenizer
{
  "settings": {
    "analysis": {
      "analyzer": {
        "ngram_analyzer": {
          "tokenizer": "ngram_tokenizer"
        }
      },
      "tokenizer": {
        "ngram_tokenizer": {
          "type": "ngram",
          "min_gram": 2,
          "max_gram": 3,
          "token_chars": ["letter"]
        }
      }
    }
  }
}

POST index_wtih_ngram_toenizer/_analyze
{
  "text": "Bond",
  "tokenizer": "ngram_analzyer"
}

// => 출력: "bo" "bon" "on" "ond" "nd"
```

### 3. 기타 토크나이저

- `whitespace`: 공백 기준 분할
- `uax_url_email`: input 내 url/email은 단독 (keyword) 토큰 + 나머지 토큰화
- `pattern`: 정규식 활용해 토큰 분할 (default: 비단어 문자 기준 분할)
- `keyword`: 분할 없이 문장/단어를 하나의 토큰 그대로
- `lowercase`: 비단어 문자 기준 분할 + 소문자화
- `path_hierarchy`: 계층적 텍스트를 경로 구분 기호에 따라 분할

---

## 7-8. 토큰 필터 (Token Filter)

- 토큰 소문자/대문자화, 동의어 제공, 어간 추출, 특정 문자/기호 제거

<details>
<summary>ex: _analyze API 호출할 때, filter 적용</summary>

```javascript
GET _analyze
{
  "tokenizer": "standard",
  "filter": ["uppercase", "reverse"],
  "text": "bond"
}

// => 출력: DNOB
```

</details>

<details>
<summary>ex: custom 토큰 필터 사용</summary>

```javascript
PUT index_with_token_filters
{
  "settings": {
    "analysis": {
      "analyzer": {
        "token_filter_analyzer": {
          "tokenizer": "standard",
          "filter": ["uppercase", "reverse"]
        }
      }
    }
  }
}
```

</details>

### 1. 스테머 필터 (Stemmer)

- **stem**은 영어로 **줄기, 어간**이라는 뜻
- 어간 추출: 단어 → 어근
- default 적용

```javascript
POST _analyze
{
  "tokenizer": "standard",
  "filter": ["stemmer"],
  "text": "barking is my life"
}

// => 출력: "bark" "is" "my" "life"
```

### 2. shingle 필터

- **shingle**은 원래 영어로 **지붕에 얹는 얇은 판자/기와 조각** 같은 뜻
- 여러 단어를 겹치듯이 묶어서 단어구를 만드는 것 (n-gram)

```javascript
PUT index_with_shingle
{
  "settings": {
    "analysis": {
      "analyzer": {
        "shingles_analyzer": {
          "tokenizer": "standard",
          "filter": ["shingles_filter"]
        }
      },
      "filter": {
        "shingles_filter": {
          "type": "shingle",
          "min_shingle_size": 2,
          "max_shingle_size": 3,
          "output_unigrams": false  // 단일단어의 출력을 끈다
        }
      }
    }
  }
}

POST index_with_shingle/_analyze
{
  "text": "java python go",
  "analyzer": "shingles_analyzer"
}

// => 출력: ["java python", "java python go", "python go"]
```

### 3. synonym 필터

```javascript
PUT index_with_synonyms
{
  "settings": {
    "analysis": {
      "filter": {
        "synonyms_filter": {
          "type": "synonym",
          "synonyms": ["soccer => football"]
        }
      }
    }
  }
}

POST index_with_synonyms/_analyze
{
  "text": "What's soccer?",
  "tokenizer": "standard",
  "filter": ["synonyms_filter"]
}

PUT index_with_synonyms_from_file_analyzer
{
  "settings": {
    "analysis": {
      "analyzer": {
        "synonyms_analyzer": {
          "type": "standard",
          "filter": ["synonyms_from_file_filter"]
        }
      },
      "filter": {
        "synonyms_from_file_filter": {
          "type": "synonym",
          "synonyms_path": "synonyms.txt"
        }
      }
    }
  }
}
```