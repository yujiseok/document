# Components and Hooks must be pure

순수함수는 계산만 수행하고 그 이상은 수행하지 않는다. 이것은 코드를 읽기 쉽게 해주며, 리액트가 자동적으로 컴포넌트와 훅을 최적화할 수 있게 해준다.

## Why does purity matter?

리액트를 리액트답게 만드는 핵심은 리액트는 순수하다는 것이다. 순수한 컴포넌트 또는 훅은 다음과 같은 특징을 갖는다:

- 멱등성: 같은 인풋(프랍, 상태, 컨텍스트, 훅의 경우 인자)에 대해 동일한 결과를 얻는다.
- 렌더링에 사이드 이펙트는 없어야한다: 사이드 이펙트의 경우 렌더링과 별개로 실행되어야한다. 유저의 상호작용에 의해 실행되는 이벤트 핸들러 또는 렌더 이후 실행되는 이펙트
- 논로컬 값 변형 x: 컴포넌트와 훅은 렌더 시에 생성된 로컬 값이 아닌 값을 변경하면 안된다.

렌더가 순수하면, 리액트는 어떤 변경이 유저에게 가장 먼저 보여지질지 알 수 있다. 이는 렌더링 순수성 덕에 가능하다. 컴포넌트가 렌더링에 사이드이펙트를 일으키지 않기 때문에 리액트는 업데이트가 중요하지 않은 컴포넌트의 렌더링을 일시 중지하고 필요할 때 돌아올 수 있다. -> 동시성?

구체적으로 이것은 렌더링 로직을 여러번 실행해 유저에게 좋은 경험을 제공할 수 있다는 것을 의미한다.
그러나, 컴포넌트에 추적되지 않은 사이드 이펙트(렌더링 중 전역 변수를 수정하는 경우)가 존재하면 리액트가 코드를 재실행 할때 원치 않는 결과를 얻을 수 있다.
이것은 종종 버그와 나쁜 유저 경험을 제공한다.

### How does React run your code?

리액트는 선언적이다. 리액트에 무엇을 렌더할지 알려주면 리액트는 최상의 방법으로 유저에게 보여준다.
이것을 수행하기 위해 리액트는 몇 가지 단계를 거친다.
리액트를 잘 쓰기 위해 모든 단계를 알 필요없다.
하지만, 높은 수준에서 어떤 코드가 렌더에 실행되며 어떤 코드가 외부에서 실행되는지 알아야한다.

**렌더링**이란 UI의 다음 버전을 어떻게 보일지 계산하는 것을 뜻한다.
렌더링 후, 이펙트가 플러쉬(더 이상 남지 않을 때까지 실행)되고 이팩트가 레이아웃에 영향을 줄 경우 계산을 업데이트한다.
리액트는 이 새로운 계산을 통해 이전 UI 버전과 비교한다. 그 후 필요한 최소한의 변경점을 돔에 **커밋**하여 최신 버전을 유지한다.

| 렌더    | 커밋           |
| ------- | -------------- |
| UI 계산 | 실제 돔에 반영 |

### How to tell if code runs in render

렌더링 중에 코드가 실행되는지 알아보는 빠른 방법 중 하나는 코드의 위치를 조사하는 것으로
아래와 같이 컴포넌트 최상단에 작성되어 있다면 렌더링 중에 실행될 가능성이 높다.

```tsx
function Dropdown() {
  const selectedItems = new Set(); // created during render
  // ...
}
```

이벤트 핸들러와 이펙트는 렌더 중에 실행되지 않는다

```tsx
function Dropdown() {
  const selectedItems = new Set();
  const onSelect = (item) => {
    // this code is in an event handler, so it's only run when the user triggers this
    selectedItems.add(item);
  };
}
```

```tsx
function Dropdown() {
  const selectedItems = new Set();
  useEffect(() => {
    // this code is inside of an Effect, so it only runs after rendering
    logForAnalytics(selectedItems);
  }, [selectedItems]);
}
```

## Components and Hooks must be idempotent

컴포넌트는 반드시 프랍, 상태, 컨텍스트와 같은 인풋에 동일한 아웃풋을 반환해야한다.
이것을 멱등성이라고 한다. 즉 같은 인풋에 대해 매번 동일한 아웃풋을 갖는다는 의미다.

즉, 이 규칙이 유지되려면 렌더링 중에 실행되는 모든 코드도 멱등이어야 한다. 예를 들어, 다음 코드는 멱등하지 않다.

```tsx
function Clock() {
  const time = new Date(); // 🔴 Bad: always returns a different result!
  return <span>{time.toLocaleString()}</span>;
}
```

`new Date()`는 호출시 매번 값이 변경되기에 멱등하지않다.
위의 컴포넌트를 렌더할 때, 화면에 표시되는 시간은 컴포넌트가 렌더링된 시간에 고정된다.
비슷하게 `Math.random()` 역시 매번 다른 값을 반환하기에 멱등하지않다.

이것은 멱등하지 않은 함수를 사용하면 안된다는 것이 아닌 - 렌더링 중에 사용하는 것을 피하라는 것이다.
최신 데이터를 이펙트를 통해 동기화할 수 있다.

```tsx
import { useState, useEffect } from "react";

function useTime() {
  // 1. Keep track of the current date's state. `useState` receives an initializer function as its
  //    initial state. It only runs once when the hook is called, so only the current date at the
  //    time the hook is called is set first.
  const [time, setTime] = useState(() => new Date());

  useEffect(() => {
    // 2. Update the current date every second using `setInterval`.
    const id = setInterval(() => {
      setTime(new Date()); // ✅ Good: non-idempotent code no longer runs in render
    }, 1000);
    // 3. Return a cleanup function so we don't leak the `setInterval` timer.
    return () => clearInterval(id);
  }, []);

  return time;
}

export default function Clock() {
  const time = useTime();
  return <span>{time.toLocaleString()}</span>;
}
```

뉴데이트를 이펙트로 감싸는 것으로, 계산이 렌더링 외부로 이동된다.

외부 상태를 리액트와 동기화할 필요가 없다면, 유저의 인터렉션에 의해 값이 변경되는 이벤트 핸들러를 사용할 수 있다.

## Side effects must run outside of render

사이드 이펙트는 반드시 렌더 외부에서 실행되어야 리액트가 컴포넌트를 여러번 렌더할 시 최상의 유저 경험을 만들 수 있다.

> Note
>
> 사이드 이펙트는 이펙트보다 광범위한 용어다. 이펙트는 유즈이펙트로 감싸진 코드를 특정하고, 사이드 이펙트의 경우 호출차에게 값을 반환하는 기본 결과 외에 관찰 가능한 모든 효과가 있는 코드에 대한 용어다.
>
> 사이드 이펙트는 이벤트 핸들러나 이펙트 내부에 작성된다. 하지만 렌더링 동안에는 절대 없다.

렌더링은 순수하게 유지되어야 하지만, 앱이 화면에 무언가를 보여주는 흥미로운 작업을 수행하려면 어느 시점에선 사이드 이펙트가 필요하다.
이 룰의 맹점은, 리액트는 여러번 실행될 수 있으니 사이드 이펙트는 절대 렌더 중에 실행되면 안된다는 것이다.
대부분 상황에서, 이벤트 핸들러로 사이드 이펙트를 처리할 수 있다.
이벤트 핸들러를 명시적으로 사용하는 것은, 리액트에게 코드가 렌더링 중에 필요 없다고 알리는 것이다. 렌더링을 순수하게 유지해라.
유즈이펙트는 진짜 최후의 보루다.

### When is it okay to have mutation?

가장 일반적인 사이드 이펙트는 뮤테이션으로 원시값이 아닌 값을 변경하는 것이다.
일반적으로, 뮤테이션은 리액트에서 멱등하지않지만 지역 뮤테이션은 완전히 괜찮다.

```tsx
function FriendList({ friends }) {
  const items = []; // ✅ Good: locally created
  for (let i = 0; i < friends.length; i++) {
    const friend = friends[i];
    items.push(<Friend key={friend.id} friend={friend} />); // ✅ Good: local mutation is okay
  }
  return <section>{items}</section>;
}
```

지역 뮤테이션을 피하기 위해 코드를 변경할 필요가 없다. 맵으로 코드를 간결하게 할 수 있지만, 렌더 중에 로컬 배열을 만드는 것에 전혀 문제없다.

우리가 아이템을 뮤테이션하는 것처럼 보이지만, 핵심은 이 코드가 지역적으로만 이를 수행한다는 것이다. 즉 컴포넌트가 리렌더링될 때 뮤테이션은 기억되지 않는다.
다시 말해 아이템은 컴포넌트가 존재하는 한 계속 존재한다. 왜냐하면, 아이템은 `<FriendList />`가 렌더될 때마다 재생성되기 때문이다.
컴포넌트는 항상 같은 결과를 반환한다.

반면, 아이템이 컴포넌트 외부에서 생성되었을 경우, 이것은 이전 값을 기억하고 있다.

```tsx
const items = []; // 🔴 Bad: created outside of the component
function FriendList({ friends }) {
  for (let i = 0; i < friends.length; i++) {
    const friend = friends[i];
    items.push(<Friend key={friend.id} friend={friend} />); // 🔴 Bad: mutates a value created outside of render
  }
  return <section>{items}</section>;
}
```

`<FriendList />`가 리렌더될 때, 우린 여전히 프렌즈를 아이템에 추가해 여러 복사된 결과를 유발한다.
이 버전은 관찰 가능한 사이드 이펙트가 있고 룰을 지키지 않는다.

### Lazy initialization

완전히 순수하지 않음에도 지연 초기화는 괜찮다.

```tsx
function ExpenseForm() {
  SuperCalculator.initializeIfNotReady(); // ✅ Good: if it doesn't affect other components
  // Continue rendering...
}
```

### Changing the DOM

컴포넌트 로직에서 사용자에게 직접 보이는 사이드 이펙트는 허용되지 않는다.
즉, 단순히 컴포넌트를 호출하는 것만으로 화면이 변경되어선 안된다.

```tsx
function ProductDetailPage({ product }) {
  document.title = product.title; // 🔴 Bad: Changes the DOM
}
```

다큐먼트의 타이틀을 변경하는 방법은, 렌더 외부에서 동기화시키는 것이다.

컴포넌트를 여러 번 호출하는 것이 안전하고 다른 컴포넌트의 렌더링에 영향을 미치지 않는 한, 리액트는 100% 순수하지 않아도 신경쓰지 않는다.
컴포넌트가 멱등적이어야 한다는 것이 더 중요하다.

## Props and state are immutable

컴포넌트의 프랍과 상태는 불변한 스냅샷이다. 절대 직접 뮤테이트하지 마라. 대신, 새로운 프랍을 전달하고 `useState`의 세터를 사용해라.

프랍과 상태를 렌더링 후 변경되는 스냅샷으로 생각할 수 있다. 이 이유로, 프랍과 상태를 직접 수정하지말고: 새로운 프랍을 전달하거나, 리액트가 컴포넌트의 다음 렌더링을 알리기 위한 리액트에서 제공되는 세터 함수를 사용해라.

### Don’t mutate Props

프랍은 불변하다. 만약 뮤테이트할 경우, 앱은 일관되지 않은 아웃풋을 생성하고, 이는 상황에 따라 작동할 수도 있고 안할 수도 있음로 디버깅이 어렵다.

```tsx
function Post({ item }) {
  item.url = new Url(item.url, base); // 🔴 Bad: never mutate props directly
  return <Link url={item.url}>{item.title}</Link>;
}
```

```tsx
function Post({ item }) {
  const url = new Url(item.url, base); // ✅ Good: make a copy instead
  return <Link url={url}>{item.title}</Link>;
}
```

### Don’t mutate State

`useState`는 상태 변수와 상태를 업데이트하기 위핸 세터를 반환한다.

```tsx
const [stateVariable, setter] = useState(0);
```

상태 변수를 직접 업데이트하는 대신, 유즈스테이트에서 반환된 세터 함수를 사용해 업데이트 해야 한다.
상태 변수의 값을 변경하는 것은 컴포넌트 업데이트를 유발하지 않고, 유저에게 오래된 UI를 보여준다.
세터 함수를 통해 리액트에게 상태가 변했다는 것을 알려주고, UI를 변경하기 위한 대기열에 넣어야한다.

```tsx
function Counter() {
  const [count, setCount] = useState(0);

  function handleClick() {
    count = count + 1; // 🔴 Bad: never mutate state directly
  }

  return <button onClick={handleClick}>You pressed me {count} times</button>;
}
```

```tsx
function Counter() {
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(count + 1); // ✅ Good: use the setter function returned by useState
  }

  return <button onClick={handleClick}>You pressed me {count} times</button>;
}
```

## Return values and arguments to Hooks are immutable

한 번 값이 훅으로 전달될 경우, 절대 수정하면 안된다.
프랍과 같이, 훅으로 전달되는 값은 불변해야한다.

```tsx
function useIconStyle(icon) {
  const theme = useContext(ThemeContext);
  if (icon.enabled) {
    icon.className = computeStyle(icon, theme); // 🔴 Bad: never mutate hook arguments directly
  }
  return icon;
}
```

```tsx
function useIconStyle(icon) {
  const theme = useContext(ThemeContext);
  const newIcon = { ...icon }; // ✅ Good: make a copy instead
  if (icon.enabled) {
    newIcon.className = computeStyle(icon, theme);
  }
  return newIcon;
}
```

리액트에서 중요한 하나의 원칙은 **지역 추론**이다: 코드만을 보고 컴포넌트나 훅이 무엇인지 이해할 수 있는 능력
훅은 호출 됐을 때, 블랙 박스처럼 다뤄져야한다.
예를 들어, 커스텀 훅이 인수를 의존성으로 사용해 훅 내부의 값을 메모이제이션할 수 있다:

```tsx
function useIconStyle(icon) {
  const theme = useContext(ThemeContext);

  return useMemo(() => {
    const newIcon = { ...icon };
    if (icon.enabled) {
      newIcon.className = computeStyle(icon, theme);
    }
    return newIcon;
  }, [icon, theme]);
}
```

만약 훅의 인수를 뮤테이트할 경우, 커스텀 훅의 메모이제이션은 정확하지 않을 것이다.
이를 피하는 것은 중요하다.

```tsx
style = useIconStyle(icon); // `style` is memoized based on `icon`
icon.enabled = false; // Bad: 🔴 never mutate hook arguments directly
style = useIconStyle(icon); // previously memoized result is returned
```

```tsx
style = useIconStyle(icon); // `style` is memoized based on `icon`
icon = { ...icon, enabled: false }; // Good: ✅ make a copy instead
style = useIconStyle(icon); // new value of `style` is calculated
```

메모된 훅의 값을 수정하지 않는 것도 비슷하다.

## Values are immutable after being passed to JSX

JSX에서 사용된 값을 뮤테이트하지 마라.
JSX가 생성되기 전으로 뮤테이션을 옮겨라.

표현식에서 JSX를 쓰는 경우, 리액트는 렌더링이 완료되기 전에 JSX를 먼저 평가할 수 있다.
즉 JSX 값이 전달된 후에 값을 변경하는 것은 UI가 오래될 수 있음을 의미한다.
리액트가 컴포넌트의 결과를 업데이트할 방법을 알지 못하기 때문이다.

```tsx
function Page({ colour }) {
  const styles = { colour, size: "large" };
  const header = <Header styles={styles} />;
  styles.size = "small"; // 🔴 Bad: styles was already used in the JSX above
  const footer = <Footer styles={styles} />;
  return (
    <>
      {header}
      <Content />
      {footer}
    </>
  );
}
```

```tsx
function Page({ colour }) {
  const headerStyles = { colour, size: "large" };
  const header = <Header styles={headerStyles} />;
  const footerStyles = { colour, size: "small" }; // ✅ Good: we created a new value
  const footer = <Footer styles={footerStyles} />;
  return (
    <>
      {header}
      <Content />
      {footer}
    </>
  );
}
```
