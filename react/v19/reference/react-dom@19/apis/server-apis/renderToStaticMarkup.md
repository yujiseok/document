# renderToStaticMarkup

`renderToStaticMarkup`은 상호작용이 없는 리액트 트리를 HTML 스트링으로 렌더한다.

```tsx
const html = renderToStaticMarkup(reactNode, options?)
```

## Reference

### renderToStaticMarkup(reactNode, options?)

서버에서 `renderToStaticMarkup`을 호출해 앱을 HTML로 렌더한다.

```tsx
import { renderToStaticMarkup } from "react-dom/server";

const html = renderToStaticMarkup(<Page />);
```

이것은 상호작용이 없는 리액트 컴포넌트의 HTML을 반환한다.

#### Parameters

- 리액트노드
- 서버 렌더를 위한 옵션
  - identifierPrefix: `useId` 훅으로 만든 id

#### Returns

HTML 문자열

#### Caveats

- `renderToStaticMarkup`의 결과는 하이드레이트 되지 않는다.
- `renderToStaticMarkup`은 제한된 서스펜스 지원을 갖는다. 만약 컴포넌트가 중단되면, `renderToStaticMarkup`은 즉시 폴백을 보낸다.
- `renderToStaticMarkup`은 브라우저에서 동작하지만, 클라이언트 코드 사용은 추천되지 않는다. 만약 컴포넌트를 브라우저에 렌더할 경우, 돔 노드를 렌더링해 HTML을 가져와라.

## Usage

### Rendering a non-interactive React tree as HTML to a string

`renderToStaticMarkup`을 호출해 앱을 서버 응답과 함께 보낼 수 있는 HTML 문자열로 렌더링한다.

```ts
import { renderToStaticMarkup } from "react-dom/server";

// The route handler syntax depends on your backend framework
app.use("/", (request, response) => {
  const html = renderToStaticMarkup(<Page />);
  response.send(html);
});
```

이것은 정적인 HTML 결과물을 생성한다.

> Pitfall
>
> 이 메서드는 하이드레이션할 수 없는 정적 HTML을 렌더한다. 정적 페이지를 만들거나, 이메일 같은 컨텐츠를 렌더링할 때 유용하다.
> 동적 앱을 만들기 위해선 `renderToString`과 `hydrateRoot`를 사용해라
