# 고급 다이어그램 기법

## 복합 다이어그램 전략

시스템 문서화 시 다이어그램을 단계적으로 조합한다:

```
1. C4 Context      → 전체 그림 (누가 무엇과 통신하는지)
2. C4 Container    → 시스템 내부 구조 (어떤 서비스가 있는지)
3. Sequence        → 핵심 흐름 (어떤 순서로 동작하는지)
4. Class / ER      → 상세 구조 (데이터가 어떻게 생겼는지)
5. State           → 상태 변화 (엔티티가 어떻게 변하는지)
6. Flowchart       → 비즈니스 규칙 (분기 조건이 무엇인지)
```

---

## 고급 Flowchart 기법

### Subgraph + 스타일링

```mermaid
flowchart TB
    subgraph client["Client Layer"]
        direction LR
        WEB[Web App]
        MOBILE[Mobile App]
    end

    subgraph gateway["API Gateway"]
        GW[Kong Gateway]
    end

    subgraph services["Microservices"]
        direction LR
        ORDER[Order Service]
        USER[User Service]
        PRODUCT[Product Service]
    end

    subgraph data["Data Layer"]
        direction LR
        PG[(PostgreSQL)]
        REDIS[(Redis)]
        ES[(Elasticsearch)]
    end

    client --> gateway
    gateway --> services
    ORDER --> PG
    USER --> PG
    PRODUCT --> ES
    ORDER --> REDIS

    classDef clientStyle fill:#E3F2FD,stroke:#1565C0
    classDef gwStyle fill:#FFF3E0,stroke:#E65100
    classDef svcStyle fill:#E8F5E9,stroke:#2E7D32
    classDef dataStyle fill:#F3E5F5,stroke:#6A1B9A

    class WEB,MOBILE clientStyle
    class GW gwStyle
    class ORDER,USER,PRODUCT svcStyle
    class PG,REDIS,ES dataStyle
```

### 스타일 정의 문법

```
classDef 스타일명 fill:#색상,stroke:#색상,stroke-width:2px,color:#텍스트색
class 노드1,노드2 스타일명
```

주요 속성: `fill`(배경), `stroke`(테두리), `stroke-width`(두께), `color`(텍스트), `stroke-dasharray`(점선)

### Subgraph 방향 제어

```mermaid
flowchart LR
    subgraph A["수직 배치"]
        direction TB
        A1 --> A2 --> A3
    end
    subgraph B["수평 배치"]
        direction LR
        B1 --> B2 --> B3
    end
    A --> B
```

`direction` 키워드로 subgraph 내부 방향을 독립 제어 가능

---

## 고급 Sequence Diagram 기법

### Critical / Break / Rect

```mermaid
sequenceDiagram
    participant C as Client
    participant A as API
    participant D as DB

    rect rgb(240, 248, 255)
        Note over C,A: 인증 구간
        C->>A: 로그인 요청
        A-->>C: JWT 토큰
    end

    C->>A: 주문 생성

    critical 데이터 정합성 확보
        A->>D: BEGIN TRANSACTION
        A->>D: INSERT order
        A->>D: UPDATE inventory
        A->>D: COMMIT
    option 재고 부족
        A->>D: ROLLBACK
        A-->>C: 409 Conflict
    end

    break 서비스 점검 중
        A-->>C: 503 Service Unavailable
    end
```

### 병렬 + 중첩

```mermaid
sequenceDiagram
    participant Client
    participant API
    participant Cache
    participant DB

    Client->>API: GET /product/123

    alt 캐시 히트
        API->>Cache: GET product:123
        Cache-->>API: 데이터
        API-->>Client: 200 OK
    else 캐시 미스
        API->>Cache: GET product:123
        Cache-->>API: null

        par DB 조회 + 캐시 갱신
            API->>DB: SELECT * FROM product
            DB-->>API: 데이터
        and
            API->>Cache: SET product:123
        end

        API-->>Client: 200 OK
    end
```

### 참가자 별칭 및 Actor 타입

```
actor User                          # 사람 아이콘
participant API as "API Server"     # 별칭 사용
participant DB as "Database"
create participant Worker           # 동적 생성
destroy Worker                      # 동적 제거
```

---

## 고급 C4 기법

### C4 Deployment (인프라 구성)

```mermaid
C4Deployment
    title 프로덕션 배포 다이어그램

    Deployment_Node(aws, "AWS", "Cloud Provider") {
        Deployment_Node(region, "ap-northeast-2", "Seoul Region") {
            Deployment_Node(vpc, "VPC") {
                Deployment_Node(alb, "ALB", "Application Load Balancer") {
                    Container(lb, "Load Balancer", "AWS ALB")
                }
                Deployment_Node(ecs, "ECS Cluster", "Fargate") {
                    Container(api1, "API Instance 1", "Spring Boot")
                    Container(api2, "API Instance 2", "Spring Boot")
                }
                Deployment_Node(rds, "RDS", "Multi-AZ") {
                    ContainerDb(primary, "Primary", "PostgreSQL 15")
                    ContainerDb(replica, "Read Replica", "PostgreSQL 15")
                }
            }
            Deployment_Node(elasticache, "ElastiCache") {
                ContainerDb(redis, "Redis Cluster", "Redis 7")
            }
        }
    }

    Rel(lb, api1, "HTTP")
    Rel(lb, api2, "HTTP")
    Rel(api1, primary, "JDBC")
    Rel(api2, replica, "JDBC (read)")
    Rel(api1, redis, "Redis Protocol")
```

### C4 관계 스타일링

```
Rel(from, to, "라벨", "프로토콜")
BiRel(a, b, "양방향 통신")
UpdateRelStyle(from, to, $textColor="blue", $lineColor="blue", $offsetY="-10")
```

---

## 고급 State Diagram 기법

### 병렬 상태 (Concurrent States)

```mermaid
stateDiagram-v2
    [*] --> Active

    state Active {
        [*] --> Working

        state Working {
            [*] --> Coding
            Coding --> Testing : 코드 완성
            Testing --> Coding : 버그 발견
            Testing --> [*] : 테스트 통과
        }

        --

        state Monitoring {
            [*] --> Watching
            Watching --> Alerting : 이상 감지
            Alerting --> Watching : 복구
        }
    }

    Active --> [*] : 종료
```

`--` 구분선으로 병렬 영역을 분리

### Choice / Fork / Join

```mermaid
stateDiagram-v2
    state check <<choice>>
    state fork_state <<fork>>
    state join_state <<join>>

    [*] --> check
    check --> Approved : 승인됨
    check --> Rejected : 거부됨

    Approved --> fork_state
    fork_state --> SendEmail
    fork_state --> UpdateDB
    fork_state --> NotifySlack

    SendEmail --> join_state
    UpdateDB --> join_state
    NotifySlack --> join_state
    join_state --> [*]

    Rejected --> [*]
```

---

## 고급 Class Diagram 기법

### 네임스페이스 / 패키지

```mermaid
classDiagram
    namespace Domain {
        class Order {
            -Long id
            -OrderStatus status
            +cancel() void
        }
        class OrderStatus {
            <<enumeration>>
            CREATED
            PAID
            SHIPPED
            CANCELLED
        }
    }

    namespace Application {
        class OrderService {
            +createOrder(dto) Order
            +cancelOrder(id) void
        }
        class OrderMapper {
            +toEntity(dto) Order
            +toDto(entity) OrderDto
        }
    }

    namespace Infrastructure {
        class JpaOrderRepository {
            +save(order) Order
            +findById(id) Order
        }
    }

    OrderService ..> Order
    OrderService ..> OrderMapper
    JpaOrderRepository ..|> OrderPort
    Order --> OrderStatus
```

### 제네릭과 추상 클래스

```
class Repository~T~ {
    <<abstract>>
    +save(entity T) T
    +findById(id Long) T
    +delete(entity T) void
}
```

`~T~`로 제네릭 표현 (꺾쇠 대신 물결표)

---

## 테마와 스타일링

### 전역 테마 설정

```mermaid
%%{init: {'theme': 'neutral'}}%%
flowchart TD
    A --> B --> C
```

사용 가능 테마: `default`, `neutral`, `dark`, `forest`, `base`

### 커스텀 설정

```mermaid
%%{init: {
    'theme': 'base',
    'themeVariables': {
        'primaryColor': '#4CAF50',
        'primaryTextColor': '#fff',
        'primaryBorderColor': '#2E7D32',
        'lineColor': '#666',
        'secondaryColor': '#E3F2FD',
        'tertiaryColor': '#FFF3E0'
    }
}}%%
flowchart TD
    A[서비스 A] --> B[서비스 B]
    B --> C[서비스 C]
```

### 개별 노드 스타일

```
style 노드ID fill:#색상,stroke:#색상,stroke-width:2px
```

### 링크(화살표) 스타일

```
linkStyle 0 stroke:#ff0000,stroke-width:2px
linkStyle default stroke:#333,stroke-width:1px
```

`linkStyle N`에서 N은 화살표의 순서 (0부터 시작, 등장 순서대로)

---

## 다이어그램 크기 관리 팁

1. **한 다이어그램에 노드 15개 이하** 유지 - 초과 시 분할
2. **subgraph로 시각적 그루핑** - 관련 노드를 묶어 가독성 확보
3. **상세도는 단계적으로** - C4 Context → Container → Component 순으로 확대
4. **레이블은 간결하게** - 노드에 상세 설명 대신 핵심 키워드만
5. **방향은 일관되게** - 한 다이어그램 내 데이터 흐름 방향 통일
