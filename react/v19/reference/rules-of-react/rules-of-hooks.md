# Rules of Hooks

훅은 자바스크립트 함수를 사용해 정의되지만, 호출되는 위치에 제한이 존재하는 특수한 유형의 재사용 가능한 UI 로직이다.

## Only call Hooks at the top level

`use`로 시작하는 함수는 리액트에서 훅으로 불린다.

훅을 루프, 조건, 중첩 함수 도는 트라이/캐치/파이널리 블락에서 호출하지마라.
대신에, 항상 얼리 리턴 전, 리액트 함수 최상단에서 호출해라.
리액트가 함수 컴포넌트를 렌더링할 때만 훅을 호출할 수 있다.

- ✅ 함수 컴포넌트 최상단 바디에서 호출
- ✅ 커스텀 훅 최상단 바디에서 호출

```tsx
function Counter() {
  // ✅ Good: top-level in a function component
  const [count, setCount] = useState(0);
  // ...
}

function useWindowWidth() {
  // ✅ Good: top-level in a custom Hook
  const [width, setWidth] = useState(window.innerWidth);
  // ...
}
```

훅의 호출이 금지된 경우:

- 🔴 조건 또는 루프에서 호출 x
- 🔴 조건의 리턴 이후 호출 x
- 🔴 이벤트 핸들러 내에서 호출 x
- 🔴 클래스형 컴포넌트에서 호출 x
- 🔴 유즈메모, 유즈리듀서 또는 유즈이펙트 내에서 호출 x
- 🔴 트/캐/파 블락에서 호출 x

```tsx
function Bad({ cond }) {
  if (cond) {
    // 🔴 Bad: inside a condition (to fix, move it outside!)
    const theme = useContext(ThemeContext);
  }
  // ...
}

function Bad() {
  for (let i = 0; i < 10; i++) {
    // 🔴 Bad: inside a loop (to fix, move it outside!)
    const theme = useContext(ThemeContext);
  }
  // ...
}

function Bad({ cond }) {
  if (cond) {
    return;
  }
  // 🔴 Bad: after a conditional return (to fix, move it before the return!)
  const theme = useContext(ThemeContext);
  // ...
}

function Bad() {
  function handleClick() {
    // 🔴 Bad: inside an event handler (to fix, move it outside!)
    const theme = useContext(ThemeContext);
  }
  // ...
}

function Bad() {
  const style = useMemo(() => {
    // 🔴 Bad: inside useMemo (to fix, move it outside!)
    const theme = useContext(ThemeContext);
    return createStyle(theme);
  });
  // ...
}

class Bad extends React.Component {
  render() {
    // 🔴 Bad: inside a class component (to fix, write a function component instead of a class!)
    useEffect(() => {});
    // ...
  }
}

function Bad() {
  try {
    // 🔴 Bad: inside try/catch/finally block (to fix, move it outside!)
    const [x, setX] = useState(0);
  } catch {
    const [x, setX] = useState(1);
  }
}
```

## Only call Hooks from React functions

일반적인 자바스크립트 함수 내에서 훅을 호출하지 말고,

✅ 리액트 함수 컴포넌트 내에서 호출
✅ 커스텀 훅 내에서 호출

이 규칙을 따르면 구성 요소의 모든 상태 논리를 소스 코드에서 명확하게 볼 수 있다.

```tsx
function FriendList() {
  const [onlineStatus, setOnlineStatus] = useOnlineStatus(); // ✅
}

function setOnlineStatus() {
  // ❌ Not a component or custom Hook!
  const [onlineStatus, setOnlineStatus] = useOnlineStatus();
}
```
