# renderToString

`renderToString`는 리액트 트리를 HTML 스트링으로 렌더한다.

```tsx
const html = renderToString(reactNode, options?)
```

## Reference

### renderToString(reactNode, options?)

서버에서 호출하여, 앱을 HTML로 렌더한다.

```tsx
import { renderToString } from "react-dom/server";

const html = renderToString(<App />);
```

클라이언트에서 `hydrateRoot`를 호출해 서버에서 생성된 HTML를 동적으로 만든다.

#### Parameters

- 리액트노드
- identifierPrefix

#### Returns

HTML 문자열

#### Caveats

- `renderToString`은제한된 서스펜스 지원을 갖는다. 만약 컴포넌트가 중단되면 즉시 폴백을 보여준다.
- `renderToString`은 브라우저에서 동작하지만, 클라이언트 코드에서 사용하는 것은 추천되지 않는다.

## Usage

### Rendering a React tree as HTML to a string

`renderToString`을 호출해 서버 응답으로 앱을 HTML로 보낼 수 있다.

```ts
import { renderToString } from "react-dom/server";

// The route handler syntax depends on your backend framework
app.use("/", (request, response) => {
  const html = renderToString(<App />);
  response.send(html);
});
```

이것은 정적 HTML 결과를 생성하지만, 클라이언트에서 `hydrateRoot`를 통해 동적 HTML이 된다.

## Alternatives

### Migrating from renderToString to a streaming render on the server

`renderToString`은 즉시 문자열을 반환하므로, 스트리밍을 지원하지 않는다.

가능하다면, 다음과 같은 모든 기능을 갖춘 대안을 사용하는 것을 추천한다:

- 노드 -> `renderToPipeableStream`
- 웹 스트림 환경의 디노 또는 모던 엣지 런타임 -> `renderToReadableStream`

만약 서버 환경이 스트림을 지원하지 않으면 사용할 수 있다.

### Migrating from renderToString to a static prerender on the server

`renderToString`은 즉시 문자열을 반환하므로, 정적 HTML 생성을 위한 데이터가 로드될 때까지 기다리는 기능은 지원되지 않는다.

다음과 같은 모든 기능을 갖춘 대안을 사용하는 것을 추천한다:

- 노드 -> `prerenderToNodeStream`
- 웹 스트림 환경의 디노 또는 모던 엣지 런타임 -> `prerender`

### Removing renderToString from the client code

## Troubleshooting

### When a component suspends, the HTML always contains a fallback

`renderToString`은 서스펜스를 완벽히 지원하지 않는다.

어떤 컴포넌트가 중단될 경우, `renderToString`은 컨텐츠가 리졸브되는 것을 기다리지 않는다.
그 대신 가장 가까운 서스펜스 바운더리의 폴백을 보여준다. 컨텐츠는 클라이언트 코드가 로드될 때까지 나타나지 않는다.

이런 문제를 해결하기 위해, 스트리밍을 사용해라. 서버 사이드 렌더링에서, 그들은 컨텐츠를 청크 단위로 리졸브하여 서버에서 클라이언트로 점진적으로 유저가 볼 수 있도록한다.
정적 생산을 위해선, 모든 컨텐츠가 리졸브되는 것을 기다린다.
