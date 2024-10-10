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
        	  "completedMission": {
        		  "completedMissionId": 1,
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