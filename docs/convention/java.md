---
title: "Spring Boot 기반 Java Coding Convention"
layout: default
---

# Spring Boot 기반 Java Coding Convention

본 문서는 Spring Boot 기반 프로젝트에서 일관된 개발을 위해 정의한 Coding Convention입니다.  
팀 전체가 동일한 규칙을 적용하여 코드 품질과 유지보수성을 높이는 것을 목표로 합니다.

[⬅️ 메인으로 돌아가기](../index.md)

---

## 1. 프로젝트 패키지 구조

### 1.1 루트 디렉토리 구조
```
project-root
├─ .database : DB 마이그레이션 및 SQL 스크립트
│ ├─ postgres/ : PostgreSQL 관련 스크립트
│ ├─ mysql/ : MySQL 관련 스크립트
│ └─ oracle/ : Oracle 관련 스크립트
│
├─ .docker : Docker 관련 설정
│ ├─ services/ : 서비스별 Dockerfile, 설정
│ └─ volumes/ : 데이터 볼륨 마운트 설정
│
├─ .github
│ └─ workflows : GitHub Actions 워크플로우 파일
│
├─ .http : REST Client 테스트 스크립트
│ └─ {modules ...}/ : 모듈... API 테스트
│
├─ .env : 환경 변수 파일(로컬 실행용)
├─ .gitignore
├─ build.gradle
├─ settings.gradle
├─ README.md
│
└─ {modules...}

```

- **.database** : DB 스크립트 (DBMS 별 구분)
- **.docker** : 서비스별 Dockerfile, volume 설정 구분
- **.github/workflows** : GitHub Actions CI/CD 정의
- **.http** : REST Client 스크립트 (도메인별 구분)
- **.env** : 환경 변수 파일 (Git에 올리지 않고 `.env.example`로 공유 권장)
- **build.gradle, settings.gradle** : 빌드 스크립트
- **README.md** : 프로젝트 설명 문서

---

### 1.2 Java 패키지 구조

```
com.obigo.{project}
├─ config : 전역 설정 (Spring, Security, DB, Swagger 등)
├─ common : 공용 유틸, 상수, 응답 Wrapper, 에러 코드
│ ├─ constants : 상수 정의
│ ├─ response : API 응답 형식을 일관되게 관리
│ ├─ error : 전역 에러 코드 관리
│ ├─ exception : 전역 예외 처리
│ ├─ util : 공용 유틸리티 클래스
│ └─ annotation : 프로젝트 전역에서 사용하는 커스텀 어노테이션
├─ domain
│ ├─ entity : JPA 엔티티
│ ├─ dto : 요청/응답 객체 (RequestDto, ResponseDto)
│ ├─ vo : 값 객체 (불변 데이터 구조)
│ └─ event : 도메인 이벤트 (Domain Event)
├─ repository : JPA Repository, MyBatis Mapper
├─ service
│ └─ impl : Service 구현체
└─ controller : REST API, GraphQL, gRPC 엔드포인트

```
- **대규모 프로젝트에서는 DDD 구조 확장 가능 (도메인별 독립 패키지 분리)**
---

## 2. 계층별 규칙

### 2.1 Controller
- **위치**: `controller`
- **역할**:
    - API 엔드포인트
    - 요청 DTO 검증 및 Service 호출
- **네이밍 규칙**: `{도메인}Controller`
- **메서드 네이밍**: `getUserById()`, `createUser()`, `updateUser()`, `deleteUser()`
- **규칙**:
    - 비즈니스 로직 포함 금지
    - `@Valid` 사용
    - 공통 응답 Wrapper (`ApiResponse<T>`) 사용

### 2.2 Service
- **위치**: `service`
- **역할**:
    - 비즈니스 로직
    - 트랜잭션 관리 (`@Transactional`)
    - 복수 Repository 및 외부 API 조합
- **네이밍 규칙**:
    - 인터페이스: `{도메인}Service`
    - 구현체: `{도메인}ServiceImpl`
- **패키지 배치 원칙**
    - **작은 규모 프로젝트**
        - 인터페이스와 구현체를 같은 패키지에 배치
        - 예:
          ```
            service/
            └─ UserService.java
            └─ UserServiceImpl.java
          ```
    - **대규모 프로젝트 / DDD 기반 설계**
        - 구현체를 별도의 `impl/` 패키지에 분리
        - 예:
          ```
            service/
            ├─ UserService.java
            └─ impl/
                └─ UserServiceImpl.java
          ```
- **규칙**:
    - **Controller → Service → Repository** 흐름 유지
    - 비즈니스 예외는 도메인 단위 예외(`UserNotFoundException`)로 정의
    - `@Transactional`은 Service 계층에서만 명시

### 2.3 Repository
- **위치**: `repository`
- **역할**:
    - DB 접근 (JPA/MyBatis)
- **네이밍 규칙**:
    - JPA: `{도메인}Repository`
    - MyBatis: `{도메인}Mapper`
- **규칙**:
    - 단일 도메인 기준
    - 반환값은 `Optional<T>` 또는 `List<T>`
    - 복잡한 쿼리는 `@Query` 또는 Querydsl 사용

### 2.4 Domain

#### 2.4.1 Entity
- **위치**: `domain.entity`
- **역할**:
    - DB 테이블과 매핑되는 핵심 데이터 모델
    - 서비스 내/외부에서 공용으로 사용되는 엔티티 정의
- **규칙**:
    1. 클래스명은 PascalCase 사용 (`TUser`, `TOrder`)
    2. `@Entity`, `@Table` 필수
        - `@Table(name = "t_user", comment = "사용자 정보 테이블")`
    3. 컬럼에는 `@Column` 및 `@Comment` 작성
        - Hibernate 6 이상: `@Comment` 사용
        - Hibernate 5 이하: `columnDefinition` 활용
    4. `@Id`는 `Long` 타입 기본키 사용, `@GeneratedValue` 전략 적용
    5. Setter 금지 → 명시적 변경 메서드 제공
    6. 연관관계 매핑은 단방향 우선, 지연 로딩(`LAZY`) 사용

- **예시**
  ```java
    @Entity
    @Table(name = "t_user", comment = "사용자 정보 테이블")
    @Getter
    @NoArgsConstructor
    public class TUser {

        @Id
        @GeneratedValue(strategy = GenerationType.IDENTITY)
        @Column(name = "id", nullable = false)
        @Comment("사용자 고유 ID")
        private Long id;

        @Column(name = "email", nullable = false, length = 100)
        @Comment("이메일 주소")
        private String email;

        public void changeEmail(String newEmail) {
            this.email = newEmail;
        }
    }
  ```

---

##### ① 내부 네이밍 규칙 (단일 서비스 기준)
- 클래스명: **PascalCase 단수형** (`TUser`, `TOrder`, `TPaymentHistory`)
- 패키지: `domain.entity` 또는 도메인 단위 분리 (`user.entity.TUser`)
- `@Table(name = "t_{snake_case}")` 사용 → DB 테이블명은 `t_user`, `t_order`
- 필드명: camelCase → 컬럼명: snake_case (`createdAt` → `created_at`)

- **PK 규칙**
    - Entity 필드명은 항상 `id`
    - DB 컬럼명도 `id` 그대로 사용
    - 간결하고 일관성 있게 유지 (`user.getId()`, `order.getId()`)
    - 접미사 `Entity`는 기본적으로 생략 (`TUserEntity ❌ → TUser ✅`)

  ```java
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
  ```

- **FK 규칙**
    - `@JoinColumn(name = "{entity명}_id")` 사용
    - DB 컬럼명은 참조하는 Entity명을 포함하여 명확하게 구분
    - 예:
        - `Order` → `User`: `@JoinColumn(name = "user_id")`
        - `Payment` → `Order`: `@JoinColumn(name = "order_id")`

---

##### ② Multi-PK (복합키) 규칙
- 복합키는 @EmbeddedId 방식 권장
    - VO처럼 관리 가능, 도메인 모델링 관점에서 유리
    - 재사용성 높음 (OrderItemId, UserRoleId)
    - equals(), hashCode() 반드시 재정의
- 단, Legacy DB 호환이나 단순 접근성이 중요한 경우 @IdClass 사용 허용
    - PK 필드가 엔티티 내부에 직접 노출됨 → getOrderId() 접근 가능
    - 단점: 필드 정의 중복, 재사용성 낮음

- **예시: @EmbeddedId**
  ```java
    @Entity
    @Table(name = "t_order_item")
    public class TOrderItem {
        @EmbeddedId
        private OrderItemId id;

        @MapsId("orderId")
        @ManyToOne
        @JoinColumn(name = "order_id")
        private Order order;

        @MapsId("productId")
        @ManyToOne
        @JoinColumn(name = "product_id")
        private Product product;
    }

    @Embeddable
    @Getter
    @EqualsAndHashCode
    public class OrderItemId implements Serializable {
        private Long orderId;
        private Long productId;
    }
  ```

- **예시: @IdClass**
  ```java
    @Entity
    @Table(name = "t_order_item")
    @IdClass(OrderItemId.class)
    public class OrderItem {
        @Id
        @Column(name = "order_id")
        private Long orderId;

        @Id
        @Column(name = "product_id")
        private Long productId;

        @ManyToOne
        @JoinColumn(name = "order_id", insertable = false, updatable = false)
        private Order order;

        @ManyToOne
        @JoinColumn(name = "product_id", insertable = false, updatable = false)
        private Product product;
    }

    @Getter
    @EqualsAndHashCode
    public class OrderItemId implements Serializable {
        private Long orderId;
        private Long productId;
    }
  ```

---

##### ③ Cross System 네이밍 규칙 (MSA/다중 시스템 기준)
- 다른 서비스의 엔티티와 혼동 방지를 위해 **접두사/역할 기반 이름** 사용
    - 외부 User → `ExternalUser`
    - 주문 내 "주문자(User)" → `Orderer` VO/DTO
- DB 컬럼 네이밍 충돌 방지를 위해 FK에는 `{entity명}_id` 규칙 고정
- 공통/Infra 엔티티는 `Entity` 접미사 허용 (`AuditLogEntity`, `SystemConfigEntity`)


##### ✅ 요약
1. **PK 컬럼명은 항상** id (Entity와 DB 모두 동일)
2. **FK 컬럼은** `{entity명}_id` (`user_id`, `order_id`)
3. **Multi-PK는 기본적으로 @EmbeddedId 권장**, 단 Legacy 호환 시 @IdClass 사용 허용
4. **서비스 내부**: 단순하고 직관적으로 (`User`, `Order`)
5. **Cross System**: `External{Entity}` 또는 역할 기반 (`Orderer`, `Payer`)

---

#### 2.4.2 DTO (Data Transfer Object)
- **위치**: `domain.dto`
- **역할**:
    - 계층 간 데이터 교환용 객체
    - Controller ↔ Service, Service ↔ 외부 API 통신 시 사용
- **규칙**:
    1. 요청/응답 객체 구분 (`UserRequestDto`, `UserResponseDto`)
    2. 불변 객체 지향 → `record`(Java 17+) 또는 `@Value` 사용
    3. Validation Annotation 적용 (`@NotNull`, `@Email`)
    4. Entity를 직접 반환하지 않고 DTO로 변환해서 사용

- **예시**
  ```java
    public record UserRequestDto(
        @NotNull(message = "이메일은 필수입니다.") String email,
        @NotNull(message = "비밀번호는 필수입니다.") String password) {}

    public record UserResponseDto(
        Long id,
        String userId,
        String email) {
        public static UserResponseDto from(User user) {
            return new UserResponseDto(user.getId(), user.getUserId(), user.getEmail());
        }
    }
  ```

#### 2.4.3 VO (Value Object)
- **위치**: `domain.vo`
- **역할**:
    - 특정 값 자체를 표현하는 불변 객체
    - 식별자(ID) 없이 값의 동등성으로만 동일성 비교
    - 예: `Email`, `PhoneNumber`, `Address`
- **규칙**:
    1. `final` 필드만 포함
    2. 불변성을 보장 (`equals`, `hashCode` 구현)
    3. Validation 로직 포함 가능 (ex: 이메일 형식 검사)
    4. Entity/DTO에서 VO를 조합해 사용

- **예시**
  ```java
    @Getter
    @EqualsAndHashCode
    public class Email {
        private final String value;

        public Email(String value) {
            if (!value.matches("^[A-Za-z0-9+_.-]+@(.+)$")) {
                throw new IllegalArgumentException("이메일 형식이 올바르지 않습니다.");
            }
            this.value = value;
        }

        @Override
        public String toString() {
            return value;
        }
    }
  ```

#### 2.4.4 Event (Domain Event)
- **위치**: `domain.event`
- **역할**:
    - 도메인 내에서 발생한 중요한 사건을 표현
    - 다른 계층/서비스에 전달되어 비동기 처리 가능
- **규칙**:
    1. `record` 또는 불변 클래스 사용
    2. 이벤트명은 과거 시제 사용 (`UserRegisteredEvent`)
    3. 이벤트 발행은 Service/Domain Layer에서만 수행
    4. Spring Event Publisher 또는 Kafka/RabbitMQ 등 메시징 시스템 활용

- **예시**
  ```java
    public record UserRegisteredEvent(Long userId, String email) {}

    @Service
    public class UserService {
        private final ApplicationEventPublisher publisher;

        public UserService(ApplicationEventPublisher publisher) {
            this.publisher = publisher;
        }

        public void registerUser(UserRequestDto request) {
            // 사용자 등록 로직...
            publisher.publishEvent(new UserRegisteredEvent(1L, request.email()));
        }
    }
  ```

### 2.5 Config

**위치**: `config`  
**역할**:
- Spring Boot 애플리케이션의 전역 환경 설정 관리
- Security, DB, CORS, Swagger, 메시징(Kafka/RabbitMQ) 등 인프라 관련 설정 포함
- `application.yml`에 정의된 설정을 타입 안전하게 바인딩하여 사용

---

#### 2.5.1 Config 클래스 작성 규칙
1. 클래스명은 `{기능}Config` 네이밍 (예: `SecurityConfig`, `SwaggerConfig`, `JpaConfig`)
2. 반드시 `@Configuration` 어노테이션을 사용
3. Bean 등록 시 `@Bean` 메서드명은 소문자 카멜케이스 사용
4. 외부 프로퍼티는 `@ConfigurationProperties` 또는 `@Value`로 주입받음
5. Config 클래스 안에 비즈니스 로직은 포함하지 않음 (순수 설정만 담당)

---

#### 2.5.2 application.yml 관리 규칙
1. `application.yml`은 **공통 속성만 정의**
    - 예: `spring.application.name`, `spring.profiles.active`, 공용 Jackson 설정 등
2. 환경별 차이가 있는 속성은 `application-{profile}.yml`에 분리
    - 예: DB URL, 계정 정보, 외부 API Key
3. 민감 정보는 yml에 직접 기록하지 않고 **환경 변수, Vault, Secret Manager**를 통해 관리
4. 중복 방지를 위해 반드시 **yml 계층 구조를 일관성 있게 유지**

---

#### 2.5.3 ConfigurationProperties 사용 규칙
1. 환경 설정을 타입 안전하게 바인딩하기 위해 `@ConfigurationProperties` 권장
2. prefix는 **서비스명 기반**으로 정의 (`myapp.db`, `myapp.external-api`)
3. `@EnableConfigurationProperties` 또는 `@ConfigurationPropertiesScan`으로 스캔
4. DTO와 동일하게 `record` 또는 `@Getter` 사용

**예시**
  ```java
    @ConfigurationProperties(prefix = "myapp.external-api")
    @Getter
    public class ExternalApiProperties {
        private String baseUrl;
        private String apiKey;
    }
  ```
  ```yml
    myapp:
      external-api:
        base-url: https://dev-api.example.com
        api-key: ${API_KEY}
  ```

#### 2.5.4 Secret 관리 규칙
1. 환경 설정을 타입 안전하게 바인딩하기 위해 `@ConfigurationProperties` 권장
2. prefix는 **서비스명 기반**으로 정의 (`myapp.db`, `myapp.external-api`)
3. `@EnableConfigurationProperties` 또는 `@ConfigurationPropertiesScan`으로 스캔
4. DTO와 동일하게 `record` 또는 `@Getter` 사용

#### 2.5.5 규칙 요약
- Config 클래스에는 설정/Bean 정의만 포함 → 비즈니스 로직 금지
- application.yml → 공통 설정만, 환경별 설정은 application-{profile}.yml
- 민감 정보는 Secret Manager/환경변수 활용
- Swagger/Security/JPA 등 핵심 인프라는 별도 Config 클래스 관리


### 2.6 Exception
- **위치**: `exception`
- **규칙**:
    - `BusinessException` 상속 구조
    - `ErrorCode` Enum 활용
    - 전역 핸들러(`@RestControllerAdvice`)에서 처리
- **네이밍**: `{도메인}NotFoundException`, `{도메인}InvalidException`

### 2.7 Common
- **위치**: `common`
- **역할**:
    - 프로젝트 전역에서 사용하는 공용 기능/객체를 관리하는 계층

#### 2.7.1 Constants
- **위치**: `common.constants`
- **역할**:
    - 상수 정의
- **규칙**:
    1. 공용 상수만 정의 (`public static final`)
    2. 상수명은 `UPPER_SNAKE_CASE`
    3. 도메인별 상수는 개별 도메인 패키지에서 관리 (Common에는 전역 상수만)

- **예시**
  ```java
    public final class GlobalConstants {
        public static final String DATE_FORMAT = "yyyy-MM-dd";
        public static final String SYSTEM_USER = "SYSTEM";
    }
  ```

#### 2.7.2 ApiResponse (공통 응답 Wrapper)
- **위치**: `common.response`
- **역할**:
    - API 응답 형식을 일관되게 관리
- **규칙**:
    1. 모든 Controller는 `ApiResponse<T>`로 응답
    2. 성공/실패 여부, 응답 코드, 메시지, 데이터 포함
    3. 제네릭을 활용하여 다양한 데이터 타입 반환 지원

- **예시**
  ```java
    @Getter
    @AllArgsConstructor
    public class ApiResponse<T> {
        private boolean success;
        private String message;
        private T data;

        public static <T> ApiResponse<T> success(T data) {
            return new ApiResponse<>(true, "OK", data);
        }

        public static <T> ApiResponse<T> fail(String message) {
            return new ApiResponse<>(false, message, null);
        }
    }
  ```

#### 2.7.3 ErrorCode
- **위치**: `common.error`
- **역할**:
    - 전역 에러 코드 관리
    - HTTP Status Code와 서비스 고유 코드를 함께 제공

- **설계 원칙**
    1. **HTTP Status Code 연동**
        - 모든 ErrorCode는 HTTP Status와 매핑
        - API 응답 시 `httpStatus`, `code`, `message` 함께 반환

    2. **서비스 코드 체계**
        - 유형별 접두사(prefix) + 3자리 숫자
        - 패턴: `{PREFIX}_{NNN}`
        - Prefix는 도메인/범주 기반으로 정의

          | Prefix | 설명 |
                  |--------|------|
          | `C`    | Common (공통/시스템 오류) |
          | `A`    | Authentication/Authorization (인증/인가) |
          | `U`    | User 도메인 |
          | `O`    | Order 도메인 |
          | `P`    | Payment 도메인 |

          예시:
            - `C001` : 잘못된 요청 (Bad Request)
            - `A001` : 인증 실패 (Unauthorized)
            - `U001` : 사용자 미존재
            - `O001` : 주문 미존재
            - `P001` : 결제 실패

    3. **메시지**
        - 사용자 친화적 메시지와 개발자용 상세 메시지 구분 가능
        - 기본적으로 `message`는 API 응답에 노출

- **예시**
    ```java
        @Getter
        @AllArgsConstructor
        public enum ErrorCode {

            // Common
            INVALID_REQUEST(HttpStatus.BAD_REQUEST, "C001", "잘못된 요청입니다."),
            INTERNAL_ERROR(HttpStatus.INTERNAL_SERVER_ERROR, "C999", "서버 내부 오류가 발생했습니다."),

            // Auth
            UNAUTHORIZED(HttpStatus.UNAUTHORIZED, "A001", "인증에 실패했습니다."),
            FORBIDDEN(HttpStatus.FORBIDDEN, "A002", "접근 권한이 없습니다."),

            // User
            USER_NOT_FOUND(HttpStatus.NOT_FOUND, "U001", "사용자를 찾을 수 없습니다."),
            USER_ALREADY_EXISTS(HttpStatus.CONFLICT, "U002", "이미 존재하는 사용자입니다."),

            // Order
            ORDER_NOT_FOUND(HttpStatus.NOT_FOUND, "O001", "주문 정보를 찾을 수 없습니다."),

            // Payment
            PAYMENT_FAILED(HttpStatus.BAD_REQUEST, "P001", "결제 처리에 실패했습니다.");

            private final HttpStatus httpStatus; // HTTP 상태 코드
            private final String code;           // 서비스 전역 에러 코드(유형별 + 숫자(3자리))
            private final String message;        // 사용자 메시지
        }
    ```

- **API 응답 예시**
    ```json
        {
            "success": false,
            "httpStatus": 404,
            "code": "U001",
            "message": "사용자를 찾을 수 없습니다."
        }
    ```

- **요약 규칙**
    - ErrorCode는 HTTP Status + 서비스 코드 체계 결합
    - 코드 체계: {Prefix}{3자리 숫자}
    - Prefix는 C(공통), A(인증/인가), U(User), O(Order), P(Payment) 등 도메인별 구분
    - 모든 에러 응답은 httpStatus, code, message를 포함

#### 2.7.4 Exception Handler
- **위치**: `common.exception`
- **역할**:
    - 전역 예외 처리
- **규칙**:
    1. @RestControllerAdvice 클래스 작성
    2. BusinessException 처리 → ApiResponse.fail() 반환
    3. 예상치 못한 예외는 INTERNAL_ERROR 코드로 응답

- **예시**
  ```java
    @RestControllerAdvice
    public class GlobalExceptionHandler {

        @ExceptionHandler(BusinessException.class)
        public ResponseEntity<ApiResponse<Void>> handleBusinessException(BusinessException ex) {
            return ResponseEntity.badRequest().body(ApiResponse.fail(ex.getMessage()));
        }

        @ExceptionHandler(Exception.class)
        public ResponseEntity<ApiResponse<Void>> handleException(Exception ex) {
            return ResponseEntity.internalServerError().body(ApiResponse.fail("시스템 오류가 발생했습니다."));
        }
    }
  ```

#### 2.7.5 Utils
- **위치**: `common.util`
- **역할**:
    - 공용 유틸리티 클래스
- **규칙**:
    1. 순수 기능만 포함 (DB 접근/비즈니스 로직 금지)
    2. `static` 메서드 형태로 작성
    3. 범용적으로 재사용 가능한 것만 포함

- **예시**
  ```java
    public final class DateUtils {
        public static String format(LocalDateTime dateTime) {
            return dateTime.format(DateTimeFormatter.ofPattern(GlobalConstants.DATE_FORMAT));
        }
    }
  ```

#### 2.7.6 Annotation
- **위치**: `common.annotation`
- **역할**:
    - 프로젝트 전역에서 사용하는 커스텀 어노테이션
- **규칙**:
    1. 로깅, 보안, 인증/인가와 관련된 공용 기능에 사용
    2. Runtime Retention 정책 사용

- **예시**
  ```java
    @Target(ElementType.METHOD)
    @Retention(RetentionPolicy.RUNTIME)
    public @interface LogExecutionTime {}
  ```

---

## 3. 네이밍 규칙

### 클래스
- Controller: `UserController`
- Service: `UserService`, `UserServiceImpl`
- Repository: `UserRepository`
- DTO: `UserRequestDto`, `UserResponseDto`
- Entity: `TUser`, `TOrder`
- Exception: `UserNotFoundException`

### 메서드
- CRUD: `findById`, `findAll`, `save`, `deleteById`
- 비즈니스 로직: `registerUser`, `loginUser`, `updatePassword`

### 패키지
- 소문자만 사용
- 복수형 지양 (`users` ❌ → `user` ✅)

---

## 4. 코드 스타일 규칙
- **들여쓰기**: 탭(크기: 4) 사용
- **라인 길이**: 120자
- **import**: 와일드카드 import 금지(`import java.util.*;` ❌)
- **중괄호**: 항상 한 줄 사용 (K&R 스타일)
- **어노테이션**:
    - 한 줄에 하나씩
    - 메서드 위 배치
- **null 비교**: `Objects.isNull(obj)` 또는 `obj == null`
- **Lombok 사용 시**
    - `@Getter`, `@Builder`, `@NoArgsConstructor` 권장
    - `@Data` 지양 (불필요한 Setter 자동생성 방지)

---

## 5. 로깅 규칙
- `slf4j` (`@Slf4j`) 사용
- `System.out.println` 금지
- 로그 레벨:
    - `DEBUG`: 개발 디버깅
    - `INFO`: 정상 흐름
    - `WARN`: 비정상 가능성
    - `ERROR`: 예외 발생
- 로그 포맷
    - `"[{traceId}] {action} - {details}"`

---

## 6. 테스트 규칙
- **위치**: `src/test/java/...`
- **네이밍**:
    - 단위 테스트: `{클래스명}Test`
    - 통합 테스트: `{클래스명}IntegrationTest`
- **JUnit5** 사용
- **Mockito**
    - 단위 테스트: `@ExtendWith(MockitoExtension.class)`
- **SpringBootTest**
    - 통합 테스트 전용
    - `@ActiveProfiles("test")` 필수
- Given-When-Then 패턴 권장
  ```java
    @Test
    void registerUser_shouldThrowException_whenEmailIsDuplicate() {
        // given
        
        // when
        
        // then
        
    }
  ```

---

## 7. DTO ↔ Entity 변환 규칙
- 변환 로직은 Mapper 계층에서 수행
- MapStruct 권장
  ```java
    @Mapper(componentModel = "spring")
    public interface UserMapper {
        UserResponseDto toDto(User user);
        User toEntity(UserRequestDto dto);
    }
  ```

## ~~8. 빌드 & 품질 관리(TBD)~~
- ~~**Gradle/Maven 플러그인**:~~
    - ~~`checkstyle` (코드 스타일 검사)~~
    - ~~`spotless` (자동 포맷팅)~~
    - ~~`jacoco` (테스트 커버리지)~~
- ~~**CI/CD**:~~
    - ~~pre-commit hook에서 `spotlessApply`~~
    - ~~PR 시 Code Style & Test 자동검증~~

---

## 9. Profile 관리 규칙

Spring Boot는 `application-{profile}.yml` 파일을 통해 환경별 설정을 분리합니다.  
팀 전체가 동일한 기준을 사용하여 운영 환경과 개발 환경을 명확히 구분합니다.

### 9.1 기본 원칙
- `application.yml`은 **공통 설정**만 포함 (DB 접속 정보, 외부 API 키 등 환경 의존적인 값은 제외)
- 환경별 설정은 `application-{profile}.yml`에 분리
- profile은 **소문자**만 사용

---

### 9.2 Profile 구분

- **local**
    - 개발자 PC 로컬 환경
    - H2 / Local DB, Mock API 서버 사용
    - 로그 레벨: `DEBUG`

- **dev**
    - 개발 서버 환경
    - 공용 개발 DB, Dev용 외부 API Key 사용
    - 로그 레벨: `DEBUG`
    - Swagger UI 활성화

- **staging**
    - 운영 전 리허설 환경
    - 운영과 동일한 DB 스키마, 외부 API Key 사용
    - 로그 레벨: `INFO`
    - 데이터는 운영과 유사하지만 별도 인프라 사용
    - 성능/부하 테스트에 사용

- **prod**
    - 운영 환경
    - 운영 DB, 운영 API Key
    - 로그 레벨: `WARN` 이상
    - Swagger UI 비활성화
    - 보안/모니터링 설정 강화

---

### 9.3 Profile 파일 구조 예시
```
src/main/resources/
├─ application.yml
├─ application-local.yml
├─ application-dev.yml
├─ application-staging.yml
└─ application-prod.yml
```

---

### 9.4 application.yml 예시
```yml
spring:
  application:
    name: my-service

---
spring:
  config:
    activate:
      on-profile: local
  datasource:
    url: jdbc:h2:mem:testdb
    username: sa
    password:
  jpa:
    hibernate:
      ddl-auto: update
  logging:
    level:
      root: DEBUG

---
spring:
  config:
    activate:
      on-profile: prod
  datasource:
    url: jdbc:postgresql://prod-db:5432/mydb
    username: ${DB_USER}
    password: ${DB_PASSWORD}
  jpa:
    hibernate:
      ddl-auto: validate
  logging:
    level:
      root: WARN
```

---

### 9.5 Profile 적용 방법

- **IntelliJ 실행/빌드 시:**
    - Run/Debug Configurations > Active Profiles 에서 지정

- **명령어 실행 시:**
    - ./gradlew bootRun --args='--spring.profiles.active=dev'
    - java -jar app.jar --spring.profiles.active=prod

---

### 9.6 Profile 적용 방법

- **운영/스테이징 설정은 절대 로컬에서 직접 사용 금지**
- **공통 설정은 application.yml에만 작성, 중복 금지**
- **민감한 정보는 .yml에 직접 기록하지 않고 환경 변수**

---