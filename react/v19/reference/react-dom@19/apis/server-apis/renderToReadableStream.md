# renderToReadableStream

`renderToReadableStream`는 리액트 트리를 읽을 수 있는 웹 스트림으로 렌더한다.

```tsx
const stream = await renderToReadableStream(reactNode, options?)
```

## Reference

### renderToReadableStream(reactNode, options?)

`renderToReadableStream`을 호출해 리액트 트리를 읽을 수 있는 웹 스트림으로 렌더한다.

```tsx
import { renderToReadableStream } from "react-dom/server";

async function handler(request) {
  const stream = await renderToReadableStream(<App />, {
    bootstrapScripts: ["/main.js"],
  });
  return new Response(stream, {
    headers: { "content-type": "text/html" },
  });
}
```

클라이언트에선 `hydrateRoot`를 통해 서버에서 생성된 HTML에 상호작용을 더한다.

#### Parameters

- `reactNode`: HTML에 렌더링하고 싶은 리액트 노드. 예를 들어, `<App />`과 같은 JSX 요소. 이것은 전체 문서를 나타내야 하므로, `App` 컴포넌트는 `<html>` 태그를 렌더해야 한다.
- `options`: 스트리밍 옵션들
  - `bootstrapScriptContent`: 명시될 경우, 인라인 `<script>` 태그로 배치된다.
  - `bootstrapScripts`: `<script>` 태그가 페이지에서 내보내는 문자열 URL 배열. `<script>`를 포함해 사용할 경우, `hydrateRoot`를 호출한다. 클라이언트에서 리액트를 사용하지 않을 경우 생략.
  - `bootstrapModules`: `bootstrapScripts`와 동일하지만, `<script type="module">`를 내보낸다.
  - `identifierPrefix`: 여러 루트가 동일한 페이지에 존재할 경우 충돌을 막기 위한 `useId`로 만들어진 접두사. `hydrateRoot`로 전달 받은 것과 동일해야 한다.
  - `namespaceURI`: 스트림을 위한 namespace URI. svg, mathML을 위한
  - `nonce`: CSP: script-src를 위한
  - `onAllReady`: 모든 렌더링이 성공한 후 실행되는 콜백으로, 쉘과 추가적인 컨텐츠를 포함한다. 크롤러와 정적 생성을 위해 `onShellReady`대신 사용할 수 있다. 이곳에서 스트리밍을 시작하면 점진적 로딩을 얻을 수 없다. 스트림은 최종 HTML을 포함할 것이다.
  - `onError`: 서버 에러가 발생할 경우 실행되는 콜백, 회복 가능하든 불가하든. 기본적으로, 이것은 `console.error`만을 호출한다. 만약 충돌 리포트를 로깅을 오버라이드 하더라도 `console.error`를 호출해야 한다. 쉘이 방출되기 전에 상태 코드를 조정하는 데 사용할 수 있다.
  - `progressiveChunkSize`: 청크 바이트의 수
  - `signal`: 서버 렌더링을 중단하기 위한 시그널. 서버 렌더링을 중단하고 나머지는 클라이언트에서 렌더링할 수 있게한다.

#### Returns

`renderToReadableStream`는 프로미스를 리턴한다.

- 만약 쉘이 성공적으로 렌더링되면, 프로미스는 웹 스트림으로 리졸브된다.
- 실패할 경우, 프로미스는 거절된다. 폴백 쉘을 보여준다.

반환된 스트림은 추가적인 프로퍼티를 갖는다:

- `allReady`: 모든 렌더링이 완료되었을 때 리졸브되는 프로미스, 쉘과 추가적인 컨텐츠를 포함한다. 크롤러와 정적 생산 전에 `await stream.allReady`을 사용할 수 있다. 이것을 사용하면, 점진적 로딩 없이 스트림은 완성된 HTML을 포함한다.

## Usage

### Rendering a React tree as HTML to a Readable Web Stream

`renderToReadableStream`을 호출해 리액트 트리를 읽을 수 있는 웹 스트림으로 렌더한다.

```tsx
import { renderToReadableStream } from "react-dom/server";

async function handler(request) {
  const stream = await renderToReadableStream(<App />, {
    bootstrapScripts: ["/main.js"],
  });
  return new Response(stream, {
    headers: { "content-type": "text/html" },
  });
}
```

루트 컴포넌트와 부트스랩 스크립트 경로를 제공해야한다.
루트 컴포넌트는 반드시 html 태그를 포함한 전체 문서를 반환해야한다.

예를 들어:

```tsx
export default function App() {
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

리액트는 독 타입과 스크립 태그들을 HTML 스트림 결과에 주입한다:

```html
<!DOCTYPE html>
<html>
  <!-- ... HTML from your components ... -->
</html>
<script src="/main.js" async=""></script>
```

클라이언트에서, 부트스트랩 스크립트는 `hydrateRoot`를 통해 하이드레이션 되어야한다.

```tsx
import { hydrateRoot } from "react-dom/client";
import App from "./App.js";

hydrateRoot(document, <App />);
```

이것은 서버에서 생성된 HTML에 상호 작용을 붙인다.

### Streaming more content as it loads

로딩이 필요한 컴포넌트를 서스펜스 바운더리로 감싼다. 그 외 쉘들은 빠르게 보여진다.

### Specifying what goes into the shell

`renderToReadableStream`에 대한 비동기 호출은 전체 쉘이 렌더링되는 즉시 스트림으로 리졸브된다. 일반적으로 해당 스트림으로 응답을 반환하고 스트리밍이 시작된다:

```tsx
async function handler(request) {
  const stream = await renderToReadableStream(<App />, {
    bootstrapScripts: ["/main.js"],
  });
  return new Response(stream, {
    headers: { "content-type": "text/html" },
  });
}
```

스트림이 반한되어도, 서스펜스 바운더리로 감싸진 컴포넌트는 데이터를 로딩 중일 것이다.

### Logging crashes on the server

### Recovering from errors inside the shell

기본 레이아웃을 보여주는 쉘에서 에러가 발생했을 경우

`renderToReadableStream`을 `try...catch`로 감싼다.
에러가 발생했을 경우 캐치 블락에서 폴백 HTML을 보여준다.

### Recovering from errors outside the shell

### Setting the status code

### Handling different errors in different ways

### Waiting for all content to load for crawlers and static generation

### Aborting server rendering

---

대부분의 내용이 `renderToPipeableStream`과 비슷하다.
동작하는 환경이 node 스트림이냐냐 웹 스트림이냐의 차이가 있다.
`renderToReadableStream`의 경우 프로미스를 리턴하여, `renderToPipeableStream`에 존재하는 에러 콜백들이 없어 `try...catch`에서 에러를 처리한다.

웹 스트림과 노드 스트림을 찾아봐야겠다.
