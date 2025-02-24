[data:image/svg+xml,%3csvg%20xmlns=%27http://www.w3.org/2000/svg%27%20version=%271.1%27%20width=%271200%27%20height=%27630%27/%3e](data:image/svg+xml,%3csvg%20xmlns=%27http://www.w3.org/2000/svg%27%20version=%271.1%27%20width=%271200%27%20height=%27630%27/%3e)

[https://www.tomray.dev/_next/image?url=%2Fstatic%2Fimages%2Fnestjs-unit-testing%2Fnestjs-unit-testing.png&w=3840&q=75](https://www.tomray.dev/_next/image?url=%2Fstatic%2Fimages%2Fnestjs-unit-testing%2Fnestjs-unit-testing.png&w=3840&q=75)

이 튜토리얼은 NestJS의 단위 테스트(테스트 더블을 사용한 모의 포함)에 대해 자세히 설명합니다.

이 튜토리얼을 최대한 활용하려면 코딩과 함께 **`npm run test:watch`**로컬로 실행하여 우리가 작성한 테스트가 실제로 실행되는 모습을 확인하는 것이 좋습니다!

이 튜토리얼의 코드를 확인하려면 [**Github repo를**](https://github.com/tomwray13/nestjs-unit-testing) 참조하세요 .

준비가 됐나요? 갑시다!

![https://www.tomray.dev/static/images/nestjs.png](https://www.tomray.dev/static/images/nestjs.png)

## **단위 테스트란 무엇입니까?**

단위 테스트는 작은 동작을 확인하는 코드의 자동화입니다.

올바르게 구현되면 단위 테스트는 탁월한 투자 수익을 얻을 수 있습니다.

단위 테스트를 추가하면 새로운 기능을 추가하거나 기존 코드를 리팩터링할 때 자신(또는 프로젝트에 참여하는 다른 동료)의 미래 버전이 빠르고 자신 있게 작업할 수 있도록 프로젝트에 투자하게 됩니다.

단위 테스트도 격리되어야 합니다. 즉, 테스트가 작동하기 위해 다른 종속성에 의존하지 않는다는 의미입니다.

제 생각에는 이러한 격리 문제를 처리하는 것이 NestJS에서 단위 테스트를 작성하는 데 가장 어려운 부분이므로 이 튜토리얼에서는 많은 예제를 다룰 것입니다.

## **NestJS에서 좋은 단위 테스트를 만드는 것은 무엇입니까?**

단위 테스트가 무엇인지 이해하고 나면 대다수의 개발자가 단위 테스트가 훌륭한 아이디어라고 생각한다고 말하는 것이 타당하다고 생각합니다.

그러나 제대로 구현되지 않은 단위 테스트는 자산보다는 책임이 더 커질 수 있습니다.

다음은 이 튜토리얼 전체의 예제에서 구현될 몇 가지 규칙입니다.

- Alignment-Act-Assert 패러다임에 따라 각 테스트를 작게 유지하십시오.
- 최종 결과/행동에 집중
- 구현 세부 사항을 테스트하지 않음으로써 취약성 방지

이 주제에 대한 책 전체가 작성되었으므로(나는 단위 테스트에 관한 [**Vladimir의 책을**](https://enterprisecraftsmanship.com/book/) 추천합니다 ), 좋은 단위 테스트를 만드는 방법에 대한 자신의 의견을 형성하려면 더 많은 책을 읽어야 합니다.

## **간단한 CRUD 예(모의 없음)**

NestJS의 단위 테스트뿐만 아니라 단위 테스트를 실습으로 사용하는 완전 초보자라면 이 섹션이 적합합니다! 테스트 더블과 모의에 대해 배우고 싶다면 다음 섹션으로 이동하세요.

우리는 트윗을 처리하기 위한 몇 가지 기본 CRUD 기능을 갖춘 서비스를 구축한 다음 이에 대한 몇 가지 단위 테스트를 작성할 것입니다.

다음과 같은 모듈을 추가하는 것부터 시작해 보겠습니다 **`tweets`**.

`nest g module tweets`

그런 다음 트윗 서비스를 추가합니다.

`nest g service tweets`

이 2개의 명령을 실행하면 내부에 3개의 파일이 포함된 디렉터리가 생성됩니다 **`tweets`**.

```jsx
src  / tweets / tweets.module.ts
								tweets.service.spec.ts
								tweets.service.ts
```

NestJS가 우리를 위해 생성한 테스트 파일을 엽니다. 파일 **`tweets.service.spec.ts`**은 다음과 같습니다.

```tsx
// **tweets.service.spec.ts**

import { Test, TestingModule } from '@nestjs/testing';
import { TweetsService } from './tweets.service';

describe('TweetsService', () => {
  let service: TweetsService;

  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      providers: [TweetsService],
    }).compile();

    service = module.get<TweetsService>(TweetsService);
  });

  it('should be defined', () => {
    expect(service).toBeDefined();
  });
});
```

아직 수행하지 않았다면 다음 명령을 사용하여 로컬에서 테스트를 실행하고 있는지 확인하세요.

`npm run test:watch`

터미널은 다음을 출력해야 합니다.

[data:image/svg+xml,%3csvg%20xmlns=%27http://www.w3.org/2000/svg%27%20version=%271.1%27%20width=%271200%27%20height=%27397%27/%3e](data:image/svg+xml,%3csvg%20xmlns=%27http://www.w3.org/2000/svg%27%20version=%271.1%27%20width=%271200%27%20height=%27397%27/%3e)

[https://www.tomray.dev/_next/image?url=%2Fstatic%2Fimages%2Fnestjs-unit-testing%2Fnpm-run-test-watch.png&w=3840&q=75](https://www.tomray.dev/_next/image?url=%2Fstatic%2Fimages%2Fnestjs-unit-testing%2Fnpm-run-test-watch.png&w=3840&q=75)

잘 하셨어요! 이제 단위 테스트가 핫 리로딩으로 실행됩니다. 코드를 변경하면 테스트가 다시 실행됩니다.

더 많은 테스트를 작성하기 전에 자동 생성된 NestJS 테스트 파일을 살펴보고 무슨 일이 일어나고 있는지 살펴보겠습니다.

우선 파일명에 가 포함되어 있으므로 **`spec.ts`**Jest [**(**](https://jestjs.io/) NestJS에서 사용하는 테스트 프레임워크)가 자동으로 테스트를 선택합니다. 프로젝트의 다른 파일은 **`spec.ts`**Jest에 의해 선택됩니다.

파일 자체 내에서는 블록으로 시작됩니다 **`describe`**.

**트윗.service.spec.ts**

`describe('TweetsService', () => { *// ...*});`

블록 의 목적은 **`describe`**관련 테스트를 그룹화하는 것이므로 여기서는 **`TweetsService`**.

다음으로 후크가 있습니다 **`beforeEach`**.

**트윗.service.spec.ts**

`import { Test, TestingModule } from '@nestjs/testing'; import { TweetsService } from './tweets.service';

describe('TweetsService', () => { let service: TweetsService;

beforeEach(async () => { const module: TestingModule = await Test.createTestingModule({ providers: [TweetsService], }).compile();

```
service = module.get<TweetsService>(TweetsService);
```

});

_// ..._});`

후크 **`beforeEach`**는 각 테스트를 실행하기 전에 발생해야 하는 모든 설정 작업을 처리합니다.

따라서 **`beforeEach`**여기서 후크가 수행하는 작업은 NestJS 내장 **`Test`**클래스를 사용하여 격리된 NestJS 런타임을 생성하는 것입니다(따라서 종속성 주입과 같은 모든 NestJS 동작을 얻을 수 있습니다).

이 런타임은 클래스를 사용할 때 정의한 내용으로 제한됩니다 **`Test`**. 위의 예에서는 **`TweetsService`**.

따라서 이 설정을 통해 **`TweetsService`**.

자동 생성된 NestJS 테스트 파일에서 검토할 마지막 부분은 테스트입니다!

다음과 같은 테스트가 표시됩니다 **`it should be defined`**.

**트윗.service.spec.ts**

`import { Test, TestingModule } from '@nestjs/testing'; import { TweetsService } from './tweets.service';

describe('TweetsService', () => { *// ...*it('should be defined', () => { expect(service).toBeDefined(); }); });`

**`expect`**Jest의 함수를 사용하는 것은 [**어설션**](https://en.wikipedia.org/wiki/Test_assertion) 입니다 .

모든 **`expect`**기능에는 특정 조건이 충족되는지 확인하는 또 다른 방법이 연결되어 있습니다.

이 예에서는 입니다 **`toBeDefined()`**. 따라서 이 테스트에서는 가 정의되었는지 확인합니다 **`TweetsService`**.

단위 테스트에 대한 좀 더 구체적인 예를 살펴보겠습니다.

파일 내부에 **`tweets.service.ts`**몇 가지 CRUD 메서드를 추가합니다.

**트윗.service.ts**

`import { Injectable } from '@nestjs/common';

@Injectable() export class TweetsService { tweets: string[] = [];

createTweet(tweet: string) { if (tweet.length > 100) { throw new Error(`Tweet too long`); } this.tweets.push(tweet); return tweet; }

updateTweet(tweet: string, id: number) { const tweetToUpdate = this.tweets[id]; if (!tweetToUpdate) { throw new Error(`This Tweet does not exist`); } if (tweet.length > 100) { throw new Error(`Tweet too long`); } this.tweets[id] = tweet; return tweet; }

getTweets() { return this.tweets; }

deleteTweet(id: number) { const tweetToDelete = this.tweets[id]; if (!tweetToDelete) { throw new Error(`This Tweet does not exist`); } const deletedTweet = this.tweets.splice(id, 1); return deletedTweet; } }`

이 예제를 단순하게 유지하기 위해 이라는 클래스에 공개 필드를 생성하여 상태가 메모리에서 처리되는 것을 확인할 수 있습니다 **`tweets`**.

트윗 서비스의 첫 번째 메소드에 대해 테스트해야 할 사항을 고려하는 것부터 시작해 보겠습니다 **`createTweet`**.

**트윗.service.ts**

`import { Injectable } from '@nestjs/common';

@Injectable() export class TweetsService { tweets: string[] = [];

createTweet(tweet: string) { if (tweet.length > 100) { throw new Error(`Tweet too long`); } this.tweets.push(tweet); return tweet; } }`

어떤 테스트를 작성해야 할지 파악하는 데 도움이 되도록 다음 질문을 스스로에게 물어볼 수 있습니다.

"여기서 의도된 동작은 무엇입니까? 코드가 내려갈 수 있는 경로가 여러 개 있습니까?"

방법 에 대한 다음 질문에 답해 보겠습니다 **`createTweets`**.

1. 유효한 트윗이 생성되면 해당 트윗이 상태에 추가됩니다.
2. 유효한 트윗이 생성되면 메서드는 해당 트윗을 반환합니다.
3. 길이가 100자를 초과하는 트윗은 허용되지 않습니다.

좋아요, 이제 이 3가지 시나리오를 다루기 위한 몇 가지 테스트 작성을 시작하겠습니다!

파일 에 **`tweets.service.spec.ts`**첫 번째 테스트를 추가합니다.

**트윗.service.spec.ts**

`import { Test, TestingModule } from '@nestjs/testing'; import { TweetsService } from './tweets.service';

describe('TweetsService', () => { let service: TweetsService;

beforeEach(async () => { const module: TestingModule = await Test.createTestingModule({ providers: [TweetsService], }).compile();

```
service = module.get<TweetsService>(TweetsService);
```

});

describe('createTweet', () => { it('should create tweet', () => { _// Arrange_ service.tweets = []; const payload = 'This is my tweet';

```
  *// Act*const tweet = service.createTweet(payload);

  *// Assert*expect(tweet).toBe(payload);
  expect(service.tweets).toHaveLength(1);
});
```

}); });`

이 테스트에서 무슨 일이 일어나고 있는지 분석해 보겠습니다.

1. **배열**
    
    : 테스트 전에 페이로드를 변수에 넣어 약간의 설정을 완료했습니다.
    
2. **Act**: Call the **`createTweet`** method, the bit of behavior we are testing
    
3. **Assert**: Declare the intended outcome. Here we've checked that the **`createTweet`** method returns the tweet that was passed into the payload. We've also tested that the in-memory state has been updated with the new tweet.
    

So we've now covered 2 of the 3 scenarios:

1. ~~When a valid tweet is created, it adds the tweet to state~~
2. ~~When a valid tweet is created, the method returns the respective tweet~~
3. A tweet greater than 100 characters in length should not be allowed

Let's add another test to handle the 3rd scenario!

**tweets.service.spec.ts**

`import { Test, TestingModule } from '@nestjs/testing'; import { TweetsService } from './tweets.service';

describe('TweetsService', () => { let service: TweetsService;

beforeEach(async () => { const module: TestingModule = await Test.createTestingModule({ providers: [TweetsService], }).compile();

```
service = module.get<TweetsService>(TweetsService);
```

});

describe('createTweet', () => { it('should create tweet', () => { _// ..._});

```
it('should prevent tweets created which are over 100 characters', () => {
  *// Arrange*const payload =
    'This is a long tweet over 100 characters This is a long tweet over 100 characters This is a long t...';

  *// Act*const tweet = () => {
    return service.createTweet(payload);
  };

  *// Assert*expect(tweet).toThrowError();
});
```

}); });`

Let's break down this test:

1. **Arrange**: We've done a bit of setup before the test by putting the payload in a variable
2. **Act**: Call the **`createTweet`** method inside a **`tweet()`** function.
3. **Assert**: Declare the intended outcome. Here we've checked that the **`createTweet`** method throws an error with the payload passed in

The [**Arrange-Act-Assert**](https://automationpanda.com/2020/07/07/arrange-act-assert-a-pattern-for-writing-good-tests/) is a good pattern to follow when writing tests. It helps keep the tests structured.

I won't include the Arrange-Act-Assert comments in any of the following examples, but I will follow this pattern so try to keep an eye out for it.

Time to practice your unit testing skills! Go ahead and write unit tests for the other methods in the **`TweetsService`**. You can compare your work with the tests I've added in the [**Github repo**](https://github.com/tomwray13/nestjs-unit-testing).

Once you've done that, let's move on to the next part of this tutorial.

![https://www.tomray.dev/static/images/nestjs.png](https://www.tomray.dev/static/images/nestjs.png)

**Free NestJS Course**Want to use NestJS to it's full potential and understand how it really works? Check out my free course which covers concepts like Dependency Injection, IoC Containers and more:**Your email addressGet free course**

## **Mocking with test doubles in NestJS**

So the previous example was quite simple. In the **`TweetsService`**, we used 0 dependencies (i.e. nothing was passed into the **`constructor`**), which simplifies our unit tests.

However, if you're working on a NestJS project, you'll soon find yourself adding dependencies to your services and controllers.

For example, the **`TweetsService`** we defined in the previous step could evolve into something like:

**tweets.service.ts**

`import { Injectable } from '@nestjs/common'; import { HttpService } from '@nestjs/axios'; import { ConfigService } from '@nestjs/config';

@Injectable() export class TweetsService { constructor( private readonly httpService: HttpService, private readonly configService: ConfigService ) {} _// ..._}`

Adding these dependencies, however, will create some challenges for our unit tests.

Our methods can now include any one of the dependencies, so the scope of the unit tests has now extended to consider the behavior of the dependencies as well. Also, the dependencies can be asynchronous (e.g. using the **`HttpService`** to make an HTTP request) which can affect the performance of our unit tests.

The solution? Test doubles.

In our unit tests, we can effectively swap out specific dependencies with a 'test double', so when the test runs it uses the test double instead of the actual dependency.

Let's dive into some examples:

### **Example: Mocking HTTP requests in NestJS**

Let's start by creating a new **`pokemon`** module:

`nest g module pokemon`

And then add a pokemon service:

`nest g service pokemon`

Running these 2 commands will create a directory called **`pokemon`** with 3 files inside:

`src / pokemon / pokemon.module.ts pokemon.service.spec.ts pokemon.service.ts`

We're going to use the [**NestJS HTTP module**](https://docs.nestjs.com/techniques/http-module) to fetch data from the [**Pokemon API**](https://pokeapi.co/).

Let's install the relevant package:

`npm i --save @nestjs/axios@0.1.0`

When this article was published, there was [**a bug**](https://github.com/axios/axios/issues/5101) between Axios and Jest which is why I've installed the **`0.1.0`** version!

In order to use the NestJS HTTP module in our service, we need to make it available for dependency injection by importing it into the module:

**pokemon.module.ts**

`import { HttpModule } from '@nestjs/axios'; import { Module } from '@nestjs/common'; import { PokemonService } from './pokemon.service';

@Module({ imports: [HttpModule], providers: [PokemonService], }) export class PokemonModule {}`

Now we can use the **`HttpModule`** in the service by passing it into the constructor.

Inside the service file let's add a simple **`getPokemon`** method that returns the name of a Pokemon:

**pokemon.service.ts**

`import { HttpService } from '@nestjs/axios'; import { BadRequestException, Injectable, InternalServerErrorException, } from '@nestjs/common';

@Injectable() export class PokemonService { constructor(private httpService: HttpService) {}

async getPokemon(id: number) { if (id < 1 || id > 151) { throw new BadRequestException(`Invalid Pokemon ID`); }

```
const { data } = await this.httpService.axiosRef({
  url: `https://pokeapi.co/api/v2/pokemon/${id}`,
  method: `GET`,
});

if (!data || !data.species || !data.species.name) {
  throw new InternalServerErrorException();
}

return data.species.name;
```

} }`

Here is some unit tests we will add for the above **`getPokemon()`** method:

1. An ID less than **`1`** should return a **`BadRequestException`**
2. An ID greater than **`151`** should return a **`BadRequestException`**
3. An ID between **`1`** and **`151`** returns the name of the respective pokemon
4. If the response from the Pokemon API isn't what we expect, then return a **`InternalServerErrorException`**

Let's dive in!

NestJS auto-generated the test file for the Pokemon service:

**pokemon.service.spec.ts**

`import { Test, TestingModule } from '@nestjs/testing'; import { PokemonService } from './pokemon.service';

describe('PokemonService', () => { let service: PokemonService;

beforeEach(async () => { const module: TestingModule = await Test.createTestingModule({ providers: [PokemonService], }).compile();

```
service = module.get<PokemonService>(PokemonService);
```

});

it('should be defined', () => { expect(service).toBeDefined(); }); });`

With **`npm run test:watch`** running, however, you should see an error stating something like **`Nest can't resolve dependencies of the Pokemon Service`**:

[data:image/svg+xml,%3csvg%20xmlns=%27http://www.w3.org/2000/svg%27%20version=%271.1%27%20width=%271200%27%20height=%27559%27/%3e](data:image/svg+xml,%3csvg%20xmlns=%27http://www.w3.org/2000/svg%27%20version=%271.1%27%20width=%271200%27%20height=%27559%27/%3e)

data:image/gif;base64,R0lGODlhAQABAIAAAAAAAP///yH5BAEAAAAALAAAAAABAAEAAAIBRAA7

You're seeing this error because Jest is trying to run the test but the testing instance is missing a dependency (the HTTP dependency, to be precise!).

So we need to provide the HTTP dependency to the testing module so that when the tests are run, the dependency we pass into the testing module can be used inside the **`PokemonService`**.

This is one of the biggest benefits of dependency injection - you can swap out the dependency with a more appropriate alternative for testing purposes.

For example, we could pass in the actual **`HttpService`** which makes HTTP requests to the Pokemon API **OR** we could pass in a 'test double' of the **`HttpService`** which essentially pretends to make the HTTP requests.

This is super powerful!

With all that being said, for demonstration purposes let's start by implementing the tests which will make the actual HTTP requests to the Pokemon API. We'll then update our tests to mock the HTTP requests.

To fix the dependency error inside the **`pokemon.service.spec.ts`** file, we just need to add the **`HttpModule`** as an import to the test module:

**pokemon.service.spec.ts**

`import { HttpModule } from '@nestjs/axios'; import { Test, TestingModule } from '@nestjs/testing'; import { PokemonService } from './pokemon.service';

describe('PokemonService', () => { let pokemonService: PokemonService; _// renamed variable to pokemonService_beforeEach(async () => { const module: TestingModule = await Test.createTestingModule({ imports: [HttpModule], providers: [PokemonService], }).compile();

```
pokemonService = module.get<PokemonService>(PokemonService);
```

});

it('should be defined', () => { expect(pokemonService).toBeDefined(); }); });`

In the above test, I also renamed the **`service`** variable to **`pokemonService`** to make it more clear.

You should no longer see any testing errors in your terminal!

Let's recap the 4 unit tests we'd like to add:

1. An ID less than **`1`** should return a **`BadRequestException`**
2. An ID greater than **`151`** should return a **`BadRequestException`**
3. An ID between **`1`** and **`151`** returns the name of the respective pokemon
4. If the response from the Pokemon API isn't what we expect, then return a **`InternalServerErrorException`**

Let's add those now:

**pokemon.service.spec.ts**

`import { HttpModule } from '@nestjs/axios'; import { BadRequestException } from '@nestjs/common'; import { Test, TestingModule } from '@nestjs/testing'; import { PokemonService } from './pokemon.service';

describe('PokemonService', () => { let pokemonService: PokemonService;

beforeEach(async () => { const module: TestingModule = await Test.createTestingModule({ imports: [HttpModule], providers: [PokemonService], }).compile();

```
pokemonService = module.get<PokemonService>(PokemonService);
```

});

it('should be defined', () => { expect(pokemonService).toBeDefined(); });

describe('getPokemon', () => { it('pokemon ID less than 1 should throw error', async () => { const getPokemon = pokemonService.getPokemon(0);

```
  await expect(getPokemon).rejects.toBeInstanceOf(BadRequestException);
});

it('pokemon ID greater than 151 should throw error', async () => {
  const getPokemon = pokemonService.getPokemon(152);

  await expect(getPokemon).rejects.toBeInstanceOf(BadRequestException);
});

it('valid pokemon ID to return the pokemon name', async () => {
  const getPokemon = pokemonService.getPokemon(1);

  await expect(getPokemon).resolves.toBe('bulbasaur');
});
```

}); });`

In the tests, we're dealing with asynchronous functions as the **`getPokemon`** method returns a promise. That's what the **`resolves`** and **`rejects`** methods from Jest are handling.

Of course, you don't need to create the **`getPokemon`** variable, but I find it a bit cleaner and makes the test easier to read instead of passing the method call directly into the Jest **`expect`**.

With **`npm run test:watch`** running, the tests should be passing!

As we've not done any mocking on the **`Http`** dependency, when the **`getPokemon`** methods are called in the 2 tests, the actual **`Http`** dependency is being used (so in this case, attempting an HTTP request to the Pokemon API).

This has the following challenges:

- 🐌 Making network requests inside your tests will cause your test suite to be slower
- 💰 Perhaps the API you're using costs money, so you only want the API to be called during runtime and not during tests
- 🧘‍♀️ The test should focus on the intended behavior of the method, regardless of any dependencies

So let's now explore how we can mock the HTTP service in NestJS.

To implement mocking in NestJS, I recommend using the [**@golevelup/ts-jest**](https://www.npmjs.com/package/@golevelup/ts-jest) package.

Using the **`createMock`** utility function from this package will give you all the properties and sub-properties for the thing you want to mock. The alternative is manually defining all the properties you need in a custom object (which can get quite repetitive).

When combined with Jest's mocking capabilities, it's very powerful.

Start by installing the package:

`npm install @golevelup/ts-jest`

So our objective here is to stop using the actual HTTP service which makes requests to the Pokemon API and instead implement a mocked version.

So instead of using the **`HttpModule`** as an import in our testing module, we're going to replace the **`HttpService`** directly using the **`createMock()`** utility function:

**pokemon.service.spec.ts**

`import { createMock, DeepMocked } from '@golevelup/ts-jest'; import { HttpService } from '@nestjs/axios'; import { BadRequestException } from '@nestjs/common'; import { Test, TestingModule } from '@nestjs/testing'; import { PokemonService } from './pokemon.service';

describe('PokemonService', () => { let pokemonService: PokemonService; let httpService: DeepMocked<HttpService>;

beforeEach(async () => { const module: TestingModule = await Test.createTestingModule({ providers: [ PokemonService, { provide: HttpService, useValue: createMock<HttpService>(), }, ], }).compile();

```
pokemonService = module.get<PokemonService>(PokemonService);
httpService = module.get(HttpService);
```

});

_// ... tests_});`

The **`DeepMocked<HttpService>`** type makes sure the **`httpService`** variable now has all the properties and sub-properties available in your IDE for auto-completion.

What impact does the above changes have on your tests?

Well, for example, one of the tests we defined was to ensure when a valid Pokemon ID is passed in, it returns the respective Pokemon name:

**pokemon.service.spec.ts**

`*// ...*it('valid pokemon ID to return the pokemon name', async () => { const getPokemon = pokemonService.getPokemon(1);

await expect(getPokemon).resolves.toBe('bulbasaur'); });`

In this test, we're calling the **`getPokemon`** method. If you look in the **`getPokemon`** method, you'll see the **`HttpService`** is used to make a request to the Pokemon API.

We've basically replaced the **`HttpService`** with a dummy function that we can control in our tests.

So if you're following on with the tutorial and still have **`npm run test:watch`** running, you'll see an error like this:

[data:image/svg+xml,%3csvg%20xmlns=%27http://www.w3.org/2000/svg%27%20version=%271.1%27%20width=%271200%27%20height=%27632%27/%3e](data:image/svg+xml,%3csvg%20xmlns=%27http://www.w3.org/2000/svg%27%20version=%271.1%27%20width=%271200%27%20height=%27632%27/%3e)

data:image/gif;base64,R0lGODlhAQABAIAAAAAAAP///yH5BAEAAAAALAAAAAABAAEAAAIBRAA7

This is because our dummy function isn't doing anything!

In this test, let's update the dummy function to 'mock' the response from the Pokemon API:

**pokemon.service.spec.ts**

`import { createMock, DeepMocked } from '@golevelup/ts-jest'; import { HttpService } from '@nestjs/axios'; import { BadRequestException } from '@nestjs/common'; import { Test, TestingModule } from '@nestjs/testing'; import { PokemonService } from './pokemon.service';

describe('PokemonService', () => { let pokemonService: PokemonService; let httpService: DeepMocked<HttpService>;

beforeEach(async () => { _// setup..._});

describe('getPokemon', () => { *// other tests...*it('valid pokemon ID to return the pokemon name', async () => { httpService.axiosRef.mockResolvedValueOnce({ data: { species: { name: `bulbasaur` }, }, headers: {}, config: { url: '' }, status: 200, statusText: '', });

```
  const getPokemon = pokemonService.getPokemon(1);

  await expect(getPokemon).resolves.toBe('bulbasaur');
});
```

}); });`

We're implementing a mock to tell the test: "Hey, whenever you run this test, make sure the httpService call returns an Axios response with the specific object inside the **`data`** property".

Your test should now be passing.

By the way, you may have noticed that we're missing a test to cover the 4th scenario:

1. ~~An ID less than 1 should return a BadRequestException~~
2. ~~An ID greater than 151 should return a BadRequestException~~
3. ~~An ID between 1 and 151 returns the name of the respective pokemon~~
4. If the response from the Pokemon API isn't what we expect, then return an **`InternalServerErrorException`**

Let's add a final test for this:

**pokemon.service.spec.ts**

`import { createMock, DeepMocked } from '@golevelup/ts-jest'; import { HttpService } from '@nestjs/axios'; import { BadRequestException } from '@nestjs/common'; import { Test, TestingModule } from '@nestjs/testing'; import { PokemonService } from './pokemon.service';

describe('PokemonService', () => { let pokemonService: PokemonService; let httpService: DeepMocked<HttpService>;

beforeEach(async () => { _// setup..._});

describe('getPokemon', () => { *// other tests...*it('if Pokemon API response unexpectedly changes, throw an error', async () => { httpService.axiosRef.mockResolvedValueOnce({ data: `Unexpected data`, headers: {}, config: { url: '' }, status: 200, statusText: '', });

```
  const getPokemon = pokemonService.getPokemon(1);

  await expect(getPokemon).rejects.toBeInstanceOf(
    InternalServerErrorException,
  );
});
```

}); });`

One final optimization we can make to our **`PokemonService`** test file is clean up the **`beforeEach`** hook a little.

Here's how it looks again:

**pokemon.service.spec.ts**

`*// imports...*describe('PokemonService', () => { let pokemonService: PokemonService; let httpService: DeepMocked<HttpService>;

beforeEach(async () => { const module: TestingModule = await Test.createTestingModule({ providers: [ PokemonService, { provide: HttpService, useValue: createMock<HttpService>(), }, ], }).compile();

```
pokemonService = module.get<PokemonService>(PokemonService);
httpService = module.get(HttpService);
```

});

_// the tests..._});`

Let's say our **`PokemonService`** grows and we add more dependencies like caching and logging:

**pokemon.service.spec.ts**

`*// imports...*describe('PokemonService', () => { let pokemonService: PokemonService; let httpService: DeepMocked<HttpService>;

beforeEach(async () => { const module: TestingModule = await Test.createTestingModule({ providers: [ PokemonService, { provide: HttpService, useValue: createMock<HttpService>(), }, { provide: CacheService, useValue: createMock<CacheService>(), }, { provide: LoggingService, useValue: createMock<LoggingService>(), }, ], }).compile();

```
pokemonService = module.get<PokemonService>(PokemonService);
httpService = module.get(HttpService);
```

});

_// the tests..._});`

Adding a new object to the **`providers`** array can get a little tedious every time we want to mock a dependency.

Thankfully, NestJS released in V8 a feature called Auto mocking which makes this code much cleaner:

**pokemon.service.spec.ts**

`*// imports...*describe('PokemonService', () => { let pokemonService: PokemonService; let httpService: DeepMocked<HttpService>;

beforeEach(async () => { const module: TestingModule = await Test.createTestingModule({ providers: [ PokemonService ], }) .useMocker(createMock) .compile();

```
pokemonService = module.get<PokemonService>(PokemonService);
httpService = module.get(HttpService);
```

});

_// the tests..._});`

Passing **`createMock`** into the **`useMocker()`** method essentially mocks any dependency you haven't defined in the testing module.

Thank you [**Thiago Martins**](https://twitter.com/thiagomni) for [**the tip**](https://trilon.io/blog/advanced-testing-strategies-with-mocks-in-nestjs) on this!

Nice work!

You've now done a deep dive into implementing test doubles in NestJS 🤟.

## **Unit testing pipes in NestJS**

[**Pipes**](https://docs.nestjs.com/pipes) in NestJS are a way to validate and transform any inputs passed into your controllers.

There are a few built-in ones, like **`ParseIntPipe`** that converts the specified parameter into an integer, returning a **`404`** if the conversion fails.

Here's an example of how the **`ParseIntPipe`** could be implemented into a controller:

**pokemon.controller.ts**

`import { Controller, Get, Param, ParseIntPipe } from '@nestjs/common'; import { PokemonService } from './pokemon.service';

@Controller('pokemon') export class PokemonController { constructor(private readonly pokemonService: PokemonService) {}

@Get(':id') getPokemon(@Param('id', ParseIntPipe) id: number) { return this.pokemonService.getPokemon(id); } }`

Let's say that as well as the **`:id`** parameter being a number, we want to ensure that the value is greater than **`0`** and less than **`152`**. We could implement something like this in the controller:

**pokemon.controller.ts**

`import { BadRequestException, Controller, Get, Param, ParseIntPipe, } from '@nestjs/common'; import { PokemonService } from './pokemon.service';

@Controller('pokemon') export class PokemonController { constructor(private readonly pokemonService: PokemonService) {}

@Get(':id') getPokemon(@Param('id', ParseIntPipe) id: number) { if (id < 0 || id > 151) { throw new BadRequestException(`Invalid Pokemon ID`); } return this.pokemonService.getPokemon(id); } }`

Or we could create a custom pipe that handles this for us! Let's add this custom pipe, then add some unit tests for it.

Using the Nest CLI, create a new pipe:

`nest g pipe parse-pokemon-id`

This will create 2 new files in the root of your **`/src`** folder:

`src/ ... parse-pokemon-id.pipe.spec.ts parse-pokemon-id.pipe.ts`

You can move these files into the **`pokemon`** directory if you like!

Let's update the pipe to achieve the specified behavior:

**parse-pokemon.pipe.ts**

`import { BadRequestException, Injectable, PipeTransform } from '@nestjs/common';

@Injectable() export class ParsePokemonIdPipe implements PipeTransform { transform(value: string): number { const id = parseInt(value); if (isNaN(id)) { throw new BadRequestException( `Validation failed (numeric string is expected)`, ); } if (id < 1 || id > 151) { throw new BadRequestException(`ID must be between 1 and 151`); } return id; } }`

This pipe we've created could do with the following tests:

1. Should throw an error for nonnumbers
2. Should throw an error if the number is less than 1 or greater than 151
3. Should return the number if between 1 and 151

Open up the pipe test file Nest created for us:

**parse-pokemon-id.pipe.spec.ts**

`import { ParsePokemonIdPipe } from './parse-pokemon-id.pipe';

describe('ParsePokemonIdPipe', () => { it('should be defined', () => { expect(new ParsePokemonIdPipe()).toBeDefined(); }); });`

Let's use a similar structure to previous unit tests: use a **`beforeEach`** hook to create a reusable **`pipe`** variable across all the tests:

**parse-pokemon-id.pipe.spec.ts**

`import { ParsePokemonIdPipe } from './parse-pokemon-id.pipe';

describe('ParsePokemonIdPipe', () => { let pipe: ParsePokemonIdPipe;

beforeEach(() => { pipe = new ParsePokemonIdPipe(); });

_// ... now we can write tests using the pipe_});`

With that set-up, let's add the tests!

**parse-pokemon-id.pipe.spec.ts**

`import { BadRequestException } from '@nestjs/common'; import { ParsePokemonIdPipe } from './parse-pokemon-id.pipe';

describe('ParsePokemonIdPipe', () => { let pipe: ParsePokemonIdPipe;

beforeEach(() => { pipe = new ParsePokemonIdPipe(); });

it('should be defined', () => { expect(new ParsePokemonIdPipe()).toBeDefined(); });

it(`should throw error for non numbers`, () => { const value = () => pipe.transform(`hello`); expect(value).toThrowError(BadRequestException); });

it(`should throw error if number less than 1`, () => { const value = () => pipe.transform(`-34`); expect(value).toThrowError(BadRequestException); });

it(`should throw error if number greater than 151`, () => { const value = () => pipe.transform(`200`); expect(value).toThrowError(BadRequestException); });

it(`should return number if between 1 and 151`, () => { const value = () => pipe.transform(`5`); expect(value()).toBe(5); }); });`

잘 하셨어요! 이제 NestJS에서 파이프를 단위 테스트하는 방법을 알았습니다.

## **Github Actions를 사용하여 CI 파이프라인에서 NestJS 단위 테스트 자동화**

로컬에서 테스트를 실행하는 것도 좋지만, 풀 요청이나 특정 브랜치에 대한 푸시 커밋과 같은 주요 트리거에서 테스트가 자동화되도록 [**지속적인 통합 워크플로를 곧 설정하고 싶을 것입니다.**](https://docs.github.com/en/actions/automating-builds-and-tests/about-continuous-integration)

[**Github Actions를**](https://docs.github.com/en/actions/learn-github-actions/understanding-github-actions) 사용하면 쉽게 설정할 수 있으므로 자세히 살펴보겠습니다.

프로젝트 루트에 **`.github/workflows/`**디렉터리를 만듭니다.

디렉터리 에서 **`.github/workflows/`**테스트.yml이라는 새 파일을 만들고 다음 코드를 추가합니다.

**테스트.yml**

`name: Tests on: pull_request jobs: build: runs-on: ubuntu-latest steps: - uses: actions/checkout@v3 - name: Install modules run: npm ci - name: Run tests run: npm run test`

이 파일의 내용을 살펴보겠습니다.

- **`on:`**
    
    자동화가 트리거될 이벤트를 정의합니다 . 따라서 열려 있는 모든 끌어오기 요청은 자동화를 실행합니다.
    
- 다음으로, 가상 머신을 가동하고 다음 단계를 실행하는 작업을 정의했습니다.
    
    1. 먼저 Github Action [**actions/checkout@v3 을**](https://github.com/actions/checkout)
        
        실행합니다 . 이는 기본적으로 자동화가 저장소에 액세스할 수 있도록 허용하는 인증 단계입니다.
        
    2. 다음으로 머신은 다음을 사용하여 종속성을 설치합니다.**`npm ci`**
        
    3. 그리고 마지막으로 테스트를 실행합니다.
        

이제 코드를 Github에 푸시하고 풀 요청을 열면 테스트가 자동으로 실행되고 풀 요청 내에서 테스트가 통과되었는지 확인할 수 있습니다.

[data:image/svg+xml,%3csvg%20xmlns=%27http://www.w3.org/2000/svg%27%20version=%271.1%27%20width=%272400%27%20height=%271782%27/%3e](data:image/svg+xml,%3csvg%20xmlns=%27http://www.w3.org/2000/svg%27%20version=%271.1%27%20width=%272400%27%20height=%271782%27/%3e)

[https://www.tomray.dev/_next/image?url=%2Fstatic%2Fimages%2Fnestjs-unit-testing%2Fautomated-test-pull-request.png&w=3840&q=75](https://www.tomray.dev/_next/image?url=%2Fstatic%2Fimages%2Fnestjs-unit-testing%2Fautomated-test-pull-request.png&w=3840&q=75)

또한 워크플로가 실행될 때마다 작업의 단계와 출력을 볼 수 있습니다.

[data:image/svg+xml,%3csvg%20xmlns=%27http://www.w3.org/2000/svg%27%20version=%271.1%27%20width=%272400%27%20height=%271377%27/%3e](data:image/svg+xml,%3csvg%20xmlns=%27http://www.w3.org/2000/svg%27%20version=%271.1%27%20width=%272400%27%20height=%271377%27/%3e)

[https://www.tomray.dev/_next/image?url=%2Fstatic%2Fimages%2Fnestjs-unit-testing%2Fgithub-jobs.png&w=3840&q=75](https://www.tomray.dev/_next/image?url=%2Fstatic%2Fimages%2Fnestjs-unit-testing%2Fgithub-jobs.png&w=3840&q=75)

프로젝트에 테스트가 의존하는 환경 변수가 있는 경우 Github 작업에서 자동화된 테스트를 실행할 때 오류가 표시됩니다.

다음과 같이 워크플로에서 환경 변수에 액세스할 수 있도록 해야 합니다.

**테스트.yml**

`name: Tests on: pull_request env: API_KEY: "an-api-key" jobs: build: runs-on: ubuntu-latest steps: - uses: actions/checkout@v3 - name: Install modules run: npm ci - name: Run tests run: npm run test`

환경 변수가 민감한 경우(예: API 키) Github Secrets를 대신 사용해야 합니다.

저장소에서 설정 탭으로 이동하여 비밀을 추가하세요.

[data:image/svg+xml,%3csvg%20xmlns=%27http://www.w3.org/2000/svg%27%20version=%271.1%27%20width=%272400%27%20height=%271643%27/%3e](data:image/svg+xml,%3csvg%20xmlns=%27http://www.w3.org/2000/svg%27%20version=%271.1%27%20width=%272400%27%20height=%271643%27/%3e)

[https://www.tomray.dev/_next/image?url=%2Fstatic%2Fimages%2Fnestjs-unit-testing%2Fgithub-secrets.png&w=3840&q=75](https://www.tomray.dev/_next/image?url=%2Fstatic%2Fimages%2Fnestjs-unit-testing%2Fgithub-secrets.png&w=3840&q=75)

그런 다음 대신 비밀을 사용하도록 워크플로를 업데이트합니다.

**테스트.yml**

`name: Tests on: pull_request env: API_KEY: ${{ secrets.API_KEY }} jobs: build: runs-on: ubuntu-latest steps: - uses: actions/checkout@v3 - name: Install modules run: npm ci - name: Run tests run: npm run test`

잘하셨습니다. 이제 프로젝트에서 끌어오기 요청이 열릴 때마다 자동 테스트를 수행할 수 있습니다!

이를 지속적인 전달 워크플로( [**Github Actions로도 설정할also set up with Github Actions**](https://www.tomray.dev/deploy-nestjs-cloud-run) 수 있음 )와 결합하면 자동화된 CI/CD 파이프라인이 생성됩니다.

[Ultimate Guide: NestJS Unit Testing and Mocking [Updated 2022]](https://www.tomray.dev/nestjs-unit-testing#automating-nestjs-unit-tests-in-a-ci-pipeline-with-github-actions)