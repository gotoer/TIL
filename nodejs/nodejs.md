# Node.js

Node.js는 Chrome V8 JavaScript 엔진으로 구동되는 비동기 이벤트 기반 서버 사이드 런타임 환경입니다.  
JavaScript를 브라우저가 아닌 서버에서 실행할 수 있도록 만든 플랫폼입니다.  

## Node.js 특징

- 비동기 & 이벤트 기반 (Non-blocking I/O)
  - 기본적으로 비동기(Asynchronous) 방식으로 동작합니다.
  - 이벤트 루프(Event Loop)를 사용해 여러 작업을 동시에 처리합니다.
    - 이벤트 루프 덕분에 node.js는 싱글 스레드이면서도 논블로킹 방식으로 여러 작업을 동시에 실행할 수 있습니다.
  - 파일 읽기, DB 쿼리 등 시간이 걸리는 작업을 실행하는 동안 다른 요청을 처리할 수 있습니다.

    [사용 예시]

    ``` javascript
    const fs = require('fs');

    fs.readFile('file.txt', 'utf8', (err, data) => {
    if (err) throw err;
    console.log(data);
    });

    console.log('파일 읽기 요청 완료'); // 먼저 실행됨
    ```

<br>
<br>

- 싱글 스레드 기반 (Single Threaded)
  - 기본적으로 하나의 메인 스레드만 사용합니다.
    - 하지만 내부적으로 멀티스레드 작업(ex: 파일 읽기, 네트워크 요청)은 백그라운드에서 실행합니다.
  - 클러스터 모드(PM2 사용 가능)를 활용하면 멀티코어 CPU도 활용 가능합니다.

<br>
<br>

- 빠른 성능 (Chrome V8 엔진)
  - Google Chrome의 V8 엔진을 사용하여 매우 빠른 실행 속도를 제공합니다.
  - C++ 기반으로 최적화되어 있어 성능이 우수합니다.

<br>
<br>

- NPM (Node Package Manager) 지원
  - NPM(Node Package Manager)을 통해 다양한 라이브러리를 쉽게 설치 가능합니다.
    - 200만 개 이상의 패키지가 제공됩니다.

<br>
<br>
<br>

## Node.js 주요 활용 분야

Node.js는 주로 서버 개발에 많이 사용되지만, 다양한 분야에서 활용 가능합니다.  

- 웹 서버
  - Express, NestJS 같은 웹 프레임워크를 이용해 API 서버 개발
  - (예시) REST API, GraphQL 서버  

- 실시간 애플리케이션
  - WebSocket을 활용한 실시간 서비스
  - (예시) 채팅, 실시간 알림, 게임 서버  

- 마이크로서비스
  - 독립적인 서비스 구조를 구현
  - (예시) AWS Lambda, Serverless  

- CLI 도구
  - 터미널에서 실행하는 커맨드라인 프로그램 개발
  - (예시) npm, eslint  

- 스크래핑 & 자동화
  - 크롤링, 데이터 수집 및 자동화
  - (예시) Puppeteer, Cheerio  

- IoT & 임베디드
  - 하드웨어 연동 가능
  - (예시) Raspberry Pi, Arduino

## Node.js 장단점

[장점]

- 비동기 & V8 엔진 덕분에 높은 처리량이 가능해 성능이 빠릅니다.
- 확장성이 높아 마이크로서비스 및 서버리스 아키텍처에 적합합니다.
- 수많은 오픈소스 라이브러리 활용 가능합니다.
- 풀스택 개발이 가능합니다.

<br>

[단점]

- 싱글 스레드라서 대규모 연산 작업에는 불리하기 때문에 CPU 집약적인 작업에 부적합합니다.
- 콜백 지옥 때문에 Promise나 async/await을 사용해야 가독성이 좋습니다.
- 오픈소스 라이브러리 사용이 많아 보안 이슈 관리가 필요합니다.
