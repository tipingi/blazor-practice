# 입력값 확인

`BlazingPizza.Shared` 프로젝트에 `Order` 클래스를 보면 `Address` 타입의 속성으로 `DeliveryAddress`가 있는 것을 알 수 있습니다. 그러나 피자 주문 흐름에서 아직 이 데이터를 채우는 것은 없으므로 모든 주문에 빈 배송 주소만 있습니다.

이제 고객이 유효한 주소를 입력해야 하는 '결제' 화면을 추가하여 이 문제를 해결해야 합니다.

## 흐름에 확인 단계 추가하기

URL `/checkout`에 해당하는 `@page` 지시문과 함께 새로운 페이지 컴포넌트인 `Checkout.razor`를 추가하는 것으로 시작합니다. 초기 마크업의 경우 `Order Review` 구성 요소를 사용하여 주문의 세부 정보를 표시합니다

```razor
<PageTitle>Blazing Pizza - Checkout</PageTitle>

<div class="main">
    <div class="checkout-cols">
        <div class="checkout-order-details">
            <h4>Review order</h4>
            <OrderReview Order="OrderState.Order" />
        </div>
    </div>

    <button class="checkout-button btn btn-warning" @onclick="PlaceOrder">
        Place order
    </button>
</div>
```

일단 `PlaceOrder`를 구현하기 위해 `Index.razor` 파일에서 같은 이름의 메서드를 `Checkout.razor` 파일로 복사합니다.

```razor
@code {
    async Task PlaceOrder()
    {
        var response = await HttpClient.PostAsJsonAsync("orders", OrderState.Order);
        var newOrderId = await response.Content.ReadFromJsonAsync<int>();
        OrderState.ResetOrder();
        NavigationManager.NavigateTo($"myorders/{newOrderId}");
    }
}
```

여기에서도 `Index.razor`에서 사용되었던 것처럼 `OrderState`, `HttpClient`, 그리고 `NavigationManager`에 대해서 `@inject` 값이 필요합니다.

다음으로 고객이 주문을 제출하려고 할 때 여기로 이동하도록 해봅시다. `Index.razor`로 돌아가서 `PlaceOrder` 메서드를 삭제했는지 확인한 다음 주문 제출 버튼을 `/checkout` URL로 연결하는 일반 HTML 링크로 변경합니다. 아래와 같이 바꾸어 주세요.

```razor
<a href="checkout" class="@(OrderState.Order.Pizzas.Count == 0 ? "btn btn-warning disabled" : "btn btn-warning")">
    Order >
</a>
```

> HTML 링크가 지원하지 않는 'disabled' 속성을 삭제하고 대신 적절한 스타일링을 추가하였습니다.

이제 앱을 실행할 때 *Order* 버튼을 클릭하면 체크아웃 페이지에 도달할 수 있으며, 여기서 *Place order*을 클릭하여 주문을 확정할 수 있습니다.

![Confirm order](https://user-images.githubusercontent.com/1874516/77242251-d2530780-6bb9-11ea-8535-1c41decf3fcc.png)

## 배달 주소 얻기

이제 배달 주소를 입력할 수 있는 UI가 들어갈 좋은 장소가 생겼습니다. 평소처럼 이것을 재사용 가능한 컴포넌트로 설계해 봅시다. 언제 다른 곳의 주소를 요청할지 모르기 때문입니다.

`BlazingPizza.Client` 프로젝트의 `Shared` 폴더에 `AddressEditor.razor`라는 새 컴포넌트를 생성합니다. 이 컴포넌트는 `Address` 인스턴스를 편집하는 일반적인 방법이므로 다음과 같은 타입의 매개 변수를 수신합니다.

```razor
@code {
    [Parameter] public Address Address { get; set; }
}
```
여기 마크업은 조금 지루할 수 있어 복사하여 붙여넣기로 추가하세요. 우리는 `Address`의 각 속성에 대한 입력 요소가 필요합니다.

```razor
<div class="form-field">
    <label>Name:</label>
    <div>
        <input @bind="Address.Name" />
    </div>
</div>

<div class="form-field">
    <label>Line 1:</label>
    <div>
        <input @bind="Address.Line1" />
    </div>
</div>

<div class="form-field">
    <label>Line 2:</label>
    <div>
        <input @bind="Address.Line2" />
    </div>
</div>

<div class="form-field">
    <label>City:</label>
    <div>
        <input @bind="Address.City" />
    </div>
</div>

<div class="form-field">
    <label>Region:</label>
    <div>
        <input @bind="Address.Region" />
    </div>
</div>

<div class="form-field">
    <label>Postal code:</label>
    <div>
        <input @bind="Address.PostalCode" />
    </div>
</div>

@code {
    [Parameter] public Address Address { get; set; }
}
```

마지막으로 실제 `Checkout.razor` 컴포넌트 안에서 `AddressEditor`를 사용할 수 있습니다.

```razor
<div class="checkout-cols">
    <div class="checkout-order-details">
        ... leave this div unchanged ...
    </div>

    <div class="checkout-delivery-address">
        <h4>Deliver to...</h4>
        <AddressEditor Address="OrderState.Order.DeliveryAddress" />
    </div>
</div>
```

이제 "확인" 화면에 배송 주소를 입력하게 되었습니다.

![Address editor](https://user-images.githubusercontent.com/1874516/77242320-79d03a00-6bba-11ea-9e40-4bf747d4dcdc.png)

이제 주문을 하면 입력한 주소 데이터는 모두 직렬화되어 서버로 전송되는 `Order` 객체의 일부이기 때문에 실제로 주문과 함께 데이터 베이스에 저장됩니다.

정말로 데이터가 저장되고 있는지 확인하고 싶다면 [DB Browser for SQLite](https://sqlitebrowser.org/)와 같은 도구를 다운로드하여 `pizza.db` 파일을 열어보세요. 하지만 꼭 해볼 필요는 없을 것 같습니다.

또는 `BlazingPizza.Server` 프로젝트의 `OrderController.PlaceOrder` 메서드에 브레이크 포인트를 설정하고 디버거를 사용하여 들어오는 `Order` 객체를 검사합니다. 그러면 백엔드 서버가 입력한 주소 데이터를 수신하는 것을 볼 수 있습니다.

## 서버측 유효성 검사 추가하기

아직까지는 고객이 '배달 주소' 입력란을 비워두고 아무데나 피자를 주문할 수 있습니다. 유효성 검사와 관련해서는 서버와 클라이언트 모두에서 규칙을 구현하는 것이 일반적입니다.

 * 클라이언트 측 유효성 검사는 사용자에 대한 배려입니다. 사용자가 양식을 편집하는 동안 즉각적인 피드백을 제공할 수 있습니다. 그러나 브라우저 개발 도구에 대한 기본 지식을 가진 사람이라면 누구나 쉽게 우회할 수 있습니다.
 * 서버 측 유효성 검사는 실제 적용이 이뤄지는 곳입니다.

따라서 일반적으로 서버 측 유효성 검사를 구현하는 것부터 시작하는 것이 가장 좋으므로 클라이언트 측에서 어떤 일이 일어나든 앱이 안정적이라는 것을 알 수 있습니다. `BlazingPizza.Server` 프로젝트의 `OrdersController.cs`을 보면 이 API 엔드포인트가 `[ApiController]` 속성으로 되어 있음을 알 수 있습니다.

```csharp
[Route("orders")]
[ApiController]
public class OrdersController : Controller
{
    // ...
}
```

[ApiController]는 `DataAnnotations` 유효성 검사 규칙 적용을 포함한 다양한 서버 측 규칙을 추가합니다. 따라서 모델 클래스에 몇 가지 `DataAnnotations` 유효성 검사 규칙을 추가하기만 하면 됩니다.

`BlazingPizza.Shared` 프로젝트의 `Address.cs`을 열고 모든 주소에 두 번째 줄이 필요한 것은 아니므로 자동 생성되는 `Id`(기본 키로 자동 생성됨)와 `Line2`를 제외한 각 속성에 `[Required]` 속성을 추가합니다. 필요하면 `[MaxLength]` 속성 또는 다른 `DataAnnotations` 규칙을 추가할 수도 있습니다.

```csharp
using System.ComponentModel.DataAnnotations;

namespace BlazingPizza
{
    public class Address
    {
        public int Id { get; set; }

        [Required, MaxLength(100)]
        public string Name { get; set; }

        [Required, MaxLength(100)]
        public string Line1 { get; set; }

        [MaxLength(100)]
        public string Line2 { get; set; }

        [Required, MaxLength(50)]
        public string City { get; set; }

        [Required, MaxLength(20)]
        public string Region { get; set; }

        [Required, MaxLength(20)]
        public string PostalCode { get; set; }
    }
}
```

이제 어플리케이션을 다시 컴파일하고 실행한 후, 서버에서 진행하는 유효성 검사를 확인할 수 있습니다. 배달 주소가 비어있는 주문을 제출하려고 하면 서버가 요청을 거부하고 브라우저의 *Network* 탭에 HTTP400("Bad Request") 오류가 표시됩니다.

![Server validation](https://user-images.githubusercontent.com/1874516/77242384-067af800-6bbb-11ea-8dd0-74f457d15afd.png)

주소 필드를 모두 작성하면 주문할 수 있습니다. 이 경우에 대해서도 모두 예상대로 작동하는지 확인해 주세요.

## 클라이언트측 유효성 검사 추가하기

블레이저는 데이터 입력 양식과 유효성 검사를 위한 포괄적인 시스템이 있습니다. 이제 이를 사용하여 서버에서 이미 적용되고 있는 것과 동일한 `DataAnnotations` 규칙을 클라이언트에 적용하겠습니다.

블레이저의 폼과 유효성 검사 시스템은 `EditContext`을 기반으로 작동합니다. `EditContext`는 편집 과정의 상태를 추적하기 때문에 어떤 필드가 수정되었는지, 어떤 데이터가 입력되었는지, 그리고 필드가 유효한지 여부를 알 수 있습니다. 다양한 내장 UI 컴포넌트는 `EditContext`에 연결되어 상태를 읽거나(예: 유효성 검사 메시지 표시) 상태에 쓰거나(예: 사용자가 입력한 데이터로 채우기) 상태를 변경할 수 있습니다.

### EditForm 사용하기

데이터 입력을 위한 가장 중요한 기본 제공 UI 컴포넌트 중 하나는 `EditForm`입니다. 이는 HTML '<form>' 태그로 렌더링되지만 `EditContext`를 설정하여 폼 내부에서 무슨 일이 일어나고 있는지 추적합니다. 이를 사용하려면 `Checkout.razor` 파일을 열고 `main` div의 전체 내용을 `EditForm`으로 둘러싸 주세요. 아래 HTML을 참고하세요.

```razor
<div class="main">
    <EditForm Model="OrderState.Order.DeliveryAddress">
        <div class="checkout-cols">
            ... leave unchanged ...
        </div>

        <button class="checkout-button btn btn-warning" @onclick="PlaceOrder">
            Place order
        </button>
    </EditForm>
</div>
```

한 번에 여러 개의 `EditForm` 컴포넌트를 가질 수 있지만 (HTML의 '<form>' 요소는 겹칠 수 없기 때문에) 중첩될 수 없습니다. `Model`을 지정하면 양식을 제출할 때 어떤 개체(이 경우 배송 주소)를 검증해야 하는지 내부의 `EditContext`에 알려줍니다.

먼저 가장 기본적이지만 멋지지 않은 방식으로 검증 메시지를 표시합니다. 바로 아래에 있는 `Edit Form` 안에 다음 두 가지 구성 요소를 추가합니다.

```razor
<DataAnnotationsValidator />
<ValidationSummary />
```

`Data Annotations Validator`는 `EditContext` 이벤트를 연결하여 `DataAnnotations` 규칙을 실행합니다. `DataAnnotations`가 아닌 다른 검증 시스템을 사용하려면 `DataAnnotationsValidator`를 다르게 바꾸어야 합니다.

`Validation Summary`는 단순히 `EditContext`의 모든 유효성 검사 메시지를 HTML `<ul>` 목록으로 렌더링 합니다.

### 제출 처리

지금 실행되어 있는 어플리케이션은 빈 양식을 제출할 수 있습니다.(서버는 여전히 HTTP 400 오류로 응답합니다) 그것은 `<button>`은 실제 `submit` 버튼이 아니기 때문입니다. `button`에 `type="submit"`을 추가하고 `@onclick` 속성을 완전히 **삭제**해 보세요.

다음으로 버튼에서 직접 `PlaceOrder`를 트리거하는 대신 `EditForm`에서 트리거해야 합니다. `EditForm`에 다음 `OnValidSubmit` 속성을 추가합니다.

```razor
<EditForm Model="OrderState.Order.DeliveryAddress" OnValidSubmit="PlaceOrder">
```

짐작하시겠지만 `<button>`은 더 이상 `PlaceOrder`를 직접 트리거하지 않습니다. 대신 버튼은 양식을 제출하도록 요청합니다. 그러면 양식은 양식이 유효한지 여부를 결정하고, **유효하다면** `PlaceOrder`를 호출합니다.

더 이상 유효하지 않은 양식을 제출할 수 없으며, `ValidationSummary`에 검증 메시지(비록 멋지지 않더라도)가 표시됩니다.

![Validation summary](https://user-images.githubusercontent.com/1874516/77242430-9d47b480-6bbb-11ea-96ef-8865468375fb.png)

### ValidationMessage 사용하기

텍스트 상자에서 너무 멀리 떨어진 곳에 모든 유효성 검사 메시지를 표시 하는 것은 그다지 좋아보이지 않습니다. 더 좋은 위치로 이동 시킵시다.

`<Validation Summary>` 컴포넌트를 완전히 삭제하는 것으로 시작합니다. 그런 다음, `AddressEditor.razor` 파일을 열고 각 양식 필드 옆에 별도의 `<ValidationMessage>` 컴포넌트를 추가합니다. 예를 들면 아래와 같습니다.

```razor
<div class="form-field">
    <label>Name:</label>
    <div>
        <input @bind="Address.Name" />
        <ValidationMessage For="@(() => Address.Name)" />
    </div>
</div>
```

모든 양식 필드에 대해 동일한 작업을 수행합니다.

궁금한 점이 있다면 `@(() => Address.Name)` 구문은 *람다식*이며 실제로 속성 값을 평가하지 않고 메타데이터를 읽을 속성을 설명하는 방법으로 이 구문을 사용합니다.

이제 훨씬 좋아보입니다.

![Validation messages](https://user-images.githubusercontent.com/1874516/77242484-03ccd280-6bbc-11ea-8dd1-5d723b043ee2.png)

원한다면 사용자 지정 메시지를 지정하여 메시지의 가독성을 높일 수 있습니다. 예를 들어 *The City field is required*을 표시하는 대신 `Address.cs` 파일에서 아래와 같이 할 수 있습니다.

```csharp
[Required(ErrorMessage = "How do you expect to receive the pizza if we don't even know what city you're in?"), MaxLength(50)]
public string City { get; set; }
```

### 빌트인 입력 컴포넌트를 사용하여 유효성 검사 UX 개선

유효성 검사 메시지가 표시되면 필드 값을 편집했더라도 *Place Order*버튼을 다시 클릭할 때까지 화면에 남아 있기 때문에 그다지 좋은 사용자 경험은 아닙니다. 사용해보고 기본적인 느낌을 확인해 보세요!

이를 개선하려면 하위 수준의 HTML 입력 요소를 Blazor의 기본 입력 요소로 교체하면 됩니다. 이들은 `EditContext`에 더 원활하게 연결됩니다.

* 편집되면 `EditContext`에 즉시 알림을 보내 유효성 검사 상태를 새로 고칠 수 있습니다.
* 또한 `EditContext`로부터 유효성에 대한 알림을 받기 때문에 사용자가 편집할 때 유효 또는 유효하지 않음으로 하이라이트 표시를 할 수 있습니다.


다시 한 번 `AddressEditor.razor` 파일을 열고 각각의 `<input>`요소를 `<InputText>`로 바꾸고 `@bind`는 `@bind-value`로 바꾸어 줍니다. 아래와 코드를 참고하세요.

```html
<div class="form-field">
    <label>Name:</label>
    <div>
        <InputText @bind-Value="Address.Name" />
        <ValidationMessage For="@(() => Address.Name)" />
    </div>
</div>
```

모든 속성에 대해 이 작업을 수행합니다. 이제 동작이 훨씬 좋아졌습니다! 포커스를 변경할 때 각 양식 필드에 대해 개별적으로 유효성 검사 메시지가 업데이트될 뿐만 아니라 각 항목에 대해 깔끔한 "유효한" 또는 "유효하지 않은" 하이라이트가 표시됩니다.

![Input components](https://user-images.githubusercontent.com/1874516/77242542-ba30b780-6bbc-11ea-8018-be022d6cac0b.png)

초록색/빨간색 스타일링은 CSS 클래스를 적용하여 처리되므로 원하는 경우 이러한 효과의 모양을 변경하거나 완전히 제거할 수 있습니다.

`InputText`뿐만 아니라 `InputCheckbox`, `InputDate`, `InputSelect` 등 다른 입력 요소가 준비되어 있습니다.

## 한 가지 더!

혹시 시간이 된다면, 실수로 폼을 두번 제출하는 것을 방지할 수 있을까요?

현재 폼 게시물이 서버에 도달하는 데 시간이 걸리는 경우, 사용자는 버튼을 여러 번 클릭하여 주문 복사본을 여러 번 보낼 수 있습니다. `true`이면 *Place order* 버튼이 비활성화되는 `bool` 형식의 `isSubmiting` 속성을 선언하고 제출이 완료되면(성공하든 실패하든) 다시 `false`로 설정해야 합니다. 그렇지 않으면 사용자가 다시 제출하지 못 하니까요.

이 방법이 잘 작동하는지 확인하려면 서버측 `OrdersController.cs` 파일의 `PlaceOrder()` 메서드 맨 위에 아래와 같은 줄을 추가하여 서버를 잠시 늦을 수 있습니다.

```cs
await Task.Delay(5000); // Wait 5 seconds
```

## 다음 세션

다음으로 [인증과 권한](06-authentication-and-authorization.md)에 대해 진행할 것입니다.

원문 읽기 - [Checkout with validation](https://github.com/dotnet-presentations/blazor-workshop/blob/main/docs/05-checkout-with-validation.md)