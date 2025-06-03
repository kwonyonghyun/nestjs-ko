### Custom Route Decorators (커스텀 라우트 데코레이터)

Nest는 **데코레이터(Decorators)** 라는 언어 기능을 기반으로 구축되어 있습니다.
데코레이터는 다양한 프로그래밍 언어에서 잘 알려진 개념이며,
**JavaScript** 에서는 비교적 새로운 기능입니다.

---

**데코레이터란?** (간단 정의)

> ES2016 데코레이터는 **함수를 반환하는 표현식(expression)** 이며,
> `target`, `name`, `property descriptor` 를 인자로 받을 수 있습니다.
> `@` 기호를 접두어로 사용해 클래스/메서드/속성에 적용합니다.

---

### Param decorators (파라미터 데코레이터)

Nest는 HTTP 라우트 핸들러에서 사용할 수 있는 다양한 **파라미터 데코레이터** 를 제공합니다:

| 데코레이터                      | Express/Fastify 객체와의 대응              |
| -------------------------- | ------------------------------------ |
| `@Request()`, `@Req()`     | `req`                                |
| `@Response()`, `@Res()`    | `res`                                |
| `@Next()`                  | `next`                               |
| `@Session()`               | `req.session`                        |
| `@Param(param?: string)`   | `req.params` / `req.params[param]`   |
| `@Body(param?: string)`    | `req.body` / `req.body[param]`       |
| `@Query(param?: string)`   | `req.query` / `req.query[param]`     |
| `@Headers(param?: string)` | `req.headers` / `req.headers[param]` |
| `@Ip()`                    | `req.ip`                             |
| `@HostParam()`             | `req.hosts`                          |

---

### 왜 커스텀 데코레이터가 필요할까?

Node.js 에서는 보통 `request` 객체에 커스텀 속성을 추가한 뒤,
라우트 핸들러에서 직접 꺼내 사용하는 경우가 많습니다:

```typescript
const user = req.user;
```

---

이렇게 **매번 수동으로 꺼내는 코드** 대신,
더 **가독성 높고 재사용 가능한** 데코레이터를 만들 수 있습니다:

#### 예: `@User()` 커스텀 데코레이터 만들기

```typescript
import { createParamDecorator, ExecutionContext } from '@nestjs/common';

export const User = createParamDecorator(
  (data: unknown, ctx: ExecutionContext) => {
    const request = ctx.switchToHttp().getRequest();
    return request.user;
  },
);
```

---

사용 예:

```typescript
@Get()
async findOne(@User() user: UserEntity) {
  console.log(user);
}
```

---

### 데이터 전달하기 (Passing data)

데코레이터의 동작이 **조건에 따라 달라져야 할 경우**,
`data` 파라미터를 통해 인자를 전달할 수 있습니다.

---

예를 들어, **인증 레이어** 에서 `req.user` 에 사용자 정보를 저장해 두었다고 가정합시다:

```json
{
  "id": 101,
  "firstName": "Alan",
  "lastName": "Turing",
  "email": "alan@email.com",
  "roles": ["admin"]
}
```

---

**특정 속성만 꺼내는** 데코레이터:

```typescript
import { createParamDecorator, ExecutionContext } from '@nestjs/common';

export const User = createParamDecorator(
  (data: string, ctx: ExecutionContext) => {
    const request = ctx.switchToHttp().getRequest();
    const user = request.user;

    return data ? user?.[data] : user;
  },
);
```

---

사용 예:

```typescript
@Get()
async findOne(@User('firstName') firstName: string) {
  console.log(`Hello ${firstName}`);
}
```

---

이렇게 하면 `@User('firstName')`, `@User('roles')` 처럼
**다양한 속성** 을 간편하게 꺼낼 수 있습니다.

---

💡 **힌트** (TypeScript 사용 시)

`createParamDecorator<T>()` 는 제네릭입니다.
명시적으로 타입을 지정해 **타입 안정성** 을 확보할 수 있습니다:

```typescript
createParamDecorator<string>((data, ctx) => ...)
```

또는

```typescript
createParamDecorator((data: string, ctx) => ...)
```

지정을 생략하면 기본적으로 `any` 타입이 됩니다.

---

### Pipes 와 함께 사용하기 (Working with pipes)

Nest는 **커스텀 파라미터 데코레이터** 를
기본 제공 데코레이터(`@Body()`, `@Param()`, `@Query()` 등)와 **동일하게 처리**합니다.

즉, **파이프(pipes)** 도 정상적으로 실행됩니다!

---

예:

```typescript
@Get()
async findOne(
  @User(new ValidationPipe({ validateCustomDecorators: true }))
  user: UserEntity,
) {
  console.log(user);
}
```

💡 **주의**
`validateCustomDecorators` 옵션을 `true` 로 설정해야 합니다.
기본적으로 `ValidationPipe` 는 커스텀 데코레이터의 인자를 검증하지 않습니다.

---

### 데코레이터 조합하기 (Decorator composition)

Nest는 여러 데코레이터를 **조합** 할 수 있도록 `applyDecorators()` 헬퍼를 제공합니다.

---

예: 인증 관련 데코레이터를 하나로 묶기:

```typescript
import { applyDecorators } from '@nestjs/common';

export function Auth(...roles: Role[]) {
  return applyDecorators(
    SetMetadata('roles', roles),
    UseGuards(AuthGuard, RolesGuard),
    ApiBearerAuth(),
    ApiUnauthorizedResponse({ description: 'Unauthorized' }),
  );
}
```

---

사용 예:

```typescript
@Get('users')
@Auth('admin')
findAllUsers() {}
```

---

이렇게 하면 **4개의 데코레이터** 를 한 번에 적용할 수 있습니다!

---

⚠️ **주의**

`@nestjs/swagger` 패키지의 `@ApiHideProperty()` 데코레이터는
`applyDecorators()` 와 **호환되지 않습니다**.
따라서 해당 데코레이터는 별도로 적용해야 합니다.