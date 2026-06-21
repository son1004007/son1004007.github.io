# SON Kiseok Technical Blog

GitHub Pages 기반 기술 블로그입니다.

이 저장소는 단순한 개발 메모장이 아니라, 실무 경험을 이력서와 포트폴리오에서 사용할 수 있는 공개 지식 자산으로 정리하기 위한 공간입니다.

## Blog URL

- https://son1004007.github.io

## Purpose

이 블로그의 목적은 다음과 같습니다.

1. 실무에서 겪은 문제 해결 과정을 재사용 가능한 문서로 정리한다.
2. ChatGPT를 활용해 대화형 문제 해결 결과를 Markdown 기반 기술 글로 전환한다.
3. 백엔드 개발, 서버 운영, 데이터 분석 시스템화, 보안/감사 전환 역량을 공개 가능한 형태로 축적한다.
4. 이력서와 포트폴리오에서 링크할 수 있는 신뢰 가능한 기술 기록을 만든다.

## Positioning

이 블로그는 "이직용 홍보 페이지"가 아니라, 개발자로서의 학습·실무·문제 해결 기록입니다.

다만 공개 글은 이력서와 포트폴리오에서 참고할 수 있도록 다음 기준으로 작성합니다.

- 문제 상황이 명확해야 한다.
- 원인 분석 과정이 드러나야 한다.
- 해결 방법이 재현 가능해야 한다.
- 민감정보가 제거되어야 한다.
- 업무 경험은 특정 고객사 중심이 아니라 일반화된 기술 지식으로 변환해야 한다.

## Category Policy

카테고리는 글의 큰 목적을 나타내고, 태그는 세부 기술과 키워드를 나타냅니다.

카테고리는 아래 7개로 고정합니다.

| Category | 용도 |
|---|---|
| `backend` | Java, Spring Boot, API, 인증/권한, 웹 애플리케이션 구조 |
| `database` | SQL, DB 설계, Oracle, PostgreSQL, Tibero, 데이터 모델링 |
| `infrastructure` | Linux, Nginx, Apache, Tomcat, Podman, 배포, 장애 대응 |
| `data-systemization` | 분석 결과를 DB, API, 화면, 운영 시스템으로 연결하는 작업 |
| `security-audit` | 보안, 접근통제, 로그관리, 내부통제, CISA, ISMS-P |
| `project-management` | PMP, 요구사항 정리, 업무일지, 보고, 고객 커뮤니케이션 |
| `career` | 이직, 포트폴리오, 기술 블로그 운영, 경력 방향 |

`study`, `linux`, `spring`, `pmp`, `cisa`, `troubleshooting`은 카테고리로 쓰지 않고 태그로 사용합니다.

자세한 기준은 `docs/category-guide.md`를 따릅니다.

## Main Topics

- Backend: Java, Spring Boot, JSP, MyBatis, REST API
- Database: PostgreSQL, Oracle, Tibero, SQL, 데이터 모델링
- Infrastructure: Linux, Rocky Linux, Ubuntu, Nginx, Apache, Tomcat, SSL, Podman
- Data Systemization: 분석 결과를 서비스/운영 환경으로 연결하는 방법
- Security & Audit: 정보보안, 내부통제, CISA, ISMS-P, 로그/접근통제
- Project Management: PMP, 업무일지, 요구사항 정리, 고객 커뮤니케이션
- Career: 이직, 포트폴리오, 기술 블로그 운영, 경력 방향

## Writing Workflow

```text
ChatGPT 대화
-> 핵심 내용 추출
-> 공개 가능 여부 검토
-> 카테고리/태그 지정
-> Markdown 글 작성
-> GitHub repo에 commit/push
-> GitHub Pages로 공개
-> 이력서/포트폴리오에 주요 글 링크 연결
```

## Standard Post Structure

기본 글 구조는 다음을 우선 사용합니다.

```markdown
## 문제점

## 원인

## 해결

## 실행 방법

## 재발 방지 / 개선 방향

## 포트폴리오 관점의 의미
```

개념 설명 글이나 학습 기록은 아래 구조를 사용합니다.

```markdown
## 배경

## 핵심 개념

## 실무 적용 기준

## 예시

## 정리
```

## Public Safety Rules

공개 글에는 아래 정보를 포함하지 않습니다.

- 고객사 내부 시스템명, 내부 IP, 계정, 비밀번호, 토큰, 경로
- 비공개 회의록 원문
- 계약상 비공개 자료
- 고객사 실명과 장애 내용을 직접 연결하는 표현
- 보안상 악용 가능한 상세 설정값
- 개인 식별정보

업무 경험은 반드시 일반화해서 작성합니다.

예시:

```text
나쁜 예: 한국지역난방공사 서버 191.xxx.xxx.xxx에서 발생한 장애
좋은 예: 공공기관 Linux 서버에서 발생한 디스크 마운트 실패 대응 절차
```

## Files for Future ChatGPT/Codex Sessions

다른 ChatGPT 또는 Codex 세션에서 이 저장소를 수정할 때는 아래 파일을 먼저 읽습니다.

1. `AGENTS.md`
2. `docs/category-guide.md`
3. `docs/blog-writing-guide.md`
4. `docs/post-template.md`
5. `_config.yml`
6. 최근 `_posts/` 문서

## Resume / Portfolio Usage

이력서에는 다음 형태로 연결합니다.

```text
기술 블로그: https://son1004007.github.io
- 백엔드 개발, Linux 서버 운영, 데이터 분석 시스템화, 보안/감사 전환 학습 및 실무 기록
- 실무 장애 대응과 시스템 구축 과정을 문제점/원인/해결/재발방지 구조로 정리
```

포트폴리오에는 주요 글을 프로젝트별 근거 자료로 연결합니다.

예시:

- Linux 서버 운영 경험: Rocky Linux 마운트 실패 대응 글
- 웹 서비스 운영 경험: Apache/Nginx/Tomcat 장애 대응 글
- 데이터 분석 시스템화 경험: 분석 결과를 웹 서비스로 연결하는 글
- 보안/감사 전환 준비: 접근통제, 로그관리, 내부통제 학습 글

## Operating Principle

글을 많이 쓰는 것보다, 실제 이력서와 면접에서 설명 가능한 글을 축적하는 것을 우선합니다.

각 글은 다음 질문에 답해야 합니다.

1. 어떤 문제를 다뤘는가?
2. 왜 문제가 발생했는가?
3. 어떻게 해결했는가?
4. 같은 문제가 다시 발생하면 어떻게 대응할 수 있는가?
5. 이 경험이 개발자 역량을 어떻게 보여주는가?
