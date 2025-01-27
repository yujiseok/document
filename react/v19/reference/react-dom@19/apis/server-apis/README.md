# Server React DOM APIs

`react-dom/server` API들은 리액트 컴포넌트들이 HTML에 서버 사이드 렌더가 가능하게 해준다. 이 API들은 앱의 최상단에서 오직 서버에서 초기 HTML을 생성하기 위해 사용된다.
프레임워크 단에서 실행해주므로, 컴포넌트에서 호출할 필요없다.

## Server APIs for Node.js Streams

이 메서드는 노드js의 스트림이 존재하는 환경에서만 사용할 수 있다.

- `renderToPipeableStream`은 리액트 트리를 pipeable한 노드js 스트림으로 렌더한다.

## Server APIs for Web Streams

웹 스트림 환경의 브라우저, 디노 또는 엣지 런타임에서 가능한 메서드.

- `renderToReadableStream`은 리액트 트리를 `Readable Web Stream`으로 렌더한다.

## Legacy Server APIs for non-streaming environments

스트림을 제공하지 않는 환경에서 사용 가능한 메서드.

- `renderToString` 리액트 트리를 문자열로 렌더링
- `renderToStaticMarkup` 상호작용 없는 리액트 트리를 문자열로 렌더링

스트리밍 API들에 비하면, 제한적인 기능을 갖는다.
