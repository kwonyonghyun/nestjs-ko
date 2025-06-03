### 컨트롤러

컨트롤러는 들어오는 요청을 처리하고 클라이언트로 응답을 반환하는 역할을 합니다.
![](https://docs.nestjs.com/assets/Controllers_1.png)


컨트롤러의 목적은 애플리케이션에서 특정 요청을 처리하는 것입니다. 라우팅 메커니즘이 어떤 컨트롤러가 각 요청을 처리할지 결정합니다. 일반적으로 컨트롤러는 여러 개의 라우트를 가지며, 각 라우트는 서로 다른 작업을 수행할 수 있습니다.

기본 컨트롤러를 생성하려면 클래스와 데코레이터를 사용합니다. 데코레이터는 클래스에 필요한 메타데이터를 연결하여, Nest가 요청과 해당 컨트롤러를 연결하는 라우팅 맵을 생성할 수 있도록 합니다.

💡 **힌트**
내장된 유효성 검사 기능이 있는 CRUD 컨트롤러를 빠르게 생성하려면 CLI의 CRUD 생성기를 사용할 수 있습니다:
`nest g resource [name]`

---

### 라우팅

다음 예제에서는 기본 컨트롤러를 정의하는 데 필수적인 `@Controller()` 데코레이터를 사용합니다. 여기서는 `cats`라는 경로 접두어(path prefix)를 선택적으로 지정합니다. 컨트롤러에서 경로 접두어를 지정하면 관련된 라우트들을 그룹화할 수 있고 반복되는 코드를 줄일 수 있습니다. 예를 들어, 고양이(cats) 엔티티와 상호작용하는 라우트들을 `/cats` 경로 아래에 묶고 싶다면, `@Controller('cats')`처럼 접두어를 지정하면 됩니다. 그러면 각 라우트에서 해당 경로를 반복해서 작성할 필요가 없습니다.

#### cats.controller.ts

```typescript
import { Controller, Get } from '@nestjs/common';

@Controller('cats')
export class CatsController {
  @Get()
  findAll(): string {
    return 'This action returns all cats';
  }
}
```

💡 **힌트**
CLI로 컨트롤러를 생성하려면 다음 명령어를 실행하세요:
`nest g controller [name]`

`@Get()` HTTP 요청 메서드 데코레이터는 `findAll()` 메서드 앞에 위치하여 특정 엔드포인트에 대한 핸들러를 만듭니다. 이 엔드포인트는 HTTP 요청 메서드(GET)와 라우트 경로로 정의됩니다. 그럼 라우트 경로란 무엇일까요? 라우트 경로는 컨트롤러에 선언한 (선택적인) 접두어와 메서드 데코레이터에 지정된 경로 문자열을 결합하여 결정됩니다. 위 예제에서는 `cats`라는 접두어를 설정했지만 메서드 데코레이터에는 별도의 경로를 지정하지 않았으므로 `GET /cats` 요청이 이 핸들러로 매핑됩니다.

위에서 언급한 것처럼, 라우트 경로는 컨트롤러 경로 접두어와 메서드 데코레이터에 지정된 경로 문자열을 합친 것입니다. 예를 들어, 컨트롤러 접두어가 `cats`이고 메서드 데코레이터가 `@Get('breed')`라면, 최종 라우트는 `GET /cats/breed`가 됩니다.

위 예제에서 `GET` 요청이 해당 엔드포인트로 들어오면 Nest는 요청을 `findAll()` 메서드로 라우팅합니다. 참고로 여기서 메서드 이름은 임의로 지정 가능합니다. 메서드를 선언해서 라우트와 연결하는 것은 필수이지만, Nest는 메서드 이름에 특별한 의미를 부여하지 않습니다.

이 메서드는 200 상태 코드와 함께 응답을 반환합니다. 여기서는 문자열을 반환하므로 그대로 전달됩니다. 왜 그런지 이해하려면 Nest에서 응답을 조작하는 방법을 알아야 합니다:

1️⃣ **표준 방식 (권장)**
핸들러가 JavaScript 객체나 배열을 반환하면 자동으로 JSON으로 직렬화됩니다. 원시 타입(string, number, boolean 등)을 반환하면 그대로 응답으로 전송됩니다. POST 요청은 기본적으로 201 상태 코드이고, 그 외에는 200입니다. 이 동작은 `@HttpCode(...)` 데코레이터로 변경할 수 있습니다.

2️⃣ **라이브러리 특화 방식**
Express 같은 라이브러리 고유의 응답 객체를 `@Res()` 데코레이터로 주입해서 사용할 수 있습니다. 예:
`findAll(@Res() response)`
이렇게 하면 `response.status(200).send()`처럼 직접 응답을 구성할 수 있습니다.

⚠️ Nest는 핸들러에서 `@Res()`나 `@Next()`를 사용하면 해당 핸들러는 라이브러리 특화 모드로 전환됩니다. 둘을 동시에 사용하면 표준 방식은 비활성화됩니다. 만약 쿠키/헤더만 직접 설정하고 나머지는 Nest가 처리하게 하려면 `@Res({ passthrough: true })` 옵션을 사용해야 합니다.

---

### 요청 객체

핸들러에서 클라이언트 요청 정보를 사용해야 하는 경우가 많습니다. Nest는 기본 플랫폼(기본적으로 Express)의 요청 객체에 접근할 수 있도록 `@Req()` 데코레이터를 제공합니다.

#### cats.controller.ts

```typescript
import { Controller, Get, Req } from '@nestjs/common';
import { Request } from 'express';

@Controller('cats')
export class CatsController {
  @Get()
  findAll(@Req() request: Request): string {
    return 'This action returns all cats';
  }
}
```

💡 **힌트**
`@types/express` 패키지를 설치하여 타입 지원을 받을 수 있습니다.

요청 객체에는 쿼리 문자열, 파라미터, HTTP 헤더, 바디 등이 포함됩니다. 하지만 대부분의 경우 이러한 속성들은 별도의 데코레이터(`@Body()`, `@Query()` 등)를 사용하는 것이 더 편리합니다.

| 데코레이터                   | 매핑되는 객체                          |
| ----------------------- | -------------------------------- |
| @Request(), @Req()      | req                              |
| @Response(), @Res()     | res                              |
| @Next()                 | next                             |
| @Session()              | req.session                      |
| @Param(key?: string)    | req.params / req.params\[key]    |
| @Body(key?: string)     | req.body / req.body\[key]        |
| @Query(key?: string)    | req.query / req.query\[key]      |
| @Headers(name?: string) | req.headers / req.headers\[name] |
| @Ip()                   | req.ip                           |
| @HostParam()            | req.hosts                        |

⚠️ `@Res()` 또는 `@Response()`를 사용하면 라이브러리 특화 모드가 되어 직접 응답을 반환해야 합니다. 그렇지 않으면 서버가 멈춥니다.

---

### 리소스

앞서 `GET` 라우트를 정의했는데, 일반적으로 `POST` 라우트도 추가합니다:

#### cats.controller.ts

```typescript
import { Controller, Get, Post } from '@nestjs/common';

@Controller('cats')
export class CatsController {
  @Post()
  create(): string {
    return 'This action adds a new cat';
  }

  @Get()
  findAll(): string {
    return 'This action returns all cats';
  }
}
```

Nest는 모든 표준 HTTP 메서드용 데코레이터를 제공합니다:
`@Get()`, `@Post()`, `@Put()`, `@Delete()`, `@Patch()`, `@Options()`, `@Head()`, 그리고 모든 요청을 처리하는 `@All()`도 있습니다.

---

### 와일드카드 경로

NestJS는 패턴 기반 라우트를 지원합니다. 예를 들어 `*`를 사용하여 경로 끝에서 어떤 문자 조합도 매칭할 수 있습니다:

```typescript
@Get('abcd/*')
findAll() {
  return 'This route uses a wildcard';
}
```

* `/abcd/`, `/abcd/123`, `/abcd/abc` 모두 매칭됩니다.
* Express 5부터는 이름 있는 와일드카드를 사용해야 합니다(예: `abcd/*splat`).
* 경로 중간에 `*`를 쓰려면 Express에서는 이름 있는 와일드카드 필요, Fastify는 지원하지 않음.

---

### 상태 코드

기본 응답 상태 코드는 `200`이고, `POST`는 `201`입니다. `@HttpCode()`로 변경할 수 있습니다:

```typescript
@Post()
@HttpCode(204)
create() {
  return 'This action adds a new cat';
}
```

---

### 응답 헤더

응답 헤더는 `@Header()` 또는 라이브러리 특화 객체로 설정할 수 있습니다:

```typescript
@Post()
@Header('Cache-Control', 'no-store')
create() {
  return 'This action adds a new cat';
}
```

---

### 리디렉션

리디렉션은 `@Redirect()` 데코레이터 또는 라이브러리 특화 객체로 구현할 수 있습니다:

```typescript
@Get()
@Redirect('https://nestjs.com', 301)
```

동적으로 리디렉션하려면 객체를 반환:

```typescript
@Get('docs')
@Redirect('https://docs.nestjs.com', 302)
getDocs(@Query('version') version) {
  if (version && version === '5') {
    return { url: 'https://docs.nestjs.com/v5/' };
  }
}
```

---

### 라우트 파라미터

정적 경로가 아닌 동적 데이터를 경로에서 받으려면 파라미터 토큰을 사용:

```typescript
@Get(':id')
findOne(@Param() params: any): string {
  console.log(params.id);
  return `This action returns a #${params.id} cat`;
}
```

더 간단히 특정 파라미터만 추출 가능:

```typescript
@Get(':id')
findOne(@Param('id') id: string): string {
  return `This action returns a #${id} cat`;
}
```

---

### 서브도메인 라우팅

`@Controller`에 `host` 옵션으로 요청 호스트를 지정 가능:

```typescript
@Controller({ host: 'admin.example.com' })
export class AdminController {
  @Get()
  index(): string {
    return 'Admin page';
  }
}
```

동적 값 캡처:

```typescript
@Controller({ host: ':account.example.com' })
export class AccountController {
  @Get()
  getInfo(@HostParam('account') account: string) {
    return account;
  }
}
```

---

### 상태 공유

Nest에서는 대부분의 리소스(데이터베이스 커넥션 풀, 싱글턴 서비스 등)가 모든 요청 간에 공유됩니다. Node.js는 멀티 스레드가 아니므로 싱글턴 사용이 안전합니다.

특정 경우(요청 기반 캐싱, 요청 추적, 다중 테넌시 등)에는 요청 단위 라이프사이클이 필요할 수 있습니다.

---

### 비동기 처리

Nest는 `async` 함수를 완벽히 지원합니다. `Promise` 또는 RxJS `Observable`을 반환할 수 있습니다:

```typescript
@Get()
async findAll(): Promise<any[]> {
  return [];
}

@Get()
findAll(): Observable<any[]> {
  return of([]);
}
```

---

### 요청 본문

POST 요청에서 클라이언트 데이터를 받으려면 DTO(Data Transfer Object)를 사용합니다. 클래스 사용 권장:

#### create-cat.dto.ts

```typescript
export class CreateCatDto {
  name: string;
  age: number;
  breed: string;
}
```

#### cats.controller.ts

```typescript
@Post()
async create(@Body() createCatDto: CreateCatDto) {
  return 'This action adds a new cat';
}
```

---

### 쿼리 파라미터

`@Query()`를 사용하여 쿼리 파라미터를 추출:

```typescript
@Get()
async findAll(@Query('age') age: number, @Query('breed') breed: string) {
  return `This action returns all cats filtered by age: ${age} and breed: ${breed}`;
}
```

예: `GET /cats?age=2&breed=Persian`

복잡한 쿼리를 위해 Express는 `query parser` 옵션 사용:

```typescript
const app = await NestFactory.create<NestExpressApplication>(AppModule);
app.set('query parser', 'extended');
```

Fastify는 `querystringParser` 사용:

```typescript
const app = await NestFactory.create<NestFastifyApplication>(
  AppModule,
  new FastifyAdapter({
    querystringParser: (str) => qs.parse(str),
  }),
);
```

---

### 오류 처리

오류 처리는 별도 챕터에서 설명합니다.

---

### 전체 리소스 예시

#### cats.controller.ts

```typescript
import { Controller, Get, Query, Post, Body, Put, Param, Delete } from '@nestjs/common';
import { CreateCatDto, UpdateCatDto, ListAllEntities } from './dto';

@Controller('cats')
export class CatsController {
  @Post()
  create(@Body() createCatDto: CreateCatDto) {
    return 'This action adds a new cat';
  }

  @Get()
  findAll(@Query() query: ListAllEntities) {
    return `This action returns all cats (limit: ${query.limit} items)`;
  }

  @Get(':id')
  findOne(@Param('id') id: string) {
    return `This action returns a #${id} cat`;
  }

  @Put(':id')
  update(@Param('id') id: string, @Body() updateCatDto: UpdateCatDto) {
    return `This action updates a #${id} cat`;
  }

  @Delete(':id')
  remove(@Param('id') id: string) {
    return `This action removes a #${id} cat`;
  }
}
```

💡 **힌트**
Nest CLI의 스키매틱(generator)을 사용하면 보일러플레이트 코드를 자동으로 생성할 수 있습니다.

---

### 시작하기

`CatsController`를 정의했더라도, Nest가 이를 인식하려면 모듈에 등록해야 합니다:

#### app.module.ts

```typescript
import { Module } from '@nestjs/common';
import { CatsController } from './cats/cats.controller';

@Module({
  controllers: [CatsController],
})
export class AppModule {}
```

---

### 라이브러리 특화 방식

라이브러리 고유 응답 객체를 사용하려면 `@Res()`로 주입:

```typescript
import { Controller, Get, Post, Res, HttpStatus } from '@nestjs/common';
import { Response } from 'express';

@Controller('cats')
export class CatsController {
  @Post()
  create(@Res() res: Response) {
    res.status(HttpStatus.CREATED).send();
  }

  @Get()
  findAll(@Res() res: Response) {
     res.status(HttpStatus.OK).json([]);
  }
}
```

단점:

* 플랫폼 의존적
* 테스트 복잡
* Nest 기능(Interceptor, `@HttpCode()`, `@Header()`)과 호환성 손실

이를 보완하려면 passthrough 사용:

```typescript
@Get()
findAll(@Res({ passthrough: true }) res: Response) {
  res.status(HttpStatus.OK);
  return [];
}
```