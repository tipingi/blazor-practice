# 내 피자 만들기

이 세션에서는 사용자가 내 피자를 만들고 그것을 주문에 추가하 수 있도록 피자 가게 앱을 업데이트 합니다.

## 이벤트 핸들링

사용자가 피자 메뉴를 클릭하면 내 기호에 맞게 변경하고 주문에 추가할 수 있는 대화상자가 나게 합니다. 블레이저 앱에서 DOM UI 이벤트를 처리하려면 이벤트를 처리할 HTML 특성에 호출할 C# 함수를 지정합니다. 함수에 이벤트 인수를 사용할 수 있지만 필수는 아닙니다.

*Pages/Index.razor* 파일에서 `@onclick` handler 핸들러를 각 피자 메뉴 항목에 추가합니다.

```html
@foreach (var special in specials)
{
    <li @onclick="@(() => Console.WriteLine(special.Name))" style="background-image: url('@special.ImageUrl')">
        <div class="pizza-info">
            <span class="title">@special.Name</span>
            @special.Description
            <span class="price">@special.GetFormattedBasePrice()</span>
        </div>
    </li>
}
```

앱을 실행하고 피자를 클릭하면 브라우저 콘솔에 피자 이름이 표시되지는 확인해 주세요.

![@onclick-event](https://user-images.githubusercontent.com/1874516/77239615-f56dbf00-6b99-11ea-8535-ddcc8bc0d8ae.png)


`@` 기호는 Razor 파일에서 C# 코드의 시작을 나타내기 위해 사용됩니다. C# 코드의 시작과 끝을 명확히 하기 위해 필요한 경우 괄호로 C# 코드를 둘러싸십시오.

*Index.razor* 파일의 `@code` 코드 블럭을 수정하여 기호에 맞게 변경한 내용을 저장하고 이 대화상자가 표시되는지 여부를 확인하 수 있는 필드를 추가합니다.

```csharp
List<PizzaSpecial> specials;
Pizza configuringPizza;
bool showingConfigureDialog;
```

피자 메뉴를 클릭할 때 이벤트를 처리할 수 있도록 `ShowConfigurePizzaDialog` 메서드를 `@code` 코드 블럭에 추가합니다.

```csharp
void ShowConfigurePizzaDialog(PizzaSpecial special)
{
    configuringPizza = new Pizza()
    {
        Special = special,
        SpecialId = special.Id,
        Size = Pizza.DefaultSize,
        Toppings = new List<PizzaTopping>(),
    };

    showingConfigureDialog = true;
}
```

`@onclick` handler를 `Console.WriteLine`대신에 `ShowConfigurePizzaDialog`를 호출할 수 있도록 수정합니다.

```html
<li @onclick="@(() => ShowConfigurePizzaDialog(special))" style="background-image: url('@special.ImageUrl')">
```

## "내 피자 만들기" 대화상자 구현

이제 사용자가 피자를 선택할 때 표시할 수 있도록 "내 피자 만들기" 대화상자를 구현해야 합니다. "내 피자 만들기" 대화상자는 피자의 크기와 원하는 토핑을 지정하고 가격를 표시하며 주문에 피자를 추가할 수 있는 새로운 컴포넌트가 됩니다.

*Shared* 디렉토리에 *ConfigurePizzaDialog.razor* 파일을 추가합니다. 이 컴포넌트는 별도의 페이지가 아니므로 '@page' 지시어가 필요하지 않습니다.

> Note: Visual Studio에서는 Solution Explorer에서 *Shared* 디렉토리를 마우스 오른쪽 버튼으로 클릭한 다음 *Add* -> *New Item*을 선택하여 *Razor Component* 항목 템플릿을 사용하여 새 Razor 구성 요소를 추가할 수 있습니다.

`ConfigurePizzaDialog`는 구성할 피자를 지정하는 `Pizza` 매개 변수가 있어야 합니다. 컴포넌트 매개 변수는 `[Parameter]` 속성으로 정의합니다. 아래와 같이 `Pizza` 매개 변수를 `ConfigurePizzaDialog`에 정의하는 코드를 `@code` 코드 블럭에 추가해 주세요.

```csharp
@code {
    [Parameter] public Pizza Pizza { get; set; }
}
```

> Note: 컴포넌트 매개 변수 값은 프레임워크에 의해 설정되므로 `public`으로 선언된 setter가 있어야 합니다. 하지만 매개 변수는 렌더링 프로세스의 일부로 반드시 프레임워크에 의해서만 설정되어야 합니다. 컴포넌트의 상태가 렌더 출력과 동기화되지 않으므로 컴포넌트 외부에서 이러한 매개 변수 값을 덮어쓰는 코드를 작성하면 안됩니다.

`ConfigurePizzaDialog`를 위한 기본 마그업을 추가합니다.

```html
<div class="dialog-container">
    <div class="dialog">
        <div class="dialog-title">
            <h2>@Pizza.Special.Name</h2>
            @Pizza.Special.Description
        </div>
        <form class="dialog-body"></form>
        <div class="dialog-buttons">
            <button class="btn btn-secondary mr-auto">Cancel</button>
            <span class="mr-center">
                Price: <span class="price">@(Pizza.GetFormattedTotalPrice())</span>
            </span>
            <button class="btn btn-success ml-auto">Order</button>
        </div>
    </div>
</div>
```

피자 메뉴를 선택하면 `ConfigurePizzaDialog`가 표시되도록 *Pages/Index.razor* 파일을 수정합니다. `ConfigurePizzaDialog`는 현재 페이지를 오버레이하도록 지정되어 있으므로 아래 코드가 어디에 있는지는 중요하지 않습니다.

```html
@if (showingConfigureDialog)
{
    <ConfigurePizzaDialog Pizza="configuringPizza" />
}
```

앱을 실행하고 피자 메뉴를 선택하여 `ConfigurePizzaDialog`이 표시되는 지 확인해 주세요.

![initial-pizza-dialog](https://user-images.githubusercontent.com/1874516/77239685-e3d8e700-6b9a-11ea-8adf-5ee8a69f08ae.png)

안타깝게도 이 시점에서는 대화상자를 닫을 수 있는 기능이 없습니다. 대화 상자 자체에 대한 구현을 시작하도록 하죠.

## 데이터 바인딩

사용자는 피자의 크기를 정할 수 있어야 합니다. `ConfigurePizzaDialog`에 피자 크기를 조정할 수 있는 슬라이더를 마크업을 추가합니다. 기존의 `<form class="dialog-body"></form>` 요소를 대체해야 합니다.

```html
<form class="dialog-body">
    <div>
        <label>Size:</label>
        <input type="range" min="@Pizza.MinimumSize" max="@Pizza.MaximumSize" step="1" />
        <span class="size-label">
            @(Pizza.Size)" (£@(Pizza.GetFormattedTotalPrice()))
        </span>
    </div>
</form>
```

이제 대화 상자에는 피자 크기를 조정하는데 사용할 수 있는 슬라이더가 표시됩니다. 하지만 지금은 조정해도 아무 것도 처리되지 않습니다.

![Slider](https://user-images.githubusercontent.com/1430011/57576985-eff40400-7421-11e9-9a1b-b22d96c06bcb.png)

슬라이더의 값이 `Pizza.Size`의 값이 되었으면 좋겠습니다. 대화상자가 열리면 슬라이더는 `Pizza.Size`에서 값을 얻어서 표시합니다. 슬라이더를 움직이면 `Pizza.Size`에 저장된 값이 수정됩니다. 이 개념을 양방향 바인딩이라고 합니다.

양방향 바인딩을 수동으로 구현하려면 다음 코드와 같이 value와 @onchange를 결합하여 구현할 수 있습니다.(더 쉬운 방법이 있으므로 이 코드를 실제로 어플리케이션을 추가할 필요는 없습니다.)

```html
<input
    type="range"
    min="@Pizza.MinimumSize"
    max="@Pizza.MaximumSize"
    step="1"
    value="@Pizza.Size"
    @onchange="@((ChangeEventArgs e) => Pizza.Size = int.Parse((string) e.Value))" />
```

블레이저에서 `@bind` 지시 속성을 사용하여 동일한 동작을 하는 양방향 바인딩을 지정할 수 있습니다. `@bind`를 사용한 마크업을 아래와 같습니다.

```html
<input type="range" min="@Pizza.MinimumSize" max="@Pizza.MaximumSize" step="1" @bind="Pizza.Size"  />
```

그러나 변경이 없는 경우에 `@bind`를 사용하면 정확히 우리가 기대하는 동작을 하지 않습니다. 한 번 시도해 보고 동작을 확인해 보세요. 업데이트 이벤트는 슬라이더가 릴리즈되어야 발생합니다.

![Slider with default bind](https://user-images.githubusercontent.com/1874516/51804870-acec9700-225d-11e9-8e89-7761c9008909.gif)

사실 슬라이더가 이동할 때 값이 업데이트 되는 것을 기대합니다. 블레이저의 데이터 바인딩은 `@bind:<eventname>` 구문을 사용하여 특정 이벤트를 지정할 수 있습니다. `oninput` 이벤트에 바인딩하려면 아래 코드처럼 작성합니다.

```html
<input type="range" min="@Pizza.MinimumSize" max="@Pizza.MaximumSize" step="1" @bind="Pizza.Size" @bind:event="oninput" />
```

이제 슬라이더를 이동하면 피자 크기가 업데이트 됩니다.

![Slider bound to oninput](https://user-images.githubusercontent.com/1874516/51804899-28e6df00-225e-11e9-9148-caf2dd269ce0.gif)

## 토핑 추가하기

`ConfigurePizzaDialog`에서 사용자는 추가 토핑을 선택할 수도 있습니다. 선택 가능한 토핑을 저장하기 위한 목록을 추가해 주세요. **BlazingPizza.Server** 프로젝트의 `PizzaApiExtensions.cs`에 정의된 minimal API(`/toppings`)에 HTTP GET 요청을 통해 사용 가능한 목록을 가져와 초기화 합니다.

```csharp
@inject HttpClient HttpClient

<div class="dialog-container">
...
</div>

@code {
    List<Topping> toppings;

    [Parameter] public Pizza Pizza { get; set; }

    protected async override Task OnInitializedAsync()
    {
        toppings = await HttpClient.GetFromJsonAsync<List<Topping>>("toppings");
    }
}

```

대화 상자에 아래 마크업을 추가하여 사용 가능한 토핑 목록과 선택한 토핑을 표시합니다. 기존의 `<div>` 태크 아래에 `<form class="dialog-body">`태그 안에 아래 마크업을 추가해 주세요.

```html
<div>
    <label>Extra Toppings:</label>
    @if (toppings == null)
    {
        <select class="custom-select" disabled>
            <option>(loading...)</option>
        </select>
    }
    else if (Pizza.Toppings.Count >= 6)
    {
        <div>(maximum reached)</div>
    }
    else
    {
        <select class="custom-select" @onchange="ToppingSelected">
            <option value="-1" disabled selected>(select)</option>
            @for (var i = 0; i < toppings.Count; i++)
            {
                <option value="@i">@toppings[i].Name - (£@(toppings[i].GetFormattedPrice()))</option>
            }
        </select>
    }
</div>

<div class="toppings">
    @foreach (var topping in Pizza.Toppings)
    {
        <div class="topping">
            @topping.Topping.Name
            <span class="topping-price">@topping.Topping.GetFormattedPrice()</span>
            <button type="button" class="delete-topping" @onclick="@(() => RemoveTopping(topping.Topping))">x</button>
        </div>
    }
</div>
```

토핑 선택 및 제거를 위한 이벤트 핸들러 역시 추가해 주세요.

```csharp
void ToppingSelected(ChangeEventArgs e)
{
    if (int.TryParse((string)e.Value, out var index) && index >= 0)
    {
        AddTopping(toppings[index]);
    }
}

void AddTopping(Topping topping)
{
    if (Pizza.Toppings.Find(pt => pt.Topping == topping) == null)
    {
        Pizza.Toppings.Add(new PizzaTopping() { Topping = topping });
    }
}

void RemoveTopping(Topping topping)
{
    Pizza.Toppings.RemoveAll(pt => pt.Topping == topping);
}
```

이제 토핑을 추가하고 제거할 수 있습니다.

![Add and remove toppings](https://user-images.githubusercontent.com/1874516/77239789-c0626c00-6b9b-11ea-9030-0bcccdee6da7.png)


## 컴포넌트 이벤트

취소 및 주문 버튼은 아직 아무 것도 동작하지 않습니다. 사용자가 피자를 주문하거나 취소할 때 `Index` 컴포넌트와 통신할 수 있는 방법이 필요합니다. 컴포넌트 이벤트를 정의하면 그렇게 할 수 있습니다. 컴포넌트 이벤트는 부모 컴포넌트가 구독할 수 있는 콜백 매개 변수 입니다.

`ConfigurePizzaDialog` 컴포넌트에 `OnCancel`과 `OnConfirm`, 두 개의 속성을 추가해 주세요. 두 속성 모두 `EventCallback` 타입이어야 합니다.

```csharp
[Parameter] public EventCallback OnCancel { get; set; }
[Parameter] public EventCallback OnConfirm { get; set; }
```

 `ConfigurePizzaDialog` 컴포넌트에 `OnCancel`과 `OnConfirm`을 `@onclick` 이벤트 핸들러에 각각 추가합니다.

```html
<div class="dialog-buttons">
    <button class="btn btn-secondary mr-auto" @onclick="OnCancel">Cancel</button>
    <span class="mr-center">
        Price: <span class="price">@(Pizza.GetFormattedTotalPrice())</span>
    </span>
    <button class="btn btn-success ml-auto" @onclick="OnConfirm">Order ></button>
</div>
```

`Index` 컴포넌트에서 대화상자를 숨기는 이벤트 핸들러를 `ConfigurePizzaDialog`에 연결하도록 `OnCancel`에 추가합니다.

```html
<ConfigurePizzaDialog Pizza="configuringPizza" OnCancel="CancelConfigurePizzaDialog" />
```

```csharp
void CancelConfigurePizzaDialog()
{
    configuringPizza = null;
    showingConfigureDialog = false;
}
```

이제 대화상자의 취소 버튼을 클릭하면 `Index.CancelConfigurePizzaDialog`가 실행되고 그러면 `Index` 컴포넌트가 다시 렌더링됩니다. 이제 `showingConfigureDialog`가 `false`이므로 대화상자가 표시되지 않게 됩니다.

일반적으로 이벤트를 트리거하면(예: 취소 버튼 클릭) 이벤트 핸들러 델리게이트를 정의한 컴포넌트가 다시 렌더링됩니다. `Action`이나 `Func<string, Task>`와 같은 델리게이트를 사용하여 이벤트를 정의할 수 있습니다. 컴포넌트에 속하지 않는 이벤트 핸들러 델리게이트를 사용하고 싶을 때가 있는데, 일반 델리게이트 유형을 사용하여 이벤트를 정의하면 아무것도 렌더링되거나 업데이트되지 않습니다.

`EventCallback`은 컴파일러에게 이러한 문제를 해결하는 방법으로 알려진 특수한 타입입니다. 이는 컴파일러에게 이벤트 핸들러를 포함하는 컴포넌트에게 이벤트가 발생하도록 지시합니다. `EventCallback`를 사용하는 몇 가지 트릭이 더 있지만 지금은 `EventCallback`를 사용하면 컴포넌트의 적당한 위치에 적절히 이벤트를 발생시킬 수 있다는 것만 기억해 주세요.

앱을 실행하고 취소 버튼을 클릭하면 대화 상자가 없어지는 것을 확인해 주세요.

`OnConfirm` 이벤트가 발생하면 토핑을 추가한 피자를 사용자의 주문에 추가합니다. `Index` 컴포넌트에 `Order`필드를 추가하여 사용자의 주문을 저장합니다.

```csharp
List<PizzaSpecial> specials;
Pizza configuringPizza;
bool showingConfigureDialog;
Order order = new Order();
```

`Index` 컴포넌트에 선택된 피자를 주문에 추가하는 `OnConfirm` 이벤트 핸들러를 추가하고 `ConfigurePizzaDialog`에 연결합니다.

```html
<ConfigurePizzaDialog
    Pizza="configuringPizza"
    OnCancel="CancelConfigurePizzaDialog"
    OnConfirm="ConfirmConfigurePizzaDialog" />
```

```csharp
void ConfirmConfigurePizzaDialog()
{
    order.Pizzas.Add(configuringPizza);
    configuringPizza = null;

    showingConfigureDialog = false;
}
```

앱을 실행하고 주문 버튼을 클릭하면 대화상자가 사라지는 지 확인합니다. 주문 정보를 표시하는 UI가 없기 때문에 주문이 추가되었는지 아직 확인할 수 없습니다. 다음에 설명하겠습니다.

## 현재 주문 표시하기

다음으로 현재 주문에 구성중인 피자를 표시하고 전체 가격을 계산하고 주문할 수 있는 방법을 제공해야 합니다.

선택된 피자를 표시하기 위한 `ConfiguredPizzaItem` 컴포넌트를 새롭게 만듭니다. 여기에는 구성된 피자와 피자가 제거된 시점에 대한 이벤트의 두 가지 매개 변수가 필요합니다.

```html
<div class="cart-item">
    <a @onclick="OnRemoved" class="delete-item">x</a>
    <div class="title">@(Pizza.Size)" @Pizza.Special.Name</div>
    <ul>
        @foreach (var topping in Pizza.Toppings)
        {
        <li>+ @topping.Topping.Name</li>
        }
    </ul>
    <div class="item-price">
        @Pizza.GetFormattedTotalPrice()
    </div>
</div>

@code {
    [Parameter] public Pizza Pizza { get; set; }
    [Parameter] public EventCallback OnRemoved { get; set; }
}
```

현재 주문의 선택된 피자을 표시하는 오른쪽 사이드 패널을 추가하기 위해 `Index` 컴포넌트의 `main` div 아래에 아래 마크업을 추가하세요.

```html
<div class="sidebar">
    @if (order.Pizzas.Any())
    {
        <div class="order-contents">
            <h2>Your order</h2>

            @foreach (var configuredPizza in order.Pizzas)
            {
                <ConfiguredPizzaItem Pizza="configuredPizza" OnRemoved="@(() => RemoveConfiguredPizza(configuredPizza))" />
            }
        </div>
    }
    else
    {
        <div class="empty-cart">Choose a pizza<br>to get started</div>
    }

    <div class="order-total @(order.Pizzas.Any() ? "" : "hidden")">
        Total:
        <span class="total-price">@order.GetFormattedTotalPrice()</span>
        <button class="btn btn-warning" disabled="@(order.Pizzas.Count == 0)" @onclick="PlaceOrder">
            Order >
        </button>
    </div>
</div>
```

또한 선택된 피자를 제거하고 주문을 넣기 위한 이벤트 핸들러를 `Index` 컴포넌트에 추가하기 위해 아래 코드를 추가해 주세요.

```csharp
void RemoveConfiguredPizza(Pizza pizza)
{
    order.Pizzas.Remove(pizza);
}

async Task PlaceOrder()
{
    await HttpClient.PostAsJsonAsync("orders", order);
    order = new Order();
}
```

이제 주문에서 선택된 피자를 추가 및 제거하고 주문을 넣을 수 있습니다.

![Order list pane](https://user-images.githubusercontent.com/1874516/77239878-b55c0b80-6b9c-11ea-905f-0b2558ede63d.png)


주문이 데이터베이스에 성공적으로 추가되었지만 UI에서는 아직 이러한 일이 발생했음을 나타내는 것이 아무 것도 없습니다. 그것은 다음 세션에서 다룰 것입니다.

다음 세션 - [주문 상태 표시하기](03-show-order-status.md)

원문 읽기 - [Customize a pizza](https://github.com/dotnet-presentations/blazor-workshop/blob/main/docs/02-customize-a-pizza.md)