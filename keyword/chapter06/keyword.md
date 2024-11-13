### **📦 실습**
---
1. 회원가입 API
    - Repository 함수
        ```javascript
        // 회원 데이터 삽입 (회원 등록) & 회원 ID 반환
        export const addMember = async(data) => {
            const member = await prisma.member.findFirst({where: {email: data.email}}); // 해당 이메일로 등록된 회원(중복 회원)이 존재하는지 확인
            if (member){ // 해당 이메일로 등록된 회원이 있을 경우
                return null;
            }
            const created = await prisma.member.create({data: data}); // 회원 생성
            return created.id; // 생성된 회원 ID 반환
        };

        // 회원 정보 조회
        export const getMember = async (memberId) => {
            const member = await prisma.member.findFirstOrThrow({ where: {id: memberId}});
            // prisma에서 member 테이블에 접근하여 해당 memberId와 일치하는, 첫 번째 레코드를 조회한다. 
            // 해당 레코드가 없을 시 예외를 던진다(에러 발생).
            return member;
        };

        // 음식 - 선호_음식_종류 매핑
        export const setFavoriteFoodKind = async(memberId, favoriteFoodKindId) => {
            await prisma.memberFavoriteFoodKind.create({
                data:{ // 새 레코드의 필드와 값을 지정한다. 
                    memberId: memberId,
                    foodKindId: favoriteFoodKindId
                },
            });
        };

        // 회원 - 선호_음식_종류 반환
        export const getMemberFavoriteFoodKindByMemberId = async (memberId) => {
            const favoriteFoodKinds = await prisma.memberFavoriteFoodKind.findMany({ // 여러 레코드 조회, 조건에 맞는 모든 레코드를 배열 형태로 반환
                select: { // 반환할 필드 명시
                    id: true,
                    memberId: true,
                    foodKindId: true,
                    foodKind: true, // 참조하는 foodKind 테이블
                },
                where: { memberId: memberId },
                orderBy: {foodKindId: "asc"}, // foodKindId 기준 오름차순 정렬
            });
            return favoriteFoodKinds;
        }
        ```
    - 테스트 결과<br/>
        ![실습-1-회원등록](images/실습-1-회원등록.png)
    - 동일한 이메일로 회원가입하는 경우<br/>
        ![images/실습-1-중복된_회원](images/실습-1-중복된_회원.png)
        - 회원 이메일로 조회했을 때 이미 데이터가 존재할 경우 에러를 발생시켰다.  
2. 특정 가게의 리뷰 목록 조회 API
    - Repository 함수
        ```javascript
        // 특정 식당의 모든 리뷰 조회
        export const getAllRestaurantReviews = async (restaurantId, cursor) => {
            const reviews = await prisma.review.findMany({ // Prisma ORM을 사용하여 review 테이블에서 여러 개의 레코드를 조회한다. 
                select: {
                    id: true,
                    member: true,
                    restaurant: true,
                    rating: true,
                    createdAt: true,
                    content: true,
                    status: true
                },
                where: { restaurantId: restaurantId, id: { gt: cursor }},
                orderBy: { id: "asc"},
                take: 5,
            })
            const formattedReviews = reviews.map(review => ({
                ...review,
                id: review.id.toString(),
                member: {
                    id: review.member.id.toString(),
                    name: review.member.name,
                    nickname: review.member.nickname,
                    birth: review.member.birth,
                    gender: review.member.gender,
                    location: review.member.location,
                    phoneNumber: review.member.phoneNumber
                },
                restaurant: {
                    id: review.restaurant.id.toString(),
                    name: review.restaurant.name
                },
            }));

            return formattedReviews;
        }
        ```
    - 테스트 결과<br/>
        ![실습-2-특정가게_리뷰목록](images/실습-2-특정가게_리뷰목록.png)
### 🎯 핵심 키워드
---
- ORM (Object-Relation Mapping)
    - 객체 지향 프로그래밍 언어에서 데이터베이스를 관리하기 위한 기술로 객체와 관계형 데이터베이스 간의 변환을 자동으로 처리하는 기능을 제공한다.
    - 개발자는 SQL을 직접 작성하지 않고(개발 속도 ↑) 데이터베이스 작업을 객체 지향 언어의 코드로 할 수 있다.
    - 특정 데이터베이스에 종속되지 않고 다양한 DBMS를 쉽게 교체할 수 있다.
    - ORM 도구: Prisma
        - Node.js 및 TypeScript를 위한 최신 ORM 도구로 데이터베이스와 상호작용하는 작업을 더 간편하고 효율적으로 수행할 수 있도록 돕는다.
        1. **Prisma Client**
            - 데이터베이스에 쿼리를 실행하기 위한 자동 생성된 타입 안전한 코드
            - 이를 통해 TypeScript를 사용할 때 타입 오류를 방지하고 코드 자동 완성 기능을 사용할 수 있다.
        2. **Prisma Migrate**
            - 데이터베이스 스키마를 관리하는 기능
            - Migration 파일을 생성하고 버전 관리를 통해 스키마 변경을 쉽게 관리할 수 있다.
        3. **Prisma Studio**
            - 데이터베이스를 시각적으로 탐색하고 관리할 수 있는 웹 기반 UI
            - 데이터를 조회, 편집, 삭제하는 작업을 직관적으로 수행할 수 있다.
        - ex. prisma client를 사용해 리뷰 데이터를 생성(리뷰 등록)하고 조회하는 코드
        ```javascript
        // prisma/schema.prisma
        
        ...
        
        // 리뷰 모델
        model Review{
          id BigInt @id @default(autoincrement())
          member Member @relation(fields: [memberId], references: [id])
          memberId BigInt @map("member_id")
          restaurant Restaurant @relation(fields: [restaurantId], references: [id])
          restaurantId BigInt @map("restaurant_id")  
          rating Decimal @db.Decimal(2, 1) @default(0.0)
          content String
          createdAt DateTime @map("created_at") @db.Timestamp(6) @default(now()) 
          updatedAt DateTime @map("updated_at") @db.Timestamp(6) @default(now())
          status Int @default(1)  
        
          replys Reply[]
          images Image[]
        
          @@index([memberId], map: "member_id")
          @@index([restaurantId], map: "restaurant_id")
          @@map("review")
        }
        
        ...
        ```
        ```javascript
        // src/db.config.js
        
        import { PrismaClient } from '@prisma/client';
        // 데이터베이스와 상호 작용하기 위한 기본 클라이언트(PrismaClient)를 가져온다. 
        
        export const prisma = new PrismaClient({ log: ["query"] });
        // PrismaClient의 새 인스턴스 생성
        // 실행되는 모든 쿼리를 콘솔에 로깅하도록 한다.
        ```
        ```javascript
        // src/repositories/review.repository.js
        
        import { prisma, pool } from "../db.config.js";
        
        // 리뷰 등록 및 리뷰 ID 반환
        export const addReview = async(data) => {
        		// 리뷰를 등록하고자 하는 식당이 존재하는지 조건을 만족하는 첫 번재 데이터를 추출하여 확인
            const restaurant = await prisma.restaurant.findFirst( {
                where: {
                    id: data.restaurant // 리뷰를 등록하고자 하는 식당의 ID로 확인한다. 
                }
            })
            if (restaurant == null){ // 식당이 존재하지 않을 시
                return null;
            } 
            const created = await prisma.review.create({ // 리뷰 생성
                data: { // 생성할 데이터 객체
                    ...data, // data 객체의 속성을 펼친다.
                    member: {
                        connect: { // member 테이블과 관계 연결
                            id: data.member 
                        }
                    },
                    restaurant:{ 
                        connect: { // restaurant 테이블과 관계 연결
                            id: data.restaurant
                        }
                    }
                }
            });
            return created.id; // 생성된 리뷰의 ID 반환
        }
        
        export const getReview = async(reviewId) => {
        		// 해당 리뷰 ID에 일치하는 리뷰 조회 (데이터가 없을 시 예외 발생)
            const review = await prisma.review.findFirstOrThrow({
                select: { // 반환할 필드 명시
                    id: true, 
                    member: true, // 참조하는 member테이블
                    restaurant: true, // 참조하는 restaurant 테이블
                    rating: true,
                    content: true,
                    createdAt: true,
                    status: true
                },
                where: {
                    id: reviewId // 해당 reviewId와 일치하는 리뷰 조회
                }
            });
            const formattedReview = { // 리뷰 데이터 형식화
                ...review, // 위에서 select한 속성들을 펼친다. 
                id: review.id.toString(), 
                // DB의 id 필드가 BigInt 타입으로 정의되어 있는데
                // javaScript에선 BigInt 타입은 JSON으로 변환할 수 없어
                // BigInt 타입의 id를 문자열로 변환해주었다. 
                member: { // 참조하는 member 테이블에서 추출할 속성
                    id: review.member.id.toString(),
                    name: review.member.name,
                },
                restaurant: {
                    id: review.restaurant.id.toString(),
                    name: review.restaurant.name,
                },
            }
            return formattedReview;
        }
        ```
        - Prisma 사용 흐름
            1. prisma/schema.prisma 파일에서 데이터베이스 모델을 정의한다.
            2. 스키마 변경 사항을 데이터베이스에 적용하기 위해 Prisma Migration을 실행한다.
            3. 데이터베이스 작업을 수행하기 위해 Prisma Client를 생성한다.
            4. Prisma Client를 사용해 쿼리를 작성하여 데이터베이스에 CRUD 작업을 수행한다.
    - 기타 ORM 도구
        1. Entity Framework: .NET 환경에서 사용되는 ORM
        2. Hibernate: Java 생태계에서 가장 많이 사용하는 ORM 프레임워크
- Prisma 문서 살펴보기
    - ex. Prisma의 Connection Pool 관리 방법
        - Prisma는 데이터베이스와의 효율적인 연결 관리를 위해 자체적인 Connection Pool을 사용한다. 이는 데이터베이스 연결을 재사용하여 성능을 향상시키고 특히 서버리스 환경에서의 연결 문제를 완화하는 데 도움을 준다.
        - 동작 방식
            - Prisma의 쿼리 엔진은 첫 번째 데이터베이스 연결 시 커넥션 풀을 생성한다.
            - 이후 쿼리가 실행될 때마다 기존 연결을 재사용하거나 필요에 따라 새로운 연결을 추가한다.
            - 이러한 방식으로 데이터베이스 연결의 효율성을 높이고 불필요한 연결 생성을 최소화한다.
        - 기본적으로 Prisma는 머신의 물리적 CPU 수를 기반으로 커넥션 풀의 크기를 결정한다.
            - 커넥션 풀 크기를 수동으로 설정하려면 데이터베이스 연결 URL에 connection_limit 파라미터를 추가한다.
                ```javascript
                datasource db {
                  provider = "mysql"
                  url      = "mysql://root:mypassword@localhost:3306/restaurant_service?connection_limit=5"
                }
                ```
                - 이 경우 커넥션 풀의 크기가 5로 제한된다.
    - ex. Prisma의 Migration 관리 방법
        - Prisma의 Migration 관리: 데이터베이스 스키마와 애플리케이션의 데이터 모델을 일관되게 유지하고 스키마 변경을 효율적으로 관리하기 위한 도구
        - Prisma 스키마 파일(schema.prisma)에서 데이터 모델을 정의하고 이를 기반으로 데이터베이스 스키마를 생성하거나 업데이트한다.
        - 스키마 변경 시, 해당 변경 사항을 반영한 SQL 마이그레이션 파일을 자동으로 생성하여 버전 관리가 가능하다.
        - 생성된 마이그레이션 파일을 개발, 테스트, 프로덕션 환경에 적용하여 스키마 변경을 일관되게 유지한다.
        - 주요 명령어
            - **prisma migrate dev**
                - 개발 환경에서 스키마 변경 시 새로운 Migration 파일을 생성하고 이를 데이터베이스에 적용한다.
                - Prisma Client를 자동으로 생성하여 코드에서 데이터베이스와 상호작용할 수 있도록 한다.
            - **prisma migrate deploy**
                - 프로덕션 환경에서 대기 중인 Migration을 순차적으로 적용한다.
                - 이 명령어는 CI/CD 파이프라인에서 사용되어 스키마 변경을 자동화한다.
            - **prisma migrate reset**
                - 개발 환경에서 데이터베이스를 초기화하고 모든 Migration을 다시 적용한다.
                - 테스트 데이터베이스를 재설정하거나 스키마를 초기 상태로 되돌릴 때 유용하다.
        - Migration 워크플로우
            1. schema.prisma 파일에서 데이터 모델을 수정한다.
            2. prisma migrate dev 명령어를 실행하여 새로운 마이그레이션 파일을 생성하고 이를 데이터베이스에 적용한다.
            3. 생성된 Migration 파일과 schema.prisma 파일을 버전 관리 시스템에 커밋하여 변경 이력을 추적한다.
            4. 프로덕션 환경에서는 prisma migrate deploy 명령어를 사용하여 대기 중인 Migration을 적용한다.
- ORM(Prisma)을 사용하여 좋은 점과 나쁜 점
    1. 좋은 점
        - TypeScript와 긴밀하게 통합되어 있어 모든 쿼리에 대해 자동으로 타입을 생성한다. 이를 통해 코드 작성 시 실수를 줄이고 안전하게 개발할 수 있다.
        - Prisma의 직관적인 API 덕분에 복잡한 데이터베이스 작업도 쉽게 구현할 수 있어 개발 속도가 빨라진다.
        - Prisma Migrate를 통해 데이터베이스 스키마를 자동으로 관리하고 버전 기록을 남길 수 있다.
        - MySQL, PostgreSQL, SQLite, SQL Server 등 다양한 관계형 데이터베이스와 호환된다.
    2. 나쁜 점
        - ORM이 자동으로 생성한 쿼리는 수작업으로 최적화된 SQL보다 느릴 수 있다. 복잡한 쿼리일수록 비효율적이 될 가능성이 크다.
        - 데이터베이스의 고유 기능을 활용하거나 세부적인 쿼리 최적화가 어려워 성능을 세밀하게 조정하기 힘들 수 있다.
        - 데이터베이스 구조가 복잡할 경우 모든 관계를 적절히 매핑하는 데 한계가 있다.
        - 자동 생성된 쿼리의 성능 문제나 오류를 해결하기가 어렵고 복잡할 수 있다.
        - 특정 ORM 라이브러리에 의존하게 되면,유지보수나 변경 시 어려움을 겪을 수 있다.
- 다양한 ORM 라이브러리 살펴보기
    - ex. Sequelize
        - Node.js에서 사용하는 ORM 라이브러리
        - 자바스크립트 코드로 데이터베이스 작업을 수행할 수 있으며 복잡한 SQL 쿼리를 작성하지 않고도 데이터베이스와 상호작용할 수 있다.
        - 데이터베이스 테이블을 모델로 정의하여 다룬다. 각 모델은 테이블의 구조를 정의하고 데이터를 쉽게 CRUD 작업할 수 있도록 지원한다.
        - 복잡한 쿼리를 쉽게 작성할 수 있는 메서드와 기능을 제공한다. findAll, create, update 등의 메서드를 통해 SQL 쿼리를 간단히 작성할 수 있다.
            ```javascript
            // 회원 데이터 생성
            async function createMember() {
                const member = await Member.create({
                    locationAddress: '서울시 관악구',
            		    email: 'ahnnn000@gmail.com',
            	      phoneNumber: '010-9836-3964',
            	      memberName: '안성진',
            	      nickname: '웬티',
            	      gender: 1,
            	      birth: 2000-04-24
                });
            }
            
            // 모든 회원 데이터 조회
            async function getMembers() {
                const members = await Member.findAll();
            }
            ```
        - Sequelize 모델
            - Sequelize 모델은 데이터베이스의 테이블을 코드 상에서 객체로 표현한 것이다. (데이터베이스의 테이블과 1:1로 매핑된다)
            - 이 모델을 통해 SQL 쿼리를 직접 작성하지 않고도 데이터베이스와 상호작용할 수 있다. CRUD 작업을 JavaScript 코드로 쉽게 수행할 수 있도록 해준다.
            - ex. Member 모델 정의
                ```javascript
                const { DataTypes, Model } = require('sequelize'); 
                // DataTypes: 데이터 타입 정의 모듈(string, integer 등)
                // Model: 모델 클래스를 확장하는 기본 클래스. 이를 상속받아 데이터베이스 테이블과 매핑되는 객체가 된다. 
                // Sequelize 라이브러리 사용
                const sequelize = require('../config/database'); 
                // 데이터베이스 설정 파일(database.js)에서 설정한 Sequelize 인스턴스를 가져온다. 
                class Member extends Model {} // Model 클래스를 상속받는다. 
                Member.init( // 모델을 초기화하고 테이블의 컬럼을 정의한다. 
                  {
                    id: {
                      **type**: DataTypes.BIGINT,
                      **primaryKey**: true,
                      **autoIncrement**: true,
                    },
                    locationAddress: {
                      type: DataTypes.TEXT,
                      **allowNull**: false,
                    },
                    email: {
                      type: DataTypes.STRING(50),
                      allowNull: false,
                      unique: true,
                    },
                    phoneNumber: {
                      type: DataTypes.STRING(15),
                      allowNull: true,
                    },
                    memberName: {
                      type: DataTypes.STRING(30),
                      allowNull: false,
                    },
                    nickname: {
                      type: DataTypes.STRING(30),
                      allowNull: false,
                    },
                    gender: {
                      type: DataTypes.INTEGER,
                      allowNull: false,
                    },
                    birth: {
                      type: DataTypes.STRING(10),
                      allowNull: false,
                    },
                    points: {
                      type: DataTypes.BIGINT,
                      defaultValue: 0,
                      allowNull: false,
                    },
                    createdAt: {
                      type: DataTypes.DATE(6),
                      allowNull: false,
                    },
                    updatedAt: {
                      type: DataTypes.DATE(6),
                      allowNull: false,
                    },
                    status: {
                      type: DataTypes.INTEGER,
                      allowNull: false,
                    },
                    inactiveAt: {
                      type: DataTypes.DATE(6),
                      allowNull: true,
                    },
                  },
                  {
                    sequelize, // Sequelize 인스턴스
                    modelName: 'Member', // 모델 이름 설정
                    tableName: 'member', // 매핑될 데이터베이스의 테이블 이름
                    timestamps: true, // createdAt, updatedAt 자동 관리 활성화
                  }
                );
                module.exports = Member; // Member 모델을 외부에서 사용하도록 내보낸다. 
                ```
    - ex. TypeORM
        - **TypeScript**와 **JavaScript**를 위한 ORM 라이브러리
        - 데코레이터를 사용하여 엔터티와 테이블, 컬럼 등을 정의한다. 이를 통해 객체와 데이터베이스 테이블 간의 매핑을 간단하게 설정할 수 있다.
        - 엔터티를 기반으로 데이터베이스 테이블을 자동 생성하고 스키마를 동기화할 수 있다.
        - 복잡한 SQL 쿼리를 TypeORM의 쿼리 빌더를 사용하여 쉽게 작성할 수 있다.
        - 데코레이터를 사용하여 컬럼에 대한 유효성 검사를 설정할 수 있다.
        - 관계된 엔터티를 지연 로딩(Lazy Loading)할 수 있는 기능을 제공한다.
- 페이지네이션을 사용햐는 다른 API 찾아보기
    - ex. https://docs.github.com/en/rest/using-the-rest-api/using-pagination-in-the-rest-api?apiVersion=2022-11-28
        - **GitHub REST API**
            - 페이지네이션을 사용해 대규모 데이터를 효율적으로 처리한다.
            - 기본적으로 한 페이지에 일정 수의 항목만 반환하며 link 헤더를 통해 다음 페이지, 이전 페이지 등으로 이동할 수 있는 URL을 제공한다.
            - per_page 매개변수를 사용해 페이지당 항목 수를 조정할 수 있다.
            - ex. 이슈 목록을 요청할 경우 (기본적으로 한 페이지에 30개 반환)
                ```javascript
                GET /repos/{owner}/{repo}/issues?per_page=10&page=2
                ```
                - per_page=10: 페이지당 10개의 이슈를 요청한다.
                - page=2: 두 번째 페이지를 요청한다.
    - ex. https://developers.notion.com/reference/intro#pagination
        - **Notion의 REST API**
            - 대량의 데이터를 효율적으로 처리하기 위해 페이지네이션을 사용한다.
            - 이는 한 번의 요청으로 반환되는 데이터 양을 제한하고 추가 데이터를 가져오기 위해 후속 요청을 수행하는 방식
            - 주요 요소
                1. **page_size**
                    - 한 번의 API 요청으로 반환되는 항목의 최대 수를 지정한다.
                    - 기본값은 100이며 최대 100까지 설정할 수 있습니다.
                    - ex. page_size=50 → 한 번의 요청으로 최대 50개의 항목 반환
                2. **start_cursor**
                    - 데이터의 특정 지점부터 반환을 시작하기 위해 사용되는 커서
                    - 이전 응답의 next_cursor 필드에서 가져온다. 이를 통해 다음 페이지의 데이터를 요청할 수 있다.
            - 데이터베이스의 항목을 조회할 경우의 페이지네이션 구현
                1. **첫 번째 요청**
                    ```javascript
                    POST https://api.notion.com/v1/databases/{database_id}/query
                    Content-Type: application/json
                    Notion-Version: 2022-06-28
                    {
                      "page_size": 100
                    }
                    ```
                    - 최대 100개의 항목을 반환하며 응답에는 next_cursor와 has_more 필드가 포함된다.
                2. **후속 요청**
                    ```javascript
                    POST https://api.notion.com/v1/databases/{database_id}/query
                    Content-Type: application/json
                    Notion-Version: 2022-06-28
                    {
                      "page_size": 100,
                      "start_cursor": "{next_cursor}"
                    }
                    ```
                    - 첫 번째 응답에서 has_more가 true이고 next_cursor가 존재하면 다음과 같이 후속 요청을 보낸다.
                    - 여기서 {next_cursor}는 이전 응답의 next_cursor 값을 사용한다. 이러한 방식으로 모든 데이터를 순차적으로 가져올 수 있다.