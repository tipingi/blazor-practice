# 상태 관리의 리펙터링

이 세션에서는 이미 작성된 코드를 더 좋게 만들어 볼 것입니다. 또한 이벤트와 그로 인해 UI가 업데이트 되는 방법에 대해 더 자세히 설명하겠습니다.

## 문제 제기

이미 알아차렸을지도 모르지만 현재 어플리케이션은 버그가 있습니다! 현재 주문한 피자 목록을 `Index` 컴포넌트에 저장하고 있기 때문에 사용자가 `Index` 페이지를 떠날 경우 사용자의 상태가 손실될 수 있습니다. 실제로 확인하려면 현재 주문에 피자를 추가한 다음(아직 주문하지 않은 상태에서) 내 주문 페이지로 이동했다가 다시 인덱스로 돌아옵니다. 그러면 주문이 비어 있는 것을 확인할 수 있습니다!

## 해결 방법

*AppState 패턴*으로 이 버그를 수정하는 방법을 소개하겠습니다. *AppState 패턴*은 관련 컴포넌트간의 상태를 조정하는데 사용할 DI 컨테이너를 추가합니다. *AppState* 객체는 DI 컨테이너에서 관리하기 때문에 컴포넌트보다 오래 지속되고 UI가 변경되어도 상태를 유지할 수 있습니다. *AppState 패턴*의 또 다른 이점은 프레젠테이션(컴포넌트)와 비지니스 로직 간의 분리가 더 쉽다는 것입니다.

## 시작 하기

클라이언트 프로젝트 루트 디렉터리에 `OrderState`라는 클래스를 새로 생성하고, DI 컨테이너에 스코프 서비스로 등록합니다. 블레이저 웹어셈블리 어플리케이션에서는 `Program` 클래스에 서비스가 등록됩니다. `await builder.Build().RunAsync();` 호출 직전에 서비스를 추가해 주세요.

```csharp
using BlazingPizza.Client;
using Microsoft.AspNetCore.Components.Web;
using Microsoft.AspNetCore.Components.WebAssembly.Hosting;

var builder = WebAssemblyHostBuilder.CreateDefault(args);
builder.RootComponents.Add<App>("#app");
builder.RootComponents.Add<HeadOutlet>("head::after");

builder.Services.AddScoped(sp => new HttpClient { BaseAddress = new Uri(builder.HostEnvironment.BaseAddress) });
builder.Services.AddScoped<OrderState>();

await builder.Build().RunAsync();
```
> Note: Singleton보다 Scoped를 선택하는 이유는 서버측 컴포넌트 어플리케이션과 대칭을 위해서입니다. 싱글톤은 일반적으로 *모든 사용자*를 의미하며, 범위는 *현재 작업 단위*를 의미합니다.

## Index 페이지의 수정

이제 DI에 등록되었으니 `Index` 페이지에서 `@inject`하여 사용할 수 있습니다.

```razor
@page "/"
@inject HttpClient HttpClient
@inject OrderState OrderState
@inject NavigationManager NavigationManager
```

`@inject`는 DI에서 타입별로 무언가를 검색하고 해당 타입의 속성을 정의할 수 있는 편리한 방법입니다.

이제 다시 앱을 실행하면 테스트를 할 수 있습니다. 만약 DI컨테이너에 없는 것을 주입하려고 하면 예외가 발생하고 `Index` 페이지가 표시되지 않습니다.

이제 `Order`와 `Pizza`의 상태를 표현하고 조작할 속성과 메서드를 이 클래스에 추가해 보겠습니다.

`configuringPizza`, `showingConfigureDialog` 그리고 `order` 필드를 `OrderState`의 속성이 되도록 이동 시킵니다. 이 속성 모두 `private set`으로 만들어 `OrderState`의 메서드를 통해서만 조작이 되도록 합니다.

```csharp
public class OrderState
{
    public bool ShowingConfigureDialog { get; private set; }

    public Pizza ConfiguringPizza { get; private set; }

    public Order Order { get; private set; } = new Order();
}
```

이제 일부 메소드를 `Index`에서 `OrderState`로 이동시켜 보겠습니다. `PlaceOrder`를 `OrderState`로 옮기면 탐색이 시작되므로 대신 `ResetOrder` 메서드만 추가하는 것으로 합니다.

```csharp
public void ShowConfigurePizzaDialog(PizzaSpecial special)
{
    ConfiguringPizza = new Pizza()
    {
        Special = special,
        SpecialId = special.Id,
        Size = Pizza.DefaultSize,
        Toppings = new List<PizzaTopping>(),
    };

    ShowingConfigureDialog = true;
}

public void CancelConfigurePizzaDialog()
{
    ConfiguringPizza = null;

    ShowingConfigureDialog = false;
}

public void ConfirmConfigurePizzaDialog()
{
    Order.Pizzas.Add(ConfiguringPizza);
    ConfiguringPizza = null;

    ShowingConfigureDialog = false;
}

public void ResetOrder()
{
    Order = new Order();
}

public void RemoveConfiguredPizza(Pizza pizza)
{
    Order.Pizzas.Remove(pizza);
}
```

`Index.razor` 파일에서 관련된 메서드를 지우는 것을 잊지 마세요. 또한 주입된 `OrderState`에서 상태 데이터를 가져오므로 `Index.razor`에서 `order`, `configuringPizza`, 그리고 `showingConfigureDialog` 속성을 삭제해야 합니다.

이때 `OrderState`에 첨부된 다양한 정보를 참조하도록 수정하여 `Index` 컴포넌트를 다시 컴파일 하여야 합니다. 예를 들어 `Index.razor`에 남아 있는 `PlaceOrder` 메서드는 아래와 같게 됩니다.

```csharp
async Task PlaceOrder()
{
    var response = await HttpClient.PostAsJsonAsync("orders", OrderState.Order);
    var newOrderId = await response.Content.ReadFromJsonAsync<int>();
    OrderState.ResetOrder();
    NavigationManager.NavigateTo($"myorders/{newOrderId}");
}
```

`OrderState.Order` 또는 `OrderState.Order.Pizzas` 같은 속성이 있는 것이 좋겠다고 생각되면 그렇게 만들면 됩니다.

수정 내용을 적용하고 모든 것이 잘 작동하는 지 확인해 주세요. 특히 원래 버그를 수정했는지 확인해 주세요. 이제 피자를 추가하고 "My Orders"으로 이동하고 다시 이동하면 주문이 더 이상 손실되지 않습니다.

## 상태 변경 탐색

블레이저에서 상태 변경 및 렌더링이 어떻게 작동하는지 그리고 `EventCallback`이 몇 가지 일반적인 문제를 어떻게 해결하는지 살펴볼 수 있는 좋은 기회입니다. `OrderState`가 관련됨에 따라 더욱 복잡해집니다.

`EventCallback`은 블레이저에게 이벤트 핸들러를 정의한 컴포넌트에 이벤트 알림(및 렌더링)을 발송하도록 지시합니다. 이벤트 핸들러가 컴포넌트(`OrderState`)에 의해 정의되지 않은 경우 이벤트 핸들러를 *hooked up* 이벤트 핸들러(`Index`) 대체합니다.

## 결론

*AppState 패턴*이 제공하는 기능을 요약해 보겠습니다.
- 공유되어야 할 컴포넌트의 상태를 `OrderState`로 이동시킵니다.
- 컴포넌트가 상태의 변화를 드리거하는 메서드를 호출합니다.
- `EventCallback`은 변경 알림을 발송합니다.

렌더링 및 이벤트에 대해 많은 것을 다루었습니다.
- 매개 변수가 변경되거나 이벤트를 수신할 때 컴포넌트가 다시 렌더링됩니다
- 이벤트 발송은 이벤트 핸들러의 위임 대상에 따라 달라집니다
- 이벤트 발신을 유연하게 하려면 `EventCallback`을 사용합니다.

다음 세션 - [입력값 확인](05-checkout-with-validation.md)

원문 읽기 - [Refactor state management](https://github.com/dotnet-presentations/blazor-workshop/blob/main/docs/04-refactor-state-management.md)