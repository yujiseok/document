# prerender

`prerender`는 웹 스트림을 이용해 리액트 트리를 정적 HTML로 렌더

```tsx
const {prelude} = await prerender(reactNode, options?)
```

## Reference

### prerender(reactNode, options?)

prerender을 호출해 앱을 정적 HTML 렌더한다.

```ts
import { prerender } from "react-dom/static";

async function handler(request) {
  const { prelude } = await prerender(<App />, {
    bootstrapScripts: ["/main.js"],
  });
  return new Response(prelude, {
    headers: { "content-type": "text/html" },
  });
}
```

클라이언트에서 하이드레이트루트가 서버에서 생성된 HTML에 인터렉션을 추가한다.

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

[default heuristic](https://github.com/facebook/react/blob/14c2be8dac2d5482fda8a0906a31d239df8551fc/packages/react-server/src/ReactFizzServer.js#L210-L225)

> progressiveChunkSize / default heuristic
> 500밀리초 간격으로 점진적 로딩이 필요하다. 그보다 빠른 업데이트는 불필요하며, 클라이언트에서 제한되어야한다.
> 또한 너무 많이 분할될 경우 오버헤드가 증가한다. 고성능 장치의 경우는 문제가 없지만, 기기를 알 수 없으므로 저성능을 우선 고려한다.
> 최대 대역폭의 80%를 사용한다고 가정하고 500밀리초마다 그 절반의 데이터를 수신할 수 있다고 본다.
> 즉 500밀리초마다 최소 12.5KB의 컨텐츠를 표시할 수 있어야한다.

#### Returns

`prerender`는 프로미스를 반환한다:

- 만약 렌더링이 성공할 경우, 다음을 포함한 객체를 반환:
  - `prelude`: HTML의 웹스트림. 응답을 청크로 보낼 수도 있으며, 전체 스트림을 문자로 읽을 수도 있다.
- 만약 렌더링이 실패하면, 프로미스는 거절된다. 폴백쉘을 사용.

> When should I use prerender?
>
> 정적 프리렌더는 정적 서버 사이드 생성시 사용된다. 렌더투스트링과 다르게, 프리렌더는 모든 데이터가 리졸브 될 때까지 기다린다.
> 이것은 서스펜스를 사용해 가져와야 하는 데이터를 포함해 전체 페이지에 대한 정적 HTML을 생성할 경우 적합하다.
> 스트림 -> 렌터투리더블스트림

## Usage

### Rendering a React tree to a stream of static HTML

리액트 트리를 정적 HTML로 렌더링하여 읽을 수 있는 웹 스트림으로 만들려면 프리렌더를 호출해라.

```tsx
import { prerender } from "react-dom/static";

async function handler(request) {
  const { prelude } = await prerender(<App />, {
    bootstrapScripts: ["/main.js"],
  });
  return new Response(prelude, {
    headers: { "content-type": "text/html" },
  });
}
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

> DEEP DIVE
> Reading CSS and JS asset paths from the build output
>
> 빌드 후 에셋은 종종 해쉬된다. 예를 들어 `styles.css`가 아니라 `styles.1234.css`를 결과로 볼 것이다.
> 해싱은 모든 고유한 빌드가 다른 파일 이름을 갖게되는 것을 보장한다. 이 기능은 정적 자산에 대한 장기 캐싱을 안전하게 활성화할 수 있어서 유용하다.
> 특정 이름을 가진 파일은 절대 컨텐츠를 변경하지 않는다.
>
> 하지만, 만약 에셋 URL을 빌드 후에도 모를 경우, 소스코드에 넣을 방법이 없다. 예를 들어, 하드코딩된 `"/styles.css"`은 적용되지 않을 것이다.
> 소스코드에서 이를 차단하려면, 루트 컴포넌트에서 실제 파일 이름을 맵 형태의 프랍으로 받아야한다.
>
> ```tsx
> export default function App({ assetMap }) {
>   return (
>     <html>
>       <head>
>         <title>My app</title>
>         <link rel="stylesheet" href={assetMap["styles.css"]}></link>
>       </head>
>       ...
>     </html>
>   );
> }
> ```
>
> 서버에서, `<App assetMap={assetMap} />`를 렌더하고 에셋 URL을 포함한 `assetMap`을 프랍으로 넘겨라:
>
> ```tsx
> // You'd need to get this JSON from your build tooling, e.g. read it from the build output.
> const assetMap = {
>   "styles.css": "/styles.123456.css",
>   "main.js": "/main.123456.js",
> };
>
> async function handler(request) {
>   const { prelude } = await prerender(<App assetMap={assetMap} />, {
>     bootstrapScripts: [assetMap["/main.js"]],
>   });
>   return new Response(prelude, {
>     headers: { "content-type": "text/html" },
>   });
> }
> ```
>
> 서버는 프랍을 가진 컴포넌트를 렌더링한다. 하이드레이션 에러를 없애기 위해 클라이언트에서 렌더해야한다. 직렬화한 에셋맵을 전달한다.
>
> ```tsx
> // You'd need to get this JSON from your build tooling.
> const assetMap = {
>   "styles.css": "/styles.123456.css",
>   "main.js": "/main.123456.js",
> };
>
> async function handler(request) {
>   const { prelude } = await prerender(<App assetMap={assetMap} />, {
>     // Careful: It's safe to stringify() this because this data isn't user-generated.
>     bootstrapScriptContent: `window.assetMap = ${JSON.stringify(assetMap)};`,
>     bootstrapScripts: [assetMap["/main.js"]],
>   });
>   return new Response(prelude, {
>     headers: { "content-type": "text/html" },
>   });
> }
> ```
>
> 예제에서 `bootstrapScriptContent`는 전역 `window.assetMap` 변수를 추가하는 인라인 스크립트 태그를 추가한다. 이것은 클라이언트 코드에서
> 동일한 에셋맵을 읽을 수 있게한다:
>
> ```tsx
> import { hydrateRoot } from "react-dom/client";
> import App from "./App.js";
>
> hydrateRoot(document, <App assetMap={window.assetMap} />);
> ```
>
> 서버와 클라이언트 둘 다 동일한 프랍을 갖기에 하이드레이션 에러가 발생하지 않는다.

### Rendering a React tree to a string of static HTML

프리렌더를 호출해 앱을 스태틱 HTML 스트링으로 렌더:

```tsx
import { prerender } from "react-dom/static";

async function renderToString() {
  const { prelude } = await prerender(<App />, {
    bootstrapScripts: ["/main.js"],
  });

  const reader = prelude.getReader();
  let content = "";
  while (true) {
    const { done, value } = await reader.read();
    if (done) {
      return content;
    }
    content += Buffer.from(value).toString("utf8");
  }
}
```

이것은 정적 HTML을 생성한다. 상호작용을 하기 위해선 하이드레이트루트를 호출해야한다.

### Waiting for all data to load

프리렌더는 정적 HTML 생성과 리졸브를 전에 모든 데이터가 로드되는 것을 기다린다.
예를 들어, 커버, 친구와 사진을 갖는 사이드바 그리고 포스트 목록을 갖는 페이지가 있다고 하자:

```tsx
function ProfilePage() {
  return (
    <ProfileLayout>
      <ProfileCover />
      <Sidebar>
        <Friends />
        <Photos />
      </Sidebar>
      <Suspense fallback={<PostsGlimmer />}>
        <Posts />
      </Suspense>
    </ProfileLayout>
  );
}
```

`<Posts />`가 데이터 로드하는 데 시간이 걸린다고 가정해보자. 이상적으로, 게시물이 완료되어 HTML에 포함될 때까지 기다리고 싶을 것이다.
이것을 구현하기 위해, 서스펜스를 통해 데이터를 기다릴 수 있다. 프리렌더는 중단된 컨텐츠가 완료될 때까지 기다린다.

## Troubleshooting

### My stream doesn’t start until the entire app is rendered

프리렌더 응답은 모든 서스펜스가 리졸브될 때까지 기다리는 것을 포함해 전체 앱의 렌더링이 완료될 때까지 기다린다.
이 기능은 정적 사이트 생성을 위해 설계되었으며, 로드하는 동안 추가 컨텐츠 스트리밍을 지원하지 않는다.

컨텐츠를 스트림하고 싶으면, 렌더투리더블스트림을 써라.
