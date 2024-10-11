### 🔥 미션
---
**홈 화면**
1. 사용자의 미션 진행 현황 및 도전 가능한 미션 목록을 조회한다. 
    1. **API Endpoint**
        - GET /member/{member-id}/missions
    2. **Request Body**
        - 필요 X
            - GET 요청이 리소스 조회를 위한 HTTP 메소드이기 때문
    3. **Request Header**
        - Authorization: Bearer Token (사용자가 인증된 상태임을 나타냄)
    4. **Query String**
        - GET /member/{member-id}/missions?sort=created_at
    5. **Path variable**
        - member-id (사용자 ID)
    6. **Response Body**
        ```json
        {
          "isSuccess": true,
          "code": "string",
          "message": "string",
          "result": {
            "memberLocation": "string",
            "completedMissionTotal": 0,
            "missions": [
        	    {
        	      "missionId": 0,
        	      "missionName": "string",
        	      "restaurantName": "string",
        	      "description": "string",
        	      "points": 500,
        	      "remainingDays": 30,
        	      "status": 1
        	    }
        	  ]
          }
        }
        ```

**마이 페이지 리뷰 작성**
1. 마이 페이지에서 “작성한 리뷰”를 클릭해 내가 작성한 리뷰를 조회한다. 
    1. **API Endpoint**
        - GET /members/{member-id}/my-page/reviews
    2. **Request Body**
        - 필요 X
    3. **Request Header**
        - Authorization: Bearer Token
    4. **Query String**
        - GET /members/{member-id}/my-page/reviews?page=1&limit=10&sort=created_at
            - page=1: 첫 번째 페이지의 리뷰 조회
            - limit=10: 한 번에 10개의 리뷰 조회
            - sort=created_at: 리뷰 작성 날짜 순으로 정렬
    5. **Path variable**
        - member-id (사용자 ID)
    6. **Response Body**
        ```json
        {
          "isSuccess": true,
          "code": "string",
          "message": "string",
          "result": {
            "reataurantName": "string",
            "reviewTotal": 0,
            "reviews": [
        	    {
        	      "reviewId": 1,
        	      "rating": 0.0,
        	      "memberName": "string",
        	      "content": "string",
        	      "createdAt": "2024-10-09"
        	      "reviewImages": [
        		      {
        			      "reviewImageId": 1,
        			      "reviewImageUrl": "string"
        		      }
        	      ],
        	      "reviewReplys": [
        		      {
        			      "reviewReplyId": 1, 
        			      "reviewReplyContent": "string",
        			      "reviewReplyCreatedAt": "2024-10-09T00:00:00"
        		      }
        	      ]
        	    }
        	  ]
          }
        }
        ```
        
2. 특정 식당에 리뷰를 남긴다. 
    1. **API Endpoint**
        - POST /members/{member-id}/restaurants/{restaurant-id}/reviews
    2. **Request Body**
        ```json
        {
        	"rating": 0.0,
        	"content": "string",
        	"status": 1
        }
        ```
    3. **Request Header**
        - Authorization: Bearer Token
    4. **Query String**
        - 필요 X
    5. **Path variable**
        - member-id(사용자 ID), restaurant-id(식당 ID)
    6. **Response Body**
        ```json
        {
          "isSuccess": true,
          "code": "string",
          "message": "string",
          "result": {
        	  "reviewId": 1
          }
        }
        ```

**미션 목록 조회 (진행 중, 진행 완료)**
1. 진행 완료한 미션 목록을 조회한다. 
    1. **API Endpoint**
        - GET /members/{member-id}/completed-missions
    2. **Request Body**
        - 필요 X
    3. **Request Header**
        - Authorization: Bearer Token
    4. **Query String**
        - GET /members/{member-id}/completed-missions?sort=created_at
    5. **Path variable**
        - member-id(사용자 ID)
    6. **Response Body**
        ```json
        {
          "isSuccess": true,
          "code": "string",
          "message": "string",
          "result": [
        	  "restaurantName": "string",
        	  "completedMission": {
        		  "completedMissionId": 1,
        		  "introduction": "string",
        		  "points": 500
        	  }
          ]
        }
        ```
        
2. 진행 중인 미션 목록을 조회한다. 
    1. **API Endpoint**
        - GET /members/{member-id}/missions-in-progress
    2. **Request Body**
        - 필요 X
    3. **Request Header**
        - Authorization: Bearer Token
    4. **Query String**
        - GET /members/{member-id}/missions-in-progress?sort=created_at
    5. **Path variable**
        - member-id(사용자 ID)
    6. **Response Body**
        ```json
        {
          "isSuccess": true,
          "code": "string",
          "message": "string",
          "result": [
        	  "restaurantName": "string",
        	  "ongoingMission": {
        		  "ongoingMissionId": 1,
        		  "introduction": "string",
        		  "points": 500
        	  }
          ]
        }
        ```

**미션 성공 누르기** 
1. 미션 수행 후 성공 요청 버튼 눌러 해당 식당의 사장님의 구분 번호를 조회한다. 
    1. **API Endpoint**
        - GET /members/{member-id}/missions-progress/{mission-id}/request-success
    2. **Request Body**
        - 필요 X
    3. **Request Header**
        - Authorization: Bearer Token
    4. **Query String**
        - 필요 X
    5. **Path variable**
        - member-id(사용자 ID), mission-id(미션 ID)
    6. **Response Body**
        ```json
        {
          "isSuccess": true,
          "code": "string",
          "message": "string",
          "result": {
        	  "ownerId": 1
          }
        }
        ```
        
2. 성공 요청 후 미션이 완료되면 미션 완료 버튼 누르기
    1. **API Endpoint**
        - PATCH /members/{member-id}/missions-progress/{mission-id}/request-success/completed
    2. **Request Body**
        ```json
        {
        	"status": 2
        }
        ```
    3. **Request Header**
        - Authorization: Bearer Token
    4. **Query String**
        - 필요 X
    5. **Path variable**
        - member-id(사용자 ID), mission-id(미션 ID)
    6. **Response Body**
        ```json
        {
          "isSuccess": true,
          "code": "string",
          "message": "string",
          "result": {
        	  "points": 500
          }
        }
        ```

**회원 가입 하기(소셜 로그인 고려 X)**
1. **API Endpoint**
    - POST /members
2. **Request Body**
    ```json
    {
    	"name": "string", 
    	"nickname": "string",
    	"gender": 0,
    	"birth": "string",
    	"location": "string",
      "email": "string",
      "phone_number": "string"
      "foodKind": [
    	  "string"
      ]
    }
    ```
3. **Request Header**
    - Content-Type: application/json
4. **Query String**
    - 필요 X
5. **Path variable**
    - 필요 X
6. **Response Body**
    ```json
    {
      "isSuccess": true,
      "code": "string",
      "message": "string",
      "result": {
        "memberId": 1,
        "accessToken": "string"
      }
    }
    ```

**스터디 후 더 공부한 내용**
1. Access token을 Bearer로 감싸서 보내는 이유
    - 간단하고 안전한 전송 방식
        - Bearer 토큰은 HTTP 요청의 Authorization 헤더에 포함되어 서버로 전달되는데 이는 별도의 복잡한 인증 과정 없이 토큰을 소지한 사람만이 리소스에 접근할 수 있도록 한다.
            - ex. Authorization: Bearer <access_token>
    - OAuth 2.0 표준 준수
        - OAuth 2.0 표준은 Bearer 토큰을 사용하여 액세스 권한을 부여하고 이를 통해 서버는 해당 토큰의 유효성을 확인할 수 있다.
    - 보안 강화 및 유효성 검증
        - Bearer 방식은 주로 SSL/TLS와 같은 안전한 통신 프로토콜과 함께 사용되어 토큰이 중간에서 도청되는 위험을 방지한다.
    - 인증 간소화
        - 클라이언트는 별도의 사용자 인증 정보를 매번 입력할 필요 없이 서버에 Bearer 토큰을 전송하기만 하면 된다. 서버는 해당 토큰의 유효성만 확인하면 그 클라이언트의 신원을 확인한 것으로 간주한다.
2. 데이터베이스에서 id를 AUTO_INCREMENT(고유한 숫자 식별자(ID)를 자동으로 생성)로 설정했을 경우, Request Body에 id 필드를 넣어 값을 지정해주어야 되는지
    - Request Body에 id 포함 여부
        - **포함하지 않아야 한다**. 데이터베이스가 새로운 레코드가 생성될 때마다 자동으로 고유한 id를 부여하기 때문에 클라이언트가 직접 id를 지정할 필요가 없다.
    - Response Body에 id 포함 여부
        - **포함하는 것이 좋다**. 새로운 리소스가 생성된 후, 서버는 데이터베이스에 의해 생성된 id를 응답으로 반환하는 것이 일반적이다. 클라이언트가 생성된 리소스의 id를 확인하고 이후에 이를 참조하거나 사용할 수 있도록 도와준다.