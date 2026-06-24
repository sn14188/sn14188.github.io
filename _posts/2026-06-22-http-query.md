---
title: "HTTP QUERY Method"
date: 2026-06-22
---

`GET`의 URL 한계를 피하려 `POST`를 쓰면 캐싱과 멱등성 문제가 생깁니다. 이 간극을 메우기 위해 `QUERY` 메서드가 제안됐는데 이 포스트에서는 `QUERY`가 무엇인지, 어떻게 동작하는지 살펴봤습니다.
<br><br>

## `GET`의 한계

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

### `POST`로 우회하기

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

## `QUERY` 메서드

`QUERY`는 `GET`과 `POST`의 장점을 합친 새로운 HTTP 메서드입니다. `POST`처럼 요청 본문에 조건을 담을 수 있으면서, `GET`처럼 안전하고 멱등적입니다.<br>
여기서 요청 본문이란 HTTP 요청에서 실제 데이터를 담는 부분을 말합니다. `GET`은 조건을 URL에만 담을 수 있어 본문이 없지만, `POST`와 `QUERY`는 본문에 데이터를 담아 보낼 수 있습니다.

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

## `QUERY` 동작 방식

### 요청 형식

`QUERY`는 조건을 본문에 담아 보내는 것이 목적이기 때문에 실질적으로 항상 본문과 함께 사용됩니다. 본문이 있을 경우 `Content-Type` 헤더로 형식을 명시해야 하며, 서버는 이를 기준으로 본문을 파싱합니다.<br>
서버가 해당 `Content-Type`을 지원하지 않거나 본문을 처리할 수 없는 경우 아래 에러를 반환합니다.

- `415 Unsupported Media Type`: 서버가 해당 `Content-Type`을 지원하지 않는 경우
- `400 Bad Request`: 본문 형식이 잘못된 경우
- `422 Unprocessable Content`: 형식은 맞지만 처리할 수 없는 내용인 경우

### 캐싱

`QUERY` 응답은 캐싱할 수 있습니다. `QUERY`는 `GET`과 달리 URL이 같아도 본문이 다르면 다른 조회이기 때문에, 캐시는 URL뿐만 아니라 요청 본문과 `Content-Type`도 함께 고려해 응답을 구분합니다.

### CORS

브라우저는 CORS 요청을 보낼 때 안전한 요청인지 먼저 판단합니다. 아래 조건을 모두 만족하면 단순 요청으로 분류해 바로 전송하고, 그렇지 않으면 preflight라는 사전 확인 요청을 먼저 보냅니다.

- 메서드가 `GET`, `POST`, `HEAD` 중 하나
- 헤더가 `Accept`, `Content-Type` 등 허용된 것만 포함
- `Content-Type`이 `application/x-www-form-urlencoded`, `multipart/form-data`, `text/plain` 중 하나

`QUERY`는 메서드 자체가 이 목록에 없기 때문에 조건 불문 항상 preflight가 발생합니다.<br>
preflight는 `OPTIONS` 메서드로 서버에 해당 메서드를 허용하는지 먼저 확인하는 요청입니다. 서버가 허용한다고 응답하면 그때 실제 `QUERY` 요청을 보냅니다. 결과적으로 요청이 두 번 오가게 됩니다.<br>
따라서 `QUERY`를 쓰려면 서버의 CORS 설정에서 `QUERY` 메서드를 명시적으로 허용해야 합니다.

## Takeaways

1. `GET`은 조건을 URL에만 담을 수 있어 조건이 복잡해질수록 문제가 생길 수 있습니다
2. `POST`로 우회할 수 있지만 캐싱이 되지 않고 멱등성이 없어 조회 목적으로는 한계가 있습니다
3. `QUERY`는 본문에 조건을 담으면서 안전하고 멱등적이라 캐싱과 자동 재시도가 모두 가능합니다
4. `QUERY`는 항상 preflight가 발생하기 때문에 서버 CORS 설정에서 명시적으로 허용해야 합니다
   <br><br>

_출처:<br>
[1] GeekNews (["HTTP QUERY 메서드 (RFC 10008)"](https://news.hada.io/topic?id=30594))<br>
[2] IETF (["The HTTP QUERY Method (RFC 10008)"](https://www.rfc-editor.org/rfc/rfc10008))<br>
[3] MDN (["Cross-Origin Resource Sharing (CORS)"](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS))<br>_
