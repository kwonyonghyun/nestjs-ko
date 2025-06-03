### 프로바이더(Providers)

프로바이더는 Nest의 핵심 개념입니다. 서비스(services), 리포지토리(repositories), 팩토리(factories), 헬퍼(helpers) 등 많은 기본 Nest 클래스들이 프로바이더로 취급될 수 있습니다. 프로바이더의 핵심 아이디어는 의존성으로 주입될 수 있다는 것입니다. 이를 통해 객체들 간에 다양한 관계를 형성할 수 있습니다. 이러한 객체들의 "연결(wiring)" 작업은 대부분 Nest 런타임 시스템이 처리합니다.
![](https://docs.nestjs.com/assets/Components_1.png)

이전 챕터에서는 간단한 `CatsController`를 만들었습니다. 컨트롤러는 HTTP 요청을 처리하고, 더 복잡한 작업은 프로바이더에 위임해야 합니다. 프로바이더는 NestJS 모듈에서 프로바이더로 선언된 일반 JavaScript 클래스입니다. 자세한 내용은 "Modules" 챕터를 참고하세요.

💡 **힌트**
Nest는 객체지향 방식으로 의존성을 설계하고 구성할 수 있도록 지원하므로, **SOLID 원칙**을 따를 것을 강력히 추천합니다.

---

### 서비스(Services)

먼저 간단한 `CatsService`를 만들어 보겠습니다. 이 서비스는 데이터 저장 및 검색을 담당하며, `CatsController`에서 사용하게 됩니다. 애플리케이션의 로직을 관리하는 역할을 하기 때문에 프로바이더로 정의하기에 적합합니다.

#### cats.service.ts

```typescript
import { Injectable } from '@nestjs/common';
import { Cat } from './interfaces/cat.interface';

@Injectable()
export class CatsService {
  private readonly cats: Cat[] = [];

  create(cat: Cat) {
    this.cats.push(cat);
  }

  findAll(): Cat[] {
    return this.cats;
  }
}
```

💡 **힌트**
CLI로 서비스를 생성하려면 다음 명령어를 실행하세요:
`nest g service cats`

---

`CatsService`는 하나의 속성과 두 개의 메서드를 가진 기본 클래스입니다. 여기서 핵심은 `@Injectable()` 데코레이터입니다. 이 데코레이터는 클래스에 메타데이터를 추가하여 Nest IoC 컨테이너에서 관리할 수 있는 클래스임을 표시합니다.

추가로 이 예제에서는 `Cat` 인터페이스도 사용합니다. 예시는 다음과 같을 것입니다:

#### interfaces/cat.interface.ts

```typescript
export interface Cat {
  name: string;
  age: number;
  breed: string;
}
```

이제 고양이 목록을 조회할 수 있는 서비스 클래스가 생겼으니, 이를 `CatsController`에서 사용해보겠습니다:

#### cats.controller.ts

```typescript
import { Controller, Get, Post, Body } from '@nestjs/common';
import { CreateCatDto } from './dto/create-cat.dto';
import { CatsService } from './cats.service';
import { Cat } from './interfaces/cat.interface';

@Controller('cats')
export class CatsController {
  constructor(private catsService: CatsService) {}

  @Post()
  async create(@Body() createCatDto: CreateCatDto) {
    this.catsService.create(createCatDto);
  }

  @Get()
  async findAll(): Promise<Cat[]> {
    return this.catsService.findAll();
  }
}
```

---

### 의존성 주입(Dependency injection)

Nest는 강력한 설계 패턴인 **의존성 주입(Dependency Injection)** 을 기반으로 설계되었습니다. 이에 대한 훌륭한 글이 Angular 공식 문서에 있으니 참고해보세요.

Nest에서는 TypeScript의 기능 덕분에 의존성 관리를 매우 쉽게 할 수 있습니다. 타입에 기반하여 의존성이 해결되기 때문입니다. 아래 예제에서는 Nest가 `catsService` 의존성을 해결할 때 `CatsService` 인스턴스를 생성해서 주입합니다(싱글턴이라면 기존 인스턴스를 재사용).

컨트롤러의 생성자에 다음과 같이 주입됩니다:

```typescript
constructor(private catsService: CatsService) {}
```

---

### 스코프(Scopes)

프로바이더는 보통 애플리케이션 수명 주기와 동일한 수명(스코프)을 가집니다.
애플리케이션이 부트스트랩될 때 모든 의존성이 해결되고, 각 프로바이더가 인스턴스화됩니다.
애플리케이션 종료 시에는 모든 프로바이더가 소멸됩니다.

하지만 특정 요청(request)에만 스코프가 묶이도록 **요청 스코프(request-scoped)** 프로바이더를 만들 수도 있습니다.
이런 고급 내용은 "Injection Scopes" 챕터에서 자세히 배울 수 있습니다.

---

### 올바른 방법을 배우세요!

* **80개 이상의 챕터**
* **5시간 이상의 영상**
* **공식 인증서**
* **심화 세션**

👉 **공식 강의 코스를 살펴보세요.**

---

### 커스텀 프로바이더(Custom providers)

Nest에는 내장된 **IoC 컨테이너(Inversion of Control container)** 가 있어 프로바이더 간의 관계를 관리합니다.
이것이 의존성 주입의 기반이 되며, 실제로는 훨씬 더 강력한 기능을 제공합니다.

다양한 방식으로 프로바이더를 정의할 수 있습니다:
단순 값, 클래스, 비동기/동기 팩토리 등을 사용할 수 있습니다.

자세한 예시는 "Dependency Injection" 챕터에서 확인하세요.

---

### 선택적 프로바이더(Optional providers)

때로는 항상 해결되지 않아도 되는 의존성이 있을 수 있습니다. 예를 들어, 클래스가 설정 객체(configuration object)에 의존하지만, 설정이 없으면 기본값을 사용하도록 만들 수 있습니다.

이 경우 해당 의존성을 **선택적(optional)** 으로 표시할 수 있으며, 설정 프로바이더가 없어도 오류가 발생하지 않습니다.

생성자 시그니처에서 `@Optional()` 데코레이터를 사용합니다:

```typescript
import { Injectable, Optional, Inject } from '@nestjs/common';

@Injectable()
export class HttpService<T> {
  constructor(@Optional() @Inject('HTTP_OPTIONS') private httpClient: T) {}
}
```

위 예제에서는 커스텀 프로바이더를 사용하기 때문에 `'HTTP_OPTIONS'`라는 커스텀 토큰을 사용합니다.
앞선 예제들은 생성자 기반 주입(constructor-based injection)을 보여줬습니다.

커스텀 프로바이더와 토큰이 어떻게 작동하는지에 대한 자세한 내용은 "Custom Providers" 챕터를 참고하세요.

---

### 속성 기반 주입(Property-based injection)

지금까지 사용한 방식은 **생성자 기반 주입**(constructor-based injection)입니다.
그러나 특정 경우에는 **속성 기반 주입**(property-based injection)이 유용할 수 있습니다.

예를 들어, 상속 구조가 깊어 `super()` 호출을 통해 모든 의존성을 전달하기 번거로운 경우, 속성에 직접 `@Inject()` 데코레이터를 사용할 수 있습니다:

```typescript
import { Injectable, Inject } from '@nestjs/common';

@Injectable()
export class HttpService<T> {
  @Inject('HTTP_OPTIONS')
  private readonly httpClient: T;
}
```

⚠️ **경고**
클래스가 다른 클래스를 상속하지 않는 경우에는 **생성자 기반 주입**을 사용하는 것이 더 좋습니다.
생성자에서 필요한 의존성을 명확히 볼 수 있어 가독성이 높고 유지보수가 쉬워지기 때문입니다.

---

### 프로바이더 등록(Provider registration)

이제 `CatsService`(프로바이더)와 `CatsController`(컨슈머)를 정의했으니, Nest가 이를 인식하도록 등록해야 합니다.

이를 위해 `app.module.ts` 파일을 수정하여 `@Module()` 데코레이터의 `providers` 배열에 `CatsService`를 추가합니다:

#### app.module.ts

```typescript
import { Module } from '@nestjs/common';
import { CatsController } from './cats/cats.controller';
import { CatsService } from './cats/cats.service';

@Module({
  controllers: [CatsController],
  providers: [CatsService],
})
export class AppModule {}
```

이제 Nest는 `CatsController` 클래스의 의존성을 해결할 수 있게 됩니다.

---

### 디렉토리 구조

현재 디렉토리 구조는 다음과 같아야 합니다:

```
src
└── cats
    ├── dto
    │   └── create-cat.dto.ts
    ├── interfaces
    │   └── cat.interface.ts
    ├── cats.controller.ts
    ├── cats.service.ts
app.module.ts
main.ts
```

---

### 수동 인스턴스화(Manual instantiation)

지금까지 Nest가 자동으로 의존성 해결을 처리하는 방법을 살펴봤습니다.
그러나 경우에 따라 Nest의 내장 **DI 시스템을 우회**하여 프로바이더를 직접 가져오거나 인스턴스화해야 할 수도 있습니다.

두 가지 기술이 있습니다:

1️⃣ **Module reference 사용**
기존 인스턴스를 가져오거나 프로바이더를 동적으로 인스턴스화할 때 사용합니다.

2️⃣ **bootstrap() 함수 내에서 사용**
(예: 독립 실행형 애플리케이션에서 또는 부트스트랩 과정에서 설정 서비스를 사용하려는 경우)
자세한 내용은 "Standalone applications" 챕터를 참고하세요.

