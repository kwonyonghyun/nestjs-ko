### 모듈(Modules)

**모듈(Module)** 은 `@Module()` 데코레이터가 적용된 클래스입니다.
이 데코레이터는 Nest가 **애플리케이션 구조를 효율적으로 구성하고 관리**하기 위한 메타데이터를 제공합니다.

---

모든 Nest 애플리케이션에는 최소한 **루트 모듈(root module)** 이 하나 존재합니다.
루트 모듈은 Nest가 **애플리케이션 그래프(application graph)** 를 구축하기 위한 시작점입니다.

**애플리케이션 그래프** 는 Nest가 **모듈과 프로바이더 간의 관계 및 의존성**을 해결하기 위해 내부적으로 사용하는 구조입니다.

---

작은 애플리케이션은 루트 모듈만으로 충분할 수 있지만, 일반적으로는 **여러 개의 모듈**을 사용하는 것이 권장됩니다.
모듈은 컴포넌트를 **효과적으로 구성**할 수 있는 강력한 도구입니다.
대부분의 애플리케이션에서는 서로 밀접하게 관련된 기능들을 **별도의 모듈로 캡슐화**하게 됩니다.

---

`@Module()` 데코레이터는 하나의 객체를 인자로 받으며, 해당 모듈을 설명하는 속성들을 포함합니다:

| 속성          | 설명                                                                           |
| ----------- | ---------------------------------------------------------------------------- |
| providers   | 이 모듈에서 Nest 인젝터에 의해 인스턴스화될 프로바이더 목록 (적어도 이 모듈 내에서 공유 가능)                     |
| controllers | 이 모듈에서 정의된 컨트롤러 목록 (인스턴스화 필요)                                                |
| imports     | 이 모듈에서 필요한 프로바이더를 export 하는 다른 모듈들의 목록                                       |
| exports     | 이 모듈이 다른 모듈에서 사용할 수 있도록 export 하는 프로바이더들의 부분 집합 (프로바이더 자체 또는 토큰으로 export 가능) |

---

**모듈은 기본적으로 프로바이더를 캡슐화**합니다.
따라서 현재 모듈 내의 프로바이더나 명시적으로 export 된 프로바이더만 주입할 수 있습니다.
**export 된 프로바이더** 는 해당 모듈의 **공개 인터페이스(public API)** 역할을 합니다.

---

### 기능 모듈(Feature modules)

예제에서 **CatsController** 와 **CatsService** 는 서로 밀접하게 관련된 기능을 담당합니다.
따라서 이를 **기능 모듈(feature module)** 로 그룹화하는 것이 좋습니다.

기능 모듈은 특정 기능과 관련된 코드를 모아서 **명확한 경계**를 유지하고 **구조적인 조직화**를 돕습니다.
이는 애플리케이션이나 팀 규모가 커질수록 특히 중요하며, **SOLID 원칙**에도 부합합니다.

---

다음으로 **CatsModule** 을 만들어 컨트롤러와 서비스를 그룹화해보겠습니다:

#### cats/cats.module.ts

```typescript
import { Module } from '@nestjs/common';
import { CatsController } from './cats.controller';
import { CatsService } from './cats.service';

@Module({
  controllers: [CatsController],
  providers: [CatsService],
})
export class CatsModule {}
```

💡 **힌트**
CLI 를 사용해서 모듈을 만들려면:

```bash
$ nest g module cats
```

---

위에서 우리는 **cats.module.ts** 파일에 **CatsModule** 을 정의했고,
cats 디렉토리로 관련 파일을 이동했습니다.

이제 마지막으로 루트 모듈인 **AppModule** 에서 이 모듈을 import 해야 합니다:

#### app.module.ts

```typescript
import { Module } from '@nestjs/common';
import { CatsModule } from './cats/cats.module';

@Module({
  imports: [CatsModule],
})
export class AppModule {}
```

---

이제 디렉토리 구조는 다음과 같습니다:

```
src
├── cats
│   ├── dto
│   │   └── create-cat.dto.ts
│   ├── interfaces
│   │   └── cat.interface.ts
│   ├── cats.controller.ts
│   ├── cats.module.ts
│   ├── cats.service.ts
├── app.module.ts
└── main.ts
```

---

### 공유 모듈(Shared modules)

Nest에서는 **모듈이 기본적으로 싱글턴(singleton)** 입니다.
따라서 **여러 모듈 간에 동일한 프로바이더 인스턴스를 쉽게 공유**할 수 있습니다.

---

모든 모듈은 자동으로 **공유 가능**하며, 다른 모듈에서 재사용할 수 있습니다.

예를 들어, **CatsService** 를 여러 다른 모듈에서 공유하고 싶다면,
먼저 해당 모듈의 `exports` 배열에 추가해야 합니다:

#### cats.module.ts

```typescript
import { Module } from '@nestjs/common';
import { CatsController } from './cats.controller';
import { CatsService } from './cats.service';

@Module({
  controllers: [CatsController],
  providers: [CatsService],
  exports: [CatsService],
})
export class CatsModule {}
```

---

이제 **CatsModule** 을 import 하는 다른 모듈들은
**CatsService** 에 접근할 수 있으며, **같은 인스턴스**를 공유합니다.

---

만약 각 모듈에서 CatsService 를 직접 등록한다면:

```typescript
providers: [CatsService]
```

→ 모듈마다 **별도의 인스턴스**가 생성됩니다.
이는 **메모리 사용량 증가** 및 **상태 불일치** 등의 문제를 일으킬 수 있습니다.

---

따라서 **CatsModule** 에서 **CatsService 를 export** 하면,
**하나의 인스턴스**가 모든 모듈에서 재사용되어 더 예측 가능한 동작을 하게 됩니다.

이것이 NestJS 같은 프레임워크에서 **모듈화(modularity)** 와 **DI(Dependency Injection)** 의 핵심 이점 중 하나입니다.

---

### 모듈 재-익스포트(Module re-exporting)

모듈은 **내부 프로바이더뿐 아니라**,
**import 한 모듈도 재-export** 할 수 있습니다.

예:

```typescript
@Module({
  imports: [CommonModule],
  exports: [CommonModule],
})
export class CoreModule {}
```

이렇게 하면 다른 모듈에서 **CoreModule** 을 import 하기만 해도,
**CommonModule** 도 사용할 수 있습니다.

---

### 의존성 주입(Dependency injection)

모듈 클래스도 프로바이더를 주입받을 수 있습니다 (예: 설정 목적):

#### cats.module.ts

```typescript
import { Module } from '@nestjs/common';
import { CatsController } from './cats.controller';
import { CatsService } from './cats.service';

@Module({
  controllers: [CatsController],
  providers: [CatsService],
})
export class CatsModule {
  constructor(private catsService: CatsService) {}
}
```

⚠️ **주의**
모듈 클래스 자체는 **프로바이더로 주입할 수 없습니다**.
(순환 의존성 문제가 발생할 수 있음)

---

### 글로벌 모듈(Global modules)

동일한 모듈을 여러 곳에 계속 import 하는 것이 번거로울 수 있습니다.
Nest에서는 프로바이더가 기본적으로 **모듈 범위(module scope)** 에만 등록됩니다.

**Angular** 와는 달리, Nest에서는 **글로벌 범위(global scope)** 에 등록하려면
명시적으로 **@Global() 데코레이터** 를 사용해야 합니다.

---

예:

```typescript
import { Module, Global } from '@nestjs/common';
import { CatsController } from './cats.controller';
import { CatsService } from './cats.service';

@Global()
@Module({
  controllers: [CatsController],
  providers: [CatsService],
  exports: [CatsService],
})
export class CatsModule {}
```

---

이렇게 하면 **CatsService** 는 애플리케이션 전체에서 사용 가능해집니다.
다른 모듈에서 **CatsModule** 을 import 할 필요가 없습니다.

---

💡 **힌트**
**모든 것을 글로벌로 만드는 것은 권장되지 않습니다**.
`imports` 배열을 사용해서 **명확하게 어떤 API 를 공개할지 제어**하는 것이 유지 보수 측면에서 더 좋습니다.

---

### 동적 모듈(Dynamic modules)

Nest에서는 **런타임 시 구성 가능한 모듈(dynamic module)** 을 만들 수 있습니다.

이는 설정에 따라 유연하게 프로바이더를 생성해야 할 때 유용합니다.

---

간단한 예:

```typescript
import { Module, DynamicModule } from '@nestjs/common';
import { createDatabaseProviders } from './database.providers';
import { Connection } from './connection.provider';

@Module({
  providers: [Connection],
  exports: [Connection],
})
export class DatabaseModule {
  static forRoot(entities = [], options?): DynamicModule {
    const providers = createDatabaseProviders(options, entities);
    return {
      module: DatabaseModule,
      providers: providers,
      exports: providers,
    };
  }
}
```

💡 **힌트**
`forRoot()` 메서드는 동기 또는 비동기로 **DynamicModule** 을 반환할 수 있습니다.

---

위 예제에서는 기본적으로 `Connection` 프로바이더를 제공하지만,
`forRoot()` 호출 시 전달된 `entities` 와 `options` 에 따라 추가 프로바이더도 동적으로 생성합니다.

→ 이렇게 하면 **정적 + 동적 프로바이더** 모두 export 됩니다.

---

**전역으로 등록**하려면:

```typescript
{
  global: true,
  module: DatabaseModule,
  providers: providers,
  exports: providers,
}
```

⚠️ **주의**
앞서 설명한 것처럼 **모든 것을 글로벌로 등록하는 것은 좋은 설계가 아닙니다**.

---

사용 예:

```typescript
import { Module } from '@nestjs/common';
import { DatabaseModule } from './database/database.module';
import { User } from './users/entities/user.entity';

@Module({
  imports: [DatabaseModule.forRoot([User])],
})
export class AppModule {}
```

---

만약 **동적 모듈을 다시 export** 하고 싶다면:

```typescript
import { Module } from '@nestjs/common';
import { DatabaseModule } from './database/database.module';
import { User } from './users/entities/user.entity';

@Module({
  imports: [DatabaseModule.forRoot([User])],
  exports: [DatabaseModule],
})
export class AppModule {}
```

---

**Dynamic modules** 에 대한 자세한 내용과 예제는 [Dynamic modules 챕터](https://docs.nestjs.com/fundamentals/dynamic-modules) 를 참고하세요.

💡 **힌트**
`ConfigurableModuleBuilder` 를 사용하면 **매우 유연한 동적 모듈**을 만들 수 있습니다.
자세한 내용은 [이 챕터](https://docs.nestjs.com/fundamentals/configurable-modules)를 참고하세요.