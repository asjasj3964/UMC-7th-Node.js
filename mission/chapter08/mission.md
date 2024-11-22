### 🔥 미션
---
> GitHub 저장소 주소 <br/>
> [https://github.com/asjasj3964/UMC-7th-Node.js-Workbook/tree/feature/frontend-connection-swagger](https://github.com/asjasj3964/UMC-7th-Node.js-Workbook/tree/feature/frontend-connection-swagger)

1. index.js에 Swagger 설정 셋팅하기
    ```javascript
    // src/index.js
    
    import express from 'express'; // ES Module
    import swaggerAutogen from 'swagger-autogen';
    import swaggerUiExpress from "swagger-ui-express"
    
    ...
    
    app.use(
      "/docs", // Swagger UI가 표시될 경로
      swaggerUiExpress.serve, // Swagger UI의 정적 파일(HTML, CSS, JS)을 제공하는 미들웨어
      swaggerUiExpress.setup({}, { // Swagger UI의 설정 초기화, Swagger 문서를 불러오는 방식 정의
        swaggerOptions: { // Swagger UI의 동작 제어
          url: "/openapi.json", // /openapi.json 경로에서 Swagger 문서를 가져오도록 설정
        },
      })
    );
    ```
    - 사용자가 브라우저에서 /docs 경로로 접속하면 Swagger UI가 표시된다.
    - Swagger UI는 /openapi.json 경로에서 OpenAPI 문서를 가져와 API 문서를 시각적으로 보여준다.
    ```javascript
    app.get("/openapi.json", async(req, res, next) => { // 클라이언트의 Swagger 문서 요청 시 /openapi.json 경로가 호출된다. 
      // #swagger.ignore = true
      const options = { // Swagger Autogen 옵션 설정
        openapi: "3.0.0", // 생성될 문서의 OpenAPI 버전
        disableLogs: true, // Swagger Autogen 실행 시 불필요한 로그 출력 비활성화
        writeOutputFile: false, // 문서를 파일로 저장하지 않는다.
      };
      const outputFile = "/dev/null"; // 생성된 Swagger 문서를 저장할 파일 경로 지정 (파일 출력은 사용하지 않는다.) 
      const routes = ["./src/index.js"]; // 문서화할 라우트가 정의된 파일 경로를 배열로 지정
      const doc = { // Swagger 문서의 메타데이터와 스키마 정의
        info: { // API의 제목과 설명 설정
          title: "UMC 7th",
          description: "UMC 7th Node.js 테스트 프로젝트"
        },
        host: "localhost:3001", // API가 실행되는 서버의 호스트 정보
        ...
      }
      ...
    }
    ```
    - 클라이언트가 /openapi.json 경로에 GET 요청을 보내면 Swagger Autogen을 통해 src/index.js의 라우트를 분석하고 메타데이터(doc)와 옵션(options)을 기반으로 OpenAPI 문서를 생성한다.
        - Swagger 문서화 되는 src/index.js의 라우트
            - POST 요청
                - /members
                - /restaurants
                - /reviews
                - /missions
                - /participated-missions
            - GET 요청
                - /restaurants/:restaurantId/reviews
                - /members/:memberId/reviews
                - /restaurants/:restaurantId/missions
                - /members/:memberId/participated-missions
            - PATCH 요청
                - /participated-missions/:participatedMissionId
        - Swagger 관련 라우트는 문서화 되지 않는다.
            - /docs
                - Swagger Autogen이 API에 실제로 영향을 미치지 않는 유틸리티 라우트를 문서화하지 않도록 설계되었기 때문에 (#swagger.ignore = true를 사용하지 않아도) 별도로 문서화하지 않는다.
            - /openapi.json
                - #swagger.ignore = true 주석을 명시적으로 추가해 라우트를 Swagger 문서의 일부로 포함하지 않는다.
    - OpenAPI 문서는 파일에 저장하지 않고 동적으로 생성된 JSON 데이터를 반환한다.
2. Component로 스키마, 응답, 파라비터 관리하기
    - component를 사용해 코드 중복을 줄이고 스키마 구조를 일관되게 관리한다.
    1. 응답(response) 관리    
        ```javascript
        const doc = { // Swagger 문서의 메타데이터와 스키마 정의
            ...
            components: { // 공통적으로 사용되는 스키마 정의
              responses: { // 응답 
                NotFoundErrorResponse: { // Not Found 에러 응답 
                  type: "object",
                  properties: {
                    resultType: { type: "string", example: "FAIL" },
                    error: { 
                      type: "object",
                      properties: {
                        errorCode: { type: "string", example: "U404" },
                        reason: { type: "string" },
                        data: { type: "object" }
                      } 
                    },
                    success: { type: "object", nullable: true, example: null }    
                  }
                },
                ForbiddenErrorResponse: { // Forbidden 에러 응답 
                  type: "object",
                  properties: {
                    resultType: { type: "string", example: "FAIL" },
                    error: { 
                      type: "object",
                      properties: {
                        errorCode: { type: "string", example: "U403" },
                        reason: { type: "string" },
                        data: { type: "object" }
                      } 
                    },
                    success: { type: "object", nullable: true, example: null }    
                  }
                },
                DuplicateErrorResponse: { // 중복 에러 응답 
                  type: "object",
                  properties: {
                    resultType: { type: "string", example: "FAIL" },
                    error: { 
                      type: "object",
                      properties: {
                        errorCode: { type: "string", example: "U001" },
                        reason: { type: "string" },
                        data: { type: "object" }
                      } 
                    },
                    success: { type: "object", nullable: true, example: null }    
                  }
                },
                ServerErrorResponse: { // 서버 내부 에러 응답 
                  type: "object",
                  properties: {
                    resultType: { type: "string", example: "FAIL" },
                    error: { 
                      type: "object",
                      properties: {
                        errorCode: { type: "string", example: "U500" },
                        reason: { type: "string" },
                        data: { type: "object" }
                      } 
                    },
                    success: { type: "object", nullable: true, example: null }    
                  }
                },
                ReviewListSuccessResponse: { // 리뷰 목록 성공 응답 
                  type: "object",
                  properties: {
                    resultType: { type: "string", example: "SUCCESS" },
                    error: { type: "object", nullable: true, example: null },
                    success: {
                      type: "object",
                      properties: {
                        data: {
                          type: "array",
                          items: {
                            type: "object",
                            properties: {
                              id: { type: "string", example: "1" },
                              restaurant: { type: "string" },
                              writer: { type: "string" },
                              content: { type: "string" },
                              rating: { type: "number", example: 4.5 },
                              status: { type: "number", example: 0 },
                              createdAt: { type: "string", format: "date-time", example: "2024-11-18T14:23:45.123456Z" }
                            }
                          }
                        },
                        pagination: {
                          type: "object",
                          properties: {
                            cursor: {
                              type: "string",
                              nullable: true,
                              example: "0"
                            }
                          }
                        }
                      }
                    }
                  }
                },
                MissionSuccessResponse: { // mission 성공 응답 
                  type: "object",
                  properties: {
                    resultType: { type: "string", example: "SUCCESS" },
                    error: { type: "object", nullable: true, example: null },
                    success: {
                      type: "object", 
                      properties: {
                        id: { type: "string", example: "1" },
                        member: { type: "string" },
                        restaurant: { type: "string" },
                        name: { type: "string" },
                        introduction: { type: "string" },
                        deadline: { type: "string", format: "date-time", example: "2025-02-01T00:00:00Z" },
                        points: { type: "string", example: "100" },
                        status: { type: "number", example: 0 }
                      }
                    }    
                  }
                },
              },
        ```
        - API 테스트 했을 때 받는 성공 응답 및 실패 응답은 API 마다 있기 때문에 컴포넌트로 감싸 중복을 줄이도록 하였다.
    2. 스키마(Schema) 관리
        ```javascript
              '@schemas': { // swagger-autogen 렌더링을 무시하고 Swagger Spec에서 직접적으로 정의된 스키마 렌더링
                MemberSchema: { // member 스키마
                  type: "object",
                  properties: {
                    name: { type: "string" },
                    nickname: { type: "string" },
                    gender: { type: "number" },
                    birth: { type: "string", example: "2000-04-24" },
                    location: { type: "string" },
                    email: { type: "string" },
                    phoneNumber: { type: "string", example: "010-0000-0000" },
                    favoriteFoodKinds: { type: "array", items: { type: "number" } }
                  }
                },
                PaginationSchema: { // 페이징 응답 스키마
                  type: "object",
                  properties: {
                    cursor: {
                      type: "string",
                      nullable: true,
                      example: null
                    }
                  }
                }
              },
        ```
        - 마찬가지로 중복적으로 사용되는 스키마를 컴포넌트로 감싸주었다.
        - 스키마의 경우엔, schema: { … } 이런 식으로 작성하면 type, properties, example와 같은 스키마 구성 요소들이 스웨거에 그대로 나타나버린다. 나는 이러한 구성 요소들이 반영된 결과를 원했기 때문에 Swagger 명세의 자동 변환 과정을 우회하기 위해 ‘@schema’: { … } 이렇게 작성해주었다.
    3. 예제(example) 관리
        ```javascript
              examples: {
                ServerErrorExample: { // 서버 내부 에러 응답 예제
                  summary: "서버 내부 오류",
                  description: "서버에서 예상치 못한 오류가 발생하였습니다.",
                  value:{
                    resultType: "FAIL",
                    error: { 
                        errorCode: "U500",
                        reason: "서버 내부 오류",
                        data: {}
                    },
                    success: null 
                  } 
                },
                MemberNotFoundErrorExample: { // member Not Found 에러 응답 예제
                  summary: "존재하지 않는 회원",
                  description: "등록되어 있지 않은 회원 ID로 조회하였습니다.",
                  value: {
                    resultType: "FAIL",
                    error: { 
                      errorCode: "U404",
                      reason: "존재하지 않는 회원",
                      data: {}
                    },
                    success: null 
                  },
                },
                RestaurantNotFoundErrorExample: { // restaurant Not Found 에러 응답 예제
                  summary: "존재하지 않는 식당",
                  description: "등록되어 있지 않은 식당 ID로 조회하였습니다.",
                  value: {
                    resultType: "FAIL",
                    error: { 
                      errorCode: "U404",
                      reason: "존재하지 않는 식당",
                      data: {}
                    },
                    success: null 
                  },
                },
                FoodKindNotFoundErrorExample: { // foodKind Not Found 에러 응답 예제
                  summary: "존재하지 않는 음식 종류",
                  description: "등록되어 있지 않은 음식 종류 ID로 조회하였습니다.",
                  value: {
                    resultType: "FAIL",
                    error: { 
                        errorCode: "U404",
                        reason: "존재하지 않는 음식 종류",
                        data: {}
                    },
                    success: null 
                  },
                },
                DeadlineErrorExample: { // 마감기한 에러 응답 예제
                  summary: "이미 종료된 미션",
                  description: "이미 마감기한이 지난 미션입니다.",
                  value: {
                    resultType: "FAIL",
                    error: { 
                      errorCode: "U403",
                      reason: "이미 종료된 미션",
                      data: {}
                    },
                    success: null 
                  },
                }
              }
        ```
        - 중복되는 에러 응답의 예시들을 일관되게 관리할 수 있도록 component로 묶어주었다.
        - 각 예제마다 summary로 간략한 설명을, value로 해당 상황의 실제 JSON 데이터를 제공한다.
    4. 파라미터(Parameter) 관리
        ```javascript
              parameters: { // 파라미터
                CursorParam: { // 커서 파라미터
                  name: "cursor",
                  in: "query",
                  description: "페이징 커서 값 입력",
                  schema: {
                    type: "integer",
                    format: "int64"
                  }
                },
                MemberIdParam: { // 회원 ID 파라미터
                  name: "memberId",
                  in: 'path',
                  required: true,
                  description: "회원의 ID 입력",
                  schema: {
                    type: "integer",
                    format: "int64"
                  }
                },
                RestaurantIdParam: { // 식당 ID 파라미터 
                  name: "restaurantId",
                  in: 'path',
                  required: true,
                  description: "식당의 ID 입력",
                  schema: {
                    type: "integer",
                    format: "int64"
                  }
                },
              }
            }
          };
        ```
        - 중복으로 사용되는 query 파라미터 및 path 파라미터를 컴포넌트로 감싸주었다.
        - 파라미터 정보를 따로 추가하지 않으면 파라미터 타입이 “string”으로 되어있는데 이를 “integer”로 바꿔주었고 64비트 정수로 처리하였다.
3. Swagger 문서 자동 생성하기
    ```javascript
      const result = await swaggerAutogen(options)(outputFile, routes, doc);
      // swaggerAutogen(options): Swagger Autogen 모듈 초기화
      // outputFile: 문서를 출력할 파일 경로
      // routes: 라우트 파일 경로 배열
      // doc: 추가 메타데이터 및 스키마
      res.json(result ? result.data : null);
      // result 객체가 생성되었는지 확인
      // result.data: Swagger Autogen이 생성한 JSON 데이터 포함
    });
    ```
    - Swagger Autogen이 초기화되고 제공된 routes와 doc 데이터를 기반으로 Swagger 문서(outputFile 경로에 저장)가 생성된다.
    - result 객체에서 Swagger JSON 데이터를 추출하고 클라이언트는 API 호출로 이 JSON 데이터를 받는다. (문서 생성에 실패한 경우엔 null을 받는다.)
4. Controller에 주석으로 Swagger 정보를 추가하기
    - member-controller
        - 주석 작성
            ```javascript
            export const handleMemberSignUp = async(req, res, next) => {
                /*
                #swagger.tags = ['member-controller']
                #swagger.summary = '회원가입 API';
                #swagger.description = '회원가입 API입니다.'
                #swagger.requestBody = {
                    required: true,
                    content: {
                        "application/json": {
                            schema: {
                                $ref: "#/components/schemas/MemberSchema"
                            }
                        }
                    }
                };
                #swagger.responses[200] = {
                    description: "회원가입 성공 응답",
                    content: {
                        "application/json": {
                            schema: {
                                type: "object",
                                properties: {
                                    resultType: { type: "string", example: "SUCCESS" },
                                    error: { type: "object", nullable: true, example: null },
                                    success: {
                                        allOf: [
                                            { $ref: "#/components/schemas/MemberSchema" },
                                            { 
                                                type: "object",
                                                properties: {
                                                    id: { type: "string", example: "1" },
                                                    favoriteFoodKinds: { type: "array", items: { type: "string" } }
                                                }
                                            },
                                        ]
                                    }    
                                }
                            }
                        }
                    }
                };
                #swagger.responses[500] = {
                    description: "회원가입 실패 응답",
                    content: {
                        "application/json": {
                            schema: {
                                $ref: "#/components/responses/NotFoundErrorResponse"
                            },
                            examples: {
                                "중복된 이메일": {
                                    summary: "중복된 이메일",
                                    description: "이미 등록된 계정으로 가입 시도를 하였습니다.",
                                    value: {
                                        resultType: "FAIL",
                                        error: { 
                                            errorCode: "U001",
                                            reason: "중복된 이메일",
                                            data: {}
                                        },
                                        success: null 
                                    },
                                },
                                "존재하지 않는 음식 종류": {
                                    $ref: "#/components/examples/FoodKindNotFoundErrorExample"
                                }, 
                                "서버 내부 오류": {
                                    $ref: "#/components/examples/ServerErrorExample"
                                } 
                            }
                        }
                    }
                };
                */
                ...
            }   
            ```
            - #swagger.tags로 태그를 지정해주어 같은 controller에 정의된 API 엔드포인트끼리 그룹화하였다.
            - 회원가입 API의 요청 본문의 schema는 $ref: "#/components/schemas/MemberSchema" 로 컴포넌트로 감싼 스키마를 참조하도록 하였다.
            - allOf를 사용해 참조한 MemberSchema 스키마에 추가로 명시해줄 ID와 foodKinds 속성을 결합해주었다.
            - 성공 응답 이외의 발생할 수도 있는 에러 응답에 대해서도 $ref 로 해당 에러 응답을 참조하도록 하였다.
            - 에러 응답 메시지의 여러 example을 설정해서 이 또한 $ref로 각각의 예시 응답을 참조하였다.
            ```javascript
            export const handleListMemberReviews = async(req, res, next) => {
                /*
                #swagger.ignore = false
                #swagger.tags = ['member-controller']
                #swagger.summary = "회원의 리뷰 목록 조회 API";
                #swagger.description = '회원의 리뷰 목록 조회 API입니다.'
                #swagger.parameters['memberId'] = {
                    $ref: "#/components/parameters/MemberIdParam"
                }
                #swagger.parameters['cursor'] = {
                    $ref: "#/components/parameters/CursorParam"
                }
                #swagger.responses[200] = {
                    description: "미션 목록 조회 성공 응답",
                    content: {
                        "application/json": {
                            schema: {
                                $ref: "#/components/responses/ReviewListSuccessResponse"
                            }
                        }
                    }
                }
                #swagger.responses[500] = {
                    description: "미션 목록 조회 실패 응답",
                    content: {
                        "application/json": {
                            schema: {
                                $ref: "#/components/responses/NotFoundErrorResponse"
                            },
                            examples: {
                                "존재하지 않는 회원": {
                                    $ref: "#/components/examples/MemberNotFoundErrorExample"
                                }, 
                                "서버 내부 오류": {
                                    $ref: "#/components/examples/ServerErrorExample"
                                } 
                            }
                        }
                    }
                }
                */
                ...
            }
            ```
            - 위와 비슷하나 파라미터를 추가해주었다는 점이 다르다. 파라미터 또한 $ref로 해당 파라미터를 참조하였고, 성공 응답으로 미션 목록 및 페이징을 참조하였다.
            ```javascript
            export const handleListMemberMission = async(req, res, next) => {
                /*
                #swagger.ignore = false
                #swagger.tags = ['member-controller']
                #swagger.summary = "회원의 진행 중인 미션 목록 조회 API";
                #swagger.description = '회원의 진행 중인 미션 목록 조회 API입니다.'
                #swagger.parameters['memberId'] = {
                    $ref: "#/components/parameters/MemberIdParam"
                }
                #swagger.parameters['cursor'] = {
                    $ref: "#/components/parameters/CursorParam"
                }
                #swagger.responses[200] = {
                    description: "미션 목록 조회 성공 응답",
                    content: {
                        "application/json": {
                            schema: {
                                type: "object",
                                properties: {
                                    resultType: { type: "string", example: "SUCCESS" },
                                    error: { type: "object", nullable: true, example: null },
                                    success: {
                                        type: "object",
                                        properties: {
                                            data: {
                                                type: "array",
                                                items: {
                                                    $ref: "#/components/responses/MissionSuccessResponse/properties/success",
                                                }
                                            },
                                            pagination: {
                                                $ref: "#/components/schemas/PaginationSchema"
                                            }     
                                        }
                                    }
                                }
                            }
                        }
                    }
                };
                #swagger.responses[500] = {
                    description: "미션 목록 조회 실패 응답",
                    content: {
                        "application/json": {
                            schema: {
                                $ref: "#/components/responses/NotFoundErrorResponse"
                            },
                            examples: {
                                "존재하지 않는 회원": {
                                    $ref: "#/components/examples/MemberNotFoundErrorExample"
                                }, 
                                "서버 내부 오류": {
                                    $ref: "#/components/examples/ServerErrorExample"
                                } 
                            }
                        }
                    }
                }
                */
                ...
            }
            ```
            - 위와 비슷하다.
        - Swagger UI 확인<br/>
            <video controls src="videos/swagger-member-controller.mp4" title="Title"></video>
    - restaurant-controller
        - 주석 작성
            ```javascript
            export const handleRestaurantRegist = async(req, res, next) => {
                /*
                #swagger.tags = ['restaurant-controller']
                #swagger.summary = '식당 등록 API';
                #swagger.description = '식당 등록 API입니다.'
                #swagger.requestBody = {
                    required: true,
                    content: {
                        "application/json": {
                            schema: {
                                type: "object",
                                properties: {
                                    "ceoId": { type: "number" },
                                    "regionId": { type: "number" },
                                    "name": { type: "string" },
                                    "introduction": { type: "string" },
                                    "foodKinds": { type: "array", items: { type: "number" } },
                                    "startTime": { type: "string", example: "09:00:00" },
                                    "endTime": { type: "string", example: "21:00:00" }
                                }
                            }
                        }
                    }
                };
                #swagger.responses[200] = {
                    description: "식당 등록 성공 응답",
                    content: {
                        "application/json": {
                            schema: {
                                type: "object",
                                properties: {
                                    resultType: { type: "string", example: "SUCCESS" },
                                    error: { type: "object", nullable: true, example: null },
                                    success: {
                                        type: "object",
                                        properties: {
                                            "id": { type: "string", example: "1" },
                                            "ceo": { type: "string" },
                                            "region": { type: "string" },
                                            "name": { type: "string" },
                                            "introduction": { type: "string" },
                                            "foodKinds": { type: "array", items: { type: "string" } },
                                            "startTime": { type: "string", example: "09:00:00" },
                                            "endTime": { type: "string", example: "21:00:00" }
                                        }
                                    }  
                                }
                            }
                        }
                    }
                };
                #swagger.responses[500] = {
                    description: "식당 등록 실패 응답",
                    content: {
                        "application/json": {
                            schema: {
                                $ref: "#/components/responses/NotFoundErrorResponse"
                            },
                            examples: {
                                "존재하지 않는 CEO": {
                                    summary: "존재하지 않는 CEO",
                                    description: "등록되어 있지 않은 CEO ID로 조회하였습니다.",
                                    value: {
                                        resultType: "FAIL",
                                        error: { 
                                            errorCode: "U404",
                                            reason: "존재하지 않는 CEO",
                                            data: {}
                                        },
                                        success: null 
                                    },
                                },
                                "존재하지 않는 위치": {
                                    summary: "존재하지 않는 위치",
                                    description: "등록되어 있지 않은 위치 ID로 조회하였습니다.",
                                    value: {
                                        resultType: "FAIL",
                                        error: { 
                                            errorCode: "U404",
                                            reason: "존재하지 않는 위치",
                                            data: {}
                                        },
                                        success: null 
                                    }
                                },
                                "존재하지 않는 음식 종류": {
                                    $ref: "#/components/examples/FoodKindNotFoundErrorExample"
                                }, 
                                "중복된 식당": {
                                    summary: "중복된 식당",
                                    description: "이미 동일한 위치와 이름의 식당이 존재합니다.",
                                    value: {
                                        resultType: "FAIL",
                                        error: { 
                                            errorCode: "U001",
                                            reason: "중복된 식당",
                                            data: {}
                                        },
                                        success: null 
                                    }                    
                                },
                                "서버 내부 오류": {
                                    $ref: "#/components/examples/ServerErrorExample"
                                } 
                            }
                        }
                    }
                };
                */
                ...
            }

            export const handleListRestaurantReviews = async(req, res, next) => {
                /*
                #swagger.ignore = false
                #swagger.tags = ['restaurant-controller']
                #swagger.summary = "식당의 리뷰 목록 조회 API";
                #swagger.description = '식당의 리뷰 목록 조회 API입니다.'
                #swagger.parameters['restaurantId'] = {
                    $ref: "#/components/parameters/RestaurantIdParam"
            
                }
                #swagger.parameters['cursor'] = {
                    $ref: "#/components/parameters/CursorParam"
                }
                #swagger.responses[200] = {
                    description: "리뷰 목록 조회 성공 응답",
                    content: {
                        "application/json": {
                            schema: {
                                $ref: "#/components/responses/ReviewListSuccessResponse"
                            }
                        }
                    }
                };
                #swagger.responses[500] = {
                    description: "리뷰 목록 조회 실패 응답",
                    content: {
                        "application/json": {
                            schema: {
                                $ref: "#/components/responses/NotFoundErrorResponse"
                            },
                            examples: {
                                "존재하지 않는 식당": {
                                    $ref: "#/components/examples/RestaurantNotFoundErrorExample"
                                },
                                "서버 내부 오류": {
                                    $ref: "#/components/examples/ServerErrorExample"
                                } 
                            }
                        }
                    }
                };
                */
                ...
            }
            
            export const handleListRestaurantMissions = async(req, res, next) => {
                /*
                #swagger.ignore = false
                #swagger.tags = ['restaurant-controller']
                #swagger.summary = "식당의 미션 목록 조회 API";
                #swagger.description = '식당의 미션 목록 조회 API입니다.'
                #swagger.parameters['restaurantId'] = {
                    $ref: "#/components/parameters/RestaurantIdParam"
                }
                #swagger.parameters['cursor'] = {
                    $ref: "#/components/parameters/CursorParam"
                }
                #swagger.responses[200] = {
                    description: "식당의 미션 목록 조회 성공 응답",
                    content: {
                        "application/json": {
                            schema: {
                                type: "object",
                                properties: {
                                    resultType: { type: "string", example: "SUCCESS" },
                                    error: { type: "object", nullable: true, example: null },
                                    success: {
                                        type: "object",
                                        properties: {
                                            data: {
                                                type: "array",
                                                items: {
                                                    type: "object",
                                                    properties: {
                                                        id: { type: "string", example: "1" },
                                                        restaurant: { type: "string" },
                                                        name: { type: "string" },
                                                        introduction: { type: "string" },
                                                        points: { type: "number", example: 4.5 },
                                                        status: { type: "number" },
                                                    }
                                                }
                                            },
                                            pagination: {
                                                $ref: "#/components/schemas/PaginationSchema"
                                            }
                                        }
                                    }
                                }                
                            }
                        }
                    }
                };
                #swagger.responses[500] = {
                    description: "미션 목록 조회 실패 응답",
                    content: {
                        "application/json": {
                            schema: {
                                $ref: "#/components/responses/NotFoundErrorResponse"
                            },
                            examples: {
                                "존재하지 않는 식당": {
                                    $ref: "#/components/examples/RestaurantNotFoundErrorExample"
                                },
                                "서버 내부 오류": {
                                    $ref: "#/components/examples/ServerErrorExample"
                                } 
                            }
                        }
                    }
                };
                */
                ...
            }
            
            ```
        - Swagger UI 확인<br/>
            <video controls src="videos/swagger-restaurant-controller.mp4" title="Title"></video>
    - mission-controller
        - 주석 작성
            ```javascript
            export const handleMissionRegist = async(req, res, next) => {
                /*
                #swagger.tags = ['mission-controller']
                #swagger.summary = '미션 등록 API';
                #swagger.description = '미션 등록 API입니다.'
                #swagger.requestBody = {
                    required: true,
                    content: {
                        "application/json": {
                            schema: {
                                type: "object",
                                properties: {
                                    restaurantId: { type: "number" },
                                    name: { type: "string" },
                                    introduction: { type: "string" },
                                    deadline: { type: "string", format: "date-time", example: "2025-02-01T00:00:00Z" },
                                    points: { type: "number", example: 100 },
                                    status: { type: "number", example: 0 }
                                }
                            }
                        }
                    }
                };
                #swagger.responses[200] = {
                    description: "미션 등록 성공 응답",
                    content: {
                        "application/json": {
                            schema: {
                                type: "object",
                                properties: {
                                    resultType: { type: "string", example: "SUCCESS" },
                                    error: { type: "object", nullable: true, example: null },
                                    success: {
                                        type: "object", 
                                        properties: {
                                            id: { type: "string", example: "1" },
                                            restaurant: { type: "string" },
                                            name: { type: "string" },
                                            introduction: { type: "string" },
                                            deadline: { type: "string", format: "date-time", example: "2025-02-01T00:00:00Z" },
                                            points: { type: "string", example: "100" },
                                            status: { type: "number", example: 0 }
                                        }
                                    }    
                                }
                            }
                        }
                    }
                };
                #swagger.responses[500] = {
                    description: "미션 등록 실패 응답",
                    content: {
                        "application/json": {
                            schema: {
                                $ref: "#/components/responses/NotFoundErrorResponse"
                            },
                            examples: {
                                "잘못된 마감기한 설정": {
                                    summary: "잘못된 마감기한 설정",
                                    description: "미션의 마감기한을 과거로 설정하였습니다.",
                                    value: {
                                        resultType: "FAIL",
                                        error: { 
                                            errorCode: "U403",
                                            reason: "마감기한을 과거로 설정하였음",
                                            data: {}
                                        },
                                        success: null 
                                    },
                                },
                                "존재하지 않는 식당": {
                                    $ref: "#/components/examples/RestaurantNotFoundErrorExample"
                                }, 
                                "중복된 미션": {
                                    summary: "중복된 미션",
                                    description: "이미 동일한 식당, 미션명, 미션 설명, 마감기한의 미션이 존재합니다.",
                                    value: {
                                        resultType: "FAIL",
                                        error: { 
                                            errorCode: "U001",
                                            reason: "중복된 미션",
                                            data: {}
                                        },
                                        success: null 
                                    },
                                }, 
                                "서버 내부 오류": {
                                    $ref: "#/components/examples/ServerErrorExample"
                                } 
                            }
                        }
                    }
                };
                */
                ...
            }
            ```
        - Swagger UI 확인<br/>
            <video controls src="videos/swagger-mission-controller.mp4" title="Title"></video>
        - 주석 작성
            ```javascript
            export const handleMemberMissionRegist = async(req, res, next) => {
                /*
                #swagger.tags = ['participated-mission-controller']
                #swagger.summary = '참여 미션 등록 API';
                #swagger.description = '참여 미션 등록 API입니다.'
                #swagger.requestBody = {
                    required: true,
                    content: {
                        "application/json": {
                            schema: {
                                type: "object",
                                properties: {
                                    memberId: { type: "number" },
                                    missionId: { type: "number" }
                                }
                            }
                        }
                    }
                };
                #swagger.responses[200] = {
                    description: "참여 미션 등록 성공 응답",
                    content: {
                        "application/json": {
                            schema: {
                                $ref: "#/components/responses/MissionSuccessResponse"
                            }
                        }
                    }
                };
                #swagger.responses[500] = {
                    description: "참여 미션 등록 실패 응답",
                    content: {
                        "application/json": {
                            schema: {
                                $ref: "#/components/responses/NotFoundErrorResponse"
                            },
                            examples: {
                                "존재하지 않는 미션": {
                                    summary: "존재하지 않는 미션",
                                    description: "등록되어 있지 않은 미션 ID로 조회하였습니다.",
                                    value: {
                                        resultType: "FAIL",
                                        error: { 
                                            errorCode: "U404",
                                            reason: "존재하지 않는 미션",
                                            data: {}
                                        },
                                        success: null 
                                    },
                                },
                                "존재하지 않는 회원": {
                                    $ref: "#/components/examples/MemberNotFoundErrorExample"
                                }, 
                                "이미 종료된 미션": {
                                    $ref: "#/components/examples/DeadlineErrorExample"
                                }, 
                                "이미 참여한 미션": {
                                    summary: "이미 참여한 미션",
                                    description: "이미 참여 미션으로 등록한 미션입니다.",
                                    value: {
                                        resultType: "FAIL",
                                        error: { 
                                            errorCode: "U001",
                                            reason: "이미 참여한 미션",
                                            data: {}
                                        },
                                        success: null 
                                    },
                                },
                                "서버 내부 오류": {
                                    $ref: "#/components/examples/ServerErrorExample"
                                } 
                            }
                        }
                    }
                };
                */
                ...
            }
            
            export const handleMissionUpdateCompleted = async(req, res, next) => {
                /* 
                #swagger.tags = ['participated-mission-controller']
                #swagger.summary = '미션 상태 업데이트 API';
                #swagger.description = '미션 상태 업데이트 API입니다.';
                #swagger.parameters['participatedMissionId'] = {
                    in: 'path',
                    required: true,
                    description: "참여한 미션의 ID 입력",
                    '@schema': {
                        type: "integer",
                        format: "int64"
                    }
                };
                #swagger.responses[200] = {
                    description: "미션 상태 업데이트 성공 응답",
                    content: {
                        "application/json": {
                            schema: {
                                type: "object",
                                properties: {
                                    resultType: { type: "string", example: "SUCCESS" },
                                    error: { type: "object", nullable: true, example: null },
                                    success: {
                                        allOf: [
                                            { $ref: "#/components/responses/MissionSuccessResponse/properties/success" },
                                            {
                                                type: "object", 
                                                properties: {
                                                    status: { type: "number", example: 1 }
                                                }   
                                            }
                                        ]
                                    }    
                                }
                            }
                        }
                    }
                };
                #swagger.responses[500] = {
                    description: "참여 미션 상태 업데이트 실패 응답",
                    content: {
                        "application/json": {
                            schema: {
                                $ref: "#/components/responses/NotFoundErrorResponse"
                            },
                            examples: {
                                "존재하지 않는 참여 미션": {
                                    summary: "존재하지 않는 참여 미션",
                                    description: "등록되어 있지 않은 참여 미션의 ID로 조회하였습니다.",
                                    value: {
                                        resultType: "FAIL",
                                        error: { 
                                            errorCode: "U404",
                                            reason: "존재하지 않는 참여 미션",
                                            data: {}
                                        },
                                        success: null 
                                    }
                                },
                                "이미 완료된 미션": {
                                    summmary: "이미 완료된 미션",
                                    description: "이미 완료된 미션입니다.",
                                    value: {
                                        resultType: "FAIL",
                                        error: { 
                                            errorCode: "U403",
                                            reason: "이미 완료된 미션",
                                            data: {}
                                        },
                                        success: null                         
                                    }
                                },
                                "이미 종료된 미션": {
                                    $ref: "#/components/examples/DeadlineErrorExample"
                                },
                                "서버 내부 오류": {
                                    $ref: "#/components/examples/ServerErrorExample"
                                } 
                            }
                        }
                    }
                };
                */
                ...
            }
            
            ```
        - Swagger UI 확인<br/>
            <video controls src="videos/swagger-participated-mission-controller.mp4" title="Title"></video>
    - review-controller
        - 주석 작성
            ```javascript
            export const handleReviewRegist = async(req, res, next) => {
                /*
                #swagger.tags = ['review-controller']
                #swagger.summary = '리뷰 등록 API';
                #swagger.description = '리뷰 등록 API입니다.'
                #swagger.requestBody = {
                    required: true,
                    content: {
                        "application/json": {
                            schema: {
                                type: "object",
                                properties: {
                                    memberId: { type: "number" },
                                    restaurantId: { type: "number" },
                                    rating: { type: "number", example: 3.5 },
                                    content: { type: "string" }
                                }
                            }
                        }
                    }
                };
                #swagger.responses[200] = {
                    description: "리뷰 등록 성공 응답",
                    content: {
                        "application/json": {
                            schema: {
                                type: "object",
                                properties: {
                                    resultType: { type: "string", example: "SUCCESS" },
                                    error: { type: "object", nullable: true, example: null },
                                    success: {
                                        type: "object", 
                                        properties: {
                                            id: { type: "string", example: "1" },
                                            member: { type: "string" },
                                            restaurant: { type: "string" },
                                            rating: { type: "number", example: 3.5 },
                                            content: { type: "string" },
                                            createdAt: { type: "string", format: "date-time", example: "2024-11-18T14:23:45.123456Z" },
                                            status: { type: "number" }
                                        }
                                    }    
                                }
                            }
                        }
                    }
                };
                #swagger.responses[500] = {
                    description: "리뷰 등록 실패 응답",
                    content: {
                        "application/json": {
                            schema: {
                                $ref: "#/components/responses/NotFoundErrorResponse"
                            },
                            examples: {
                                "존재하지 않는 식당": {
                                    $ref: "#/components/examples/RestaurantNotFoundErrorExample"
                                }, 
                                "존재하지 않는 회원": {
                                    $ref: "#/components/examples/MemberNotFoundErrorExample"
                                }, 
                                "서버 내부 오류": {
                                    $ref: "#/components/examples/ServerErrorExample"
                                } 
                            }
                        }
                    }
                };
                */
                ...
            }
             
            ```
        - Swagger UI 확인<br/>
            <video controls src="videos/swagger-review-controller.mp4" title="mission/chapter08/videos/swagger-review-controller.mp4"></video>

<br/><br/>
- Swagger 작성하면서 알게 된 것
    1. Swagger/OpenAPI Spec에서는 examples의 value 안에서 $ref를 사용할 수 없다. Spec의 제한 사항으로, value 필드에는 직접적인 JSON 형식의 응답 데이터를 작성해야 한다.
    2. allOf는 여러 ‘스키마’를 결합하여 새로운 스키마를 생성하는 역할을 하는데, 같은 속성을 여러 스키마에서 정의한 경우 마지막에 정의된 값으로 치환된다. 
    3. @schemas는 Swagger Autogen이 OpenAPI 표준에 맞게 스키마를 components.schemas 섹션에 등록하도록 강제하여 $ref 참조가 항상 올바르게 동작하도록 보장한다. 다른 컴포넌트(examples, parameters, responses)는 이미 고유한 위치에 등록되므로 별도의 접두사(@)가 필요하지 않다.