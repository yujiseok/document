# prerenderToNodeStream

`prerenderToNodeStream`는 노드 웹스트림을 이용해 리액트 트리를 정적 HTML로 렌더

`const {prelude} = await prerenderToNodeStream(reactNode, options?)`

## Reference

### prerenderToNodeStream(reactNode, options?)

`prerenderToNodeStream`를 호출해 앱을 정적 HTML로 렌더해라

```tsx
import { prerenderToNodeStream } from "react-dom/static";

// The route handler syntax depends on your backend framework
app.use("/", async (request, response) => {
  const { prelude } = await prerenderToNodeStream(<App />, {
    bootstrapScripts: ["/main.js"],
  });

  response.setHeader("Content-Type", "text/plain");
  prelude.pipe(response);
});
```

하이드레이트루트에서 서버에서 생성된 HTML에 상호작용을 추가한다.

#### Parameters

- `reactNode`: HTML에 렌더하고 싶은 리액트 노드. 예를 들어, JSX (`<App />`). 전체 앱을 요하므로, 앱 컴포넌트는 `<html>` 태그를 포함해야한다.

- `options`: 정적 생산에 필요한 옵션 객체
  - `bootstrapScriptContent`: 명시할 경우, 인라인 `<script>`로 치환됨
  - `bootstrapScripts`: 문자열 URL 배열로 페이지의 `<script>` 태그로 내보내짐. 포함될 경우 하이드레이트 루트가 실행됨. 리액트가 클라이언트에서 실행되는 것이 싫다면 생략.
  - `bootstrapModules`: `bootstrapScripts`과 비슷하지만, `<script type="module">`로 내보내짐
  - identifierPrefix, namespaceURI
  - `onError`: 서버에서 에러가 발생할 경우 실행되는 콜백. 회복가능하든 안하든 상관없음. `console.error`를 호출하고, 크래쉬 리포트를 만들고 싶다면 오버라이드할 수 있지만 `console.error`는 호출해야함. 상태코드 역시 수정할 수 있다.
  - `progressiveChunkSize`: 청크의 바이트 수
  - `signal`: 중단 시그널로 서버 렌더링을 중단하고 클라이언트 렌더링을 진행

#### Returns

`prerenderToNodeStream`은 프로미스를 반환한다:

- 만약 성공할 시, 프로미스는 해결되고 객체를 반환한다:
  - `prelude`: HTML의 노드 스트림. 해당 스트림으로 응답을 청크로 보낼 수 있으며, 전체 스트림을 문자열로 읽을 수 있다.
- 렌더링이 실패할 시, 프로미스는 거절된다. 폴백 쉘을 사용

SSG 시 사용해라.

## Usage

### Rendering a React tree to a stream of static HTML

`prerenderToNodeStream`는 노드 웹스트림을 이용해 리액트 트리를 정적 HTML로 렌더

```tsx
import { prerenderToNodeStream } from "react-dom/static";

// The route handler syntax depends on your backend framework
app.use("/", async (request, response) => {
  const { prelude } = await prerenderToNodeStream(<App />, {
    bootstrapScripts: ["/main.js"],
  });

  response.setHeader("Content-Type", "text/plain");
  prelude.pipe(response);
});
```

루트 컴포넌트(앱)에 스크립트 경로를 제공해야한다. 루트 컴포넌트는 반드시 루트 `<html>` 태그를 포함해야한다.

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

리액트는 독타입과 부트스랩된 스크립트 태그들을 HTML에 주입한다.

```tsx
<!DOCTYPE html>
<html>
  <!-- ... HTML from your components ... -->
</html>
<script src="/main.js" async=""></script>
```

클라이언트에서 부트스트랩된 스크립트는 하이드레이트루트를 호출해 하이드레이션을 진행해야한다.

```tsx
import { hydrateRoot } from "react-dom/client";
import App from "./App.js";

hydrateRoot(document, <App />);
```

이것은 정적으로 서버에서 생성된 HTML에 이벤트 리스너를 부착한다.

### Rendering a React tree to a string of static HTML

프리렌더를 호출해 정적 HTML 스트링으로 렌더:

```tsx
import { prerenderToNodeStream } from "react-dom/static";

async function renderToString() {
  const { prelude } = await prerenderToNodeStream(<App />, {
    bootstrapScripts: ["/main.js"],
  });

  return new Promise((resolve, reject) => {
    let data = "";
    prelude.on("data", (chunk) => {
      data += chunk;
    });
    prelude.on("end", () => resolve(data));
    prelude.on("error", reject);
  });
}
```

이것은 정적 HTML을 생성한다. 상호작용을 하기 위해선 하이드레이트루트를 호출해야한다.

### Waiting for all data to load

프리렌더랑 동일함

## Troubleshooting

### My stream doesn’t start until the entire app is rendered

프리렌더랑 동일함
