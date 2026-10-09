---
layout: post
title: "Python FastAPI API 개발 면접 질문과 핵심 답변 37개"
description: "FastAPI, Python 비동기 처리, Pydantic, 의존성 주입, 데이터베이스, 인증·인가, AI API 실무 질문을 완결형 문장으로 정리한 면접 암기 노트입니다."
date: 2026-10-09
categories: [backend]
tags: [Python, FastAPI, REST API, ASGI, Backend, Interview, Study]
permalink: /backend/fastapi-api-interview-questions/
---

Python과 FastAPI로 API를 구현한 개발자를 대상으로 핵심 개념과 실무 설계 질문을 정리했습니다. 모든 답변은 **동작 주체를 명시한 완결형 문장**으로 작성했습니다.

## 1. FastAPI 기본 개념

### Q1. FastAPI란 무엇인가요?
**답변:** FastAPI는 Python 타입 힌트와 Pydantic을 활용하여 요청 검증, 응답 직렬화, OpenAPI 문서 생성을 지원하는 ASGI 기반 웹 API 프레임워크입니다.

### Q2. FastAPI를 선택하는 이유는 무엇인가요?
**답변:** 개발자는 Python의 데이터 처리·AI 생태계를 활용하면서 타입 선언 기반의 요청 검증과 API 문서화를 빠르게 구현할 수 있습니다.

### Q3. API 요청은 어떤 순서로 처리되나요?
**답변:** ASGI 서버는 HTTP 요청을 FastAPI에 전달하고, FastAPI는 라우팅과 의존성 해결 및 입력 검증을 수행한 후 경로 처리 함수의 반환값을 HTTP 응답으로 변환합니다.

### Q4. ASGI란 무엇인가요?
**답변:** ASGI는 Python 웹 애플리케이션과 서버 사이의 표준 인터페이스로서 비동기 HTTP 연결과 WebSocket 통신 등을 지원합니다.

### Q5. Uvicorn이란 무엇인가요?
**답변:** Uvicorn은 ASGI 애플리케이션을 실행하고 클라이언트의 네트워크 요청을 애플리케이션에 전달하는 서버입니다.

### Q6. `async`와 `await`는 무엇인가요?
**답변:** `async def`는 코루틴 함수를 정의하고, `await`는 대기 가능한 비동기 작업이 완료될 때까지 현재 코루틴의 실행을 일시 중단하여 이벤트 루프가 다른 작업을 수행할 기회를 제공합니다.

### Q7. FastAPI에서 `def`와 `async def`는 어떻게 다른가요?
**답변:** FastAPI는 일반 `def` 경로 처리 함수를 스레드풀에서 실행하고, `async def` 경로 처리 함수를 이벤트 루프에서 실행합니다. 개발자는 사용하는 라이브러리의 동기·비동기 I/O 방식에 맞춰 함수를 선택합니다.

### Q8. 동시성과 병렬성은 어떻게 다른가요?
**답변:** 동시성은 여러 작업의 진행 시간을 겹치도록 관리하는 방식입니다. 병렬성은 여러 실행 자원을 사용하여 여러 작업을 실제로 동시에 수행하는 방식입니다.

### Q9. Pydantic은 무엇인가요?
**답변:** Pydantic은 타입 선언과 필드 제약조건을 바탕으로 입력 데이터를 파싱하고 검증하는 라이브러리입니다. FastAPI는 Pydantic 모델을 요청 본문과 응답 모델에 활용합니다.

### Q10. `Depends()`는 무엇인가요?
**답변:** FastAPI의 `Depends()`는 인증 사용자 조회, DB 세션 생성, 공통 설정 주입과 같은 의존성을 선언하고 요청 처리 과정에서 필요한 값을 전달합니다.

### Q11. `response_model`은 무엇인가요?
**답변:** FastAPI의 `response_model`은 API 응답의 스키마를 정의하고 반환 데이터를 검증·직렬화하며 선언한 필드를 기준으로 응답을 필터링합니다.

### Q12. `APIRouter`는 무엇인가요?
**답변:** FastAPI의 `APIRouter`는 사용자, 주문, 조회 등 기능별 경로를 모듈로 분리하고 메인 애플리케이션에 결합하는 구성요소입니다.

### Q13. FastAPI는 API 문서를 어떻게 생성하나요?
**답변:** FastAPI는 경로, 타입 선언, 요청·응답 모델을 바탕으로 OpenAPI 스키마를 생성하고 Swagger UI와 ReDoc 화면을 제공합니다.

## 2. Python 언어와 실행 모델

### Q14. GIL은 무엇인가요?
**답변:** GIL은 일반적인 GIL 활성화 CPython 실행 환경에서 한 번에 하나의 스레드가 Python 바이트코드를 실행하도록 조정하는 잠금 장치입니다. 개발자는 I/O 작업에 스레드를 활용하고 CPU 집약적인 작업에는 다중 프로세스를 검토합니다.

### Q15. Thread와 Process는 어떻게 다른가요?
**답변:** 같은 프로세스의 스레드들은 메모리 공간을 공유하며 실행됩니다. 각 프로세스는 독립된 메모리 공간을 사용하며 별도의 실행 자원으로 동작합니다.

### Q16. 코루틴이란 무엇인가요?
**답변:** 코루틴은 실행 중 비동기 대기 지점에서 제어권을 이벤트 루프에 넘기고 이후 해당 지점에서 실행을 재개할 수 있는 작업 단위입니다.

### Q17. Decorator란 무엇인가요?
**답변:** Python 데코레이터는 함수나 클래스를 감싸거나 등록하여 추가 동작을 적용하는 기능입니다. FastAPI는 `@app.get()` 같은 데코레이터를 사용하여 경로를 등록합니다.

### Q18. `yield`는 어떤 역할을 하나요?
**답변:** Python의 `yield`는 제너레이터 실행 상태를 유지하면서 값을 전달합니다. FastAPI의 `yield` 의존성은 요청에 DB 세션을 제공하고 정리 코드를 수행하는 데 활용됩니다.

### Q19. 타입 힌트는 어떤 역할을 하나요?
**답변:** Python 타입 힌트는 함수 인자와 반환값의 예상 타입을 표현합니다. 정적 분석 도구와 FastAPI·Pydantic은 타입 정보를 활용하여 개발 지원과 데이터 검증을 수행합니다.

## 3. API 설계와 운영 실무

### Q20. FastAPI 프로젝트의 코드는 어떻게 분리하나요?
**답변:** Router는 HTTP 요청과 응답을 처리하고, Service는 업무 규칙을 처리하며, Repository는 데이터 접근을 담당하도록 역할을 분리합니다.

### Q21. HTTP 메서드는 어떻게 사용하나요?
**답변:** GET은 자원을 조회하고, POST는 서버에 처리를 요청하거나 자원을 생성하며, PUT은 자원 전체를 교체하고, PATCH는 일부를 변경하며, DELETE는 자원을 삭제합니다.

### Q22. 주요 HTTP 상태 코드는 무엇인가요?
**답변:** 200은 요청 성공, 201은 자원 생성 성공, 400은 잘못된 요청, 401은 인증 필요, 403은 접근 권한 거부, 404는 자원 미발견, 422는 요청 데이터 검증 오류, 500은 서버 내부 오류를 의미합니다.

### Q23. FastAPI의 입력값 검증 실패는 어떻게 처리되나요?
**답변:** FastAPI는 Pydantic 기반 요청 검증 오류를 기본적으로 HTTP 422와 오류 상세 정보로 응답합니다. 개발자는 `RequestValidationError` 핸들러로 응답 형식을 조정할 수 있습니다.

### Q24. DB 연결은 어떻게 관리하나요?
**답변:** 애플리케이션은 커넥션 풀을 통해 연결을 재사용하고, 요청 범위의 세션과 트랜잭션을 관리하며 처리 완료 시 연결 자원을 반환합니다.

### Q25. 트랜잭션은 어떻게 처리하나요?
**답변:** Service 계층은 하나의 업무 단위를 트랜잭션으로 묶고 정상 완료 시 커밋하며 오류 발생 시 롤백하여 데이터 일관성을 유지합니다.

### Q26. 예외 처리는 어떻게 구현하나요?
**답변:** 경로 처리 함수는 예상 가능한 HTTP 오류를 `HTTPException`으로 표현하고, 공통 Exception Handler는 오류 응답 형식과 서버 로그 기록을 통합합니다.

### Q27. 인증과 인가는 어떻게 구분하나요?
**답변:** 인증은 요청한 사용자의 신원을 확인하는 절차입니다. 인가는 인증된 사용자가 특정 자원에 수행할 수 있는 작업을 판단하는 절차입니다.

### Q28. JWT 기반 API 인증은 어떻게 처리하나요?
**답변:** 인증 계층은 JWT의 서명, 만료시간, 발급자와 대상 서비스 등을 검증하여 사용자를 식별합니다. 인가 계층은 해당 사용자의 역할과 자원 소유권을 확인합니다.

### Q29. 외부 API 호출이 지연되면 어떻게 대응하나요?
**답변:** 애플리케이션은 연결·응답 시간 제한을 설정하고 요청의 멱등성과 오류 유형을 기준으로 제한된 재시도를 적용하며 실패 상태를 기록합니다.

### Q30. API 응답이 느리면 어떻게 분석하나요?
**답변:** 개발자는 요청별 처리시간을 계측하고 SQL 실행시간, DB 연결 대기, 외부 HTTP 호출, CPU 사용량과 이벤트 루프 블로킹 시간을 구분하여 병목을 찾습니다.

### Q31. FastAPI API는 어떻게 테스트하나요?
**답변:** 개발자는 pytest와 FastAPI TestClient를 사용하여 정상·입력 오류·권한·경계 시나리오를 검증하고 실제 DB를 연결한 통합 테스트로 런타임 동작을 확인합니다.

### Q32. FastAPI는 어떻게 배포하나요?
**답변:** 개발자는 Uvicorn 기반 ASGI 서버를 배포 환경에 실행하고 컨테이너 설정, 환경변수, 네트워크 노출, 프록시, 헬스체크, 로그와 워커 구성을 관리합니다.

### Q33. 오래 걸리는 작업은 어떻게 처리하나요?
**답변:** 애플리케이션은 짧은 응답 후처리에 FastAPI `BackgroundTasks`를 활용합니다. 장시간 실행과 재시도·작업 보존이 필요한 처리는 작업 큐, 별도 Worker, 상태 조회 API로 구성합니다.

## 4. AI·데이터 서비스 실무

### Q34. LLM 기능을 FastAPI 서비스에 어떻게 연동하나요?
**답변:** FastAPI 서비스는 사용자 요청을 검증하고 모델 호출 인터페이스에 전달하며, 모델 응답에 대한 구조·권한·업무 규칙 검증을 거쳐 최종 결과를 반환합니다.

### Q35. LLM이 생성한 SQL은 어떻게 실행하나요?
**답변:** 서버는 SQL 파서를 사용하여 단일 조회문과 허용 테이블 여부를 검증합니다. 조회 실행 계층은 읽기 전용 DB 권한과 실행시간·결과 행 수 제한을 적용합니다.

### Q36. Text2SQL 결과의 정확성은 어떻게 평가하나요?
**답변:** 평가 계층은 SQL 생성, 정책 검증, DB 실행, 실제 조회 결과의 정확성을 각각 측정합니다. 정확성 평가는 기대한 열과 행의 의미를 실제 결과와 비교합니다.

### Q37. Spring Boot와 FastAPI는 어떻게 비교하나요?
**답변:** Spring Boot는 Java 기반의 웹·보안·트랜잭션 생태계를 제공하며, FastAPI는 Python 타입 힌트·Pydantic·ASGI를 활용한 API 구현과 Python 데이터·AI 라이브러리 연동을 지원합니다. 개발자는 서비스 요구사항과 기존 시스템의 기술 구성을 기준으로 프레임워크를 선택합니다.

## 5. 최소 코드 예시

아래 코드는 입력 모델, 요청 처리, 응답 모델을 보여주는 독립 학습 예제입니다.

```python
from fastapi import FastAPI
from pydantic import BaseModel, Field

app = FastAPI()

class ItemCreate(BaseModel):
    name: str = Field(min_length=1)

class ItemResponse(BaseModel):
    id: int
    name: str

@app.post("/items", response_model=ItemResponse, status_code=201)
def create_item(payload: ItemCreate) -> ItemResponse:
    return ItemResponse(id=1, name=payload.name)
```

- `ItemCreate`는 요청 본문을 검증합니다.
- `ItemResponse`는 응답 데이터의 구조를 정의합니다.
- `@app.post`는 POST 요청 경로와 상태 코드를 등록합니다.
- Uvicorn은 `app`을 실행해 HTTP 요청을 처리할 수 있게 합니다.

## 6. 공개 프로젝트로 설명할 수 있는 구현 사례

공개 독립 프로젝트 [Text2SQL Workspace](https://github.com/son1004007/text2sql-workspace)는 Python, FastAPI, PostgreSQL로 자연어 질의를 조회 API에 연결한 사례입니다.

1. FastAPI는 인증된 사용자의 Workspace와 요청을 처리합니다.
2. Text2SQL 모델 인터페이스는 자연어 질문으로부터 SQL 후보를 생성합니다.
3. SQLGlot 기반 검증 계층은 SQL의 문장 수, 조회 유형, 테이블 접근 범위를 검사합니다.
4. 조회 실행 계층은 PostgreSQL 읽기 전용 계정과 실행 제한을 적용합니다.
5. 서비스는 요청과 실행 시도 이력을 별도로 저장합니다.
6. 평가 계층은 생성·검증·실행·결과 정확성을 분리해 확인합니다.
7. pytest와 Docker/PostgreSQL 통합 테스트는 사용자 격리, SQL 검증, 조회 권한을 검증합니다.

이 공개 프로젝트는 합성 데이터와 재현 가능한 테스트를 활용한 **독립 구현 사례**입니다.

**면접용 요약 답변:**

> 저는 Python과 FastAPI로 자연어 질문을 SQL 조회 결과에 연결하는 API를 구현했습니다. 서버는 모델이 생성한 SQL을 파싱하고 허용된 조회 조건을 검증한 뒤, PostgreSQL 읽기 전용 계정으로 실행합니다. 또한 서버는 사용자별 Workspace 권한과 실행 이력을 관리하며, pytest와 Docker 기반 통합 테스트로 정상 처리와 오류 처리를 검증했습니다.

## 7. 암기 순서

1. **기본 구조:** FastAPI → ASGI → Uvicorn → Router → Service → Repository
2. **요청 처리:** Pydantic → Depends → 업무 검증 → response_model
3. **동시성:** async def → await → 이벤트 루프 / 동기 def → 스레드풀
4. **데이터와 보안:** 인증 → 인가 → 트랜잭션 → DB 권한
5. **운영과 검증:** 예외 → Timeout → 로그 → pytest → Docker
6. **AI 연동:** LLM 호출 → 결과 검증 → 제한된 실행 → 정확성 평가

## 8. 공식 참고 문서

- [FastAPI - async / await](https://fastapi.tiangolo.com/async/)
- [FastAPI - Dependencies](https://fastapi.tiangolo.com/tutorial/dependencies/)
- [FastAPI - Dependencies with yield](https://fastapi.tiangolo.com/tutorial/dependencies/dependencies-with-yield/)
- [FastAPI - Response Model](https://fastapi.tiangolo.com/tutorial/response-model/)
- [FastAPI - Error Handling](https://fastapi.tiangolo.com/tutorial/handling-errors/)
- [FastAPI - Bigger Applications](https://fastapi.tiangolo.com/tutorial/bigger-applications/)
- [FastAPI - Testing](https://fastapi.tiangolo.com/tutorial/testing/)
- [FastAPI - Background Tasks](https://fastapi.tiangolo.com/tutorial/background-tasks/)
- [FastAPI - OAuth2 and JWT](https://fastapi.tiangolo.com/tutorial/security/oauth2-jwt/)
- [Python - Threading and GIL](https://docs.python.org/3/library/threading.html)
- [Python - yield expressions](https://docs.python.org/3/reference/expressions.html)
