# createRoot

`createRoot`는 브라우저 돔 노드에 리액트 컴포넌트를 표시하는 루트를 만들 수 있다.

```tsx
const root = createRoot(domNode, options?)
```

## Reference

### createRoot(domNode, options?)

`createRoot`를 호출해 브라우저 돔 요소에 컨텐츠를 표시하기 위한 리액트 루트를 생성한다.

```tsx
import { createRoot } from "react-dom/client";

const domNode = document.getElementById("root");
const root = createRoot(domNode);
```

리액트는 `domNode`를 위한 루트를 생성하고, 그 돔 내부의 관리를 맡는다. 루트를 생성한 후, `root.render`를 호출해 리액트 컴포넌트를 표시해야한다.

```tsx
root.render(<App />);
```

완전히 리액트로 만들어진 앱의 경우 보통 루트 컴포넌트를 위한 오직 한 번의 `createRoot` 호출이 있다. 만약 리액트로 부분부분 만들어진 페이지의 경우 여러 루트가 존재한다.

#### Parameters

- `domNode`: 돔 요소. 리액트는 이 돔 요소를 위해 루트를 생성하고 `render`와 같은 리액트 컨텐츠를 보여주기 위한 함수를 실행할 수 있게 한다.
- 선택적 옵션: 리액트 루트를 위한 옵션
  - `onCaughtError`: 에러 바운더리에서 에러를 잡았을 경우 호출될 콜백. 에러 바운더리에서 잡은 에러와 `componentStack`을 포함한 `errorInfo` 객체가 존재할 경우 호출된다.
  - `onUncaughtError`: 에러 바운더리에서 에러를 잡지 못했을 경우 호출될 콜백. 에러가 던져졌을 때와 `componentStack`을 포함한 `errorInfo` 객체가 존재할 경우 호출된다.
  - `onRecoverableError`: 리액트가 자동으로 에러에서 회복되었을 경우 호출될 콜백. 리액트가 던진 에러와 `componentStack`을 포함한 `errorInfo` 객체가 존재할 경우 호출된다. 어떤 회복 가능한 에러는 원본 에러의 `error.cause`를 포함한다.
  - `identifierPrefix`: `useId` 훅으로 생성된 아이디 프리픽스. 동일 페이지에 여러 루트가 있을 경우 충돌을 방지한다.

#### Returns

`createRoot`는 `render`와 `unmount`를 갖는 객체를 반환한다.

#### Caveats

- 만약 앱이 SSR일 경우, `createRoot()` 사용은 제공되지 않는다. 대신에 `hydrateRoot()`를 사용해라.
- `createRoot`는 단 한번만 호출될 가능성이 크다. 프레임워크를 사용한다면, 그것이 호출할 것이다.
- 만약 자식 컴포넌트가 아닌 다른 돔 트리에 JSX를 렌더하고 싶다면, `createPortal`을 사용해라.

### root.render(reactNode)

`root.render`를 호출해 JSX(리액트 노드들)을 리액트 루트의의 브라우저 돔 노드에 표시한다.

```tsx
root.render(<App />);
```

리액트는 루트에 `<App />`을 표시하고, 그 내부의 돔을 관리한다.

#### Parameters

- `ReactNode`: 표시하고 싶은 리액트 노드. JSX의 조각일 수도 있고, `createElement()`로 만들어진 리액트 엘리먼트, 문자, 넘버, 널, 언디파인드도 전달할 수 있다.

#### Returns

언디파인드

#### Caveats

- `render`가 처음 실행되면, 리액트는 리액트의 컴포넌트를 렌더링하기 전에 리액트 루트 내의 모든 HTML 컨텐츠를 지운다.
- 만약 루트 돔 노드가 빌드 또는 리액트를 통해 서버에서 생성된 HTML 컨텐츠를 포함한다면, 기존 HTML에 이벤트 핸들러를 부착하는 `hydrateRoot()`를 사용해라.
- 동일한 루트에서 `render`를 여러번 호출할 경우, 리액트는 전달되는 최신 JSX를 반영하기 위해 필요에 따라 돔을 업데이트한다. 리액트는 돔의 어떤 부분이 재사용이 가능한지와 이전 트리와 비교를 통해 재생산해야할 부분을 결정한다. 렌더를 여러번 호출하는 것은 셋 함수를 루트 컴포넌트에서 실행하는 것과 비슷하다. 리액트는 불필요한 돔 변경을 피한다.

### root.unmount()

리액트 루트에 생성된 트리를 파괴하기 위해 호출해라.

```tsx
root.unmount();
```

완전히 리액트로 생성된 앱의 경우 호출할 필요가 없다.

주로 리액트 루트의 돔 노드 혹은 그 조상이 다른 코드에 의해 돔에서 제거될 수 있는 경우 유용하다.

해당 메서드 호출 시 모든 컴포넌트가 마운트 해체되고 이벤트 핸들러와 상태도 제거된다.

#### Parameters

없음

#### Returns

언디파인드

#### Caveats

- 호출 시 모든 컴포넌트가 언마운트되고 루트 돔 노드에서 리액트를 분리한다.
- 언마운트 호출 시, 동일한 루트에서 렌더를 호출할 수 없다. 시도할 경우 `Cannot update an unmounted root` 에러를 발생한다. 하지만, 동일 돔 노드에 새로운 루트를 생성하면 가능하다.

## Usage

### Rendering an app fully built with React

만약 앱이 완전히 리액트로 만들어졌다면, 전체 앱을 위한 단일 루트를 생성해라.

```tsx
import { createRoot } from "react-dom/client";

const root = createRoot(document.getElementById("root"));
root.render(<App />);
```

보통, 첫 실행에 해당 코드의 호출이 필요하다.

1. 브라우저의 돔 노드를 HTML에서 정의한다.
2. 앱 내부에 리액트 컴포넌트를 표시한다.

만약 앱이 완전히 리액트로 만들어졌다면, 추가 루트를 생성할 필요가 없으며 렌더를 재호출할 필요도 없다.

이때부터 리액트가 전체 앱의 돔을 제어한다. 컴포넌트를 추가하기 위해선, 앱 컴포넌트에 중첩해라.
UI를 변경하고 싶다면, 각 컴포넌트에서 상태를 사용해라. 돔 노드 외부에서 모달 또는 툴팁을 보여주고 싶을 경우, 포탈을 사용해라.

> **Note**
> HTML이 비었을 경우, 유저는 자바스크립트가 로드되고 실행되기 전까진 빈 화면을 보게된다.
>
> ```html
> <div id="root"></div>
> ```
>
> 이것은 매우 느리게 느껴진다. 이 문제를 해결하기 위해, 서버 또는 빌드 시 HTML을 생성할 수 있다. 그럼 유저는 텍스트를 읽을 수 있고, 이미지를 볼 수 있으며, 자바스크립트가 로드되기 전에 링크를 클릭할 수 있다. 프레임워크를 사용하면 별도의 설치 없이 해당 최적화를 사용할 수 있다.
> 어디서 실행되는지에 따라 SSR과 SSG로 불린다.

> **Pitfall**
> 서버 또는 정적으로 생성될 경우 반드시`hydrateRoot`를 호출해야한다.
> 그래야, 리액트가 HTML의 돔 노드를 파괴하는 것이 아닌 하이드레이트(재사용)를 실행한다.

### Rendering a page partially built with React

### Updating a root component

### Show a dialog for uncaught errors

기본적으로, 리액트는 잡히지 않은 모든 에러를 콘솔에 로깅한다.
커스텀한 에러 리포팅을 구현하기 위해선, `onUncaughtError` 옵션을 루트 옵션에 제공하면 된다.

```tsx
import { createRoot } from "react-dom/client";

const root = createRoot(document.getElementById("root"), {
  onUncaughtError: (error, errorInfo) => {
    console.error("Uncaught error", error, errorInfo.componentStack);
  },
});
root.render(<App />);
```

`onUncaughtError` 옵션은 두 인자와 함께 호출된다:

1. error
2. componentStack을 포함한 errorInfo

`onUncaughtError` 루트 옵션을 사용해 에러 다이알로그를 보여줄 수 있다:

> 19 버전부터 스테이블하게 되었다.
> vite로 앱 만들면 v18이라 안되어서 한참 삽질했네.. 🥺
> [ref](https://github.com/DefinitelyTyped/DefinitelyTyped/pull/69022/files#diff-81e131cae318073e369358c0c4439ff98f037d5b9af2e1382d61ca5937a584caR37-R54)

### Displaying Error Boundary errors

기본적으로, 리액트는 리액트는 에러 바운더리에서 발견된 모든 오류를 콘솔 에러에 기록한다.
이를 오버라이드 하기 위해선, `onCaughtError` 옵션을 전달해 에러 바운더리에서 잡은 에러를 다룰 수 있다:

```tsx
import { createRoot } from "react-dom/client";

const root = createRoot(document.getElementById("root"), {
  onCaughtError: (error, errorInfo) => {
    console.error("Caught error", error, errorInfo.componentStack);
  },
});
root.render(<App />);
```

`onCaughtError`는 두 인자와 함께 호출된다:

1. 바운더리로 잡은 에러
2. componentStack을 포함한 errorInfo

`onCaughtError`를 옵션을 통해 에러 다이얼로그를 보여주거나 로깅에서 알려진 에러를 필터링한다.

### Displaying a dialog for recoverable errors

리액트는 자동으로 컴포넌트를 두 번 렌더해 렌더에 발생한 에러 해결을 시도한다.
만약 성공한다면, 리액트는 회복 가능한 에러를 콘솔에 보여준다.
이것을 오버라이드 하기위해 `onRecoverableError` 옵션을 전달할 수 있다.

```tsx
import { createRoot } from "react-dom/client";

const root = createRoot(document.getElementById("root"), {
  onRecoverableError: (error, errorInfo) => {
    console.error(
      "Recoverable error",
      error,
      error.cause,
      errorInfo.componentStack
    );
  },
});
root.render(<App />);
```

`onRecoverableError`는 두 인자와 함께 호출된다:

1. 리액트가 던진 에러. 어떤 에러는 원본 에러의 cause를 포함한다.
2. componentStack을 포함한 errorInfo

---

rootOption이 매우 흥미롭다.
넥스트에서 뭔가 에러가 발생하면 저 rootOption을 사용해서 에러 다이얼로그를 보여주는 걸까?
