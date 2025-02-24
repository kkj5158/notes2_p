
참고강의 : Spring MVC 2 _김영한 _ 검증
[Validation 검증 ](https://hhhhicode.tistory.com/5#)

## 목차
[[Validation _ V1 ( Only Java)]]
[[Validation _ V2 (BindingResult)]]
[[Validation _ V3 (Validator)]]
# V1 __ only java의 한계

앞선 검증은 다음과 같은 한계점을 가지고 있다

1. 타입 오류 처리의 진행이 일어나지 않는다. 

아이템 가격은 int형인데 , 만약 string 형이 들어온다면 스프링은 400에러페이지를 보일것이다. 이는 서비스에서는 발생해서는 안되는 일이다. 

2. 사용자가 입력한 값들도 별도로 관리가 되어야한다. 

사용자가 기존에 기입한 문구와 함께  예외처리 메시지를 봐야 어떤 부분을 잘 못 입력했는지 한번에 파악이 가능하기 때문, 예를 들어서 키보드를 안 보고 1991을 year 필드에 입력을 했느데 한칸 아래 문자들을 입력하여 qppq를 입력했을 경우 예외 처리 문구만 보고서는 자기가 무엇을 잘못입력했는지 파악하기 어렵다. 

# BindingResult

스프링에서는 검증 오류가 발생했을 때 오류 내용을 보관하는 BindingResult라는 객체를 제공한다. 

BindingResult가 있으면 @ModelAttribute에데이터 바인딩 시 오류가 발생해도 오류 정보를 FieldError 개체를 BindingResult가 담은 뒤 컨트롤러가 호출된다. ( 이는 1. 타입오류의 문제를 해결한다.)


**주의**
BindingResult 객체의 파라미터 위치는 반드시 @ModelAttribute 어노테이션이 붙은 객체 다음에 위치해야한다. 

## Binding Result의 장점

- errors를 별도로 선언하는 대신 스프링 프레임워크에서 제공하는 BindingResult 객체를 사용.
- BindingResult는 Model에 자동으로 포함되기 때문에 Model 객체를 파라미터에 추가하지 않아도 된다. 

```java
@PostMapping("/add")  
public String addItemV1(@ModelAttribute Item item, BindingResult bindingResult, RedirectAttributes redirectAttributes, Model model) {  
  
    //검증 로직  
    if (!StringUtils.hasText(item.getItemName())) {  
        bindingResult.addError(new FieldError("item", "itemName", "상품 이름은 필수 입니다."));  
    }  
    if (item.getPrice() == null || item.getPrice() < 1000 || item.getPrice() > 1000000) {  
        bindingResult.addError(new FieldError("item", "price", "가격은 1,000 ~ 1,000,000 까지 허용합니다."));  
    }  
    if (item.getQuantity() == null || item.getQuantity() >= 9999) {  
        bindingResult.addError(new FieldError("item", "quantity", "수량은 최대 9,999 까지 허용합니다."));  
    }  
  
    //특정 필드가 아닌 복합 룰 검증  
    if (item.getPrice() != null && item.getQuantity() != null) {  
        int resultPrice = item.getPrice() * item.getQuantity();  
        if (resultPrice < 10000) {  
            bindingResult.addError(new ObjectError("item", "가격 * 수량의 합은 10,000원 이상이어야 합니다. 현재 값 = " + resultPrice));  
        }  
    }  
  
    //검증에 실패하면 다시 입력 폼으로  
    if (bindingResult.hasErrors()) {  
        log.info("errors={} ", bindingResult);  
        return "validation/v2/addForm";  
    }  
  
    //성공 로직  
    Item savedItem = itemRepository.save(item);  
    redirectAttributes.addAttribute("itemId", savedItem.getId());  
    redirectAttributes.addAttribute("status", true);  
    return "redirect:/validation/v2/items/{itemId}";  
}


```
Binding Result의 최대 장점은 Binding 진행할때 , 타입오류가 발생한다면 400에러를 발생시키지 않고 이를 BindingResult에 보관하여서 컨트롤러단을 정상 호출할 수 있도록 하는 것이다. 
d
### 1번, 타입 오류의 해결 , 400에러가 아니라 컨트롤러단 정상호출 

@ModelAttribute에 바인딩 시 타입 오류가 발생한다면 ? 
- BindingResult 가 없으면 -> 400오류가 발생하면서 컨트롤러가 호출되지 않고 , 오류페이지로 이동 
- BindingResult가 있으면 -> 오류 정보(Field Error)를 BindingResult에 담어서 컨트롤러를 정상 호출한다. 

### 2번 , 사용자 입력값의 저장 

#### FieldError , ObjectError 

bindingResult에 에러를 더해가는 과정상에서, FieldError와 ObjectError를 사용하면 기존 사용자가 입력한 값을 저장해 둘 수 있다 .

```java
new FieldError("item", "price", item.getPrice(), false, null, null, "가격은 1,000 ~
1,000,000 까지 허용합니다.")` 
```

Field Error에는 다음의 파라미터 목록이 존재한다. 

파라미터 목록
`objectName` : 오류가 발생한 객체 이름
`field` : 오류 필드
`rejectedValue` : 사용자가 입력한 값(거절된 값)
`bindingFailure` : 타입 오류 같은 바인딩 실패인지, 검증 실패인지 구분 값
`codes` : 메시지 코드
`arguments` : 메시지에서 사용하는 인자
`defaultMessage` : 기본 오류 메시지

여기에서 `rejectedValue`를 사용하면 화면단에서 사용자가 입력한 값을 출력시킬 수 있다. 

## 그럼에도, Binding Result를 활용한 검증의 여전한 한계 


- 여전히 방대한 컨트롤러단의 코드 -> 필드값에 대한 검증로직으로 컨트롤러단이 방대해진다. 

이러한 문제를 해결하기 위해서는 복잡한 검증로직을 별도로 분리할 필요가 있다. 
-> Validator의 분리 

[[Validation _ V3 (Validator)]]
