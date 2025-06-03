# Controllers

Controller는 들어오는 request를 처리하고, client로 response를 반환하는 역할을 한다. Controller의 목적은 애플리케이션에서 특정 요청을 처리하는 것이다. 이때, 라우팅 메커니즘이 어떤 controller가 각 요청을 처리할지 결정한다. 일반적으로 controller는 여러 개의 route를 가지고, 각 route는 서로 다른 작업을 수행할 수 있다.

# Routing
다음 예제에서는 기본 컨트롤러를 정의한 데 필수적인 @Controller() 데코레이터를 사용한다. 여기서는 cats라는 path prefix를 지정한다. 컨트롤러에서 path prefix를 지정하면 관련된 route들을 그룹화 할 수 있고 반복되는 코드를 줄일 수 있다. 예를 들어, cats entity와 상호작용하는 route들을 /cats 경로 아래로 묶고 싶다면, @Controller('cats')처럼 지정하면 된다. 그러면 각 라우트에서 해당 경로를 반복해서 작성할 필요가 없다.

cats.controller.ts
```ts
import { Controller, Get } from '@nestjs/common';

@Controller('cats')
export class CatsController {
  @Get()
  findAll(): string {
    return 'This action returns all cats';
  }
}

```
