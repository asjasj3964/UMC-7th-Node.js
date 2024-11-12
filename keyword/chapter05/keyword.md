### **📦 실습**
---

### 🎯 핵심 키워드
---
- 환경 변수 (Environment Variables)
    - 운영체제나 실행 환경에서 설정된 변수
    - 소프트퉤어 애플리케이션 동작에 영향을 미치는 중요한 정보를 저장하는 데 사용된다. 즉, 중요한 정보를 소스코드에 직접 포함하지 않아 보안을 강화한다.
    - 민감하거나 자주 변경될 수 있는 설정 값을 코드에 하드코딩하지 않고 외부에서 관리할 수 있도록 한다.
        - ex. 데이터베이스 연결 정보, API 키, 서버 포트 번호 등
    - ex. Node.js 프로젝트에서의 .env 파일
        - dotenv 라이브러리를 사용해 .env 파일에 환경변수를 정의하고 로드할 수 있다.
        ```javascript
        DB_HOST=localhost
        DB_POST=3306
        DB_USER=root
        DB_PASSWORD=mypassword
        PORT=3000
        ```
        - Node.js에서 process.env를 통해 환경변수에 접근할 수 있다.
        ```javascript
        const dbHost = process.env.DB_HOST
        const dbPort = process.env.DB_PORT
        ```     
- CORS (Cross-Origin Resource Sharing)
    - 웹 브라우저에서 실행되는 웹 애플리케이션이 다른 도메인, 프로토콜, 포트에서 실행되는 리소스에 접근할 수 있도록 허용하는 보안 기능
    - 기본적으로 브라우저는 보안상의 이유로 동일한 출처 정책(Same-Origin Policy)을 적용해 한 출처에서 로드된 웹 페이지가 다른 출처의 리소스에 접근하는 것을 제한한다.
        - ex. http://umc.com에서 로드된 웹 애플리케이션은 기본적으로 http://api.umc.com과 같은 다른 도메인에 있는 API에 접근할 수 없다. → Cors는 이러한 제한을 완화한다.
        - 웹 애플리케이션이 외부 API를 통해 데이터를 가져오거나 다른 도메인에 있는 서버와 상호작용해야 할 때 CORS 설정으로 이러한 요청을 허용할 수 있다.
    1. 간단한 요청 (Simple Request)
        - 브라우저는 GET, POST, HEAD 메서드를 사용하는 단순한 HTTP 요청에 대해 origin 헤더를 추가한다. 서버는 이 요청을 받고 응답 헤더에 Access-Control-Allow-Origin을 포함시켜 요청을 허용할 지를 결정한다.
        - ex.<br/>
            요청 헤더: Origin: http://umc.com <br/>
            응답 헤더: Access-Control-Allow-Origin: http://umc.com 
    2. 예비 요청 (Preflight Request)
        - 보안상의 이유로 브라우저는 단순하지 않은 요청(ex. PUT, DELETE 메서드, 사용자 지정 헤더가 포함된 요청)을 실행하기 전에 서버에 예비 요청(OPTIONS 메서드)을 보낸다. 이는 실제 요청을 보내기 전에 서버가 해당 요청을 허용할지를 확인하는 과정이다.
    - CORS 관련 헤더
        1. Access-Control-Allow-Origin
            - 클라이언트가 리소스에 접근할 수 있도록 허용된 출처(Origin)를 정의한다.
            - ex. Access-Control-Allow-Origin: * →  모든 출처를 허용한다.
        2. Access-Control-Allow-Methods
            - 허용된 HTTP 메서드를 지정한다.
            - ex. Access-Control-Allow-Methods: GET, POST, PUT, DELETE
        3. Access-Control-Allow-Headers
            - 클라이언트가 요청할 때 사용할 수 있는 헤더를 명시한다.
            - ex. Access-Control-Allow-Headers: Content-Type, Authorization
        4. Access-Control-Allow-Credentials
            - 클라이언트에서 쿠키와 같은 자격 증명을 포함한 요청을 할 수 있도록 허용할 지에 대한 여부를 설정한다.
            - ex. Access-Control-Allow-Credentials: true
    - ex. CORS 설정 (Node.js/Express)
        ```javascript
        import express from 'express'; 
        import cors from 'cors';;
        const app = express();
        
        app.use(cors()); // 모든 출처에 대해 cors 허용
        
        // 특정 출처만 허용하는 경우
        app.use(cors({ // cors 미들웨어 추가
          origin: 'http://umc.com', // 허용된 출처(Cross-Origin) 지정
          methods: 'GET, POST', // 허용된 HTTP 지정
          credentials: true // 클라이언트가 서버에 쿠키, 인증 헤더 등 자격 증명(credentials)을 포함할 수 있다.
        }));
        
        app.listen(3000, () => {
          console.log('Server is running on port 3000');
        });
        ```
        - http://umc.com에서 오는 요청만 CORS 규칙에 의해 허용된다.
        - Cross-Origin: 동일 출처 정책을 우회할 출처를 지정한다.
            - 위의 예시에선 웹 브라우저가 http://umc.com에서 실행되는 애플리케이션이 서버에 요청을 보내는 것을 허용한다.
        - 서버는 GET과 POST 메서드로만 요청을 받을 수 있다.
        - Credential 옵션이 true로 설정될 경우 Access-Control-Allow-Origin을 *로 설정할 수 없고 특정 출처를 지정해야 한다.
- DB Connection, DB Connection Pool
    1. DB Connection (데이터베이스 연결)
        - 애플리케이션이 데이터를 읽거나 쓰기 위해 데이터베이스에 접근할 때 필요한 데이터베이스 연결을 해준다.
        - 각 연결은 네트워크를 통해 데이터베이스와 상호작용하고 쿼리를 실행하며 결과를 가져오는 데 사용된다.
        - 기본 흐름
            1. 애플리케이션이 데이터베이스 연결을 설정한다.
            2. 쿼리를 보내고 응답을 받는다. 
            3. 작업이 끝나면 연결을 종료한다.
    2. DB Connection Pool (데이터베이스 연결 풀)
        - 여러 데이터베이스 연결을 미리 생성해두고 이를 애플리케이션에서 재사용(성능, 효율 ↑)할 수 있도록 관리하는 메키니즘
        - 작동 원리
            1. 초기화: 애플리케이션이 시작될 때 일정한 수(과부하 방지)의 데이터베이스 연결을 미리 생성해놓는다. 
            2. 재사용: 애플리케이션이 데이터베이스 작업을 요청할 때 연결 풀에서 사용 가능한 연결을 가져다 사용한다. (시간 및 리소스 절약)
            3. 반환: 작업이 끝나면 연결이 닫히는 것이 아니라 풀로 반환되어 다른 요청이 재사용할 수 있다.
            4. 확장/축소: 연결 풀이 필요한 경우 더 많은 연결을 생성하거나 사용되지 않는 연결을 종료하여 리소스를 관리할 수 있다. 
    - Node.js에선 mysql2 라이브러리를 사용해 데이터베이스 연결 풀을 생성할 수 있다.
        ```javascript
        import mysql from 'mysql2/promise'; // MySQL 데이터베이스와 연결
        
        export const pool = mysql.createPool({ // 데이터베이스 연결 풀(connection pool) 생성
            host: process.env.DB_HOST || 'localhost', // mysql의 hostname
            user: process.env.DB_USER || 'root', // 사용자 이름
            port: process.env.DB_PORT || 3306, // 포트 번호
            database: process.env.DB_TABLE || 'restaurant_service', // 데이터베이스 이름
            password: process.env.DB_PASSWORD || 'mypassword', // 비밀번호
            waitForConnections: true, 
            // Pool에 획득할 수 있는 connection이 없을 때 
            // true면 요청을 queue에 넣고 connection을 사용할 수 있게 되면 요청을 실행한다.
            // false면 즉시 오류를 내보내고 다시 요청한다. 
            connectionLimit: 10, // 몇 개의 커넥션을 가지게끔 할 것인지 
            queueLimit: 0, // getConnection에서 오류가 발생하기 전에 Pool에 대기할 요청의 개수 한도
        });
        
        // 연결 사용
        pool.query('SELECT * FROM users', (error, results) => { // sql 쿼리 실행
          if (error) throw error;
          console.log(results);
        });
        ```
        
- 비동기 (async, await)
    - **동기 프로그래밍 (Synchronous Programming)**
        - 작업들이 순서대로 실행되어 이전 작업이 끝나야만 다음 작업을 시작한다.
        - 시간이 오래 걸리는 작업이 있으면 그동안 프로그램이 멈춘 상태로 대기한다.
            ```javascript
            console.log("Start");
            const result = fetchData();  // 오래 걸리는 작업
            console.log(result);
            console.log("End");
            ```
            1. fetchData()가 완료될 때까지 기다린 후
            2. ”End”가 출력된다. 
            - “Start” → fetchData() → “End”
    - **비동기 프로그래밍 (Asynchronous Programming)**
        - 작업을 동시에 실행되도록 하여 하나의 작업이 끝날 때까지 기다리지 않고 다음 작업을 수행하는 방식
        - 오래 걸리는 작업을 처리하는 동안 애플리케이션이 다른 작업을 계속 수행할 수 있도록 도와준다. (각 작업이 병렬적으로 진행된다)
        - 완료된 작업은 나중에 콜백, Promise 또는 async/await을 통해 처리된다. 
            ```javascript
            console.log("Start");
            setTimeout(() => {
            	console.log("Fetching data... (completed)");
            }, 2000); // 2초 후 실행
            console.log("End");	
            ```
            - setTimeout 함수에서 2초 기다리는 동안 다음 코드(”End” 출력)를 먼저 실행한다.
            - “Start” → setTimeout() 실행 → “End” → (2초 후) ”Fetching data... (completed)"
    - **비동기 프로그래밍의 주요 패턴**
        1. **콜백 함수 (Callback Function)**
            - 작업이 완료된 후 특정 함수를 호출한다.
            ```javascript
            function fetchData(callback){
            	setTimeout(() => {
            		**callback("Data loaded")**;
            	}, 1000);
            }
            console.log("Start");
            fetchData(**(data) => console.log(data)**);
            console.log("End");
            ```
            - “Start” → setTimeout() 실행 → “End” → (1초 후) “Data loaded”
            - 문제점: 콜백 함수(Callback Hell)가 중첩되면 콜백 지옥이라는 복잡한 코드가 발생할 수 있다.
        2. **Promise**
            - 작업의 성공/실패를 표현하며 then과 catch로 결과를 처리한다.
            ```javascript
            const fetchData = new **Promise**((resolve, reject) => {
            	setTimeout(() => **resolve**("Data loaded"), 1000);
            });
            console.log("Start");
            fetchData.**then**((data) => console.log(data));
            console.log("End");
            ```
            > resolve: 작업이 성공적으로 완료되었을 때 호출된다.
            > reject: 작업이 실패했을 때 호출된다. 
            - setTimeout으로 1초 후에 resolve 함수를 호출한다. 이 작업 성공 시 Promise가 해결(resolved)된다.
            - fetchData는 Promise이므로 then 메서드를 사용해 비동기 작업이 완료되었을 때 결과(data)를 처리한다.
                - 1초 후 resolve 함수가 호출되면 값("Data loaded")이 then에 전달되어 data가 출력된다.
            - “Start” → setTimeout() 실행 → “End” → (1초 후) “Data loaded”
        3. **async/await**
            - 자바스크립트의 비동기 프로그래밍을 직관적이고 간결하게 처리할 수 있도록 도와주는 문법
            - 비동기 작업 순서와 흐름을 동기적 코드처럼 작성할 수 있다.
            - async 키워드를 함수 앞에 붙여 해당 함수를 비동기 함수로 만든다.
                ```javascript
                async function example() {
                	return "Hello, Wenty!";
                }
                example().then((result) => console.log(result)); // 호출 시 Promise로 반환된다. 
                ```
                - 비동기 함수는 항상 Promise 객체를 반환한다.
                - 반환 값이 일반 값이면 자동으로 Promise.resolve(…)으로 감싸서 반환된다.
            - await는 비동기 함수 내에서만 사용할 수 있다.
                ```javascript
                async function fetchData() {
                	let data = await **new Promise**((resolve) => 
                		setTimeout(() => resolve("Data loaded"), 1000)
                	);
                	console.log(data); // 1초 후 출력
                }
                fetchData();
                ```
                - Promise가 해결될 때까지 기다렸다가 해당 결과를 반환 받아 다음 코드를 실행한다.
                - await 뒤에는 Promise가 반환되는 비동기 작업이 와야 한다.
            - ex.
                ```javascript
                **async** function loadData(){
                	console.log("Start");
                	let data = **await** new **Promise**(
                		(resolve) => setTimeout(() => resolve("Data loaded"), 1000));
                	);
                	console.log(data);
                	condole.log("End");
                }
                loadData();
                ```
                - await는 Promise가 해결될 때까지 대기하므로 이후의 코드는 실행되지 않는다.
                - 1초 후 Promise가 해결된 후에 resolve()가 호출되면서 await가 해제되고 “Data loaded”가 data 변수에 저장된다.
                - “Start” → (1초 후) “Data loaded” → “End”
        4. async/await + Promise.all
            - 여러 개의 비동기 작업을 병렬로 실행하려면 Promise.all과 함께 사용한다.
            - async/await만 사용하면 비동기 작업들이 순차적으로 실행되어 시간이 더 오래 걸린다. 그러나 Promise.all과 async/await을 함께 사용하면 작업들을 동시에 실행할 수 있어 더 효율적이다.
                - 이 경우 전체 실행 시간은 가장 오래 걸리는 작업의 시간만큼만 필요합니다.
            - async/await만 사용할 경우
                ```javascript
                async function getStudent1() {
                	return new Promise((resolve) => setTimeout(() => resolve("Wenty", 1000));
                }
                async function getStudent2() {
                	return new Promise((resolve) => setTimeout(() => resolve("Bongoo"), 2000));
                }
                async function loadData() {
                	console.log("Start"); // 1
                	let student1 = await getStudent1();
                	console.log(`Student 1: ${student1}`); // 2
                	let student2 = await getStudent2();
                	console.log(`Student 2: ${student2}`); // 3
                	console.log("End"); // 4
                }
                loadData();
                ```
                - getStudent1()를 기다렸다가 getStudent2()를 기다린다.
                    - 총 1 + 2 = 3초 소요된다.
            - async/await + Promise.all을 사용할 경우
                ```javascript
                async function getStudent1() {
                	return new Promise((resolve) => setTimeout(() => resolve("Wenty", 1000));
                }
                async function getStudent2() {
                	return new Promise((resolve) => setTimeout(() => resolve("Bongoo"), 2000));
                }
                async function loadData() {
                	console.log("Start"); // 1
                	let [student1, student2] = await Promise.all([getStudent1(), getStuent2()]); // 병렬로 실행되며 가장 오래 걸리는 작업 시간(2초)만 기다린다. 
                	console.log(`Student 1: ${student1}`); // 2
                	console.log(`Student 2: ${student2}`); // 3
                	console.log("End"); // 4
                }
                loadData();
                ```
                - Promise.all을 사용해 student1과 student2 정보를 동시에 가져온다.
                - 두 작업이 병렬로 진행되어 가장 오래 걸리는 작업 시간만 소요된다.
                    - getStudent1()과 getStudent2() 중 작업 시간이 더 긴 시간 즉, 2초 소요된다.
- try/catch/finally
    - 코드에서 발생할 수 있는 오류를 처리하고 오류가 발생했을 때 어떻게 동작할 지를 제어하는 데 사용된다.
    - 예외 처리(Exception Handling)을 통해 코드 실행 중 오류를 탐지하고 이를 적절히 처리할 수 있도록 한다.
    ```javascript
    function riskyFunction() { // 오류가 날 수도 있는 코드
      throw new Error("Error"); 
    }
    
    try { // try 블록 - 실행할 코드
      let result = riskyFunction(); // 오류 발생 시
      console.log("Result:", result); // 이 코드는 실행되지 않는다.
    } catch (error) { // 오류 발생 시 이 코드로 이동
      console.error("Error:", error.message); // 오류 처리
    } finally {
      console.log("End"); // 항상 실행
    }
    ```
    - 오류가 발생할 가능성이 있는 코드를 try에 넣어 예외 처리에 대비한다.
    - catch의 매개변수: 일반적으로 error로 사용되며 발생한 오류에 대한 정보가 포함된다.
    - finally: 오류 발생 여부와 상관없이 항상 실행된다. try 블록과 catch 블록이 끝난 후에 실행된다.