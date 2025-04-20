# React calls Components and Hooks

리액트는 유저 경험을 최적화 하기 위해 컴포넌트 렌더링과 훅의 실행을 책임진다.
리액트는 선언적이다: 리액트에게 특정 컴포넌트 로직을 렌더링 시키면, 리액트는 최적의 방법을 찾아 유저에게 보여준다.

## Never call component functions directly

컴포넌트는 오직 JSX에서만 사용되어야한다. 일반적인 함수처럼 호출하지마라. 리액트가 호출할 것이다.

리액트는 반드시 렌더링 중 컴포넌트를 호출하는 것을 결정해야한다. 리액트에서 JSX를 사용하면 가능하다.

```tsx
function BlogPost() {
  return (
    <Layout>
      <Article />
    </Layout>
  ); // ✅ Good: Only use components in JSX
}
```

```tsx
function BlogPost() {
  return <Layout>{Article()}</Layout>; // 🔴 Bad: Never call them directly
}
```

만약 컴포넌트가 훅을 포함한다면, 컴포넌트를 루프나 조건에서 직접 호출했을 경우 훅의 규칙을 어기기 쉽다.

리액트가 알아서 렌더링하도록 하는 것은 여러 이점이 존재한다:

- 컴포넌트는 함수 이상의 것이 된다. 리액트는 트리에 컴포넌트의 아이디에 연결된 훅을 통해 로컬 상태와 같은 기능으로 이를 확장할 수 있다.
- 컴포넌트 타입은 재조정에 들어간다. 리액트가 컴포넌트를 호출하도록 하는 것은, 트리의 개념적 구조에 대해 더 자세히 알릴 수 있다. 예를 들어, 렌더링이 피드에서 프로필 페이지로 이동했을 때 리액트는 재사용 시도를 하지 않는다.
- 리액트는 유저 경험을 향상한다. 예를 들어, 컴포넌트 호출 사이에 일부 작업을 수행하여 거대한 컴포넌트 트리를 리렌더링해도 메인 스레드가 차단되지 않는다.
- 더 나은 디버깅 이력. 컴포넌트가 라이브러리에서 인식할 수 있는 일급 객체일 경우, 개발 과정에서 내부적으로 개발자 도구를 구축할 수 있다.
- 더 효율적인 재조정. 리액트는 정확히 트리의 어떤 컴포넌트가 리렌더링 되고 스킵될지 결정한다. 이것은 앱을 빠르고 스내피하게 만들어준다.

### Never pass around Hooks as regular values

훅은 반드시 컴포넌트 또는 훅 안에서 호출되어야한다. 일반 값처럼 전달하지마라.
훅을 사용하면 컴포넌트가 리액트 기능을 쓸 수 있게한다. 그들은 항상 함수로 실행되어야하고, 일반 값처럼 전달되면 안된다.
이를 통해 컴포넌트의 지역 추론이 가능해지고, 개발자는 해당 컴포넌트를 격리하여 해당 컴포넌트가 수행하는 작업을 이해할 수 있다.

룰을 어기면 리액트가 자동으로 최적화를 진행하지 못하게 한다.

### Don’t dynamically mutate a Hook

훅은 가능하면 정적이어야한다. 즉 동적으로 뮤테이트하지 말아야한다. 예를 들어, 고차훅:

```tsx
function ChatInput() {
  const useDataWithLogging = withLogging(useData); // 🔴 Bad: don't write higher order Hooks
  const data = useDataWithLogging();
}
```

훅은 불변하고 뮤테이트되면 안된다. 훅을 뮤테이트하지 말고, 정적 버전의 훅을 만들어라.

```tsx
function ChatInput() {
  const data = useDataWithLogging(); // ✅ Good: Create a new version of the Hook
}

function useDataWithLogging() {
  // ... Create a new version of the Hook and inline the logic here
}
```

## Don’t dynamically use Hooks

또한, 훅은 동적으로 사용되면 안된다. 예를 들어, 훅을 컴포넌트의 프랍에 값으로 전달하는 것:

```tsx
function ChatInput() {
  return <Button useData={useDataWithLogging} />; // 🔴 Bad: don't pass Hooks as props
}
```

반드시 컴포넌트 내에서 인라인으로 훅을 호출하고 로직을 다뤄라.

```tsx
function ChatInput() {
  return <Button />;
}

function Button() {
  const data = useDataWithLogging(); // ✅ Good: Use the Hook directly
}

function useDataWithLogging() {
  // If there's any conditional logic to change the Hook's behavior, it should be inlined into
  // the Hook
}
```

이 방법은 버튼을 더 이해하기 쉽고 디버깅을 용이하게 해준다. 훅을 동적으로 사용하면, 복잡도가 높아지며 앱은 매우 제한적이고 지역 추론을 방해하게되어 팀의 생산성을 낮춘다. 또 훅의 규칙을 어기기 쉬워진다. 가능하면 e2e로 테스트 하는 것이 더 좋다.
