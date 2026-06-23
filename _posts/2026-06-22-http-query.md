---
title: "HTTP QUERY Method"
date: 2026-06-22
---

## GET의 한계

HTTP에서 데이터를 조회할 때는 `GET`을 사용합니다. `GET`에 조회 조건을 포함하려면 아래와 같이 URL에 담을 수 있습니다.

```
GET /search?q=hello&category=tech&sort=date&order=desc&page=1&size=20&lang=ko&from=2024-01-01&to=2024-12-31&author=서웅덕
```

쿼리스트링은 `?`로 시작하고 조건들을 `&`로 이어 붙이는 방식이라 조건이 늘어날수록 그대로 길어집니다.
단순한 조건이라면 괜찮지만, 조건이 복잡해질수록 아래와 같은 문제들이 생길 수 있습니다.

- URI 길이 제한: 브라우저와 서버마다 URI 길이 제한이 있어 조건이 많아지면 요청 자체가 실패할 수 있습니다

  | 종류 | 이름 | 제한 | 출처 |
  |---|---|---|---|
  | Browser | Chrome | 주소창 기준 약 2,048자 | [Link](https://www.lineserve.net/blog/ultimate-guide-to-url-length-limits-browsers-http-specs-and-best-practices) |
  | Browser | Firefox | 약 65,536자 | [Link](https://www.lineserve.net/blog/ultimate-guide-to-url-length-limits-browsers-http-specs-and-best-practices) |
  | Browser | Safari | 약 80,000자 | [Link](https://www.lineserve.net/blog/ultimate-guide-to-url-length-limits-browsers-http-specs-and-best-practices) |
  | Web Server | Apache | 기본 8,190바이트 | [Link](https://httpd.apache.org/docs/2.4/mod/core.html) |
  | Web Server | Nginx | 기본 8,192바이트 | [Link](https://nginx.org/en/docs/http/ngx_http_core_module.html) |
- 로그 노출: URL은 텍스트 그대로 여러 곳에 기록되기 때문에 민감한 검색 조건이 노출될 수 있습니다
  - 서버 로그: 웹 서버는 들어오는 요청 URL을 로그로 남깁니다
  - 브라우저 히스토리: 주소창에 나타나는 URL이 방문 기록에 저장됩니다
  - 프록시 로그: 중간에 프록시 서버가 있으면 거기에서도 URL이 기록됩니다
- 인코딩 필요: URL에는 알파벳, 숫자, 일부 특수문자만 쓸 수 있어 그 외의 문자는 `%`와 16진수로 변환해야 합니다
  - 위 URL의 `author=서웅덕`은 실제로 `author=%EC%84%9C%EC%9B%85%EB%8D%95`으로 변환되어 전송됩니다
  - JSON 구조처럼 복잡한 조건을 담을수록 가독성이 크게 떨어지고 디버깅도 어려워집니다

### POST로 우회하기

`POST`는 URL 대신 요청 본문에 조건을 담기 때문에 `GET`의 한계를 피할 수 있습니다.<br>
대표적인 사례로 Elasticsearch가 있습니다. 검색 조건을 JSON으로 표현하는 특성상 구조가 복잡해져 URL에 담을 수 없어, 처음부터 `POST`와 JSON body 방식을 채택했습니다.

```
POST /search
{
  "query": {
    "bool": {
      "must": [{ "match": { "title": "hello" } }],
      "filter": [
        { "term": { "category": "tech" } },
        { "term": { "author": "서웅덕" } },
        { "term": { "lang": "ko" } },
        { "range": { "date": { "gte": "2024-01-01", "lte": "2024-12-31" } } }
      ]
    }
  },
  "sort": [{ "date": "desc" }],
  "from": 0,
  "size": 20
}
```

하지만 `POST`를 조회 목적으로 쓰면 다른 문제가 생깁니다.

- 캐싱 불가: 대부분의 브라우저와 프록시는 `POST` 응답을 캐싱하지 않습니다
- 자동 재시도 불가: `POST`는 멱등성이 없기 때문에 네트워크 오류 발생 시 자동으로 재시도할 수 없습니다
- 의미가 명확하지 않음: 이 `POST`가 조회인지 변경인지 코드만 봐서는 바로 알기 어렵습니다

## QUERY 메서드

`QUERY`는 `GET`과 `POST`의 장점을 합친 새로운 HTTP 메서드입니다. `POST`처럼 요청 본문에 조건을 담을 수 있으면서, `GET`처럼 안전하고 멱등적입니다.

- 안전: 서버 상태를 변경하지 않기 때문에 부작용이 없습니다
- 멱등: 같은 요청을 여러 번 보내도 서버 상태가 동일하게 유지됩니다

```
QUERY /search
Content-Type: application/json

{
  "query": {
    "bool": {
      "must": [{ "match": { "title": "hello" } }],
      "filter": [
        { "term": { "category": "tech" } },
        { "term": { "author": "서웅덕" } },
        { "term": { "lang": "ko" } },
        { "range": { "date": { "gte": "2024-01-01", "lte": "2024-12-31" } } }
      ]
    }
  },
  "sort": [{ "date": "desc" }],
  "from": 0,
  "size": 20
}
```

위 두 가지 특성 덕분에 `QUERY`는 `GET`과 `POST` 모두가 가지지 못했던 것들을 동시에 갖습니다. 요청 본문에 복잡한 조건을 담으면서도, 응답을 캐싱할 수 있고 네트워크 오류 시 자동으로 재시도할 수 있습니다.
