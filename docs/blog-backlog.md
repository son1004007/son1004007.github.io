# Blog Backlog

이 문서는 기존 ChatGPT 대화와 GitHub Pages 블로그 작업 내용을 기준으로 정리한 블로그 작성 후보 목록입니다.

목적은 단순 소재 저장이 아니라, 이력서와 포트폴리오에 연결할 수 있는 공개 기술 글을 체계적으로 축적하는 것입니다.

## 작성 기준

블로그 글 후보는 아래 기준을 만족해야 합니다.

1. 실무 문제 해결 과정이 있다.
2. 기술 역량을 보여준다.
3. 민감정보를 제거하고 일반화할 수 있다.
4. 이력서/면접에서 설명할 수 있다.
5. 같은 문제를 겪는 사람이 참고할 수 있다.

## 상태 값

| 상태 | 의미 |
|---|---|
| `done` | 작성 완료 및 push 완료 |
| `todo-1` | 1순위 작성 대상 |
| `todo-2` | 2순위 작성 대상 |
| `later` | 필요 시 작성 |

## 표준 카테고리

카테고리는 `docs/category-guide.md` 기준을 따른다.

| Category | 용도 |
|---|---|
| `backend` | Java, Spring Boot, API, 인증/권한, 웹 애플리케이션 구조 |
| `database` | SQL, DB 설계, Oracle, PostgreSQL, Tibero, 데이터 모델링 |
| `infrastructure` | Linux, Nginx, Apache, Tomcat, Podman, 배포, 장애 대응 |
| `data-systemization` | 분석 결과를 DB, API, 화면, 운영 시스템으로 연결하는 작업 |
| `security-audit` | 보안, 접근통제, 로그관리, 내부통제, CISA, ISMS-P |
| `project-management` | PMP, 요구사항 정리, 업무일지, 보고, 고객 커뮤니케이션 |
| `career` | 이직, 포트폴리오, 기술 블로그 운영, 경력 방향 |

---

# 작성 완료 글

| 상태 | 제목 | Category | 비고 |
|---|---|---|---|
| done | ChatGPT를 활용해 GitHub Pages 기술 블로그를 시작합니다 | `career` | 블로그 목적 설명 |
| done | Rocky Linux 디스크 마운트 실패 대응 절차 | `infrastructure` | Linux 운영 장애 대응 |
| done | Apache VirtualHost 404 장애 원인 분석 | `infrastructure` | 웹서버 장애 대응 |
| done | Podman 기반 CloudBeaver 운영 검토 | `infrastructure` | 분석 서버 도구 운영 |
| done | 분석 결과를 웹 서비스로 시스템화할 때 고려할 점 | `data-systemization` | 분석 결과 시스템화 기준 |
| done | Nginx + Tomcat + Let’s Encrypt 기반 웹 서비스 운영 구조 | `infrastructure` | 웹 서비스 운영/배포 구조 |

---

# 1순위 작성 대상

| 우선순위 | 제목 | Category | 포트폴리오 의미 |
|---:|---|---|---|
| 1 | Spring Boot + JSP 운영 환경에서 로그인 루프를 점검하는 방법 | `backend` | 백엔드 운영 장애 대응 |
| 2 | 분석가가 만든 결과물을 웹 화면과 API로 연결하는 절차 | `data-systemization` | 분석-개발 연결 역량 |
| 3 | 공공 데이터 분석 프로젝트에서 분석 범위와 개발 범위를 구분하는 방법 | `project-management` | 요구사항/범위 조율 역량 |
| 4 | GitHub Pages 기술 블로그를 이력서 포트폴리오로 운영하는 방법 | `career` | 포트폴리오 운영 전략 |
| 5 | VS Code Remote SSH를 개발 서버 환경으로 사용할 때의 장단점 | `infrastructure` | 개발환경 설계 역량 |
| 6 | CloudBeaver, VS Code Server, Jupyter 중 분석 서버 도구 선택 기준 | `data-systemization` | 분석환경 아키텍처 판단 |
| 7 | Apache와 Nginx가 섞인 서버에서 포트와 프록시 역할을 정리하는 방법 | `infrastructure` | 운영환경 정리 역량 |
| 8 | 데이터 분석 프로젝트에서 원천 로그를 요청할 때 필요한 항목 | `data-systemization` | 데이터 요구사항 정의 역량 |
| 9 | 분석 결과를 운영 DB 테이블로 설계할 때 고려할 점 | `database` | 데이터 모델링/서비스화 역량 |

---

# Backend 후보

| 상태 | 제목 | 설명 |
|---|---|---|
| todo-1 | Spring Boot + JSP 운영 환경에서 로그인 루프를 점검하는 방법 | SSO, 세션, 권한, anonymous JSON, redirect 문제 일반화 |
| todo-2 | Spring Boot WAR 배포 구조와 Tomcat 운영 시 확인할 항목 | JDK, Tomcat, profile, WAR 배포 기준 |
| todo-2 | MyBatis에서 SQL 오류를 줄이기 위한 쿼리 작성 기준 | ORA-01790, ORA-00979, ORA-01722 등 일반화 |
| later | 관리자/학생 권한이 섞인 웹 서비스에서 권한 필터를 설계하는 방법 | ADMIN/STUDENT 권한, URL 패턴, 접근제어 |
| later | Chart.js 기반 대시보드 화면에서 백엔드 API를 설계하는 기준 | 화면 지표와 API 응답 구조 연결 |
| later | Spring Boot 운영 환경에서 profile을 dev/prod로 나누는 기준 | 운영/개발 설정 분리 |
| later | JSP + Ajax 기반 서비스에서 JSON 응답 오류를 점검하는 방법 | MIME mismatch, 인증 오류, 응답 구조 문제 |

---

# Database 후보

| 상태 | 제목 | 설명 |
|---|---|---|
| todo-1 | 분석 결과를 운영 DB 테이블로 설계할 때 고려할 점 | 분석 결과, 화면 조회, 이력관리 테이블 구분 |
| todo-1 | 분석가의 산출물과 화면 설계를 DB 테이블로 연결하는 방법 | 데이터 모델링/요구사항 연결 |
| todo-2 | Oracle SQL 오류를 원인별로 분류하는 방법 | ORA-01790, ORA-00935, ORA-01722, ORA-00979 |
| later | 실측값과 예측값을 비교하는 결과 테이블 설계 기준 | 예측/실측/기준일자/모델버전 |
| later | 데이터 분석 결과를 key-value 테이블로 저장할 때의 장단점 | 유연성 vs 조회/검증 어려움 |
| later | PostgreSQL, Oracle, Tibero를 함께 고려해야 하는 프로젝트의 SQL 작성 기준 | DB 호환성/문법 차이 |
| later | 통계 데이터만 있을 때와 원천 로그가 있을 때 가능한 분석 차이 | 페이지뷰 통계 vs 세션 로그 |

---

# Infrastructure 후보

| 상태 | 제목 | 설명 |
|---|---|---|
| done | Nginx + Tomcat + Let’s Encrypt 기반 웹 서비스 운영 구조 | Apache 혼선 정리 후 Nginx 중심 운영 |
| todo-1 | Apache와 Nginx가 섞인 서버에서 포트와 프록시 역할을 정리하는 방법 | 80/443, 8080, AJP, 리버스 프록시 |
| todo-1 | VS Code Remote SSH를 개발 서버 환경으로 사용할 때의 장단점 | 노트북은 문서/학습, 서버는 개발환경 |
| todo-2 | Rocky Linux에서 nginx 업그레이드 후 확인할 항목 | nginx.org repo, 모듈, 설정 경로 |
| todo-2 | Podman rootless 환경에서 서비스 컨테이너를 운영할 때 고려할 점 | root 없이 컨테이너 운영 |
| later | Ubuntu 부팅 중 initramfs 진입 시 대응 절차 | 파일시스템 점검 일반화 |
| later | UEFI Fast Boot 때문에 BIOS 진입이 안 될 때 대응 방법 | 개발 PC 운영 팁 |
| later | KVM over IP와 기존 KVM 스위치를 함께 사용할 수 있는지 판단 기준 | 장비 운영/원격 제어 |
| later | Wi-Fi USB 어댑터가 Linux/Windows에서 인식되지 않을 때 점검 순서 | 드라이버/칩셋/커널 |
| later | 서버 디스크 100% 장애 발생 시 서비스 복구 우선순위 | ModelOps 장애 경험 일반화 |

---

# Data Systemization 후보

| 상태 | 제목 | 설명 |
|---|---|---|
| todo-1 | 분석가가 만든 결과물을 웹 화면과 API로 연결하는 절차 | 분석 코드 → 결과 테이블 → API → UI |
| todo-1 | 데이터 분석 프로젝트에서 원천 로그를 요청할 때 필요한 항목 | 세션ID, 시간, 메뉴, 검색어, 다운로드 |
| todo-1 | CloudBeaver, VS Code Server, Jupyter 중 분석 서버 도구 선택 기준 | 분석환경 도구 비교 |
| todo-2 | 홈페이지 이용 로그 분석을 위해 필요한 데이터 구조 | 페이지뷰 통계 한계와 세션 기반 분석 |
| todo-2 | 통합검색 키워드와 메뉴 이동 로그를 함께 분석하는 방법 | UX/메뉴 개선 근거 |
| later | 공공기관 홈페이지 퀵메뉴 개선을 데이터로 판단하는 방법 | 사용 빈도, 중요도, 접근성 |
| later | 일회성 분석 코드와 운영 배치 코드의 차이 | Notebook → 배치화 |
| later | 분석 결과 시각화 요구사항을 개발 요구사항으로 바꾸는 방법 | 화면 설계/데이터 설계 연결 |
| later | ERP 엑셀 첨부 파일을 분석 시스템에 반영할 때의 위험 | 엑셀 업로드/수정/이력관리 범위 |
| later | 데이터 분석 과제에서 분석 중심 추진과 업무 기능 중심 추진을 구분하는 방법 | 분석/업무 기능 범위 분리 |

---

# Security / Audit 후보

| 상태 | 제목 | 설명 |
|---|---|---|
| todo-2 | 개발자가 IT 내부통제 직무로 확장할 때 준비해야 할 기술 역량 | 경력 전환 관점 |
| todo-2 | 접근통제 관점에서 분석 서버를 운영할 때 고려할 점 | 계정, 권한, DB 접근, 로그 |
| todo-2 | CloudBeaver 같은 웹 DB 도구를 운영할 때 보안상 주의할 점 | DB 계정/네트워크/권한 |
| todo-2 | GitHub 블로그에 실무 경험을 공개할 때 제거해야 할 민감정보 | 공개 범위 기준 |
| later | CISA 학습 내용을 IT 내부통제 관점으로 정리하는 방법 | 자격증 학습 실무화 |
| later | ISMS-P 학습을 웹 서비스 운영 기준으로 연결하는 방법 | 보안 인증/운영 통제 |
| later | 공공 프로젝트에서 방화벽 허용 요청을 정리할 때 필요한 정보 | 출발지/목적지/포트/계정/용도 |
| later | 개인정보 유출 과징금 기준을 개발자가 이해해야 하는 이유 | 개인정보보호 실무 관점 |

---

# Project Management 후보

| 상태 | 제목 | 설명 |
|---|---|---|
| todo-1 | 공공 데이터 분석 프로젝트에서 분석 범위와 개발 범위를 구분하는 방법 | 분석/개발 범위 조율 |
| todo-1 | 분석 범위와 개발 범위가 섞였을 때 요구사항을 정리하는 방법 | 자금계획관리 고도화 경험 일반화 |
| todo-2 | 고객 인터뷰 후 질의서를 정리하는 실무 절차 | 홈페이지 로그 분석 인터뷰 일반화 |
| todo-2 | 일일보고를 기술 산출물로 남기는 방법 | 업무일지/보고서 작성 체계 |
| later | 회의록에서 개발 요구사항과 분석 요구사항을 분리하는 방법 | 회의 내용 구조화 |
| later | 공공 데이터 분석 프로젝트에서 방화벽 요청을 늦게 해야 하는 이유 | 과제 정리 후 일괄 요청 |
| later | PMP 학습 내용을 실제 프로젝트 관리 언어로 변환하는 방법 | 자격증 학습/실무 연결 |
| later | 고객에게 범위 외 항목을 설명하는 방법 | ERP 수정/이력관리 범위 문제 |
| later | 분석가에게 코드 리뷰 피드백을 전달할 때의 기준 | 정책 repo 기준 코드 리뷰 경험 일반화 |

---

# Career 후보

| 상태 | 제목 | 설명 |
|---|---|---|
| todo-1 | GitHub Pages 기술 블로그를 이력서 포트폴리오로 운영하는 방법 | 현재 블로그 구축 과정 |
| todo-2 | SI 개발자가 더 나은 근무환경으로 이동하기 위해 준비할 포트폴리오 | 이직 전략 |
| todo-2 | 백엔드 개발자에서 IT 내부통제/감사 직무로 확장하는 경로 | 장기 커리어 |
| todo-2 | 개발자 경력과 보안 경력을 함께 활용하는 이직 전략 | 보안기사 + 개발 실무 |
| later | 자격증을 보험처럼 활용하는 커리어 전략 | PMP, 빅분기, CISA, ISMS-P |
| later | 공공기관 AI/AX 또는 보안 직무로 이직할 때의 장단점 | 공공 채용 검토 |
| later | 70세까지 일하기 위한 개발자 커리어 설계 | 장기 생존 전략 |
| later | 기술 블로그를 만들 때 Obsidian/Quartz보다 GitHub Pages가 적절한 이유 | 도구 선택 의사결정 기록 |

---

# 작성 전략

현재 가장 좋은 블로깅 전략은 다음 순서입니다.

```text
1. 운영 장애 대응 글로 기술 신뢰 확보
2. 분석 결과 시스템화 글로 차별점 확보
3. 요구사항/범위 조율 글로 프로젝트 수행 역량 확보
4. 보안/감사 글로 장기 커리어 방향 확보
5. career 글로 이력서/포트폴리오 연결
```

## 다음 작성 추천 5개

1. Spring Boot + JSP 운영 환경에서 로그인 루프를 점검하는 방법
2. 분석가가 만든 결과물을 웹 화면과 API로 연결하는 절차
3. 공공 데이터 분석 프로젝트에서 분석 범위와 개발 범위를 구분하는 방법
4. GitHub Pages 기술 블로그를 이력서 포트폴리오로 운영하는 방법
5. VS Code Remote SSH를 개발 서버 환경으로 사용할 때의 장단점
