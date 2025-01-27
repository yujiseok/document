# hydrateRoot

`hydrateRoot`는 리액트 컴포넌트가 `react-dom/server`로 이미 만들어진 HTML 브라우저 돔 노드에 표시될 수 있게 해준다.

```tsx
const root = hydrateRoot(domNode, reactNode, options?)
```

## Reference

### hydrateRoot(domNode, reactNode, options?)

이미 리액트 서버 환경에서 생성도니 HTML에 `hydrateRoot`를 호출해 리액트를 붙일 수 있다.

```tsx
import { hydrateRoot } from "react-dom/client";

const domNode = document.getElementById("root");
const root = hydrateRoot(domNode, reactNode);
```

리액트는 `domNode`를 위한 루트를 생성하고, 그 돔 내부의 관리를 맡는다.
완전히 리액트로 만들어진 앱의 경우 보통 루트 컴포넌트를 위한 오직 한 번의 `hydrateRoot` 호출이 있다.

#### Parameters

- `domNode`: 돔 요소. 서버에서 렌더되는 루트 요소.
- `reactNode`: 기존 HTML을 렌더링하는 데 사용되는 리액트 노드. JSX의 조각. 리액트 돔 서버의 API에 의해 렌더링됨. `renderToPipeableStream` 같은
- 선택적 옵션: 리액트 루트를 위한 옵션
  - `onCaughtError`: 에러 바운더리에서 에러를 잡았을 경우 호출될 콜백. 에러 바운더리에서 잡은 에러와 `componentStack`을 포함한 `errorInfo` 객체가 존재할 경우 호출된다.
  - `onUncaughtError`: 에러 바운더리에서 에러를 잡지 못했을 경우 호출될 콜백. 에러가 던져졌을 때와 `componentStack`을 포함한 `errorInfo` 객체가 존재할 경우 호출된다.
  - `onRecoverableError`: 리액트가 자동으로 에러에서 회복되었을 경우 호출될 콜백. 리액트가 던진 에러와 `componentStack`을 포함한 `errorInfo` 객체가 존재할 경우 호출된다. 어떤 회복 가능한 에러는 원본 에러의 `error.cause`를 포함한다.
  - `identifierPrefix`: `useId` 훅으로 생성된 아이디 프리픽스. 동일 페이지에 여러 루트가 있을 경우 충돌을 방지한다.

#### Returns

`hydrateRoot`는 렌더와 언마운트를 갖는 객체를 반환한다.

#### Caveats

- `hydrateRoot`는 렌더된 컨텐츠와 서버에서 렌더된 컨텐츠가 동일하다고 기대한다. 그러므로, 미스매치 에러를 해결해야한다.
- 개발 환경에서, 리액트는 하이드레이션 과정에 미스매치를 알려준다. 미스매치로 인해 속성 차이가 패치될 것이라는 보장은 없다. 대부분의 앱에서 미스매치가 발생하는 경우는 드물기 때문에 모든 마크업을 검증하는 것은 엄청난 비용이 들기 때문이다.
- 오직 하나의 `hydrateRoot` 호출이 필요하다면, 프레임워크를 사용해라.
- 클라이언트 렌더라면 `createRoot`를 사용해라.

### root.render(reactNode)

브라우저 돔 요소의 하이드레이트된 리액트 내에서 컴포넌트를 업데이트하기위해 루트 렌더를 호출한다.

```tsx
root.render(<App />);
```

#### Parameters

- `ReactNode`: 표시하고 싶은 리액트 노드. JSX의 조각일 수도 있고, `createElement()`로 만들어진 리액트 엘리먼트, 문자, 넘버, 널, 언디파인드도 전달할 수 있다.

#### Returns

언디파인드

#### Caveats

하이드레이트가 완료되지 않았을 때, 루트 렌더를 실행하면, 리액트는 서버에서 렌더된 HTML 컨텐츠와 전체 앱을 클라리언트 렌더링으로 교체한다.

### root.unmount()

## Usage

### Hydrating server-rendered HTML

만약 앱의 HTML이 `react-dom/server`에 의해 생성되었다면, 클라이언트에서 하이드레이트가 필요하다.

```tsx
import { hydrateRoot } from "react-dom/client";

hydrateRoot(document.getElementById("root"), <App />);
```

이것은 리액트 컴포넌트가 존재하는 브라우저 돔 노드의 서버 HTML을 하이드레이트한다.
프레임워크를 사용하면, 알아서 해준다.

앱을 하이드레이트하기 위해 리액트는 서버에서 생성된 HTML에 컴포넌트 로직을 부착한다.
하이드레이션은 서버의 스냅샷을 완전히 상호작용할 수 있게 해준다.

> Pitfall
> 하이드레이트루트에 넘긴 리액트 트리는 서버에서 생성된 것과 동일한 결과를 생성해야한다.
>
> 이것은 UX에 중요하다. 유저들은 자바스크립트가 로드 되기 전, 서버에서 생성된 HTML을 본다.
> 서버 렌더링은 HTML 스앱샷을 보여줌으로 앱이 빠르게 로드된다는 착각을 일으킨다. 갑작스러운 일치하지 않는 컨텐츠는 이런 착각을 부순다. 이것이 서버 렌더 결과물이 클라이언트에서 첫 렌더 결과물과 같아야하는 이유다.
>
> 하이드레이션 에러를 유발하는 경우:
>
> - 루트 노드 내 리액트가 생성한 HTML의 추가적인 공백(줄 바꿈)
> - `typeof window !== 'undefined'`가 렌더링 로직에 포함될 때
> - 브라우저에서 만 사용되는 `window.matchMedia`가 렌더링 로직에 포함될 때
> - 서버와 클라이언트에서 다른 데이터를 렌더링할 때
>
> 리액트는 하이드레이션 에러를 리커버하지만, 다른 버그처럼 고쳐야한다. 좋은 경우, 속도가 저하되고, 최악의 경우 잘못된 요소에 이벤트 핸들러가 부착된다.

### Hydrating an entire document

완벽하게 리액트로 만들어진 앱의 경우, `<html>` 태그를 포함해 전체 문서를 JSX로 렌더한다.

```tsx
function App() {
  return (
    <html>
      <head>
        <meta charSet="utf-8" />
        <meta name="viewport" content="width=device-width, initial-scale=1" />
        <link rel="stylesheet" href="/styles.css"></link>
        <title>My app</title>
      </head>
      <body>
        <Router />
      </body>
    </html>
  );
}
```

전체 앱을 하이드레이트 하기 위해, `hydrateRoot`의 첫 인자로 글로벌 다큐먼트를 전달해라:

```tsx
import { hydrateRoot } from "react-dom/client";
import App from "./App.js";

hydrateRoot(document, <App />);
```

### Suppressing unavoidable hydration mismatch errors

만약 한 요소의 속성 또는 텍스트가 불가피하게 서버와 클라이언트에서 다를 경우(타임스탬프), 하이드레이션 미스매치 경고를 끌 수 있다.

하이드레이션 미스매치 경고를 끄기 위해 요소에 `suppressHydrationWarning={true}`를 추가해라:

이것은 오직 1뎁스에 적용되고, 비상구이다. 과하게 사용하지 마라.
텍스트라도, 리액트는 변경하지 않을 것이다. 따라서 업데이트까지는 일관성이 유지되지 않을 수 있다.

### Handling different client and server content

서버와 클라이언트에서 다른 것을 렌더링해야 한다면, two-pass 렌더링을 적용할 수 있다.
클라이언트에서 다르게 렌더되는 경우 상태 변수(`isClient`)의 상태를 이펙트 내에서 참으로 변경할 수 있다.

이렇게 하면 초기 렌더 패스는 서버와 동일한 컨텐츠를 렌더링하여 불일치를 방지하지만, 하이드레이트 직후에 추가 패스가 동기적으로 발생한다.

> Pitfall
>
> 이 접근은 렌더링을 두 번하기에 하이드레이션을 늦춘다. 연결 속도가 느린 사용자 경험을 유의해라. 자바스크립트 코드는 초기 HTML 렌더링보다 상당히 늦게 로드될 수 있다. 그러므로 하이드레이션 이후 다른 UI를 즉시 보여주는 것은 유저에게 덜그럭거리는 경험을 줄 수 있다.

### Updating a hydrated root component

루트가 하이드레이트 되었다면, `root.render`를 호출해 루트 리액트 컴포넌트를 변경할 수 있다. `createRoot`와 다르게, 초기 컨텐츠가 HTML로 렌더되었으므로 사용할 필요가 없다.

하이드레이션 이후 호출할 경우, 컴포넌트 트리는 이전 렌더와 비교한다. 리액트는 상태를 보존한다.

### Show a dialog for uncaught errors

### Displaying Error Boundary errors

### Show a dialog for recoverable hydration mismatch errors

## Troubleshooting

### I’m getting an error: “You passed a second argument to root.render”

---

하이드레이션 미스매치 에러가 왜 발생하는지 이해했다.
또 어떤 결과를 초래하는지 알게되었다.

미스매치를 해결하기 위한 방법을 각 상황에 맞게 잘 적용해야겠다.
