# 📝 React Day 4: useContext 학습 회고

> **Date**: 2026-01-14  
> **Tags**: #React #useContext #ContextAPI #전역상태

---

## 💡 핵심 요약

React의 네 번째 Hook인 **useContext**를 학습했습니다. 컴포넌트가 깊어질수록 발생하는 Props Drilling 문제를 해결하기 위해, Context API를 활용해 '전역 보관소'를 만들어 필요한 컴포넌트만 데이터를 꺼내 쓰는 방식입니다.

---

## 🔑 주요 포인트

### 1. Props Drilling vs Context API

| 방식           | 비유             | 특징                              |
| -------------- | ---------------- | --------------------------------- |
| Props Drilling | 층층이 택배 전달 | 중간 컴포넌트도 props를 받아야 함 |
| useContext     | 무전기 채널      | 필요한 컴포넌트만 채널에 접속     |

### 2. Context API 3단계

```jsx
// 1. 채널 생성
export const UserContext = createContext();

// 2. Provider로 범위 지정
<UserContext.Provider value={{ user, login }}>{children}</UserContext.Provider>;

// 3. useContext로 꺼내 쓰기
const { user } = useContext(UserContext);
```

### 3. 로컬 상태 vs 전역 상태

- **로컬 (tempName)**: 입력 중인 임시 값. 한 글자씩 변해도 다른 컴포넌트에 영향 없음.
- **전역 (user.name)**: 로그인 버튼을 눌러 확정된 값. 변경 시 구독 중인 모든 컴포넌트가 리렌더링.

---

## 🛠️ 실습 내용

| 파일              | 역할                                   |
| ----------------- | -------------------------------------- |
| UserContext.jsx   | 전역 상태 저장소 (user, login, logout) |
| App.jsx           | UserProvider로 앱 전체 감싸기          |
| Header.jsx        | 로그인 상태에 따라 환영 메시지 표시    |
| LoginPractice.jsx | 닉네임 입력 후 전역 상태 업데이트      |

---

## 📌 향후 계획

- 백엔드 연동 시 JWT 토큰을 Context에서 관리하는 패턴 학습
- Custom Hook으로 Context 로직 분리하기
