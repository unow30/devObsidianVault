- 클라이언트와 서버가 통신하는 방법 중 하나
- 클라이언트가 요청을 보내면 서버가 응답을 반환한다. 요청을 보내는 쪽이 클라이언트
- 일반적으로 데이터를 JSON구조로 보낸다.
## HTTP 요청의 구성요소
- url: 요청을 보내는 주소
- method: 요청의 종류, 타입
- header: 요청의 메타데이터(데이터에 대한 데이터, 요청에 대한 데이터)
- body: 요청에 관련된 데이터
![[Pasted image 20241119124444.png]]
## url의 구성요소
- scheme
- host
- path
- query parameter
- path parameter

## Method
- 같은 url에 여러개의 Method가 존재할 수 있다.
- GET 요청은 데이터를 조회할 때 사용한다.
- POST 요청은 데이터를 생성할 때 사용한다.
- PUT 요청은 데이터를 업데이트 또는 생성할 때 사용한다.
- PATCH 요청은 데이터를 업데이트할 때 사용한다.
- DELETE 요청은 데이터를 삭제할 때 사용한다.

## Header
- 메다데이터를 정의한다.
- 요청에 대한 정보를 정의한다.
- key/value 형태로 정의되고 모두 string 형태다.
- 라이브러리 프레임워크 환경에 의해 자동생성되는 값들이 많고 직접 값을 변경하는 경우는 Body보단 상대적으로 적다.

## Body
- 요청에 대한 로직 수행에 직접적으로 필요한 정보를 의미한다.
- 새로운 자원을 생성하는 POST 요청을 한다면 그 내용을 Body에 입력한다.
- 일반적으로 JSON 구조를 사용한다.
- Header는 요청 자체에 대한 정보를 담고 있고 Body는 요청 수행에 필요한 데이터를 담고 있다