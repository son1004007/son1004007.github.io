# Post Template

아래 템플릿을 복사해서 `_posts/YYYY-MM-DD-title.md` 파일로 작성합니다.

## Troubleshooting / Operations Template

```markdown
---
layout: post
title: "문제 또는 장애 제목"
date: YYYY-MM-DD
categories: [infrastructure]
tags: [Linux, Troubleshooting]
---

## 문제점

어떤 문제가 발생했는지 작성합니다.

- 증상:
- 영향 범위:
- 발생 시점:
- 확인된 로그/메시지:

## 원인

원인을 어떻게 좁혔는지 작성합니다.

- 가능성 1:
- 가능성 2:
- 최종 원인:

## 해결

실제 해결 방향을 작성합니다.

## 실행 방법

명령어, 설정, 점검 순서를 작성합니다.

```bash
# example
systemctl status nginx
```

## 검증 방법

정상화 여부를 확인하는 방법을 작성합니다.

## 재발 방지 / 개선 방향

같은 문제가 다시 발생하지 않도록 할 개선 방향을 작성합니다.

## 포트폴리오 관점의 의미

이 경험이 어떤 역량을 보여주는지 작성합니다.
```

## Study / Concept Template

```markdown
---
layout: post
title: "개념 또는 학습 제목"
date: YYYY-MM-DD
categories: [study]
tags: [PMP, CISA]
---

## 배경

이 개념을 왜 정리하는지 작성합니다.

## 핵심 개념

핵심 내용을 정리합니다.

## 실무 적용 기준

실무에서 어떻게 판단하고 적용할지 작성합니다.

## 예시

간단한 예시를 작성합니다.

## 주의점

오해하기 쉬운 부분이나 한계를 작성합니다.

## 정리

핵심 결론을 작성합니다.
```

## Public Review Checklist

공개 전 아래 항목을 확인합니다.

- [ ] 고객사명 또는 내부 조직명이 직접 노출되지 않는다.
- [ ] IP, 계정, 비밀번호, 토큰, 키가 없다.
- [ ] 내부 경로와 시스템명이 불필요하게 노출되지 않는다.
- [ ] 회의록 원문이 없다.
- [ ] 문제, 원인, 해결, 실행 방법이 구분되어 있다.
- [ ] 제목이 이력서/포트폴리오 링크로 사용하기 적절하다.
- [ ] 과장된 표현이 없다.
