*### TDD는 소프트웨어 설계 방법론이다.

---

TDD는 소프트웨어 설계 **방법론**이기 때문에 너무 엄격하고 딱딱하게 접근을 하게 되면 TDD가 주는 교훈을 잘못 이해하고, 배우나마나 한 것이 되는 것 같습니다.

누군가는 TDD로 개발하는 것이 좋다고 생각할 수도, 누군가는 싫다고 할 수도 있습니다. 다만, 온보딩 과정에서는 목표로 해야하는 것은 TDD가 맞다,안맞다 좋다 안좋다를 생각하는 것이 아닌, **TDD를 이해해보고 경험해보는 것을 목표로 해야합니다.**

### 테스트 시나리오 또한 언제든 바뀔 수 있다.

---

처음에 정한 테스트 시나리오(유저 시나리오)를 바꾸지 않는 것이 Best Case 겠지만, 처음에 만든 시나리오에 종속되어 좋지 않은 구조로 소프트웨어를 개발하게 되는 것은 지양해야합니다.

아주 완벽한 시나리오를 짜는 것은 힘들다고 생각합니다. 더구나 처음 시나리오를 짜본다면 더욱 힘든 것 같습니다. 책에서도 계속해서 테스트 해야 할 경우나 시나리오를 수정해가는 모습을 볼 수 있습니다. **우리의 목표는 좋은 소프트웨어를 설계하는 것이지, 시나리오 대로 Test Code를 만드는 일이 아닙니다.** TDD는 짧은 주기로 iteration을 도는 애자일과 가깝습니다.

### 테스트 코드에서는 “실패” 케이스가 우선되어야 합니다.

---

완벽하게 맞는 말은 아닐 수 있겠지만, 테스트 코드는 실패 케이스를 우선으로 한다고 생각합니다. 예를 들어, 살펴보겠습니다.

저는 결제를 위해 사용자, 상품, 주문서, 결제서가 필요하다고 생각했고 객체를 만들고 테스트 코드를 만들었다고 해보겠습니다.

```jsx
// 객체 예시
export default class Order{
    /*
      멤버 변수
    * 주문번호(unique), 유저아이디
    * 상품번호, 상품명, 상품가격,
    * 주문시간, 주문수량, 주문 가격
    * 배송지,
    * 상태(결제 요청, 결제 승인, 결제 취소, 환불)
    * */

    constructor(orderNo, userID, productNo, productName, productPrice, orderTime, orderCount, addr, state) {
        this._orderNo = orderNo;
        this._userID = userID;
        this._productNo = productNo;
        this._productName = productName;
        this._productPrice = productPrice;
        this._orderTime = orderTime;
        this._orderCount = orderCount;
        this._orderPrice = orderCount * productPrice; or
        this._addr = addr;
        this._state = state;
    }
```

```java
describe("정보를 생성할 수 있다.", ()=>{
    const user = createUser("ID", "PW", "name", "addr");
    const product = createProduct("productNo", "productName", 1000, 1, true, true);
    const order = createOrder("orderNo", user.userID, product.productNo, product.productName, product.productPrice, 1, user.userAddr, OrderState.PayRequest);
    const pay = createPay("payNo", PayMethod.Card, order.orderPrice);
    const refund = createRefund("refundNo", pay.payPrice, user.userID, pay.payMethod);

    test("사용자 정보를 생성할 수 있다", ()=>{
        expect(user).toEqual( new User("ID", "PW", "name", "addr"));
    })
    test("상품 정보를 생성할 수 있다", ()=>{
        expect(product).toEqual(new Product("productNo", "productName", 1000, 1, true, true));
    })
    test("주문 정보를 생성할 수 있다.", ()=>{
        expect(order).toEqual(new Order("orderNo", user.userID, product.productNo, product.productName, product.productPrice, order.orderTime  , order.orderCount, user.userAddr, OrderState.PayRequest));
    })
    test("결제 정보를 생성할 수 있다.", ()=>{
        expect(pay).toEqual(new Pay("payNo", PayMethod.Card, pay.payPrice, pay.payTime));
    })
    test("환불 정보를 생성할 수 있다.", ()=>{
        expect(refund).toEqual(new Refund("refundNo", pay.payPrice, user.userID, pay.payMethod, refund.refundTime))
    })l
})
```

사실 위와 같이 테스트는 크게 의미가 없다고 생각합니다. createOrder이 동작하는 걸 그대로 가져다가 Test했기 때문입니다. 위와 같은 코드를 작성할 때 테스트 그 자체에 의미를 부여하고 목적을 두면 무언가 이상하다는 느낌을 받습니다.

위와 같이 객체가 만들어지는 테스트 코드에서 가질 수 있는 의미는 2개 정도가 있는 것 같습니다.

1. 기술서
    
    내가 남의 코드를 볼 때, 모든 코드를 보면서 이해하는 것이 아닌, 시나리오를 이해할 수 있는 테스트 코드를 통해 객체가 어떤 꼴로 생성되는지 이해해볼 수 있습니다.
    
2. TDD 시작하는 코드
    
    TDD로 개발을 하다 보면 조금은 자연스럽게 독립적인 행위를 기반으로 설계를 하게 되는 것 같습니다. 그런 부분에서 “~~한 정보들이 필요하겠지?” 하면서 시작하는 코드 정도
    

위와 같은 테스트를 하는 행위에 매몰되다 보면 왜 이런 테스트 케이스까지 해야하는지, 또 수정해야 할 코드들이 많아질 때(객체 생성 파라미터를 바꾼다거나) 일을 늘리기만 하는 느낌을 받을 수 있습니다.

```jsx
// 성공 케이스 보다는 실패 케이스에 초점
describe("정보의 생성과 실패", ()=>{
    describe("각 모델의 정보를 생성하는 **예시**들은 다음과 같다.", ()=>{
        const user = createUser("ID", "PW", "name", "addr");
        const product = createProduct("productNo", "productName", 1000, 1, true, true);

        test("user 정보 생성 예시", ()=>{
            expect(user).toEqual(new User(user.userID, user.userPW, user.userName, user.userAddr));
        })

        test("product 정보 생성 예시", ()=>{
            expect(product).toEqual(new Product(product.productNo, product.productName, product.productPrice, product.productCount, product.isValid, product.canRefund));
        })

        test("order 정보 생성 예시", ()=>{
            const order = createOrder(user, product, 1);
            expect(order).toEqual(new Order(order.orderNo,order.userID, order.productNo, order.productName, order.productPrice, order.orderTime, order.orderCount, order.addr, order.state ))
        })

        test("pay 정보 생성 예시", ()=>{
            const order = createOrder(user, product, 1);
            const pay = createPay(user, order, PayMethod.Card);
            expect(pay).toEqual(new Pay(pay.payNo, pay.payMethod, pay.payPrice, pay.payTime));
        })

        test("refund 정보 생성 예시", ()=>{
            const order = createOrder(user, product, 1);
            const pay = createPay(user, order, PayMethod.Card);
            const refund = createRefund(user, order, pay);
            expect(refund).toEqual(new Refund(refund.refundNo, refund.refundPrice, refund.userID, refund.payMethod, refund.refundTime))
        })
    });
		
    **// 생성할 수 없는 경우에 집중!**
    describe('아래와 같은 경우 각 정보를 생성할 수 없다', ()=>{
      // 이 안에서 여러 정보를 생성하는 경우를 생각해볼 수 있습니다.
			// 정보를 생성할 수 없는 에러 케이스들은 어떤 것이 있을까 생각하면, 데이터베이스에서 Schema를 강제 한다거나
		  // 타입, 꼴을 강제한다 거나 하는 것들로 이해해볼 수 있습니다.
    });
})
```

정보를 생성할 수 없는 에러 케이스들은 어떤 것이 있을까 생각하면, 데이터베이스에서 Schema를 강제 한다거나 타입, 꼴을 강제한다 거나 하는 것들로 이해해 볼 수 있습니다.

[또 다른 예시]

```jsx
describe("각 status는 아래와 같은 경우 다음 status로 넘어가지 못한다.", ()=>{
    describe("다음가 같은 경우 '결제 요청'이 불가능하다.", ()=>{
        test("인증 실패", ()=>{
            const user = createUser("heonil10", "PW", "name", "addr");
            const product = createProduct("productNo", "productName", 1000, 100, true, true);
            expect(()=>requestPay(user, product, 1)).toThrow(AuthError);
        });

        test("재고 부족", ()=>{
            const user = createUser("heonil1", "PW", "name", "addr");
            const product = createProduct("productNo", "productName", 1000, 1, true, true);
            expect(()=>requestPay(user, product, 10)).toThrow(ProductError);
        })

        test("구매 불가 상품", ()=>{
            const user = createUser("heonil1", "PW", "name", "addr");
            const product = createProduct("productNo", "productName", 1000, 10, false, true);
            expect(()=>requestPay(user, product, 1)).toThrow(ProductError);
        });
    });

    describe("다음과 같은 경우 '결제 승인'이 불가능하다.", ()=>{
        test("인가 실패", ()=>{
            const user = createUser("heonil1", "PW", "name", "addr");
            const product = createProduct("productNo", "productName", 1000, 100, true, true);
            const order = requestPay(user, product, 1);

            const card = createCard("cardCompany", "cardNUm", true);
            user._userID = "heonil10"; //
            expect(()=>approvePay(user, order, card, Date.now())).toThrow(AuthEㄱror);
        });
        test("카드사 점검 시간", ()=>{
            const user = createUser("heonil1", "PW", "name", "addr");
            const product = createProduct("productNo", "productName", 1000, 100, true, true);
            const order = requestPay(user, product, 1);
            const card = createCard("cardCompany", "cardNUm", true);

            expect(()=>approvePay(user, order, card, 5)).toThrow(CardError)
        });

        describe("카드사 처리 실패", ()=>{
            test("카드가 유효하지 않은 경우 결제 승인 실패", ()=>{
                const user = createUser("heonil1", "PW", "name", "addr");
                const product = createProduct("productNo", "productName", 1000, 100, true, true);
                const order = requestPay(user, product, 1);
                const card = createCard("cardCompany", "cardNUm", false);

                expect(()=>approvePay(user, order, card, Date.now())).toThrow(CardError)
            })

            test("카드의 금액이 부족한 경우 결제 승인 실패", ()=>{
                const user = createUser("heonil1", "PW", "name", "addr");
                const product = createProduct("productNo", "productName", 1000, 100, true, true);
                const order = requestPay(user, product, 1);
                const card = createCard("cardCompany", "cardNUm", true, 1);

                expect(()=>approvePay(user, order, card, Date.now())).toThrow(CardError);
            })
    });

	...
	...
	...
	...
});

```

기본적인 로직을 완성해두고, 에러가 날 수 있는 가능한 모든 경우들을 생각해 test code를 작성하고 동작하는 코드에서 예외처리를 하는 방식으로 코드를 작성해 나갈 수 있습니다.

### 확장 가능성 있는 코드

---

위에서도 말했지만, 일을 2번 하는 듯한 느낌의 생성 그 자체 코드에 대해 매몰되다 보면 수정해야 할 코드들이 늘면서, 또는 테스트 코드 그 자체를 완성하기 위해 확장성 있는 코드를 놓치게 되기도 했던 것 같습니다.

user, product, order, pay, refund 객체들이 있는데 객체 생성에 대해서 아래와 같이 객체 생성 코드를 만들었을 때, 각 파라미터가 고정되는데 이런 코드를 테스트를 올바르게 통과시키게 하기 위해 의도적으로 이런 꼴로 만들어지게 되는 경우가 있습니다.

```jsx
const user = createUser("ID", "PW", "name", "addr");
const product = createProduct("productNo", "productName", 1000, 1, true, true);
const order = createOrder("orderNo", user.userID, product.productNo, product.productName, product.productPrice, 1, user.userAddr, OrderState.PayRequest);
const pay = createPay("payNo", PayMethod.Card, order.orderPrice);
const refund = createRefund("refundNo", pay.payPrice, user.userID, pay.payMethod);
```

User와 Product를 통해 order를 만드는 꼴인데 product를 생성하는 꼴이 바뀌거나 user 객체의 생성 파라미터가 바뀌거나 하면 order를 생성하는 것에 대한 모든 파라미터를 바꿔야 할 수도 있습니다.

그래서 확장성 있는 코드를 위해 객체 그 자체로 매개변수를 넘기고 생성 그 자체에 대한 테스트 코드에 대해서는 힘을 빼는게 좋을 것 입니다. 나중에 최종적으로 생성 그 자체에 대한 테스트 코드를 완성시켜도 될 것입니다.(어떤 꼴로 객체가 만들어지는지 기술하는 목적이 더 크다고 보기 때문)

```jsx
createOrder(user, product, ..)
createPay(user, order, ..)
createRefund(user, order, pay, ...)
```

### 테스트 커버리지 ⇒ 모든 경우를 테스트 하는 것이 아니다

모든 함수들에 대해서 테스트 코드가 필요한 것은 아니다. 위에서도 이야기 했지만, 단순 생성의 경우는 테스트를 할 필요가 없을 수도 있다. Test Coverage를 높일 수 있으면 좋다. 다만, 100%의 Test Coverage를 가지는 것이 좋은 설계가 되는 것인지는 의문이 든다.

- 처음에 TDD를 잘못이해하고, 모든 코드에 대해 테스트 코드를 만들다 보면 당연하게도 100%의 커버리지를 가지게 된다. 좋은건지는 .. 의문!!
    
    ---
    

### 좋은 글들?

- [“**테스트 코드 없이 레거시 코드를 다 감수하시겠습니까?**](https://techblog.woowahan.com/2613/)
- [http://cloudrain21.com/test-driven-development](http://cloudrain21.com/test-driven-development)*