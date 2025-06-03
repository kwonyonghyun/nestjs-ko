### Pipes (파이프)

**파이프(Pipe)** 는 `@Injectable()` 데코레이터가 적용된 클래스이며, `PipeTransform` 인터페이스를 구현합니다.

---

**파이프의 대표적인 2가지 사용 사례는 다음과 같습니다:**

✅ **변환(transformation)**: 입력 데이터를 원하는 형식으로 변환 (예: 문자열 → 정수)
✅ **검증(validation)**: 입력 데이터를 평가한 뒤 유효하면 그대로 통과시키고, 그렇지 않으면 예외를 발생

---

두 경우 모두, 파이프는 **컨트롤러 라우트 핸들러의 인자(arguments)** 에 대해 동작합니다.
Nest는 메서드가 호출되기 **직전에** 파이프를 개입시켜, 해당 인자를 전달하기 전에 파이프가 먼저 처리하도록 합니다.
변환 또는 검증 작업은 이 시점에서 이루어지며, 이후 (변환된) 인자를 가지고 라우트 핸들러가 호출됩니다.

---

Nest에는 **내장된 파이프**들이 여러 개 제공됩니다.
또한 **사용자 정의 커스텀 파이프**도 만들 수 있습니다.

이 챕터에서는 먼저 **내장 파이프** 사용법을 소개한 후,
**커스텀 파이프 작성 방법**을 설명합니다.

---

💡 **힌트**
파이프는 **예외 처리 영역(exceptions zone)** 내에서 실행됩니다.
따라서 파이프에서 예외가 발생하면 글로벌 예외 필터(global exception filter) 또는 현재 컨텍스트에 적용된 예외 필터가 이를 처리합니다.

👉 파이프에서 예외가 발생하면 해당 컨트롤러 메서드는 실행되지 않습니다.
따라서 외부 입력 데이터를 검증하는 데 있어 파이프를 사용하는 것이 모범 사례입니다.

---

### 내장 파이프(Built-in pipes)

Nest에는 다음과 같은 **내장 파이프**가 기본 제공됩니다:

* `ValidationPipe`
* `ParseIntPipe`
* `ParseFloatPipe`
* `ParseBoolPipe`
* `ParseArrayPipe`
* `ParseUUIDPipe`
* `ParseEnumPipe`
* `DefaultValuePipe`
* `ParseFilePipe`
* `ParseDatePipe`

이들은 `@nestjs/common` 패키지에서 export 됩니다.

---

간단한 예제로 `ParseIntPipe` 를 사용해보겠습니다.
이 파이프는 **변환(transformation)** 용도로 사용됩니다.

메서드 핸들러 인자를 **정수(integer)** 로 변환하며, 변환에 실패하면 예외를 발생시킵니다.

---

### 파이프 바인딩(Binding pipes)

파이프를 사용하려면, 해당 파이프 인스턴스를 **적절한 컨텍스트에 바인딩**해야 합니다.

예제: `ParseIntPipe` 를 특정 라우트 핸들러에 바인딩

```typescript
@Get(':id')
async findOne(@Param('id', ParseIntPipe) id: number) {
  return this.catsService.findOne(id);
}
```

---

위와 같이 바인딩하면, 두 가지 경우만 발생합니다:

1️⃣ `findOne()` 메서드의 `id` 파라미터가 **숫자(number)** 로 변환되어 전달됨
2️⃣ 변환 실패 시 **예외 발생** → 핸들러 실행되지 않음

---

예: `GET localhost:3000/abc` 호출 시 발생하는 예외 응답:

```json
{
  "statusCode": 400,
  "message": "Validation failed (numeric string is expected)",
  "error": "Bad Request"
}
```

---

위 예제에서는 **클래스(파이프 클래스)** 를 전달했습니다.
Nest가 인스턴스를 생성하므로 **DI(의존성 주입)** 을 활용할 수 있습니다.

**옵션**을 전달해야 한다면, 직접 인스턴스를 생성하여 전달하면 됩니다:

```typescript
@Get(':id')
async findOne(
  @Param('id', new ParseIntPipe({ errorHttpStatusCode: HttpStatus.NOT_ACCEPTABLE }))
  id: number,
) {
  return this.catsService.findOne(id);
}
```

---

다른 `Parse*` 파이프들도 동일하게 사용합니다:

**쿼리 파라미터 예시:**

```typescript
@Get()
async findOne(@Query('id', ParseIntPipe) id: number) {
  return this.catsService.findOne(id);
}
```

**UUID 파이프 예시:**

```typescript
@Get(':uuid')
async findOne(@Param('uuid', new ParseUUIDPipe()) uuid: string) {
  return this.catsService.findOne(uuid);
}
```

💡 **힌트**
`ParseUUIDPipe()` 는 기본적으로 UUID **버전 3, 4, 5** 를 파싱합니다.
특정 버전만 허용하려면 옵션에서 지정할 수 있습니다.

---

**ValidationPipe** 사용법은 조금 다르므로, 다음 섹션에서 다룹니다.

💡 **힌트**
검증 파이프 사용법은 [Validation techniques](https://docs.nestjs.com/techniques/validation)에서 다양한 예제를 볼 수 있습니다.

---

### 커스텀 파이프(Custom pipes)

Nest는 강력한 `ParseIntPipe`, `ValidationPipe` 를 내장하고 있지만,
**커스텀 파이프**도 작성할 수 있습니다.

먼저 단순한 **ValidationPipe** 를 직접 만들어 보겠습니다:

#### validation.pipe.ts

```typescript
import { PipeTransform, Injectable, ArgumentMetadata } from '@nestjs/common';

@Injectable()
export class ValidationPipe implements PipeTransform {
  transform(value: any, metadata: ArgumentMetadata) {
    return value;
  }
}
```

💡 **힌트**
`PipeTransform<T, R>` 는 제네릭 인터페이스입니다:

* `T`: 입력값 타입
* `R`: `transform()` 메서드 반환 타입

---

**transform()** 메서드는 다음 두 가지 인자를 받습니다:

* `value`: 현재 처리 중인 메서드 인자 값
* `metadata`: 해당 인자의 메타데이터

`ArgumentMetadata` 타입:

```typescript
export interface ArgumentMetadata {
  type: 'body' | 'query' | 'param' | 'custom';
  metatype?: Type<unknown>;
  data?: string;
}
```

속성 설명:

| 속성       | 설명                                  |
| -------- | ----------------------------------- |
| type     | 인자의 위치 (body, query, param, custom) |
| metatype | 인자의 메타타입 (예: `String`)              |
| data     | 데코레이터에 전달한 값 (@Body('key'))         |

⚠️ **주의**
**TypeScript 인터페이스는 트랜스파일 시 사라지므로**, 인자의 타입이 인터페이스일 경우 `metatype` 은 `Object` 가 됩니다.

---

### 스키마 기반 검증(Schema based validation)

예를 들어 `CatsController` 의 `create()` 메서드에서는 **POST 요청 바디**를 검증하고 싶습니다:

```typescript
@Post()
async create(@Body() createCatDto: CreateCatDto) {
  this.catsService.create(createCatDto);
}
```

`CreateCatDto`:

```typescript
export class CreateCatDto {
  name: string;
  age: number;
  breed: string;
}
```

---

검증을 **핸들러 내부에서 직접 수행**하면 **SRP(Single Responsibility Principle)** 를 위반하게 됩니다.
따라서 **파이프**로 처리하는 것이 가장 깔끔한 방법입니다.

---

**Zod** 라이브러리를 이용한 예시:

```bash
$ npm install --save zod
```

#### ZodValidationPipe

```typescript
import { PipeTransform, ArgumentMetadata, BadRequestException } from '@nestjs/common';
import { ZodSchema } from 'zod';

export class ZodValidationPipe implements PipeTransform {
  constructor(private schema: ZodSchema) {}

  transform(value: unknown, metadata: ArgumentMetadata) {
    try {
      const parsedValue = this.schema.parse(value);
      return parsedValue;
    } catch (error) {
      throw new BadRequestException('Validation failed');
    }
  }
}
```

#### Zod 스키마 예시:

```typescript
import { z } from 'zod';

export const createCatSchema = z
  .object({
    name: z.string(),
    age: z.number(),
    breed: z.string(),
  })
  .required();

export type CreateCatDto = z.infer<typeof createCatSchema>;
```

#### 바인딩:

```typescript
@Post()
@UsePipes(new ZodValidationPipe(createCatSchema))
async create(@Body() createCatDto: CreateCatDto) {
  this.catsService.create(createCatDto);
}
```

💡 **힌트**
`@UsePipes()` 데코레이터는 `@nestjs/common` 패키지에서 가져옵니다.

⚠️ **주의**
`zod` 사용 시 `tsconfig.json` 에서 **strictNullChecks** 설정이 필요합니다.

---

### class-validator 사용

⚠️ **주의**
이 방법은 TypeScript 기반에서만 가능합니다. (Vanilla JS에서는 불가)

---

설치:

```bash
$ npm i --save class-validator class-transformer
```

#### CreateCatDto 수정:

```typescript
import { IsString, IsInt } from 'class-validator';

export class CreateCatDto {
  @IsString()
  name: string;

  @IsInt()
  age: number;

  @IsString()
  breed: string;
}
```

---

#### ValidationPipe:

```typescript
import { PipeTransform, Injectable, ArgumentMetadata, BadRequestException } from '@nestjs/common';
import { validate } from 'class-validator';
import { plainToInstance } from 'class-transformer';

@Injectable()
export class ValidationPipe implements PipeTransform<any> {
  async transform(value: any, { metatype }: ArgumentMetadata) {
    if (!metatype || !this.toValidate(metatype)) {
      return value;
    }
    const object = plainToInstance(metatype, value);
    const errors = await validate(object);
    if (errors.length > 0) {
      throw new BadRequestException('Validation failed');
    }
    return value;
  }

  private toValidate(metatype: Function): boolean {
    const types: Function[] = [String, Boolean, Number, Array, Object];
    return !types.includes(metatype);
  }
}
```

---

#### 사용:

```typescript
@Post()
async create(
  @Body(new ValidationPipe()) createCatDto: CreateCatDto,
) {
  this.catsService.create(createCatDto);
}
```

**Parameter-scoped** 파이프는 특정 파라미터에만 적용됩니다.

---

### 글로벌 파이프(Global scoped pipes)

`ValidationPipe` 는 매우 범용적이므로, **전역 파이프**로 등록하면 좋습니다:

```typescript
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.useGlobalPipes(new ValidationPipe());
  await app.listen(process.env.PORT ?? 3000);
}
bootstrap();
```

⚠️ **주의**
`useGlobalPipes()` 는 **게이트웨이(gateways)** 및 **마이크로서비스(microservices)** 에는 적용되지 않습니다.
일반 HTTP 앱에는 적용됩니다.

---

DI가 필요한 경우 모듈에서 **APP\_PIPE** 사용:

```typescript
import { Module } from '@nestjs/common';
import { APP_PIPE } from '@nestjs/core';

@Module({
  providers: [
    {
      provide: APP_PIPE,
      useClass: ValidationPipe,
    },
  ],
})
export class AppModule {}
```

---

### 내장 ValidationPipe

Nest에는 이미 **ValidationPipe** 가 내장되어 있습니다.
직접 구현할 필요 없이 다양한 옵션을 제공합니다.
자세한 내용과 예제는 [공식 문서](https://docs.nestjs.com/techniques/validation)에서 확인하세요.

---

### 변환(Transformation) 사용 사례

**파이프는 입력 데이터를 변환**할 수도 있습니다.
`transform()` 메서드가 반환하는 값이 최종 인자로 사용됩니다.

예시: 문자열을 정수로 변환하는 `ParseIntPipe` (커스텀 버전):

#### parse-int.pipe.ts

```typescript
import { PipeTransform, Injectable, ArgumentMetadata, BadRequestException } from '@nestjs/common';

@Injectable()
export class ParseIntPipe implements PipeTransform<string, number> {
  transform(value: string, metadata: ArgumentMetadata): number {
    const val = parseInt(value, 10);
    if (isNaN(val)) {
      throw new BadRequestException('Validation failed');
    }
    return val;
  }
}
```

바인딩:

```typescript
@Get(':id')
async findOne(@Param('id', new ParseIntPipe()) id) {
  return this.catsService.findOne(id);
}
```

---

다른 예시: User 엔터티 변환 파이프

```typescript
@Get(':id')
findOne(@Param('id', UserByIdPipe) userEntity: UserEntity) {
  return userEntity;
}
```

---

### 기본값 제공(Providing defaults)

`Parse*` 파이프들은 **null** 또는 **undefined** 값에서 예외를 발생시킵니다.
따라서 기본값을 제공하려면 `DefaultValuePipe` 를 사용하면 됩니다:

```typescript
@Get()
async findAll(
  @Query('activeOnly', new DefaultValuePipe(false), ParseBoolPipe) activeOnly: boolean,
  @Query('page', new DefaultValuePipe(0), ParseIntPipe) page: number,
) {
  return this.catsService.findAll({ activeOnly, page });
}
```