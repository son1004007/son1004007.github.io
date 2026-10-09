---
layout: post
title: "React 기본 개념과 용어 정리: 컴포넌트, Props, State, Hook, 렌더링"
description: "React를 처음 사용하는 개발자를 위해 필수 용어의 영문 전체 명칭, 한 줄 정의, 동작 흐름, TypeScript 예제와 백엔드 API 연동 구조를 정리합니다."
date: 2026-10-09
categories: [backend]
tags: [React, JavaScript, TypeScript, Frontend, Study]
permalink: /backend/react-fundamentals/
---

React를 학습할 때는 **용어 암기 → 동작 원리 → 코드 해석 → API 연동** 순서로 이해하면 효율적이다. 이 글은 Java/Spring 및 Python/FastAPI 개발자가 React 화면 코드를 읽고 구현하기 위한 기본 개념을 정리한다.

> **핵심 암기 문장:** React는 컴포넌트로 UI를 구성한다. 부모 컴포넌트는 Props를 전달한다. 컴포넌트는 State를 관리한다. React는 변경된 State를 기준으로 UI를 다시 렌더링한다.

## 1. 핵심 용어: 한 줄 정의

| 용어 | 영문 전체 명칭 / 의미 | 암기할 정의 |
| --- | --- | --- |
| **React** | React | React는 컴포넌트 기반으로 사용자 인터페이스를 렌더링하는 JavaScript 라이브러리이다. |
| **UI** | User Interface, 사용자 인터페이스 | UI는 사용자가 프로그램과 상호작용하는 화면과 입력 요소이다. |
| **Component** | 컴포넌트 | 컴포넌트는 화면을 구성하는 재사용 가능한 독립적 UI 단위이다. |
| **JSX** | JavaScript XML | JSX는 JavaScript 코드 안에서 HTML과 유사한 구문으로 UI를 표현하는 문법 확장이다. |
| **Props** | Properties, 속성 | Props는 부모 컴포넌트가 자식 컴포넌트에 전달하는 읽기 전용 데이터이다. |
| **State** | 상태 | State는 컴포넌트가 기억하고 화면 갱신에 사용하는 데이터이다. |
| **Hook** | 훅 | Hook은 함수형 컴포넌트에서 State와 다른 React 기능을 사용하는 함수이다. |
| **Rendering** | 렌더링 | 렌더링은 React가 컴포넌트 함수를 호출하여 표시할 UI를 계산하는 과정이다. |
| **Re-rendering** | 리렌더링 | 리렌더링은 React가 새로운 상태와 Props 등을 반영하기 위해 컴포넌트를 다시 실행하는 과정이다. |
| **Event Handler** | 이벤트 핸들러 | 이벤트 핸들러는 클릭과 입력 등 이벤트에 대응하여 실행되는 함수이다. |

**암기 순서:** Component → JSX → Props → State → Event → Render

## 2. 화면 갱신과 데이터 흐름 관련 용어

| 용어 | 영문 전체 명칭 / 의미 | 암기할 정의 |
| --- | --- | --- |
| **DOM** | Document Object Model, 문서 객체 모델 | DOM은 브라우저가 HTML 문서를 객체 트리로 표현하고 조작하도록 제공하는 인터페이스이다. |
| **Virtual DOM** | 가상 문서 객체 모델 | Virtual DOM은 React의 UI 표현을 메모리에서 다루고 변경을 계산하는 방식을 설명할 때 사용하는 개념적 용어이다. |
| **Reconciliation** | 재조정 | 재조정은 React가 이전 UI와 새 UI의 차이를 파악하여 변경할 부분을 결정하는 과정이다. |
| **Commit** | 커밋 | 커밋은 React가 계산한 변경 사항을 실제 DOM에 반영하는 단계이다. |
| **One-way Data Flow** | 단방향 데이터 흐름 | 부모 컴포넌트는 Props를 통해 자식 컴포넌트에 데이터를 전달한다. |
| **Immutability** | 불변성 | 개발자는 기존 State 객체를 직접 수정하는 대신 새 객체나 배열을 만들어 상태 변경을 요청한다. |
| **Conditional Rendering** | 조건부 렌더링 | 컴포넌트는 조건에 따라 서로 다른 UI를 반환한다. |
| **Key** | 목록 식별자 | Key는 React가 목록의 각 항목을 식별하는 데 사용하는 고유한 값이다. |
| **Context** | 컨텍스트 | Context는 여러 하위 컴포넌트가 공통 데이터를 읽도록 제공하는 React 기능이다. |
| **Controlled Input** | 제어 컴포넌트의 입력 | React 컴포넌트는 입력값을 State로 관리하고 이벤트 핸들러로 변경한다. |

### React의 동작 흐름

```text
사용자가 버튼을 클릭한다.
  ↓
이벤트 핸들러가 실행된다.
  ↓
State 변경 함수가 새 상태를 예약한다.
  ↓
React가 컴포넌트를 다시 렌더링한다.
  ↓
React가 새 UI와 이전 UI를 비교한다.
  ↓
React가 필요한 DOM 변경 사항을 커밋한다.
  ↓
브라우저가 갱신된 화면을 표시한다.
```

**한 줄 암기:** 사용자 이벤트 → State 변경 → Render → Reconciliation → Commit → 화면 갱신

React의 **렌더링**은 컴포넌트를 호출해 UI를 계산하는 단계이고, **커밋**은 필요한 DOM 변경을 적용하는 단계이다. React는 같은 결과가 유지되는 DOM 요소를 그대로 재사용할 수 있다.

**State 스냅샷:** React는 각 렌더링에 대응하는 State 값을 컴포넌트에 제공한다. 개발자는 이전 상태를 기준으로 값을 누적할 때 `setCount(previous => previous + 1)` 형태의 업데이터 함수를 사용한다.

## 3. React Hook 핵심 정리

| Hook | 역할 | 주된 적용 상황 |
| --- | --- | --- |
| `useState` | 컴포넌트의 State를 선언하고 업데이트한다. | 입력값, 선택 항목, 모달 표시 상태 |
| `useEffect` | 컴포넌트를 외부 시스템과 동기화한다. | 브라우저 API, 연결 및 구독 관리 |
| `useRef` | 렌더링 간 참조값을 유지한다. | DOM 요소 접근, 타이머 식별자 보관 |
| `useContext` | 컴포넌트가 Context 값을 읽는다. | 테마, 공통 설정 |
| `useMemo` | 계산 결과를 의존성 기준으로 재사용한다. | 계산 비용이 큰 데이터 가공 |
| `useCallback` | 함수 참조를 의존성 기준으로 재사용한다. | 함수 Props의 참조 안정화 |
| `useReducer` | Reducer 함수로 상태 변경을 관리한다. | 복잡한 상태 전이 |

**Hook 암기:** State = 상태 / Effect = 외부 동기화 / Ref = 참조 / Context = 공유 값 / Memo = 계산값 재사용 / Callback = 함수 참조 재사용 / Reducer = 상태 전이

### `useState`: 상태 관리

```tsx
const [count, setCount] = useState(0);
setCount(previousCount => previousCount + 1);
```

- `count`는 현재 렌더링의 State 값이다.
- `setCount`는 State 업데이트를 요청하는 함수이다.
- React는 변경된 State를 반영하는 렌더링을 예약한다.

### `useEffect`: 외부 시스템 동기화

```tsx
useEffect(() => {
  document.title = `현재 값: ${count}`;
}, [count]);
```

- React는 DOM 변경을 커밋한 뒤 Effect를 실행하여 문서 제목을 `count` 값과 동기화한다.
- `[count]`는 React가 Effect의 재실행 여부를 판단할 때 비교하는 의존성 배열이다.
- 개발자는 외부 구독이나 연결을 생성하는 Effect에 정리(cleanup) 함수를 반환하여 자원을 해제한다.

### `useRef`: 참조값 유지

```tsx
const inputRef = useRef<HTMLInputElement>(null);
inputRef.current?.focus();
```

- `inputRef`는 렌더링 사이에서 유지되는 참조 객체이다.
- `inputRef.current`는 연결된 입력 DOM 요소를 가리킨다.

## 4. 코드로 이해하는 Component, Props, State

Vite의 React + TypeScript 프로젝트에서 아래 코드를 `src/App.tsx`에 작성할 수 있다.

```tsx
import { useState } from "react";

// 자식 컴포넌트는 Props를 받아 화면을 표시한다.
type CounterProps = {
  count: number;
  onIncrease: () => void;
};

function Counter({ count, onIncrease }: CounterProps) {
  return (
    <div>
      <p>현재 값: {count}</p>
      <button onClick={onIncrease}>증가</button>
    </div>
  );
}

// 부모 컴포넌트는 State와 이벤트 핸들러를 관리한다.
export default function App() {
  const [count, setCount] = useState(0);

  function handleIncrease() {
    setCount(previousCount => previousCount + 1);
  }

  return (
    <Counter count={count} onIncrease={handleIncrease} />
  );
}
```

| 코드 | 해석 |
| --- | --- |
| `App()` | 부모 컴포넌트 |
| `Counter()` | 자식 컴포넌트 |
| `CounterProps` | TypeScript로 선언한 Props 타입 |
| `useState(0)` | 초기 State 값을 0으로 선언 |
| `count` | 현재 렌더링의 State 값 |
| `setCount()` | State 변경 요청 함수 |
| `count={count}` | 부모가 자식에 데이터를 전달하는 Props |
| `onIncrease={handleIncrease}` | 부모가 자식에 전달하는 이벤트 처리 함수 |
| `onClick={onIncrease}` | 버튼과 이벤트 처리 함수를 연결 |
| `return (...)` | JSX로 화면 구조를 반환 |

**동작 순서:** 사용자가 증가 버튼을 누른다 → Counter가 `onIncrease`를 호출한다 → App의 `handleIncrease`가 State를 업데이트한다 → React가 화면을 갱신한다.

## 5. JavaScript와 주변 기술 용어

| 약어·용어 | 영문 전체 명칭 | 역할 |
| --- | --- | --- |
| **HTML** | HyperText Markup Language | HTML은 웹 문서의 구조를 정의한다. |
| **CSS** | Cascading Style Sheets | CSS는 웹 문서의 표현과 스타일을 정의한다. |
| **JS** | JavaScript | JavaScript는 웹 화면의 동작과 로직을 구현한다. |
| **TS** | TypeScript | TypeScript는 JavaScript에 정적 타입 검사를 제공한다. |
| **TSX** | TypeScript + JSX | TSX 파일은 TypeScript와 JSX를 함께 사용한다. |
| **SPA** | Single-Page Application | SPA는 하나의 기본 문서를 중심으로 클라이언트 화면을 전환하는 애플리케이션이다. |
| **CSR** | Client-Side Rendering | CSR에서는 브라우저가 JavaScript를 실행하여 UI를 렌더링한다. |
| **SSR** | Server-Side Rendering | SSR에서는 서버가 HTML을 생성하여 클라이언트에 전달한다. |
| **HTTP** | Hypertext Transfer Protocol | HTTP는 클라이언트와 서버가 요청과 응답을 교환하는 프로토콜이다. |
| **API** | Application Programming Interface | API는 프로그램 사이의 기능과 데이터 교환을 위한 인터페이스이다. |
| **REST** | Representational State Transfer | REST는 자원과 표현 중심으로 시스템 인터페이스를 설계하는 아키텍처 스타일이다. |
| **JSON** | JavaScript Object Notation | JSON은 구조화된 데이터를 교환하는 텍스트 형식이다. |
| **Vite** | Vite(도구 이름) | Vite는 개발 서버와 프런트엔드 빌드 기능을 제공한다. |
| **npm** | npm(패키지 관리자 이름) | npm은 Node.js 생태계의 패키지 설치와 스크립트 실행을 지원한다. |

React Router는 URL별 컴포넌트 표시를 관리하는 라우팅 라이브러리이다. TanStack Query는 서버 데이터의 조회, 캐싱, 동기화와 업데이트를 지원하는 라이브러리이다.

## 6. 백엔드와 React의 역할

```text
사용자
  ↓ 입력 · 클릭
React (Frontend)
  - Component로 화면 구성
  - State로 화면 상태 관리
  - HTTP로 API 호출
  ↓ HTTP 요청
Spring Boot / FastAPI (Backend)
  - Controller 또는 Route가 요청 수신
  - Service가 비즈니스 로직 실행
  - Repository 또는 Mapper가 데이터 접근
  ↓ SQL
PostgreSQL / Oracle
  ↓ 조회 결과
Backend가 JSON 응답 생성
  ↓
React가 응답 데이터를 State에 반영
  ↓
React가 화면 갱신
```

**실무 기준:** React는 사용자 입력과 화면 상태를 관리한다. 백엔드는 인증·인가, 비즈니스 규칙, 데이터 처리와 저장을 담당한다.

예를 들어 React는 `fetch("/api/users")`로 사용자 목록을 요청한다. Spring Boot 또는 FastAPI 서버는 JSON 배열을 반환한다. React는 응답을 State에 저장하고 JSX로 목록을 표시한다. 개발자는 실행 환경에 따라 프록시 또는 동일 출처의 API 경로를 설정한다.

## 7. 실행 방법: React + TypeScript 프로젝트

Vite 공식 프로젝트 생성 명령은 다음과 같다.

```bash
npm create vite@latest my-react-app -- --template react-ts
cd my-react-app
npm install
npm run dev
```

개발자는 Vite가 출력한 로컬 주소에서 화면을 확인한다.

**학습 순서**

1. JavaScript: `const`, `let`, 화살표 함수, 구조 분해 할당, `map`, `async/await`
2. JSX와 Component: 화면 분해와 조합
3. Props와 State: 부모·자식 데이터 전달 및 상태 관리
4. Event와 Rendering: 사용자 입력과 화면 갱신
5. Hook: `useState`, `useEffect`, `useRef`
6. API 연동: HTTP 호출, JSON 처리, 로딩·오류 상태
7. Routing: URL별 화면 전환
8. TypeScript: Props와 API 응답 타입
9. 공통·서버 상태: Context, TanStack Query 등

## 8. 면접·암기용 핵심 답변

| 질문 | 1문장 정답 |
| --- | --- |
| React란 무엇인가? | React는 컴포넌트를 조합하여 사용자 인터페이스를 렌더링하는 JavaScript 라이브러리이다. |
| Component란 무엇인가? | Component는 재사용할 수 있는 독립적인 UI 단위이다. |
| JSX란 무엇인가? | JSX는 JavaScript 안에서 HTML과 유사한 문법으로 UI를 표현하는 구문 확장이다. |
| Props란 무엇인가? | Props는 부모 컴포넌트가 자식 컴포넌트에 전달하는 읽기 전용 데이터이다. |
| State란 무엇인가? | State는 컴포넌트가 기억하고 화면 갱신에 사용하는 데이터이다. |
| Props와 State의 차이는 무엇인가? | 부모는 Props로 데이터를 전달하고, 컴포넌트는 State로 자체 상태를 관리한다. |
| Hook이란 무엇인가? | Hook은 함수형 컴포넌트가 State와 React의 다른 기능을 사용하도록 제공하는 함수이다. |
| 리렌더링은 언제 발생하는가? | React는 State 업데이트 등 렌더링 트리거가 발생하면 컴포넌트를 다시 실행하여 UI를 계산한다. |
| useEffect는 언제 사용하는가? | 컴포넌트는 useEffect로 브라우저 API, 연결, 구독 등 외부 시스템과 동기화한다. |
| Key는 왜 사용하는가? | React는 Key로 목록 항목을 식별하여 항목의 변경과 이동을 처리한다. |

## 9. 최종 정리

- **React = 컴포넌트 기반 UI 라이브러리**
- **Component = UI 단위**
- **JSX = JavaScript에서 UI를 표현하는 구문**
- **Props = 부모가 자식에 전달하는 데이터**
- **State = 컴포넌트가 관리하는 데이터**
- **Hook = React 기능을 사용하는 함수**
- **Render = UI 계산**
- **Commit = 실제 DOM 변경 반영**
- **useState = 상태 관리**
- **useEffect = 외부 시스템 동기화**

React 학습에서는 **Props → State → Event → Render**의 관계를 먼저 이해한다. 개발자는 이후 API 연동, 타입 정의, 라우팅과 공통 상태 관리로 학습 범위를 확장한다.

## 공식 참고 자료

- [React 공식 학습 문서](https://ko.react.dev/learn)
- [React: UI 표현하기](https://ko.react.dev/learn/describing-the-ui)
- [React: Props 전달](https://react.dev/learn/passing-props-to-a-component)
- [React: 스냅샷으로서의 State](https://ko.react.dev/learn/state-as-a-snapshot)
- [React: 렌더링 그리고 커밋](https://ko.react.dev/learn/render-and-commit)
- [React: useEffect](https://react.dev/reference/react/useEffect)
- [React: useMemo](https://react.dev/reference/react/useMemo)
- [React: useCallback](https://react.dev/reference/react/useCallback)
- [Vite: 시작하기](https://vite.dev/guide/)
