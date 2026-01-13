# 📝 React Day 3: useRef 학습 회고

> **Date**: 2026-01-13  
> **Tags**: #React #useRef #WebRTC #DOM

---

## 💡 핵심 요약

React의 세 번째 Hook인 **useRef**를 학습했습니다. useState와 달리 값이 변해도 리렌더링을 일으키지 않으며, 주로 DOM 요소에 직접 접근할 때 사용합니다. 이는 WebRTC 프로젝트에서 video 태그를 제어하는 핵심 기술이 됩니다.

---

## 🔑 주요 포인트

### 1. 변수 선택 기준

| 상황 | 사용할 것 |
|------|----------|
| 화면에 보여야 하고, 변하면 업데이트 필요 | `useState` |
| 렌더링 사이에 값 유지 필요 (화면에 안 보임) | `useRef` |
| 기존 state에서 파생되는 값 | 일반 `const` |
| DOM 요소 직접 접근 | `useRef` |

### 2. CS 관점에서의 차이

- **useState**: Observer Pattern 적용. 값 변경 시 React에 알림 → 리렌더링
- **useRef**: 일반 JS 객체. 값 변경 시 아무 일도 안 일어남

둘 다 힙(Heap) 메모리에 저장되지만, React의 "감시 여부"가 다릅니다.

### 3. useRef 사용법 (3단계)

```jsx
const videoRef = useRef(null);     // 1. ref 생성
<video ref={videoRef} />           // 2. JSX에 연결
videoRef.current.play();           // 3. .current로 DOM 접근
```

---

## 🛠️ 실습 내용

| 실습 | 배운 것 |
|------|---------|
| FocusPractice | `input.focus()` 호출 |
| VideoPractice | `video.play()`, `pause()` 제어 |
| WebcamPractice | `getUserMedia()` + video 연결 |

세 번째 실습은 WebRTC의 첫 단계로, 내 카메라 스트림을 video 요소에 연결하는 것이었습니다.

---

## 📌 향후 계획

- Day 4: useContext, Custom Hook 학습
- WebRTC 프로젝트에 오늘 배운 카메라 연결 적용
