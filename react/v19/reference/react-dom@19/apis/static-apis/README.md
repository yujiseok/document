# Static React DOM APIs

`react-dom/static`는 리액트 컴포넌트에 대한 정정 HTML을 생성하게 해준다.
스트리밍 API들에 비해 제약이 존재한다. 프레임워크들이 호출해줄 것.
대부분의 컴포넌트들에서 사용할 필요 없다.

## Static APIs for Web Streams

Web Streams 환경에서 동작하는 API. (디노, 엣지 런타임)

- `prerender` 리더블 웹 스트림을 이용해 리액트 트리를 정적 HTML로 렌더

## Static APIs for Node.js Streams

Node.js Streams 환경에서 동작하는 API.

- `prerenderToNodeStream` 노드의 스트림을 사용해 리액트 트리를 정적 HTML로 렌더
