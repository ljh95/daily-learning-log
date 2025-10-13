출처: https://lapidix.dev/posts/fsd-ddd-clean-architecture

## 1. 현대 프론트엔드 개발의 복잡성 문제와 새로운 접근법

프론ㅌ엔드 아키텍처 방법론인 FSD를 구조적 기반으로 삼고,
여기에 DD의 모델링 기법과 클린 아키텍처의 철학을 통합하는 고도화된 아키텍처 모델을 제안함

이 3가지 패턴을 유기적으로 결합함으로써 얻는 시너지는 명확함
-> FSD가 제공하는 예측 가능한 구조 위에 DDD로 비즈니스 복잡성을 정밀하게 모델링하고, 클린 아키텍처의 의존성 규칙으로 핵심 비즈니스 로직을 외부 기술 변화로부터 완벽하게 보호하여
구조적 명확성.. 등을 확보하는것

## 2. 핵심 아키텍처 패턴 분석: FSD, DDD, 클린 아키텍처

### 2.1 FSD: 예측 가능한 프론트엔드 구조 설계

FSD는 프론트엔드 애플리케이션의 복잡도 증가 문제를 체계적으로 해결하기 위한 설계 방법론
app, pages, widgets, features, entities, shared의 레이어로 나눔

- entities: 핵심 비즈니스 개체(예: User, Post)와 관련된 데이터 및 UI를 정의하는 레이어
- features: 사용자 상호작용을 포함하는 구체적인 비즈니스 기능(예: 게시글 작성)

FSD는 2가지 핵심 원칙을 통해 구조적 안정성을 보장

- 단방향 의존성 규칙
  - 항상 상위 레이어에서 하위 레이어로만 의존성이 흐름
  - 이 원칙은 순환 참조를 원천적으로 방지, 코드베이스를 예측 가능하고 안정적으로 만듬
  - 특정 기능의 변경이 다른 기능에 미치는 영향을 최소화함
- Public API와 슬라이스 격리
  - 각 슬라이스는 외부로 노출할 모듈을 indes.ts파일(Barrerl Export)을 통해 명시적으로 정의함
  - 외부 모듈을 오직 이 Public API를 통해서만 해당 슬라이스에 접근할 수 있음
  - 이 엄격한 계약은 모듈들이 예측 불가능한 방식으로 결합되는 스파게티 의존성의 출현을 방지
  - 리팩토링과 테스트를 용이하게 함

### 2.2 DDD: 비즈니스 복잡성 모델링

### 2.3 클린 아키텍처: 비즈니스 로직 보호

## 3. 통합 아키텍처 모델: FSD 구조와 클린 아키텍처의 결합

FSD의 계층적 폴더 구조를 클링 아키텍처의 동신원 모델에 매핑하고
그 안에서 DDD의 전술적 패턴을 활용해서 비즈니스 로직을 견고하게 구축하는 방법을 제안

### 3.1 아키텍처 매핑: FSD 레이어와 클린 아키텍처 동심원 구조의 연결

- Enterprice Business Rules(최내부): 아키텍처의 가장 심장부에는 순수 비즈니스 정채깅 자리함, FSD의 entities슬라이스 내 core 디렉토리에 위치하는 Entities Domain Core가 여기에 해당, DDD의 Entity, Value Object 등 순수 도메인 모델을 정의함. 또한 데이터 영속성을 위한 추상화 계층인 Entities Repository(Interface)도 이곳ㅇ에 정의, 이 인터페이스는 도메인이 "무엇을" 필요로 하는지만 기술하면, "어떻게" 데이터를 가져올지에 대한 구체적인 방법은 알지 못함
- Application Business Rules: 최내부 레이어를 감싸는 이 계층은 애플리케이션의 구체적인 기능, 즉 유스케이스를 정의함. FSD의 features 슬라이스에 위치하는 Features SErvice(Implementation)이 이 역할을 함. 이 서비스는 Repository 인터페이스를 주입받아 도메인 객체를 조율하고, 애플리케이션의 핵심 비즈니스 흐름을 처리함
- Interface Adapters: 이 계층은 내부 비즈니스 로직과 외부 세계 사이의 데이터 변환을 책임지는 어댑터들의 집합. 여기서 클린 아키텍처의 제어의 역전 원칙이 물리적으로 실현됨. Enterprice Business Rules 계층에 정의된 Entities Repository의 구체적인 구현체가 FSD의 entities 슬라이스 내 infrastructure 디렉토리에 위치함. 또한 UI와 유스케이스를 연결하는 FEatures Hooks역시 이 계층에 속하며, 프레임워크 종속적인 코드를 비즈니스 로직으로부터 격리함.
- Frameworks & Drivers(최외부): 가장 바깥쪽 레이어는 웹 프레임워크, UI 컴포넌트 라이블러ㅣ, 외부 API 서버 등 변동성이 가장 높은 구체적인 기술 구현체들로 구성됨. FSD 상위 레이어인 App, Pages, Widgets 가 여기에 해당, 사용자와 직접 상호작용하는 모든 세부사항을 포함함

### 3.2 의존성 흐름과 제어의 역전

## 4. 구현 전략 및 코드 예시 분석

### 4.1 DDD 전술적 패턴의 적용

```ts
// /src/entities/user/core/user.domain.ts

export class User implements UserEntity {
  constructor(
    private _id: string,
    private _username: string,
    private _email: string
  ) {}

  updateUsername(newUsername: string) {
    if (!this.isValidUsername(newUsername)) {
      throw BaseError.validation("Invalid username format");
    }
    this._username = newUsername;
  }

  private isValidUsername(username: string): boolean {
    return /^[a-zA-Z0-9_.]{3,20}$/.test(username);
  }
}
```

- updateUsername 메서드는 유효성 검증 로직을 포함하여 User 객체의 상태가 비즈니스 규칙에 어긋나는 것을 원천적으로 방지합니다.
- 이 설계는 비즈니스 로직이 여러 서비스나 컴포넌트에 흩어지는 것을 막고
- 규칙 변경 시 해당 Entity만 수정하면 되므로 코드의 안전성과 유지보수성이 크게 향상됨

Value Object
원칙: Value Object 패턴은 원시 타입을 비즈니스 개념을 담은 불변 객체로 대체하여 타입 안정성과 도메인 표현력을 높입니다. 증거: comment-body.vo.ts는 댓글 본문을 단순 문자열이 아닌, 생성 시점에 유효성 검증이 완료된 불변 객체로 다룹니다.

```ts
// /src/shared/domain/value-object.ts (추상 클래스)
export abstract class ValueObject<T> {
  protected readonly _value: T;
  constructor(value: T) {
    this.validate(value);
    this._value = this.deepFreeze(value); // 불변성 보장
  }
  // ...
}

// /src/entities/comment/value-objects/comment-body.vo.ts
export class CommentBody extends ValueObject<string> {
  private static readonly MAX_LENGTH = 100;

  protected validate(value: string): void {
    if (!value || !value.trim()) {
      throw new Error("Comment body cannot be empty");
    }
    if (value.length > CommentBody.MAX_LENGTH) {
      throw new Error(
        `Comment body cannot exceed ${CommentBody.MAX_LENGTH} characters`
      );
    }
  }
}
```

- ValueObject 추상 클래스는 deepFreeze를 통해 완벽한 불변성을 보장하며
- CommentBody는 생성자에서 비즈니스 규칙(최대 길이, 빈 값 금지)을 강제합니다.
- 이로써 CommentBody 타입의 객체는 항상 유효한 상태임을 보장받을 수 있어

Factory

- 원칙: Factory 패턴은 복잡한 객체 생성 로직을 캡슐화하여 클라이언트 코드의 책임과 복잡도를 줄입니다.
- 증거: post.factory.ts는 새로운 게시글 생성(createNew)과
- 외부 데이터로부터 도메인 객체를 복원(createFromDto)하는
- 두 가지 상이한 생성 로직을 중앙에서 관리합니다.

```ts
// /src/entities/post/core/post.factory.ts
export class PostFactory {
  static createNew(
    title: string,
    body: string,
    user: UserReference,
    image: string = ""
  ): Post {
    return new Post(
      "", // ID는 서버에서 할당
      user,
      title,
      body,
      image,
      0,
      0, // 좋아요, 댓글 수 초기화
      new Date().getTime(),
      new Date().getTime() // 생성, 수정 시간 설정
    );
  }

  static createFromDto(dto: PostDto): Post {
    return new Post(
      dto.id,
      dto.user,
      dto.title,
      dto.body,
      dto.image || "",
      dto.likes || 0, // 기본값 처리
      dto.totalComments || 0,
      dto.createdAt || new Date().getTime(),
      dto.updatedAt || new Date().getTime()
    );
  }
}
```

- createNew은 새 게시글의 초기 상태를 정의하는 도메인 지식을
- createFromDto는 외부 데이터의 불완전성을 처리하는 방어적 로직을
- Factory를 사용하면 객체 생성 로직이 애플리케이션 전반에 흩어지는 것을 방지하고
- 관련 비즈니스 규칙 변경 시 Factory만 수정하면 되므로 코드의 일관성과 유지보수성이 향상됩니다

Repository (Interface)

- 원칙: Repository 인터페이스는 데이터 영속성 메커니즘을 추상화하여 도메인 계층을 인프라스트럭처로부터 격리시킵니다.
- 증거: comment.repository.ts는 데이터 접근 방식을 CRUD를 넘어 비즈니스 관점에서 정의합니다.

```ts
// /src/entities/comment/core/comment.repository.ts
export interface CommentRepository {
  getByPostId(postId: string): Promise<Comment[]>;
  create(comment: Comment): Promise<Comment>;
  // ...
  like(id: string, userId: string): Promise<boolean>;
  unlike(id: string, userId: string): Promise<boolean>;
}
```

- like, unlike와 같이 비즈니스 의미가 담긴 메서드를 정의함으로써, 도메인 로직의 순수성을 지킵니다
- Repository 인터페이스는 도메인 계층이 실제 데이터가 API에서 오는지, 로컬 스토리지에서 오는지를 알 필요 없게 만들어

### 4.2 클린 아키텍처 레이어별 구현

Application Layer (Use Case)

- 원칙: Application Layer는 Repository 인터페이스에 의존하여 도메인 객체를 조율하고, 특정 유스케이스를 수행합니다.
- 증거: post.service.ts의 updatePost 메서드는 게시글 수정 유스케이스를 구현하며, PostRepository 인터페이스에만 의존합니다.

```ts
// /src/features/post/services/post.service.ts
export const PostService = (
  postRepository: PostRepository // 인터페이스에 의존
  // ...
): PostUseCase => ({
  updatePost: async (
    id: string,
    title: string,
    body: string
  ): Promise<PostDto> => {
    try {
      // 1. Repository를 통해 기존 Entity 조회
      const existingPost = await postRepository.getById(id);
      if (!existingPost) {
        throw BaseError.notFound("Post", id);
      }
      // 2. Entity의 비즈니스 로직 호출
      existingPost.updateTitle(title);
      existingPost.updateBody(body);
      // 3. 변경된 Entity를 Repository를 통해 영속화
      const updatedPost = await postRepository.update(existingPost);
      // ...
      return PostMapper.toDto(updatedPost);
    } catch (error) {
      /* ... */
    }
  },
});
```

- updatePost 메서드는 유스케이스의 전형적인 흐름(조회 → 도메인 로직 실행 → 영속화)을 명확히 보여줍니다.
- 의존성 주입을 통해 PostRepository의 구체적인 구현을 알지 못하므로,
- 테스트 시에는 Mock Repository를 주입하여 비즈니스 로직만을 독립적으로 검증할 수 있어 높은 테스트 용이성을 보장합니다.

Infrastructure & Adapters Layer

- 원칙: Infrastructure와 Adapters 레이어는 인터페이스를 구현하고 데이터 변환을 책임져 외부 세계와의 상호작용을 캡슐화합니다.
- 증거: post.api.repository.ts는 PostRepository 인터페이스를 구현하여 실제 API와 통신하고,
- post.mapper.ts는 DTO와 도메인 모델 간의 변환을 담당합니다.

```ts
// /src/entities/post/infrastructure/repository/post.api.repository.ts
export class PostApiRepository implements PostRepository {
  private api: ReturnType<typeof PostAdapter>;
  constructor(apiClient: ApiClient) {
    this.api = PostAdapter(apiClient);
  }

  async getById(id: string): Promise<Post> {
    const postDto = await this.api.getById(id);
    // 외부 API 응답(DTO)을 도메인 모델로 변환
    return PostMapper.toDomain(postDto);
  }
  // ...
}

// /src/entities/post/mapper/post.mapper.ts
export class PostMapper {
  static toDomain(dto: PostDto): Post {
    return new Post({
      /* ... DTO to Domain mapping ... */
    });
  }

  static toCreateDto(post: Post): CreatePostDto {
    return {
      /* ... Domain to DTO mapping ... */
    };
  }
}
```

- PostApiRepository는 외부 API 통신을,
- PostMapper는 데이터 변환을 각각 책임짐으로써
- 만약 백엔드 API의 응답 형식이 변경되더라도, PostMapper만 수정하면 되므로
-

## 5. 실용적 관점에서의 고찰 및 트레이드오프

### 5.1 모든 데이터를 도메인 모델로 변환해야 하는가?

- 결정 지점:
  - API로부터 받은 모든 데이터를 도메인 객체로 변환할 것인가,
  - 아니면 필요한 경우 DTO를 직접 사용할 것인가?
- 상충하는 요소:

  - 아키텍처 일관성(Architectural Consistency) vs.
  - 실용적 성능(Pragmatic Performance).

- 분석:
  - 게시글 목록처럼 단순히 화면에 나열하기만 하는 데이터까지 도메인 객체로 변환했다가 다시 UI용 데이터로 변환하는 과정은 명백한 오버헤드를
  - 반면, updateTitle()과 같은 비즈니스 메서드가 필요한 게시글 상세 정보의 경우
  - 도메인 객체는 관련 로직을 캡슐화하여 코드의 응집도를 높이는 강력한 이점을 제공합니다.
- 결론:
  - 실무에서는 데이터의 사용 목적에 따라 접근 방식을 달리하는 것이 합리적입니다.
  - 비즈니스 로직이 필요한 데이터는 도메인 객체로 변환하고
  - 단순 조회용 데이터는 DTO를 직접 사용하는 것이 실용적인 선택입니다
  - 본 백서에서는 아키텍처 패턴의 일관성을 명확히 보여주기 위한 실험적 목적으로 모든 데이터를 도메인 객체로 변환하는 방식을 채택했습니다.

### 5.2 Value Object는 어디까지 적용하는 것이 적절한가?

- 결정 지점:
  - 단순한 유효성 검증이 필요한 모든 원시 타입 필드를 Value Object(VO)로 모델링할 것인가?
- 상충하는 요소:

  - 패턴의 순수성(Pattern Purity) vs. 개발 속도(Development Velocity).

- 분석:
  - 댓글 본문을 나타내는 CommentBody VO처럼, 단순 문자열 검증을 위해 클래스를 만드는 것은 초기 개발 단계에서 과도한 추상화로 느껴질 수 있습니다
  - 하지만 TDD(Test-Driven Development)나 장기적 확장성 관점에서 보면,
  - VO는 해당 비즈니스 규칙을 독립적으로 테스트하기 용이하게 하고
  - 도메인 규칙을 한곳에 모아 코드의 응집도와 유지보수성을 높이는 장점이 있습니다.
- 결론:
  - 테스트 전략과 도메인 확장 가능성을 중요하게 고려한다면,
  - 핵심적인 비즈니스 개념을 나타내는 필드에 VO 패턴을 적극적으로 도입하는 것은 가치 있는 투자입니다
  - 그러나 모든 필드에 무분별하게 적용하기보다는
  - 도메인에서 중요한 의미를 갖거나
  - 복잡한 유효성 검증이 필요한 경우에 선택적으로 적용하는 것이 바람직합니다.

### 5.3 Hooks Facotry 패턴은 과도한 추상화인가?

- 결정 지점:
  - UI와 비즈니스 로직을 연결하는
  - Adapter인 React Custom Hooks에
  - 일관성을 위해 팩토리 패턴을 적용할 것인가?
- 상충하는 요소:

  - 설계의 일관성(Design Consistency) vs. 구현의 단순성(Implementation Simplicity).

- 분석:
  - 비즈니스 로직 레이어까지 적용된 팩토리 패턴을
  - Adapter 계층인 Hooks에도 확장하여 전체적인 설계의 일관성을 유지하려는 시도는
  - 하지만 실제 구현 결과
  - 훅은 각 서비스를 직접 의존하여 구현하는 방식이 복잡도를 낮추면서도 Adapter로서의 역할을 충분히 수행할 수 있었습니다.
- 결론:
  - Hooks에 대한 추가적인 추상화(팩토리 패턴)는
  - 실질적인 이점보다 복잡성을 더 많이 증가시키는 것으로 판단했습니다
  - 아키텍처 패턴은 맹목적으로 적용하는 것이 아니라, 각 계층의 특성과 실용성을 고려하여 필요한 부분에만 선택적으로 적용하는 것이 더 효과적입니다.

### 5.4 Repository 의존성 주입은 프론트엔드에 과한가?

- 결정 지점:
  - Service가 Repository를 직접 import할 것인가,
  - 아니면 의존성 주입(DI)을 통해 받을 것인가?
- 상충하는 요소:

  - 즉각적인 편의성(Immediate Convenience) vs. 장기적인 유연성(Long-term Flexibility)

- 분석:
  - 직접 import 방식은 코드 작성이 간단하고 직관적입니다.
  - 반면, 의존성 주입(DI) 방식은 초기 설정이 다소 복잡하게 느껴질 수 있습니다
  - 그러나 DI는 구현체 교체의 용이성과 테스트 유연성이라는 강력한 장점을 제공합니다.
  - 더 나아가 이는 장기적인 전략적 옵션을 확보하는 수단이 됩니다.
  - 예를 들어, DI를 통해 데이터 페칭 라이브러리를 REST 클라이언트에서 GraphQL로 원활하게 전환하거나,
  - 성능 최적화를 위해 캐싱 Repository를 비즈니스 로직 수정 없이 추가할 수 있습니다.
- 결론:
  - DI는 단순한 기술적 선택을 넘어, 장기적인 유지보수성과 확장성을 보장하는 전략적 아키텍처 결정입니다.
  - 프로젝트의 규모가 크고 장기적으로 운영될 예정이라면, 초기 복잡성을 감수하더라도 DI를 도입하는 것이 현명한 투자입니다.

### 5.5 프론트엔드에서 Aggregate 패턴은 필요한가?

- 결정 지점:
  - 프론트엔드에서 데이터 일관성 보장을 위해 Aggregate 패턴을 도입할 것인가?
- 상충하는 요소:

  - DDD 원칙 준수(Adherence to DDD Principles) vs. 책임 분리(Separation of Concerns).

- 분석:
  - DDD에서는 데이터 일관성 보장을 위해
  - 연관된 객체들을 Aggregate로 묶어 관리합니다.
  - 그러나 프론트엔드의 주된 책임은 데이터의 표현이며,
  - 데이터의 최종적인 일관성을 보장하는 책임은 대부분 서버에 있습니다.
  - 프론트엔드에서 Aggregate를 도입하여 복잡한 일관성 규칙과 트랜잭션을 직접 관리하는 것은
  - 서버의 책임을 침범하는 것이며, 불필요한 복잡성을 야기합니다.
- 결론:
  - 프론트엔드에서는 Aggregate 패턴을 도입하는 대신,
  - 유스케이스(Service) 레벨에서
  - 여러 Repository를 조합하여 비즈니스 로직을 처리하는 수준으로 충분합니다.
  - 복잡한 데이터 정합성 관리는
  - 서버에 위임하는 것이 명확한 책임 분리 원칙에 부합하는 현실적인 선택입니다.

## 6. 결론: 아키텍처의 본질과 사고의 확장

## 7. 참고 자료
