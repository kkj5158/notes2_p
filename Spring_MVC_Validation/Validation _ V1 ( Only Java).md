참고강의 : Spring MVC 2 _김영한 _ 검증 

## 목차
[[Validation _ V1 ( Only Java)]]
[[Validation _ V2 (BindingResult)]]
[[Validation _ V3 (Validator)]]

# 검증의 필요성

데이터의 검증은 시스템의 안정성을 위해서 필수적이다. Spring은 이런 검증과정을 객체지향의 원칙에 맞게 (특히 SRP 원칙에 맞춰서) 검증로직에만 집중할 수 있는 Bena Validator 기능을 제공한다. 우리는 Spring의 기능없이 검증로직을 우선 구현해보고. 이를 스프링이 제공하는 기능을 통해서 간략화 시켜나가는 방식으로 리팩토링 해나갈것이다. 우선 검증의 시나리오이다. 

### 시나리오 

요구사항: 검증 로직 추가
타입 검증
가격, 수량에 문자가 들어가면 검증 오류 처리
필드 검증
상품명: 필수, 공백X
가격: 1000원 이상, 1백만원 이하
수량: 최대 9999
특정 필드의 범위를 넘어서는 검증
가격 * 수량의 합은 10,000원 이상

### Dto
```java
package hello.itemservice.domain.item;  
  
import lombok.Data;  
  
@Data  
public class Item {  
  
    private Long id;  
    private String itemName;  
    private Integer price;  
    private Integer quantity;  
  
    public Item() {  
    }  
    public Item(String itemName, Integer price, Integer quantity) {  
        this.itemName = itemName;  
        this.price = price;  
        this.quantity = quantity;  
    }  
}
```

### Controller

```java
package hello.itemservice.web.validation;  
  
import hello.itemservice.domain.item.Item;  
import hello.itemservice.domain.item.ItemRepository;  
import lombok.RequiredArgsConstructor;  
import org.springframework.stereotype.Controller;  
import org.springframework.ui.Model;  
import org.springframework.util.StringUtils;  
import org.springframework.web.bind.annotation.*;  
import org.springframework.web.servlet.mvc.support.RedirectAttributes;  
  
import java.util.HashMap;  
import java.util.List;  
import java.util.Map;  
  
@Controller  
@RequestMapping("/validation/v1/items")  
@RequiredArgsConstructor  
public class ValidationItemControllerV1 {  
  
    private final ItemRepository itemRepository;  
  
    @GetMapping  
    public String items(Model model) {  
        List<Item> items = itemRepository.findAll();  
        model.addAttribute("items", items);  
        return "validation/v1/items";  
    }  
  
    @GetMapping("/{itemId}")  
    public String item(@PathVariable long itemId, Model model) {  
        Item item = itemRepository.findById(itemId);  
        model.addAttribute("item", item);  
        return "validation/v1/item";  
    }  
  
    @GetMapping("/add")  
    public String addForm(Model model) {  
        model.addAttribute("item", new Item());  
        return "validation/v1/addForm";  
    }  
  
    @PostMapping("/add")  
    public String addItem(@ModelAttribute Item item, RedirectAttributes redirectAttributes) {  
          
        Item savedItem = itemRepository.save(item);  
        redirectAttributes.addAttribute("itemId", savedItem.getId());  
        redirectAttributes.addAttribute("status", true);  
        return "redirect:/validation/v1/items/{itemId}";  
    }  
  
    @GetMapping("/{itemId}/edit")  
    public String editForm(@PathVariable Long itemId, Model model) {  
        Item item = itemRepository.findById(itemId);  
        model.addAttribute("item", item);  
        return "validation/v1/editForm";  
    }  
  
    @PostMapping("/{itemId}/edit")  
    public String edit(@PathVariable Long itemId, @ModelAttribute Item item) {  
        itemRepository.update(itemId, item);  
        return "redirect:/validation/v1/items/{itemId}";  
    }  
  
}
```

해당 코드에서는 검증이 이뤄지지 않는다. 만약 서비스로직에 맞지 않는 데이터 값이 들어와도 아무런 예외를 발생시키지 않을것이다. 

# 검증의 직접 처리

검증은 보통 컨트롤러 계층에서 이루어질것이다. 이때 검증을 직접 처리하려면 자바에서의 로직을 직접 이용해서 검증을 처리하게 된다. 이렇게 된다면 컨트롤러단의 코드가 검증코드로만 가득차게 될것이다. 

실무에서의 필드양은 매우 방대하고 이 필드 하나하나를 일일히 검증하는 코드들을 컨트롤러단에 단순히 자바코드로 나열하게 되면 컨트롤러단은 스파게티 코드가 되어버릴것이다. 이는 컨트롤러 계층의 역할에 맞지 않는다. (SRP의 위배) ( 컨트롤러의 역할은 컨트롤러이지. 검증 Class가 아니다.)

다음의 예시가, Spring에서 제공하는 Bean Validation 검증을 사용하지 않고서 수동으로 검증을 진행하는 코드의 예시이다. 

## 수동검증처리의 예시 ( 검증 V1 )

### 진행 

검증은 사용자에게 올바르지 못한 정보가 입력되었다는 오류메시지와 함께 기존에 입력되었던 데이터를 dto로 묶어서 반환하는 과정을 모두 포함한다. 이를 스프링이 제공하는 기능없이 JAVA 코드로만 진행해보자 . 

![[검증과정.png]]


```java
@PostMapping("/add")  
public String addItem(@ModelAttribute Item item, RedirectAttributes redirectAttributes, Model model) {  
  
    // 검증 오류 결과를 보관  
  
    Map<String, String> errors = new HashMap<>();  
  
    if(!StringUtils.hasText(item.getItemName())){  
       errors.put("itemName", "상품 이름은 필수입니다.");  
    }  
  
    if(item.getPrice() == null || item.getPrice() < 1000 || item.getPrice() > 1000000 ) {  
        errors.put("price", "가격은 1000 ~ 1000000 까지 허용합니다. ");  
    }  
  
    if(item.getQuantity() == null || item.getQuantity() > 9999){  
        errors.put("quantity", "수량은 최대 9.999 까지 허용합니다.");  
    }  
  
    // 특정 필드가 아닌 복합 룰 검증  
  
    if(item.getPrice() != null && item.getQuantity() != null ){  
        int resultPrice = item.getPrice() * item.getQuantity();  
        if(resultPrice < 10000){  
            errors.put("globalError", "가격 * 수량의 합은 10,000원 이상이어야 합니다. 현재 값 = " + resultPrice);  
  
        }  
    }  
  
    // 검증에 실패하면 다시 입력 폼으로  
  
    if(!errors.isEmpty()){  
        log.info("erros = {} ", errors);  
        model.addAttribute("errors", errors );  
        return "validation/v1/addForm";  
    }  
  
    // 성공 로직  
  
  
    Item savedItem = itemRepository.save(item);  
    redirectAttributes.addAttribute("itemId", savedItem.getId());  
    redirectAttributes.addAttribute("status", true);  
    return "redirect:/validation/v1/items/{itemId}";  
}
```


V1의 해당 검증 방식은 JAVA의 Util을 사용하여서 Dto에 담겨진 값들을 일일히 검증하면서 상황에 따라서 MAP의 예외가 발생한 검증필드들을 key로, 예외메시지를  Value로 담아서 이를 다시 사용자의 화면단에 넘기는 방식으로 검증을 진행한다. 위에서 보듯이 필드가 3개인 경우에도 벌써부터 컨트롤러단의 코드는 어마무시하게 방대해진다. 이런 방법이 아니라 다른 방법으로 코드를 최적화 하는 방법을 구상해야한다. 

### 한계 
- 방대해지는 컨트롤러단의 코드 
- 타입 에러를 잡아내지 못한다 


[[Validation _ V2 (BindingResult)]]
