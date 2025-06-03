### Exception filters (예외 필터)

Nest에는 **내장된 예외 계층(exceptions layer)** 이 있습니다.
이 계층은 애플리케이션 전역에서 처리되지 않은 모든 예외를 처리하는 역할을 합니다.

애플리케이션 코드에서 예외가 처리되지 않으면, 해당 예외는 이 계층에서 **가로채어(caught)** 적절하고 사용자 친화적인 응답을 자동으로 전송합니다.

---

기본적으로 이 작업은 **내장 글로벌 예외 필터(global exception filter)** 에 의해 수행됩니다.
이 필터는 `HttpException` (및 그 하위 클래스) 타입의 예외를 처리합니다.

만약 인식할 수 없는 예외(`HttpException` 또는 그 하위 클래스가 아닌 경우)가 발생하면,
내장 예외 필터는 다음과 같은 **기본 JSON 응답**을 생성합니다:

```json
{
  "statusCode": 500,
  "message": "Internal server error"
}
```

💡 **힌트**
**글로벌 예외 필터는 `http-errors` 라이브러리를 부분적으로 지원**합니다.
`statusCode` 와 `message` 속성이 포함된 예외가 발생하면, 기본 `InternalServerErrorException` 대신 해당 속성으로 응답이 구성됩니다.

---

### 표준 예외 던지기(Throwing standard exceptions)

Nest는 내장된 `HttpException` 클래스를 제공합니다.
이 클래스는 `@nestjs/common` 패키지에서 가져올 수 있습니다.

일반적인 HTTP REST / GraphQL API 기반 애플리케이션에서는 오류가 발생했을 때 **표준 HTTP 응답 객체**를 보내는 것이 모범 사례입니다.

---

예를 들어, `CatsController` 의 `findAll()` 메서드(GET 라우트 핸들러)가 있다고 가정해봅시다.
이 핸들러에서 예외를 발생시키도록 **하드코딩** 해보겠습니다:

#### cats.controller.ts

```typescript
@Get()
async findAll() {
  throw new HttpException('Forbidden', HttpStatus.FORBIDDEN);
}
```

💡 **힌트**
`HttpStatus`는 `@nestjs/common` 패키지에서 가져오는 **헬퍼 enum** 입니다.

---

클라이언트가 해당 엔드포인트를 호출하면 응답은 다음과 같습니다:

```json
{
  "statusCode": 403,
  "message": "Forbidden"
}
```

---

`HttpException` 생성자는 다음과 같은 2개의 필수 인수를 받습니다:

1️⃣ `response` 인수: JSON 응답 본문을 정의합니다. 문자열 또는 객체가 될 수 있습니다.
2️⃣ `status` 인수: HTTP 상태 코드를 정의합니다.

기본적으로 JSON 응답 본문은 다음 2개 속성을 포함합니다:

* `statusCode`: `status` 인수로 제공된 HTTP 상태 코드
* `message`: 상태 코드에 따른 HTTP 오류 설명 문자열

---

**메시지 부분만 재정의**하려면 `response` 인수에 문자열을 제공하면 됩니다.
**전체 응답 본문을 재정의**하려면 `response` 인수에 객체를 전달하면 됩니다.

`status` 인수는 유효한 HTTP 상태 코드여야 하며, `HttpStatus` enum 을 사용하는 것이 좋습니다.

---

또한, **세 번째 선택적 인수(options)** 를 통해 **error cause(원인)** 를 제공할 수 있습니다.
이 cause 객체는 응답 본문에는 직렬화되지 않지만, **로깅 목적으로 유용한 내부 오류 정보를 포함**할 수 있습니다.

---

예시 - 전체 응답 본문과 error cause 제공:

#### cats.controller.ts

```typescript
@Get()
async findAll() {
  try {
    await this.service.findAll()
  } catch (error) {
    throw new HttpException({
      status: HttpStatus.FORBIDDEN,
      error: 'This is a custom message',
    }, HttpStatus.FORBIDDEN, {
      cause: error
    });
  }
}
```

응답 예시:

```json
{
  "status": 403,
  "error": "This is a custom message"
}
```

---

### 예외 로깅(Exceptions logging)

기본 예외 필터는 `HttpException` (및 그 하위 클래스) 같은 **내장 예외**는 로깅하지 않습니다.
이런 예외가 발생해도 **콘솔에 출력되지 않습니다**, 왜냐하면 이는 정상적인 애플리케이션 흐름의 일부로 간주됩니다.

`WsException`, `RpcException` 등 다른 내장 예외도 마찬가지입니다.

---

이러한 예외들은 모두 `IntrinsicException` 클래스를 상속합니다.
이 클래스는 `@nestjs/common` 패키지에서 가져올 수 있으며, **정상 흐름에 속하는 예외와 그렇지 않은 예외를 구분**하는 데 도움이 됩니다.

---

만약 이런 예외도 **로깅**하고 싶다면, **커스텀 예외 필터**를 만들 수 있습니다.
이는 다음 섹션에서 설명합니다.

---

### 커스텀 예외(Custom exceptions)

대부분의 경우, 커스텀 예외를 만들 필요 없이 Nest의 내장 HTTP 예외만으로 충분합니다.
만약 **커스텀 예외**가 필요하다면, **HttpException** 을 상속한 계층 구조를 만들면 좋습니다.

이렇게 하면 Nest가 예외를 인식하여 자동으로 응답 처리를 수행합니다.

#### forbidden.exception.ts

```typescript
export class ForbiddenException extends HttpException {
  constructor() {
    super('Forbidden', HttpStatus.FORBIDDEN);
  }
}
```

---

`ForbiddenException` 이 `HttpException` 을 상속하므로,
내장 예외 핸들러와 **문제없이 동작**하며 다음과 같이 사용할 수 있습니다:

#### cats.controller.ts

```typescript
@Get()
async findAll() {
  throw new ForbiddenException();
}
```

---

### 내장 HTTP 예외(Built-in HTTP exceptions)

Nest는 `HttpException` 을 상속하는 **표준 예외 클래스**들을 제공합니다.
이들은 `@nestjs/common` 패키지에서 가져올 수 있으며, 다양한 일반적인 HTTP 예외를 나타냅니다:

* `BadRequestException`
* `UnauthorizedException`
* `NotFoundException`
* `ForbiddenException`
* `NotAcceptableException`
* `RequestTimeoutException`
* `ConflictException`
* `GoneException`
* `HttpVersionNotSupportedException`
* `PayloadTooLargeException`
* `UnsupportedMediaTypeException`
* `UnprocessableEntityException`
* `InternalServerErrorException`
* `NotImplementedException`
* `ImATeapotException`
* `MethodNotAllowedException`
* `BadGatewayException`
* `ServiceUnavailableException`
* `GatewayTimeoutException`
* `PreconditionFailedException`

---

이 내장 예외들은 `options` 인수를 통해 **error cause** 와 **error description** 도 제공할 수 있습니다:

```typescript
throw new BadRequestException('Something bad happened', {
  cause: new Error(),
  description: 'Some error description',
});
```

응답 예시:

```json
{
  "message": "Something bad happened",
  "error": "Some error description",
  "statusCode": 400
}
```

---

### 예외 필터(Exception filters)

기본(내장) 예외 필터로 대부분의 상황을 처리할 수 있지만,
**예외 계층을 완전히 제어**하고 싶을 때는 **예외 필터(exception filters)** 를 사용합니다.

예를 들어:

✅ 로깅 추가
✅ 동적 조건에 따라 JSON 스키마 변경
✅ 예외 처리 흐름을 직접 제어

이런 작업이 가능합니다.

---

다음은 `HttpException` 인스턴스를 잡아서 **커스텀 응답 로직**을 구현하는 예제입니다.

이때 Request/Response 객체에 접근하여 **요청 URL** 을 로깅에 포함하고,
`response.json()` 메서드로 응답을 직접 구성합니다.

#### http-exception.filter.ts

```typescript
import { ExceptionFilter, Catch, ArgumentsHost, HttpException } from '@nestjs/common';
import { Request, Response } from 'express';

@Catch(HttpException)
export class HttpExceptionFilter implements ExceptionFilter {
  catch(exception: HttpException, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse<Response>();
    const request = ctx.getRequest<Request>();
    const status = exception.getStatus();

    response
      .status(status)
      .json({
        statusCode: status,
        timestamp: new Date().toISOString(),
        path: request.url,
      });
  }
}
```

💡 **힌트**
모든 예외 필터는 `ExceptionFilter<T>` 인터페이스를 구현해야 합니다.
따라서 반드시 `catch(exception: T, host: ArgumentsHost)` 메서드를 정의해야 합니다.

⚠️ **경고**
Fastify 사용 시에는 `response.send()` 를 사용하고, Fastify 타입을 올바르게 import 해야 합니다.

---

`@Catch(HttpException)` 데코레이터는 이 필터가 `HttpException` 타입의 예외만 처리한다는 **메타데이터**를 설정합니다.
`@Catch()` 는 단일 타입 또는 여러 타입을 인자로 받을 수 있습니다.

---

### ArgumentsHost

`catch()` 메서드의 인자는:

* `exception`: 현재 처리 중인 예외 객체
* `host`: `ArgumentsHost` 객체

---

`ArgumentsHost` 는 매우 강력한 **유틸리티 객체**입니다.
현재는 HTTP context 에서 Request/Response 객체를 가져오는 데 사용하지만,
**Microservices** 나 **WebSockets** 에서도 사용할 수 있습니다.

Execution context 챕터에서 더 자세히 배우게 됩니다.

---

### 예외 필터 바인딩(Binding filters)

다음은 `CatsController` 의 `create()` 메서드에 `HttpExceptionFilter` 를 적용하는 방법입니다:

#### cats.controller.ts

```typescript
@Post()
@UseFilters(new HttpExceptionFilter())
async create(@Body() createCatDto: CreateCatDto) {
  throw new ForbiddenException();
}
```

💡 **힌트**
`@UseFilters()` 데코레이터는 `@nestjs/common` 패키지에서 가져옵니다.

---

`@UseFilters()` 는 단일 필터 인스턴스 또는 여러 필터 인스턴스를 전달할 수 있습니다.

클래스 자체를 전달하여 DI를 사용할 수도 있습니다:

```typescript
@Post()
@UseFilters(HttpExceptionFilter)
async create(@Body() createCatDto: CreateCatDto) {
  throw new ForbiddenException();
}
```

💡 **힌트**
가능하면 **클래스**를 전달하는 것이 좋습니다.
Nest가 인스턴스를 재사용하여 **메모리 사용량이 감소**합니다.

---

컨트롤러 전체에 필터 적용:

```typescript
@Controller()
@UseFilters(new HttpExceptionFilter())
export class CatsController {}
```

---

**글로벌 필터** 적용은 `main.ts` 에서 다음과 같이 합니다:

```typescript
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.useGlobalFilters(new HttpExceptionFilter());
  await app.listen(process.env.PORT ?? 3000);
}
bootstrap();
```

⚠️ **경고**
`useGlobalFilters()` 는 **gateways** 나 **hybrid applications** 에는 적용되지 않습니다.

---

글로벌 필터에 **DI를 적용하려면**, 모듈에서 `APP_FILTER` 토큰을 사용합니다:

#### app.module.ts

```typescript
import { Module } from '@nestjs/common';
import { APP_FILTER } from '@nestjs/core';

@Module({
  providers: [
    {
      provide: APP_FILTER,
      useClass: HttpExceptionFilter,
    },
  ],
})
export class AppModule {}
```

💡 **힌트**
이렇게 하면 **전역 필터**가 되고, DI도 적용됩니다.

---

### 모든 예외 잡기(Catch everything)

모든 예외를 잡으려면 `@Catch()` 를 빈 인자로 사용하면 됩니다:

```typescript
@Catch()
export class CatchEverythingFilter implements ExceptionFilter {
  ...
}
```

다음은 플랫폼 독립적인 예제입니다:

```typescript
import {
  ExceptionFilter,
  Catch,
  ArgumentsHost,
  HttpException,
  HttpStatus,
} from '@nestjs/common';
import { HttpAdapterHost } from '@nestjs/core';

@Catch()
export class CatchEverythingFilter implements ExceptionFilter {
  constructor(private readonly httpAdapterHost: HttpAdapterHost) {}

  catch(exception: unknown, host: ArgumentsHost): void {
    const { httpAdapter } = this.httpAdapterHost;

    const ctx = host.switchToHttp();

    const httpStatus =
      exception instanceof HttpException
        ? exception.getStatus()
        : HttpStatus.INTERNAL_SERVER_ERROR;

    const responseBody = {
      statusCode: httpStatus,
      timestamp: new Date().toISOString(),
      path: httpAdapter.getRequestUrl(ctx.getRequest()),
    };

    httpAdapter.reply(ctx.getResponse(), responseBody, httpStatus);
  }
}
```

⚠️ **경고**
특정 타입에 바인딩된 필터와 "Catch anything" 필터를 함께 사용할 때는 **Catch anything 필터를 먼저 선언**해야 합니다.

---

### 상속(Inheritance)

일반적으로는 **맞춤형 예외 필터**를 직접 작성합니다.
하지만 경우에 따라 기본 글로벌 예외 필터를 **상속**하여 일부 동작만 재정의하고 싶을 수 있습니다.

---

이때는 `BaseExceptionFilter` 를 상속하고, `catch()` 메서드를 호출하면 됩니다:

#### all-exceptions.filter.ts

```typescript
import { Catch, ArgumentsHost } from '@nestjs/common';
import { BaseExceptionFilter } from '@nestjs/core';

@Catch()
export class AllExceptionsFilter extends BaseExceptionFilter {
  catch(exception: unknown, host: ArgumentsHost) {
    super.catch(exception, host);
  }
}
```

⚠️ **경고**
메서드 범위 또는 컨트롤러 범위에서 `BaseExceptionFilter` 를 상속한 필터는 **new** 로 인스턴스화하지 않고,
Nest가 자동으로 인스턴스를 생성하도록 해야 합니다.

---

**글로벌 필터**는 두 가지 방법으로 기본 필터를 확장할 수 있습니다:

1️⃣ `HttpAdapter` 참조를 직접 주입하여 인스턴스화:

```typescript
async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  const { httpAdapter } = app.get(HttpAdapterHost);
  app.useGlobalFilters(new AllExceptionsFilter(httpAdapter));

  await app.listen(process.env.PORT ?? 3000);
}
bootstrap();
```

2️⃣ `APP_FILTER` 토큰을 사용하는 방법도 있습니다.
