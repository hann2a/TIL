> 요약
>> GET은 조회, POST는 생성/변경에 최적화.
>> 캐시/히스토리/보안/멱등성 관점에서 차이를 이해하고 상황에 맞게 선택.
>> 민감 정보는 절대 URL로 보내지 말고, HTTPS와 적절한 헤더로 안전하게 전송할 것.

# TIL — HTTP Request Methods 정리

## HTTP란?

- HTTP(HyperText Transfer Protocol): 클라이언트(브라우저/앱)와 서버가 요청-응답으로 리소스를 주고받는 약속.
- 요청에는 어떤 작업을 원하는지를 나타내는 메서드가 포함됨.

## 요청 메서드 한눈에 보기

- GET → 조회(Read)
- POST → 생성/상태 변경(Create/Change)

* (참고) PUT/PATCH → 수정, DELETE → 삭제

### GET — 조회용

- 검색, 페이지 로드, API 데이터 조회 등 읽기 전용 상황에서 사용.
- 특징
  - 데이터 전달: URL 쿼리 문자열(Query String) 로 전송
  예)
  ```bash
  GET /articles?title=제목&content=내용
  ```
  - 제한: URL 길이 제한 → 대량 데이터 전송에 부적합
  - 히스토리: 요청 URL이 브라우저 히스토리에 남음
  - 캐싱: 캐시 가능

  - 브라우저/프록시가 응답을 로컬(브라우저 캐시: 메모리/디스크) 에 저장
  - 동일한 URL 재요청 시 저장된 응답을 재사용하여 빠른 로딩/뒤로 가기 가능

    - 🔎 팁: 캐시 동작은 서버의 Cache-Control, ETag, Last-Modified 헤더 설정에 따라 달라진다.
    - ⚠️ 보안: 민감 정보(비밀번호, 토큰 등) 를 쿼리 문자열에 절대 담지 말 것. URL은 로그/히스토리/분석 도구에 남는다.

### POST — 생성/상태 변경

- 회원가입, 로그인, 글 작성, 파일 업로드처럼 서버의 상태가 바뀌는 상황에 사용.
- 특징
  - 데이터 전달: HTTP Body 로 전송 (JSON/폼/파일 등 대용량 가능)
    예)
    ```bash
      POST /articles
      Content-Type: application/json

      { "title": "제목", "content": "내용" }
    ```

  - 제한: 사실상 크기 제한 적음(서버/클라이언트 설정 범위 내)
  - 히스토리: 데이터가 URL에 없어 브라우저 히스토리에 직접 남지 않음
  - 캐싱: 기본적으로 캐시 대상 아님
  - URL에 파라미터가 없고 의미상 부작용이 있어 캐시가 꺼져 있는 게 일반적

    - 🧪 멱등성(idempotency): POST는 일반적으로 멱등 아님(같은 요청을 여러 번 보내면 여러 개 생성될 수 있음). 반면 GET은 멱등이어야 한다(여러 번 조회해도 결과가 바뀌지 않아야 함).

### 언제 무엇을 쓸까?

- 데이터 읽기/검색/리스트 조회 → GET
  캐시 이점 활용 가능, URL 공유/북마크 가능

- 리소스 생성/변경(로그인 포함) → POST
  본문에 안전하게 데이터 담기, 대용량 전송 가능

### 미니 예시 (cURL)
```bash
GET: 게시글 목록 조회 (캐시 가능)
curl -X GET "https://api.example.com/articles?page=1"

POST: 새 게시글 생성 (본문에 JSON)
curl -X POST "https://api.example.com/articles" \
  -H "Content-Type: application/json" \
  -d '{ "title": "제목", "content": "내용" }'
```