### Interceptors (인터셉터)

**인터셉터(Interceptor)** 는 `@Injectable()` 데코레이터가 적용된 클래스이며,
`NestInterceptor` 인터페이스를 구현합니다.

---

인터셉터는 **AOP(Aspect Oriented Programming, 관점 지향 프로그래밍)** 에서 영감을 받은 다양한 유용한 기능들을 제공합니다:

* 메서드 실행 **전/후에 추가 로직 바인딩**
* 함수가 반환하는 결과를 **변환**
* 함수에서 발생한 예외를 **변환**
* 기본 함수 동작을 **확장**
* 특정 조건에 따라 함수 실행을 **완전히 오버라이드** (예: 캐싱 목적)

---

### 기본 구조(Basics)

각 인터셉터는 `intercept()` 메서드를 구현해야 하며,
이 메서드는 **두 개의 인자**를 받습니다:

1️⃣ `ExecutionContext` 인스턴스
(guards 와 동일한 객체 — ArgumentsHost 를 상속)

2️⃣ `CallHandler`

---

**ExecutionContext** 는 앞서 **Exception Filters** 챕터에서 설명한 `ArgumentsHost` 를 상속합니다.
`ArgumentsHost` 는 **원래 핸들러에 전달된 arguments 를 감싸는 래퍼(wrapper)** 입니다.
애플리케이션 타입에 따라 다양한 arguments 배열을 포함합니다.

자세한 내용은 **Exception Filters** 챕터를 참고하세요.

---

#### Execution context

`ExecutionContext` 는 `ArgumentsHost` 를 확장하여,
현재 실행 프로세스에 대한 **추가적인 헬퍼 메서드** 를 제공합니다.

이 덕분에 보다 **범용적인 인터셉터** 를 구현할 수 있으며,
다양한 컨트롤러/메서드/ExecutionContext 에서 재사용이 가능합니다.

자세한 내용은 [ExecutionContext](https://docs.nestjs.com/fundamentals/execution-context) 문서 참고.

---

#### Call handler

두 번째 인자인 `CallHandler` 는 `handle()` 메서드를 구현합니다.

인터셉터 내에서 `handle()` 메서드를 호출하면
**해당 라우트 핸들러가 실행됩니다**.

👉 만약 `handle()` 을 호출하지 않으면
**라우트 핸들러는 실행되지 않습니다!**

---

결과적으로 `intercept()` 메서드는 **요청/응답 스트림을 감싸는(wrapper)** 구조가 됩니다.

* `handle()` 호출 **전(before)** 에는 인터셉터 내부에서 커스텀 로직 실행 가능
* `handle()` 호출 **후(after)** 에는 **RxJS 연산자** 를 사용해 응답 스트림을 조작 가능

이러한 개념은 **AOP 용어로 Pointcut** (포인트컷) 이라고 부릅니다.
추가 로직이 삽입되는 지점을 의미합니다.

---

예를 들어 `POST /cats` 요청이 들어왔을 때,
인터셉터가 `handle()` 을 호출하지 않으면 `create()` 핸들러는 실행되지 않습니다.

`handle()` 호출 후에는 `create()` 핸들러가 실행되고,
**Observable 응답 스트림** 을 통해 후처리를 진행할 수 있습니다.

---

### AOP 방식 인터셉션(Aspect interception)

첫 번째 예제로 **사용자 상호작용 로깅** 인터셉터를 만들어 보겠습니다.
(사용자 호출 저장, 이벤트 비동기 디스패치, 타임스탬프 계산 등 가능)

#### logging.interceptor.ts

```typescript
import { Injectable, NestInterceptor, ExecutionContext, CallHandler } from '@nestjs/common';
import { Observable } from 'rxjs';
import { tap } from 'rxjs/operators';

@Injectable()
export class LoggingInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    console.log('Before...');

    const now = Date.now();
    return next
      .handle()
      .pipe(
        tap(() => console.log(`After... ${Date.now() - now}ms`)),
      );
  }
}
```

💡 **힌트**
`NestInterceptor<T, R>` 는 제네릭 인터페이스입니다:

* `T`: Observable<T> 의 타입
* `R`: Observable<R> 에 래핑된 반환 값의 타입

---

⚠️ **주의**
인터셉터도 **생성자에서 DI(의존성 주입)** 을 사용할 수 있습니다.
(컨트롤러, 프로바이더, 가드 등과 동일)

---

위 예제에서 `handle()` 은 RxJS Observable 을 반환하므로
`tap()` 연산자를 사용해 스트림 종료 시점에 **로깅** 합니다.
응답 사이클에는 영향을 주지 않습니다.

---

### 인터셉터 바인딩(Binding interceptors)

인터셉터를 적용하려면
`@nestjs/common` 패키지에서 제공하는 `@UseInterceptors()` 데코레이터를 사용합니다.

인터셉터는 **컨트롤러 범위(controller-scoped)**,
**메서드 범위(method-scoped)**,
**글로벌 범위(global-scoped)** 로 설정할 수 있습니다.

---

#### 예시 (컨트롤러 범위):

```typescript
@UseInterceptors(LoggingInterceptor)
export class CatsController {}
```

💡 **힌트**
`@UseInterceptors()` 는 `@nestjs/common` 에서 import.

---

위와 같이 하면 `CatsController` 에 정의된 모든 라우트 핸들러에서
**LoggingInterceptor** 가 적용됩니다.

---

예상 출력:

```
Before...
After... 1ms
```

---

인스턴스를 직접 생성해서 전달할 수도 있습니다:

```typescript
@UseInterceptors(new LoggingInterceptor())
export class CatsController {}
```

---

만약 특정 **메서드만** 인터셉트하고 싶다면,
해당 메서드에 `@UseInterceptors()` 를 적용하면 됩니다.

---

### 글로벌 인터셉터(Global interceptors)

전역 인터셉터는 Nest 애플리케이션 전체에 적용됩니다:

```typescript
const app = await NestFactory.create(AppModule);
app.useGlobalInterceptors(new LoggingInterceptor());
```

---

⚠️ **주의**
전역 인터셉터를 이렇게 등록하면 **DI가 불가능**합니다.
(모듈 외부에서 인스턴스를 생성하므로)

---

DI 를 사용하려면 `APP_INTERCEPTOR` 토큰을 사용합니다:

#### app.module.ts

```typescript
import { Module } from '@nestjs/common';
import { APP_INTERCEPTOR } from '@nestjs/core';

@Module({
  providers: [
    {
      provide: APP_INTERCEPTOR,
      useClass: LoggingInterceptor,
    },
  ],
})
export class AppModule {}
```

💡 **힌트**
이 방식으로 등록하면 인터셉터는 **전역(global)** 으로 적용됩니다.
가급적 인터셉터가 정의된 모듈에 작성하는 것이 좋습니다.

---

### 응답 매핑(Response mapping)

`handle()` 은 Observable 을 반환하므로
**RxJS map() 연산자** 를 사용해 응답을 쉽게 변환할 수 있습니다.

⚠️ **주의**
응답 매핑은 **라이브러리 고유의 응답 객체 (@Res()) 를 사용할 경우 동작하지 않습니다**.

---

예: 응답을 `{ data: originalResponse }` 형식으로 변환

#### transform.interceptor.ts

```typescript
import { Injectable, NestInterceptor, ExecutionContext, CallHandler } from '@nestjs/common';
import { Observable } from 'rxjs';
import { map } from 'rxjs/operators';

export interface Response<T> {
  data: T;
}

@Injectable()
export class TransformInterceptor<T> implements NestInterceptor<T, Response<T>> {
  intercept(context: ExecutionContext, next: CallHandler): Observable<Response<T>> {
    return next.handle().pipe(map(data => ({ data })));
  }
}
```

---

예: `GET /cats` 호출 시 반환 예시:

```json
{
  "data": []
}
```

---

전역 응답 변환도 가능합니다.
예: **null 값을 빈 문자열로 변환**:

```typescript
import { Injectable, NestInterceptor, ExecutionContext, CallHandler } from '@nestjs/common';
import { Observable } from 'rxjs';
import { map } from 'rxjs/operators';

@Injectable()
export class ExcludeNullInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next
      .handle()
      .pipe(map(value => value === null ? '' : value ));
  }
}
```

---

### 예외 매핑(Exception mapping)

RxJS `catchError()` 연산자를 사용해
**예외를 변환**할 수도 있습니다:

#### errors.interceptor.ts

```typescript
import {
  Injectable,
  NestInterceptor,
  ExecutionContext,
  BadGatewayException,
  CallHandler,
} from '@nestjs/common';
import { Observable, throwError } from 'rxjs';
import { catchError } from 'rxjs/operators';

@Injectable()
export class ErrorsInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next
      .handle()
      .pipe(
        catchError(err => throwError(() => new BadGatewayException())),
      );
  }
}
```

---

### 스트림 오버라이드(Stream overriding)

때때로 **핸들러를 아예 호출하지 않고** 다른 값을 반환해야 할 때가 있습니다.
대표적인 예: **캐싱**.

---

#### cache.interceptor.ts

```typescript
import { Injectable, NestInterceptor, ExecutionContext, CallHandler } from '@nestjs/common';
import { Observable, of } from 'rxjs';

@Injectable()
export class CacheInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const isCached = true;
    if (isCached) {
      return of([]);
    }
    return next.handle();
  }
}
```

---

위 예시에서는 `isCached` 값에 따라:

* 캐시되어 있다면 → `of([])` 를 반환 → 핸들러 호출 X
* 캐시되어 있지 않다면 → `next.handle()` 호출 → 핸들러 실행

---

실제 구현 시:

* TTL(Time To Live)
* 캐시 무효화(invalidation)
* 캐시 크기 등 고려해야 함

---

### 추가 RxJS 연산자 활용(More operators)

RxJS 연산자를 통해 다양한 기능 구현 가능:
예: **요청 타임아웃 처리**.

---

#### timeout.interceptor.ts

```typescript
import { Injectable, NestInterceptor, ExecutionContext, CallHandler, RequestTimeoutException } from '@nestjs/common';
import { Observable, throwError, TimeoutError } from 'rxjs';
import { catchError, timeout } from 'rxjs/operators';

@Injectable()
export class TimeoutInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(
      timeout(5000),
      catchError(err => {
        if (err instanceof TimeoutError) {
          return throwError(() => new RequestTimeoutException());
        }
        return throwError(() => err);
      }),
    );
  };
};
```

---

위 예시에서는 **5초 이상 응답이 없으면**:

* **RequestTimeoutException** 을 발생시킴

필요하다면 예외 발생 전 리소스 정리 등의 **커스텀 로직** 도 추가 가능.