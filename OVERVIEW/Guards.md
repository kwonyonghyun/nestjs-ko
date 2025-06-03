### Guards (가드)

**가드(Guard)** 는 `@Injectable()` 데코레이터가 적용된 클래스이며, `CanActivate` 인터페이스를 구현합니다.

---

가드는 **단일 책임(single responsibility)** 을 가집니다.
즉, **주어진 요청이 라우트 핸들러에 의해 처리될지 여부**를 런타임 시점의 조건(권한, 역할, ACL 등)에 따라 결정합니다.
이것은 흔히 **인가(authorization)** 라고 합니다.

---

**인가(authorization)** 와 짝을 이루는 개념인 **인증(authentication)** 은 전통적인 Express 애플리케이션에서는 종종 **미들웨어(middleware)** 로 처리되었습니다.

* **미들웨어는 인증(authentication)** 을 처리하기에 적합합니다.
  (토큰 유효성 검사, request 객체에 속성 추가 등은 특정 라우트 컨텍스트와 강하게 연결되어 있지 않음)

---

그러나 **미들웨어는 한계가 있습니다**:

* 미들웨어는 `next()` 호출 이후 **어떤 핸들러가 실행될지 알지 못합니다**.

반면, **가드(Guards)** 는 `ExecutionContext` 인스턴스에 접근할 수 있기 때문에,
**어떤 핸들러가 실행될지 정확히 알 수 있습니다**.

---

가드는 **예외 필터(Exception Filters)**, **파이프(Pipes)**, **인터셉터(Interceptors)** 처럼
**요청/응답 사이클의 적절한 시점에 처리 로직을 선언적으로 추가**하도록 설계되었습니다.
이를 통해 **코드를 DRY** 하고 **선언적(declarative)** 으로 유지할 수 있습니다.

---

💡 **힌트**
가드는 **모든 미들웨어가 실행된 후**,
**인터셉터 및 파이프가 실행되기 전에** 실행됩니다.

---

### 인가 가드(Authorization guard)

앞서 설명한 것처럼 **인가(authorization)** 는 가드의 주요 사용 사례입니다.
특정 라우트는 **권한이 충분한 사용자만 접근 가능**해야 합니다.

---

아래 **AuthGuard** 는 **인증된 사용자**(즉, 요청 헤더에 토큰이 첨부된 경우)를 전제로 합니다.
토큰을 추출하고 검증한 뒤, 요청을 계속 진행할지 여부를 결정합니다.

#### auth.guard.ts

```typescript
import { Injectable, CanActivate, ExecutionContext } from '@nestjs/common';
import { Observable } from 'rxjs';

@Injectable()
export class AuthGuard implements CanActivate {
  canActivate(
    context: ExecutionContext,
  ): boolean | Promise<boolean> | Observable<boolean> {
    const request = context.switchToHttp().getRequest();
    return validateRequest(request);
  }
}
```

💡 **힌트**
실제 인증(auth) 구현 예제는 [이 챕터](https://docs.nestjs.com/security/authentication)를 참고하세요.
좀 더 복잡한 인가(authorization) 예제는 [여기](https://docs.nestjs.com/security/authorization)를 참고하세요.

---

`validateRequest()` 함수의 구현은 상황에 따라 단순하거나 복잡할 수 있습니다.
여기서는 **가드가 요청/응답 사이클에 어떻게 개입하는지**를 보여주는 것이 핵심입니다.

---

모든 가드는 반드시 **`canActivate()`** 메서드를 구현해야 합니다.
이 메서드는 **현재 요청이 허용되는지 여부(boolean)** 를 반환해야 합니다.

* `true` 반환 → 요청 처리됨
* `false` 반환 → 요청 거부됨

`canActivate()` 는 동기(synchronous), 비동기(Promise), 또는 Observable 을 반환할 수 있습니다.

---

### Execution context

`canActivate()` 메서드는 `ExecutionContext` 인스턴스를 인자로 받습니다.
`ExecutionContext` 는 `ArgumentsHost` 를 상속합니다.

---

우리는 이전에 **Exception Filters** 챕터에서 `ArgumentsHost` 를 봤습니다.
여기서도 동일한 헬퍼 메서드를 사용해 **Request 객체**를 가져옵니다.

---

`ExecutionContext` 는 `ArgumentsHost` 를 확장하여
현재 실행 프로세스에 대한 **추가적인 헬퍼 메서드** 를 제공합니다.
이를 활용하면 더 **범용적인 가드**를 구현할 수 있습니다.

자세한 내용은 [ExecutionContext](https://docs.nestjs.com/fundamentals/execution-context) 챕터에서 확인하세요.

---

### 역할 기반 인증(Role-based authentication)

이번에는 **특정 역할(role)** 을 가진 사용자만 접근할 수 있도록 하는 가드를 만들어봅시다.

기본 템플릿:

#### roles.guard.ts

```typescript
import { Injectable, CanActivate, ExecutionContext } from '@nestjs/common';
import { Observable } from 'rxjs';

@Injectable()
export class RolesGuard implements CanActivate {
  canActivate(
    context: ExecutionContext,
  ): boolean | Promise<boolean> | Observable<boolean> {
    return true;
  }
}
```

---

### 가드 바인딩(Binding guards)

가드는 **컨트롤러 범위(controller-scoped)**, **메서드 범위(method-scoped)**,
또는 **글로벌 범위(global-scoped)** 로 바인딩할 수 있습니다.

---

**컨트롤러 범위** 예시:

```typescript
@Controller('cats')
@UseGuards(RolesGuard)
export class CatsController {}
```

💡 **힌트**
`@UseGuards()` 데코레이터는 `@nestjs/common` 패키지에서 가져옵니다.

---

여기서 **RolesGuard 클래스(클래스 자체)** 를 전달했습니다.
Nest가 인스턴스를 생성하므로 **DI(의존성 주입)** 이 가능합니다.

인스턴스를 직접 생성해서 전달할 수도 있습니다:

```typescript
@Controller('cats')
@UseGuards(new RolesGuard())
export class CatsController {}
```

---

위 구문은 **컨트롤러 내 모든 핸들러**에 가드를 적용합니다.
특정 메서드에만 적용하려면 **메서드 레벨**에서 `@UseGuards()` 를 사용하면 됩니다.

---

**글로벌 가드** 설정:

```typescript
const app = await NestFactory.create(AppModule);
app.useGlobalGuards(new RolesGuard());
```

⚠️ **주의**
**Hybrid 앱**에서는 `useGlobalGuards()` 가 기본적으로
게이트웨이(gateways) 및 마이크로서비스(microservices)에는 적용되지 않습니다.

자세한 내용은 [Hybrid application](https://docs.nestjs.com/faq/hybrid-application) 문서를 참고하세요.

---

**전역 가드**는 애플리케이션 전체에서, 모든 컨트롤러와 라우트 핸들러에 적용됩니다.

단, **DI** 가 필요한 경우, 모듈에서 `APP_GUARD` 토큰을 사용하여 등록해야 합니다:

#### app.module.ts

```typescript
import { Module } from '@nestjs/common';
import { APP_GUARD } from '@nestjs/core';

@Module({
  providers: [
    {
      provide: APP_GUARD,
      useClass: RolesGuard,
    },
  ],
})
export class AppModule {}
```

💡 **힌트**
이렇게 등록하면 어느 모듈에서든 **전역 가드**가 됩니다.
가드가 정의된 모듈에서 이 구성을 하는 것이 좋습니다.

---

### 핸들러 별 역할 설정(Setting roles per handler)

현재 RolesGuard 는 아직 단순합니다.
핸들러 별로 어떤 역할이 허용되는지 알지 못합니다.

---

**CatsController** 의 각 라우트는 서로 다른 권한 체계를 가질 수 있습니다:

* 일부는 **admin** 사용자 전용
* 일부는 **모든 사용자** 가능

---

역할 정보를 핸들러에 매핑하려면 **커스텀 메타데이터(custom metadata)** 를 사용하면 됩니다.
Nest는 이를 위해:

* `Reflector.createDecorator()` 메서드
* 내장된 `@SetMetadata()` 데코레이터

를 제공합니다.

---

예시: **Roles 데코레이터** 만들기:

#### roles.decorator.ts

```typescript
import { Reflector } from '@nestjs/core';

export const Roles = Reflector.createDecorator<string[]>();
```

Roles 데코레이터는 `string[]` 타입 인자를 받습니다.

---

사용 예시:

#### cats.controller.ts

```typescript
@Post()
@Roles(['admin'])
async create(@Body() createCatDto: CreateCatDto) {
  this.catsService.create(createCatDto);
}
```

이렇게 하면 **create() 메서드** 에 **admin 역할**만 허용된다고 명시됩니다.

---

참고: `@SetMetadata()` 를 사용하여 동일한 효과를 얻을 수도 있습니다.
자세한 내용은 [여기](https://docs.nestjs.com/security/authorization#custom-metadata) 참고.

---

### Putting it all together (종합 구현)

이제 **RolesGuard** 를 완성해봅시다.
현재는 무조건 `true` 를 반환하고 있지만,
이제는 **현재 사용자 역할** 과 **요청한 라우트의 허용된 역할** 을 비교해야 합니다.

---

이를 위해 `Reflector` 를 사용하여 **핸들러의 메타데이터(roles)** 를 가져옵니다:

#### roles.guard.ts

```typescript
import { Injectable, CanActivate, ExecutionContext } from '@nestjs/common';
import { Reflector } from '@nestjs/core';
import { Roles } from './roles.decorator';

@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    const roles = this.reflector.get(Roles, context.getHandler());
    if (!roles) {
      return true;
    }
    const request = context.switchToHttp().getRequest();
    const user = request.user;
    return matchRoles(roles, user.roles);
  }
}
```

💡 **힌트**
Node.js 에서는 인증된 사용자를 `request.user` 에 저장하는 것이 일반적입니다.
여기서는 `request.user.roles` 에 사용자의 역할 정보가 있다고 가정합니다.

**matchRoles() 함수**는 원하는 만큼 단순하거나 복잡하게 구현할 수 있습니다.

---

**권한 부족 시** Nest는 다음과 같은 응답을 자동으로 반환합니다:

```json
{
  "statusCode": 403,
  "message": "Forbidden resource",
  "error": "Forbidden"
}
```

가드가 `false` 를 반환하면, Nest는 내부적으로 **`ForbiddenException`** 을 발생시킵니다.

---

원한다면 다른 예외를 직접 발생시킬 수도 있습니다:

```typescript
throw new UnauthorizedException();
```

가드에서 발생한 모든 예외는 **예외 처리 계층**(Global Exception Filter 또는 현재 컨텍스트의 Exception Filter)에서 처리됩니다.

---

💡 **힌트**
실제 애플리케이션에서 **인가 구현 예제**는 [이 챕터](https://docs.nestjs.com/security/authorization)를 참고하세요.