---
layout: post
title: "React를 사용한 Universal rendering web app의  local dev server 구축하기"
author: "Tylor Shin"
categories: frontend
tags: [React, Redux, universal-rendering, isomorphic-rendering, universal rendering, isomorphic rendering, ReactJS, redux]
image: localServer.png
---

## 도입
지난 6개월간 Universal rendering Web app을 다루면서 제일 힘들었던 것 중 하나는 서버사이드 렌더링 디버깅이었다.  
특히 scinapse는 별도로 FrontEnd용 서버를 운영하지 않고, AWS Lambda로 서버를 대신 사용하고 있다.  
이 경우 코드를 스테이지 서버에 올려보기 전까지 어떤 잠재적 문제가 있는지 확인하기 어려웠는데, 특히 코드 변화가 커서 여러 군데가 잘못된 경우에는 버그 해결이 꽤 큰 짐으로 다가왔다.

```
스테이지 서버에 코드 반영(1~3분 소요) -> 서버사이드 렌더링 버그 -> 고쳐서 다시 스테이지 서버에 반영(1~3분 소요) -> 또 다른 버그 -> 고쳐서 ... (전부 안정화 될 때까지 무한 반복)
```

또한 에러가 아닌 버그들도 있었다.  
Server side에서 렌더링 된 결과물이 client side에서 다시 로드될 때와 다른 모습을 보이거나,  
Server side에서 넘겨준 Redux State(보통 window.__INITIAL_STATE__에 담아 사용)가 제대로 파싱이 안된다거나 하는 문제가 간혹 발생했다.  

애초에 개발 환경에서 실시간으로 이러한 문제점들을 진단할 수 있으면 삽질을 안 해도 될 것 같아서 개발 환경을 다시 세팅해보기로 했다.  

## 기술적 배경
[원래 세팅]
  - [Webpack-dev-server](https://github.com/webpack/webpack-dev-server)를 통한 Client-Side bundle 및 Hot Reload 사용.
  - TypeScript 사용
  - Browser용 bundle.js와 NodeJS용 bundle.js를 따로 빌드 중(Webpack에서 target option을 다르게 설정해주면 된다.)

원래 세팅에서도 Node에서 사용할 수 없는 변수를 사용하거나, Browser에서 사용할 수 없는 리소스를 사용하면 Build 단계에서 에러가 발생하긴 했었다.  
하지만 에러가 발생하지 않는 문제들과 서버/클라이언트 단의 렌더링 결과물 차이 문제들을 전부 해결할 수는 없었다.  

## 문제 정의
우선 AWS Lambda가 하는 일을 생각해보자.  
람다가 하는 일은 유저가 요청한 path에 따라서 serverSideRender 함수를 실행하는 게 전부다.  
현재 함수는 다음과 같이 선언되어 있다. (언제라도 https://github.com/pluto-net/web-client 에서 코드를 확인할 수 있다.)

```ts
export async function serverSideRender({ requestUrl, scriptPath, queryParamsObject }: ServerSideRenderParams) {
  // DO SERVER SIDE RENDER
})
```
각 파라메터는 다음 역할을 한다.  
  - requestUrl: user가 요청한 pathname을 얻음.
  - queryParamsObject: Lambda에서는 queryParam을 파싱해서 string이 아닌 object로 전달하므로(혹은 undefined) 별도로 파라메터로 처리했다.
  - scriptPath: client side에서 번들링 된 자바스크립트를 로딩하기 위한 주소이다. ```<script src="${scriptPath}"></script>```의 형태로 삽입된다.


결국, 변화를 감지할 수 있는 로컬 서버를 돌려서 람다가 하는 것처럼 serverSideRender 함수를 실행시키고 결과 string을 html로 돌려주면 문제가 해결될 것 같았다.  

순서대로 정의해보면 아래 문제들만 해결하면 된다.

1. server용 bundle.js 변화를 감지해서 Restart 시켜줄 수 있는 로컬 서버를 무엇으로 돌릴 것인가?
2. scriptPath를 어디로 지정할 것인가? scriptPath의 target이 되는 browser용 bundle.js에 대한 변화 감지도 되어야 한다.
3. 이 둘을 어떻게 동시에 실행시키고 연결할 것인가?

1은 깊게 고민하지 않고 [Express](https://expressjs.com/) + [Nodemon](https://github.com/remy/nodemon)으로 해결하기로 했다.  

물론 굳이 Express를 쓸 필요도 없이 NodeJS의 기본 HTTP로도 처리할 수 있지만, 앞으로 기능 추가나 userAgent handling 같은 것들을 편하게 하기 위해서 Express를 사용했다.   
(어차피 개발 환경에서만 사용하기 때문에 용량이나 최적화 문제도 발생하지 않는다.)

2는 Webpack-dev-server를 사용하기로 했다. 변화를 감지해서 스스로 빌드하고, https://localhost:8080 으로 결과물도 실시간으로 serving 해주기 때문이다.  

3은 아직 뾰족한 해결법을 찾지 못하고 있다.

## 해결

```
  // package.json
  "start": "npm-run-all -p start:server start:build dev",
  "start:server": "nodemon ./dist/bundle.js --watch \"./dist\"",
  "start:build": "webpack --config webpack.local.dev.config --watch",
  "dev": "webpack-dev-server"
```
위와 같이 npm script를 구성했다.  

[npm run all](https://github.com/mysticatea/npm-run-all)은 편하게 병렬로 npm script들을 실행시키기 위해 사용했다.  3가지 스크립트를 동시에 돌리는데, 작동 시나리오는 다음과 같다.  

- express는 `localhost:3000` 으로 서버사이드 렌더링 결과물을 렌더링하게 설정.
- nodemon을 통해서 dist폴더에 bundle.js 변화에 따라 express server를 restart 하도록 설정.
- webpack을 --watch 모드를 걸어 실시간으로 반응하도록 만듦. 
  - 이 output이 dist/bundle.js 이므로, 소스코드가 변화할 때마다 webpack --watch가 trigger되고 webpack이 bundling을 완료하면, dist/bundle.js 를 바라보는 nodemon이 trigger 하고 express server를 restart.
- webpack-dev-server는 소스 코드의 변화에 따라 browser용 bundle.js를 실시간으로 만들고 `localhost:8080/bundle.js`으로 서빙함.

```
소스코드 변화 -> webpack(서버용) && webpack-dev-server(브라우저용)이 작동 -> nodemon restart -> express restart
```

이후 express가 제공하는 localhost:3000 으로 접속해보면 세팅해놓은 게 정상적으로 작동하는 걸 볼 수 있다.  
더 깊은 이해를 위해 `localServer.ts` 코드를 첨부한다.  

```ts
import * as express from "express";
import { serverSideRender, renderJavaScriptOnly } from "../app/server";

const server = express();

server.get("/*", async (req: express.Request, res: express.Response) => {

  const normalRender = async () => {
    const resultHTML = await serverSideRender({
      requestUrl: req.url,
      scriptPath: "http://localhost:8080/bundle.js",
    });

    return resultHTML;
  };

  const safeTimeout = new Promise((resolve, _reject) => {
    const jsOnlyHTML = renderJavaScriptOnly("http://localhost:8080/bundle.js");

    setTimeout(
      () => {
        resolve();
      },
      7000,
      jsOnlyHTML,
    );
  });

  const html = await Promise.race([normalRender(), safeTimeout]);
  res.send(html);
});

const port: number = Number(process.env.PORT) || 3000;

server
  .listen(port, () => console.log(`Express server listening at ${port}! Visit https://localhost:${port}`))
  .on("error", err => console.error("LOCAL_SERVER_ERROR =======================", err));
```

## 논의 및 한계점
- webpack이 2개나 돌아가므로(병렬로 돌아가긴 하지만) 시스템 자원을 많이 차지한다.
  - Browser build와 server build를 따로 하므로 현재로서는 어쩔 수 없는 선택인 듯하다.
- HMR이나 하다못해 Auto refresh도 지원하지 않는다.
  - 소스 코드 변화를 확인할 때마다 refresh를 눌러야 해서 고통스럽긴 하지만, 현재로서는 방법을 잘 모르겠다. Bash script로 강제로 refresh 할 수 있게 할 수 있을 거 같긴 한데, 그렇게 깊게 파고들지는 않았다. 

## 결론
이 세팅을 해놓은 덕분에 조금 불편해지기는 했지만 universal rendering을 하는데 있어서 안정성이 굉장히 높아졌다.  
또한, 현재 Lambda로 serving하는 Fronend 서버를 추후에 NodeJS 기반 웹 프레임워크로 돌리기도 쉬워져서 마음이 한결 편해졌다.  

이러한 로직이 적용된 프로젝트는 Pluto의 [scinapse](https://scinapse.io)이고, 소스 코드 확인 및 contribute는 누구나, 언제든지 https://github.com/pluto-net/web-client 저장소에서 할 수 있다.  
