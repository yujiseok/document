# renderToPipeableStream

`renderToPipeableStream`은 pipeable 노드js 스트림으로 리액트 트리를 렌더한다.

## Reference

### renderToPipeableStream(reactNode, options?)

`renderToPipeableStream`을 호출해 리액트 트리를 노드js 스트림으로 렌더링

```tsx
import { renderToPipeableStream } from "react-dom/server";

const { pipe } = renderToPipeableStream(<App />, {
  bootstrapScripts: ["/main.js"],
  onShellReady() {
    response.setHeader("content-type", "text/html");
    pipe(response);
  },
});
```

클라이언트에선 `hydrateRoot`를 통해 서버에서 생성된 HTML을 상호작용하도록 한다.

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
  - `onShellReady`: 초기 쉘이 렌더되고 실행되는 콜백. 상태 코드를 설정할 수 있고 `pipe`를 호출해 스트리밍을 시작할 수 있다. 리액트는 HTML 로딩 폴백을 컨텐츠로 대체하는 인라인 스크립트 태그와 함께 쉘 이후 추가 컨텐츠를 스트리밍한다.
  - `onShellError`: 초기 쉘 렌더에 에러가 발생 시 실행되는 콜백. 에러를 인자로 받는다. 아직 스트림에서 바이트가 내보내지지 않았고, `onAllReady`도 `onShellReady`도 호출되지 않았다. 폴백 HTML 쉘을 반환할 수 있다.
  - `progressiveChunkSize`: 청크 바이트의 수

#### Returns

`renderToPipeableStream`는 두 메서드를 갖는 객체를 반환한다:

- `pipe`: 읽기 가능한 노드js 스트림으로 HTML을 반환한다. 스트리밍이 가능할 경우 `pipe`를 `onShellReady`에서 호출하고, `onAllReady`에서 크롤러와 정정 생산을 위해 호출해라.
- `abort`: 서버 렌더링을 중단하고 클라이언트 렌더링

## Usage

### Rendering a React tree as HTML to a Node.js Stream

`renderToPipeableStream`을 호출해 리액트 트리를 노드js 스트림으로 렌더링

```tsx
import { renderToPipeableStream } from "react-dom/server";

// The route handler syntax depends on your backend framework
app.use("/", (request, response) => {
  const { pipe } = renderToPipeableStream(<App />, {
    bootstrapScripts: ["/main.js"],
    onShellReady() {
      response.setHeader("content-type", "text/html");
      pipe(response);
    },
  });
});
```

루트 컴포넌트(App)와 함께 부트스트랩(main.js)리스트를 제공해야 한다.
루트 컴포넌트는 반드시 html 태그를 포함한 전체 도큐먼트를 반환해야한다.

예를 들면:

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

리액트는 독타입과 부트스랩 스크립 태그를 HTML 스트림에 주입한다:

```html
<!DOCTYPE html>
<html>
  <!-- ... HTML from your components ... -->
</html>
<script src="/main.js" async=""></script>
```

클라이언트에서, 부트스트랩 스크립트는 `hydrateRoot`로 하이드레이트되어야 한다.

```tsx
import { hydrateRoot } from "react-dom/client";
import App from "./App.js";

hydrateRoot(document, <App />);
```

이것은 서버에서 생성된 HTML에 이벤트 리스너를 부착해 상호작용이 가능하도록 한다.

#### Deep Dive

Reading CSS and JS asset paths from the build output

최종 에셋 URL은 빌드 후 종종 해시된다. 예를 들어, style.css 대신 style.1234.css가 된다. 정적 에셋 파일 이름을 해싱하면 동일한 에셋의 모든 빌드가 다른 파일 이름을 갖는다. 이것은 정정 에셋에 대한 장기 캐싱을 안전하게 활성화할 수 있기 때문에 유용하다. 특정 이름을 가진 파일은 절대 컨텐츠를 변경하지 않는다.

하지만, 빌드 종료 후 에셋의 URL을 알 수 있기에, 소스 코드에 주입할 방법이 없다. 예를 들어, 하드 코딩된 style.css 주입은 동작하지 않을 것이다. 소스 코드에서 이를 방지하기 위해, 루트 컴포넌트가 프랍으로 전달된 맵에서 실제 파일의 이름을 읽을 수 있다.

```tsx
export default function App({ assetMap }) {
  return (
    <html>
      <head>
        ...
        <link rel="stylesheet" href={assetMap["styles.css"]}></link>
        ...
      </head>
      ...
    </html>
  );
}
```

서버에서 `<App assetMap={assetMap} />`을 렌더하고, `assetMap`을 넘겨라:

```ts
// You'd need to get this JSON from your build tooling, e.g. read it from the build output.
const assetMap = {
  "styles.css": "/styles.123456.css",
  "main.js": "/main.123456.js",
};

app.use("/", (request, response) => {
  const { pipe } = renderToPipeableStream(<App assetMap={assetMap} />, {
    bootstrapScripts: [assetMap["main.js"]],
    onShellReady() {
      response.setHeader("content-type", "text/html");
      pipe(response);
    },
  });
});
```

서버가 `<App assetMap={assetMap} />`을 렌더링하므로, 하이드레이션 에러를 피하기 위해 `assetMap`을 같이 클라이언트에 렌더해야한다. `assetMap`을 다음과 같이 직렬화하여 넘길 수 있다.

```ts
// You'd need to get this JSON from your build tooling.
const assetMap = {
  "styles.css": "/styles.123456.css",
  "main.js": "/main.123456.js",
};

app.use("/", (request, response) => {
  const { pipe } = renderToPipeableStream(<App assetMap={assetMap} />, {
    // Careful: It's safe to stringify() this because this data isn't user-generated.
    bootstrapScriptContent: `window.assetMap = ${JSON.stringify(assetMap)};`,
    bootstrapScripts: [assetMap["main.js"]],
    onShellReady() {
      response.setHeader("content-type", "text/html");
      pipe(response);
    },
  });
});
```

`bootstrapScriptContent`는 추가적인 코드를 스크립트 태그에 추가한다. `window.assetMap` 전역 변수를 클라이언트에서 설정한다. 이것은 클라이언트 코드가 `assetMap`을 읽을 수 있도록 한다:

```tsx
import { hydrateRoot } from "react-dom/client";
import App from "./App.js";

hydrateRoot(document, <App assetMap={window.assetMap} />);
```

### Streaming more content as it loads

스트리밍은 모든 데이터가 완전히 로드 되기 전에 유저가 컨텐츠를 볼 수 있게 해준다.
예를 들어:

```tsx
function ProfilePage() {
  return (
    <ProfileLayout>
      <ProfileCover />
      <Sidebar>
        <Friends />
        <Photos />
      </Sidebar>
      <Posts />
    </ProfileLayout>
  );
}
```

`<Posts />`를 위한 데이터의 로딩에 시간이 걸릴 경우. 이상적으로, 포스트를 제외한 프로필 페이지의 나머지 컨텐츠는 보여줄 수 있다. 이것을 하기 위해, 포스트를 서스펜스 바운더리로 감싸면 된다.

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

이것은 리액트에게 포스트가 데이터를 로드하기 전에 HTML 스트리밍을 시작하도록 한다.
리액트는 HTML을 로딩 폴백을 우선적으로 보여준 후, 포스트 데이터의 로딩이 끝났을 경우, 리액트는 로딩 폴백을 대체하는 인라인 스크립트 태그와 나머지 HTML을 전송한다.
유저 관점에서 초기에 폴백을 보고 포스트로 대체될 것이다.

중첩 서스펜스를 통해 세분화된 로딩 시퀀스를 만들 수 있다:

```tsx
function ProfilePage() {
  return (
    <ProfileLayout>
      <ProfileCover />
      <Suspense fallback={<BigSpinner />}>
        <Sidebar>
          <Friends />
          <Photos />
        </Sidebar>
        <Suspense fallback={<PostsGlimmer />}>
          <Posts />
        </Suspense>
      </Suspense>
    </ProfileLayout>
  );
}
```

위 예에서 리액트는 페이지의 스트리밍을 더 일찍 시작한다. 레이아웃과 커버는 서스펜스로 감싸지지 않았으므로, 반드시 우선적으로 렌더링된다.
하지만, 사이드바, 프렌즈, 포토의 경우 데이터 로딩이 필요하므로, 리액트는 폴백을 대신 보낸다.
데이터가 사용 가능해질 경우 더 많은 컨텐츠가 보여진다.

스트리밍은 리액트 자체가 브라우저에 로드되거나 앱이 상호작용이 일어날 때까지 기다릴 필요없다.
스크립트 태그가 로드되기 전에 서버에서 제공되는 HTML 컨텐츠가 점진적으로 공개된다.

### Specifying what goes into the shell

앱에서 서스펜스 바운더리 외부의 모든 것을 쉘이라 부른다:

```tsx
function ProfilePage() {
  return (
    // 쉘
    <ProfileLayout>
      // 쉘
      <ProfileCover />
      <Suspense fallback={<BigSpinner />}>
        <Sidebar>
          <Friends />
          <Photos />
        </Sidebar>
        <Suspense fallback={<PostsGlimmer />}>
          <Posts />
        </Suspense>
      </Suspense>
    </ProfileLayout>
  );
}
```

이것은 유저에 보여지는 가장 빠른 로딩 상태를 결정한다:

```tsx
<ProfileLayout>
  <ProfileCover />
  <BigSpinner />
</ProfileLayout>
```

만약 모든 앱을 서스펜스로 감쌀 경우, 쉘은 오직 스피너만을 가질 것이다.
하지만, 이것은 유저에게 좋은 경험이 아니다. 하나의 큰 스피너를 보는 것은 느리게 느껴지며 조금의 레이아웃이 보이는 것보다 짜증나다. 이것이 일반적으로 서스펜스 바운더리를 배치해 쉘의 최소한이지만 완전한 느낌을 주도록하는 이유다.

`onShellReady`은 전체 쉘이 렌더된 후 실행된다. 보통 이후에 스트리밍을 시작할 수 있다:

```ts
const { pipe } = renderToPipeableStream(<App />, {
  bootstrapScripts: ['/main.js'],
  onShellReady() {
    response.setHeader('content-type', 'text/html');
    pipe(response);
  }
```

`onShellReady`가 호출된 시점에, 서스펜스로 감싸진 컴포넌트들은 여전히 데이터를 로딩하고 있을 것이다.

### Logging crashes on the server

onError에서 커스텀한 에러 로깅을 오버라이드할 수 있음

### Recovering from errors inside the shell

onError는 로깅을, onShellError는 폴백 HTML 전달을 위해 사용

### Recovering from errors outside the shell

1. 가장 가까운 서스펜스에서 로딩 폴백을 보여준다.
2. 서버에서 컨텐츠를 렌더하길 포기한다.
3. 자바스크립트 코드 로드가 완료되면, 클라이언트 렌더링을 시도한다.

만약 클라이언트에서도 실패한다면, 가까운 에러 바운더리에서 에러를 보여준다.
즉 유저는 에러가 회수되지 않을 때까지 로딩을 본다는 것.

만약 클라이언트에서 성공할 경우 서버에서의 로딩 폴백은 클라이언트 렌더링으로 대체된다.

### Setting the status code

스트리밍이 시작되면 상태 코드를 설정할 수 없다.
하지만, 각 onShellReady, onShellError 등 에서 상태코드를 설정할 수 있다.

### Handling different errors in different ways

커스텀한 에러 클래스를 만들어서 에러를 체크할 수 있다.

### Waiting for all content to load for crawlers and static generation

스트리밍은 컨텐츠가 나오는 대로 볼 수 있어 좋은 유저 경험을 제공한다.

하지만, 크롤러가 페이지에 방문했을 경우, 빌드 시간에 페이지를 생성하는 경우, 점진적으로 컨텐츠를 공개하는 대신, 모든 컨텐츠를 로드한 다음 최종 HTML을 생성하는 것이 좋다.

`onAllReady` 콜백을 사용해 모든 컨텐츠가 로드될때까지 기다릴 수 있다.

```tsx
let didError = false;
let isCrawler = // ... depends on your bot detection strategy ...

const { pipe } = renderToPipeableStream(<App />, {
  bootstrapScripts: ['/main.js'],
  onShellReady() {
    if (!isCrawler) {
      response.statusCode = didError ? 500 : 200;
      response.setHeader('content-type', 'text/html');
      pipe(response);
    }
  },
  onShellError(error) {
    response.statusCode = 500;
    response.setHeader('content-type', 'text/html');
    response.send('<h1>Something went wrong</h1>');
  },
  onAllReady() {
    if (isCrawler) {
      response.statusCode = didError ? 500 : 200;
      response.setHeader('content-type', 'text/html');
      pipe(response);
    }
  },
  onError(error) {
    didError = true;
    console.error(error);
    logServerCrashReport(error);
  }
});
```

일반 방문자의 경우 스트리밍 되는 컨텐츠를 볼 것이다. 크롤러는 최종 HTML 결과를 받게된다.
하지만, 이것은 크롤러가 모든 데이터가 로드 되기를 기다려야한다는 것이다. 어떤 것은 느리거나 에러를 발생할 수 있다. 앱에 따라 쉘이 준비 되었을 경우 크롤러에게 보낼 수 있다.

### Aborting server rendering

타임아웃 이후 서버 렌더링을 포기할 수 있다:

```tsx
const { pipe, abort } = renderToPipeableStream(<App />, {
  // ...
});

setTimeout(() => {
  abort();
}, 10000);
```

리액트는 나머지 HTML을 로딩 폴백으로 플러쉬하고, 나머지를 클라이언트 렌더를 시도한다.

---

컨텐츠를 통으로 주는 것이 아닌 조각조각 주는 방식.
서스펜스를 사용하면 좀 더 유려하게 처리할 수 있다.
에러가 발생할 경우 클라이언트 까지 가서 에러 해결 하는 것이 신기하다.
한 번더 읽어봐야겠다.
