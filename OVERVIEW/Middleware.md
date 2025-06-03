### 미들웨어(Middleware)

**미들웨어(Middleware)** 는 **라우트 핸들러가 호출되기 전에 실행되는 함수**입니다.
미들웨어 함수는 **요청(request) 객체**와 **응답(response) 객체**, 그리고 애플리케이션의 요청-응답 사이클에서 **다음(next) 미들웨어 함수**에 접근할 수 있습니다.
보통 다음 미들웨어 함수는 `next`라는 변수로 표현합니다.

---

Nest의 미들웨어는 기본적으로 **Express 미들웨어와 동일한 동작**을 합니다.
다음은 공식 **Express 문서**에서 설명하는 미들웨어의 기능입니다:

미들웨어 함수는 다음 작업을 수행할 수 있습니다:

✅ 임의의 코드를 실행할 수 있다.
✅ 요청(request) 및 응답(response) 객체를 수정할 수 있다.
✅ 요청-응답 사이클을 종료할 수 있다.
✅ 스택의 다음 미들웨어 함수를 호출할 수 있다.

현재 미들웨어 함수가 요청-응답 사이클을 종료하지 않으면 반드시 `next()`를 호출하여 다음 미들웨어로 제어를 넘겨야 합니다.
그렇지 않으면 요청이 **멈춘 상태**로 남게 됩니다.

---

**Nest에서는** 사용자 정의 미들웨어를:

* 함수(function) 또는
* `@Injectable()` 데코레이터가 적용된 **클래스(class)** 로 구현할 수 있습니다.

클래스를 사용할 경우 `NestMiddleware` 인터페이스를 구현해야 합니다.
함수를 사용할 경우 특별한 요구사항은 없습니다.

---

먼저 **클래스 기반** 미들웨어를 구현해봅시다.

⚠️ **경고**
Express와 Fastify는 미들웨어를 서로 다르게 처리하며 **메서드 시그니처**도 다릅니다. 자세한 내용은 [여기](https://docs.nestjs.com/middleware#express-vs-fastify)에서 확인하세요.

---

#### logger.middleware.ts

```typescript
import { Injectable, NestMiddleware } from '@nestjs/common';
import { Request, Response, NextFunction } from 'express';

@Injectable()
export class LoggerMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    console.log('Request...');
    next();
  }
}
```

---

### 의존성 주입(Dependency injection)

Nest의 미들웨어는 **의존성 주입(Dependency Injection)** 을 완벽하게 지원합니다.
프로바이더나 컨트롤러처럼, 동일한 모듈 내에서 사용할 수 있는 의존성을 주입할 수 있습니다.
항상 그렇듯, **생성자(constructor)** 를 통해 주입합니다.

---

### 미들웨어 적용(Applying middleware)

`@Module()` 데코레이터에는 **미들웨어 설정 공간이 없습니다**.
대신, **모듈 클래스의 `configure()` 메서드**를 사용하여 설정합니다.

미들웨어를 포함하는 모듈은 반드시 `NestModule` 인터페이스를 구현해야 합니다.

---

다음은 **AppModule 수준**에서 `LoggerMiddleware`를 설정하는 예제입니다:

#### app.module.ts

```typescript
import { Module, NestModule, MiddlewareConsumer } from '@nestjs/common';
import { LoggerMiddleware } from './common/middleware/logger.middleware';
import { CatsModule } from './cats/cats.module';

@Module({
  imports: [CatsModule],
})
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer
      .apply(LoggerMiddleware)
      .forRoutes('cats');
  }
}
```

위 예제에서는 `/cats` 경로에 대해 `LoggerMiddleware` 를 설정했습니다.
이 `/cats` 경로는 이전에 `CatsController` 내부에서 정의되었습니다.

---

특정 요청 메서드에만 미들웨어를 적용하려면,
`forRoutes()` 메서드에 **경로(path)** 와 **요청 메서드(method)** 를 포함한 객체를 전달할 수 있습니다.

아래 예제에서는 `RequestMethod` 열거형(enum)을 가져와서 사용합니다:

#### app.module.ts

```typescript
import { Module, NestModule, RequestMethod, MiddlewareConsumer } from '@nestjs/common';
import { LoggerMiddleware } from './common/middleware/logger.middleware';
import { CatsModule } from './cats/cats.module';

@Module({
  imports: [CatsModule],
})
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer
      .apply(LoggerMiddleware)
      .forRoutes({ path: 'cats', method: RequestMethod.GET });
  }
}
```

💡 **힌트**
`configure()` 메서드는 `async/await` 를 사용하여 **비동기 함수**로 만들 수 있습니다 (예: `configure()` 메서드 내부에서 비동기 작업 완료를 기다릴 때).

---

⚠️ **경고**
**Express 어댑터 사용 시**, NestJS 앱은 기본적으로 `body-parser` 패키지에서 `json` 및 `urlencoded` 미들웨어를 등록합니다.
`MiddlewareConsumer` 를 통해 이 미들웨어를 커스터마이징하려면, 앱 생성 시 `bodyParser` 플래그를 `false`로 설정해야 합니다:

```typescript
const app = await NestFactory.create(AppModule, { bodyParser: false });
```

---

### 라우트 와일드카드(Route wildcards)

NestJS 미들웨어는 **패턴 기반 라우트**도 지원합니다.

예를 들어, **이름 있는 와일드카드(named wildcard) `*splat`** 를 사용하여 경로의 어떤 문자 조합에도 일치시킬 수 있습니다.

```typescript
forRoutes({
  path: 'abcd/*splat',
  method: RequestMethod.ALL,
});
```

💡 **힌트**
`splat`는 그냥 **와일드카드 변수명**일 뿐이며 특별한 의미는 없습니다.
`*wildcard` 처럼 원하는 이름을 쓸 수 있습니다.

---

`'abcd/*'` 경로는 다음과 같은 요청에 일치합니다:

* `abcd/1`
* `abcd/123`
* `abcd/abc`

하지만 `abcd/` (추가 문자가 없는 경우) 에는 일치하지 않습니다.
이 경우, 와일드카드를 중괄호로 감싸 **옵셔널** 로 만들어야 합니다:

```typescript
forRoutes({
  path: 'abcd/{*splat}',
  method: RequestMethod.ALL,
});
```

---

### 미들웨어 컨슈머(Middleware consumer)

`MiddlewareConsumer` 는 **헬퍼 클래스** 입니다.
미들웨어를 관리하기 위한 다양한 내장 메서드를 제공하며, 모든 메서드는 **체이닝(fluent style)** 으로 사용 가능합니다.

---

`forRoutes()` 메서드는 다음과 같은 값을 받을 수 있습니다:

✅ 단일 문자열 (string)
✅ 여러 문자열 (string\[])
✅ `RouteInfo` 객체
✅ 컨트롤러 클래스
✅ 여러 컨트롤러 클래스

---

일반적으로는 **컨트롤러 클래스 목록**을 전달하는 경우가 많습니다.
아래는 **단일 컨트롤러**를 사용하는 예제입니다:

#### app.module.ts

```typescript
import { Module, NestModule, MiddlewareConsumer } from '@nestjs/common';
import { LoggerMiddleware } from './common/middleware/logger.middleware';
import { CatsModule } from './cats/cats.module';
import { CatsController } from './cats/cats.controller';

@Module({
  imports: [CatsModule],
})
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer
      .apply(LoggerMiddleware)
      .forRoutes(CatsController);
  }
}
```

💡 **힌트**
`apply()` 메서드는 단일 미들웨어 또는 **여러 개의 미들웨어를 쉼표(,)로 구분하여** 지정할 수 있습니다.

---

### 특정 라우트 제외(Excluding routes)

경우에 따라 특정 라우트에는 **미들웨어 적용을 제외**하고 싶을 수 있습니다.
이때는 `exclude()` 메서드를 사용하면 됩니다.

`exclude()` 메서드는 다음과 같은 값을 받을 수 있습니다:

✅ 단일 문자열
✅ 여러 문자열
✅ `RouteInfo` 객체

---

예제:

```typescript
consumer
  .apply(LoggerMiddleware)
  .exclude(
    { path: 'cats', method: RequestMethod.GET },
    { path: 'cats', method: RequestMethod.POST },
    'cats/{*splat}',
  )
  .forRoutes(CatsController);
```

💡 **힌트**
`exclude()` 메서드는 **path-to-regexp** 패키지를 사용하여 **와일드카드** 지원도 합니다.

---

위 예제에서 `LoggerMiddleware` 는 `CatsController`의 모든 라우트에 바인딩되지만, `exclude()`에 전달된 3개 경로에는 적용되지 않습니다.

이 방식은 특정 라우트 또는 패턴에 따라 유연하게 미들웨어 적용 여부를 조정할 수 있습니다.

---

### 함수형 미들웨어(Functional middleware)

지금까지 사용한 `LoggerMiddleware` 클래스는 매우 간단했습니다.
멤버 변수도 없고, 추가 메서드나 의존성도 없었습니다.

그렇다면, 굳이 클래스로 작성할 필요 없이 **함수로 작성**할 수 있습니다.
이런 방식을 **함수형 미들웨어(functional middleware)** 라고 합니다.

---

클래스 기반 미들웨어를 함수형으로 변경해 보겠습니다:

#### logger.middleware.ts

```typescript
import { Request, Response, NextFunction } from 'express';

export function logger(req: Request, res: Response, next: NextFunction) {
  console.log(`Request...`);
  next();
};
```

그리고 **AppModule** 에서 사용:

```typescript
consumer
  .apply(logger)
  .forRoutes(CatsController);
```

💡 **힌트**
미들웨어에 **의존성이 필요 없는 경우**는 함수형 미들웨어를 사용하는 것이 더 간단합니다.

---

### 다중 미들웨어(Multiple middleware)

앞서 언급했듯이, 여러 미들웨어를 순차적으로 실행하려면 `apply()` 메서드에 **쉼표(,)로 구분된 목록**을 전달하면 됩니다:

```typescript
consumer.apply(cors(), helmet(), logger).forRoutes(CatsController);
```

---

### 전역 미들웨어(Global middleware)

모든 등록된 라우트에 **전역적으로 미들웨어**를 적용하고 싶다면
`INestApplication` 인스턴스에서 제공하는 `use()` 메서드를 사용하면 됩니다:

#### main.ts

```typescript
const app = await NestFactory.create(AppModule);
app.use(logger);
await app.listen(process.env.PORT ?? 3000);
```

💡 **힌트**
전역 미들웨어에서는 **DI 컨테이너 접근이 불가능**합니다.
따라서 전역에서는 함수형 미들웨어 사용이 권장됩니다.

또는 클래스 기반 미들웨어를 `AppModule` 또는 다른 모듈에서 `.forRoutes('*')` 로 적용할 수도 있습니다.
