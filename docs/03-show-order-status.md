# 주문 상태 표시하기

고객은 이제 피자를 주문할 수 있지만 주문 상태를 볼 수 있는 방법은 없습니다. 이 세션에서는 여러 주문이 나열된 "내 주문" 페이지와 개별 주문의 내용 및 상태를 보여주는 "주문 세부 정보" 보기를 구현합니다.

## 탐색 링크 추가

`Shared/MainLayout.razor` 파일을 열고 `NavLink` 컴포넌트를 사용하지 않고 새 링크 요소를 추가해 보겠습니다. `myorders`를 가리키는 일반 HTML `<a>` 태그를 추가합니다.

```html
<div class="top-bar">
    (leave existing content in place)

    <a href="myorders" class="nav-tab">
        <img src="img/bike.svg" />
        <div>My Orders</div>
    </a>
</div>
```

> 링크하는 URL이 `/`로 시작하지 않는 지 확인하세요. 만약 `/myorders`를 링크했다면 동일하게 동작하는 것처럼 보이지만 앱을 root가 아닌 URL에 배포한다면 링크가 끊어집니다. `<base href="/">` 태그는 어떤 컴포넌트가 렌더링하는지에 관계없이 앱의 모든 접두사가 아닌 URL의 접두사를 지정합니다.

지금 앱을 실행하면 기대한 대로 스타일화된 링크가 표시됩니다.

![My orders link](https://user-images.githubusercontent.com/1874516/77241321-a03ba880-6bad-11ea-9a46-c73be397cb5e.png)

이것은 `<NavLink>`를 반드시 사용할 필요는 없다는 것을 보여줍니다. 그럼 사용해야 하는 이유에 대해서 바로 알아보겠습니다.


## "내 주문" 페이지 추가하기

"My Orders"를 클릭하면 "죄송합니다. 이 주소에는 아무 것도 없습니다."라는 페이지가 표시됩니다. 이는 URL `myorders`와 일치하는 페이지를 아직 추가하지 않았기 때문입니다. 그러나 매우 자세히 관찰하고 있다면 이 작업이 클라이언트측 네비게이션(SPA-style)이 아니고 전체 페이지 새로고침이 일어나는 것을 알 수 있습니다.

실제로 일어나고 있는 것은 아래와 같습니다.

1. `myorders` 링크를 클릭합니다.
2. 클라이언트에서 실행중인 블레이저는 `@page` 지시어 속성을 기반으로 이를 클라이언트측 컴포넌트를 찾으려고 시도합니다.
3. 그리고 일치하는 URL이 서버측 코드에 의해 처리되는 경우 블레이저는 전체 페이지 로드 탐색을 진행합니다.
4. 하지만 서버에도 이와 일치하는 항목이 없기 때문에 클라이언트측 블레이저 어플리케이션을 렌더링 합니다.
5. 이 시점에 블레이저는 클라이언트 *또는* 서버에서 일치하는 항목이 없으므로 `App.razor` 컴포넌트에서 `NotFound` 블록을 렌더링 합니다.

원한다면 `App.razor` 컴포넌트의 `NotFound` 블럭에 있는 내용을 변경하여 이 메시지를 원하는 내용으로 수정할 수 있습니다.

짐작하시겠지만 이 경로에 맞는 컴포넌트를 추가하여 링크를 실제로 동작시킬 것입니다. `Pages` 폴더에 `MyOrders.razor` 파일을 만들고 다음 내용을 추가해 주세요.

```html
@page "/myorders"

<div class="main">
    My orders will go here
</div>
```

이제 앱을 실행하면 다음 페이지를 방문할 수 있습니다.

![My orders blank page](https://user-images.githubusercontent.com/1874516/77241343-fc9ec800-6bad-11ea-8176-febf614ed4ad.png)

또한 이번에는 URL이 클라이언트측 SPA내에서 완전히 일치하므로 탐색할 때 전체 페이지 로드가 발생하지 않습니다. 따라서 탐색은 바로 반응합니다.

## 페이지 제목 추가하기

브라우저에 새 페이지의 제목은 **Blazing Pizza**로 표시되는데 "My Orders" 페이지이므로 타이틀에도 반영되는 것이 좋겠습니다. 이 때 `PageTitle` 컴포넌트를 이용하여 `MyOrders.razor` 페이지의 제목을 업데이트 할 수 있습니다.

```html
@page "/myorders"

<PageTitle>Blazing Pizza - My Orders</PageTitle>

<div class="main">
    My orders will go here
</div>
```

이는 `Program.cs` 파일 내부에서 피자 가게 어플리케이션에 `HeadOutlet` 컴포넌트를 추가하였기 때문에 동작합니다. 블레이져는 이 `HeadOutlet`을 HTML 페이지의 헤더 안의 컨텐츠를 작성하는데 이용합니다.

```csharp
builder.RootComponents.Add<HeadOutlet>("head::after");
```

## 내비게이션 위치 하이라이팅

상단 메뉴를 자세히 봐 주세요. "My Orders"에 있을 때 링크가 노란색으로 강조되어 표시되지 않습니다. 어떻게 해야 사용자가 이 페이지에 있을 때 이 링크를 강조할 수 있을 까요? 일반 `<a>` 태그 대신 `NavLink` 컴포넌트를 사용하면 됩니다. `NavLink` 컴포넌트의 유일한 특이점은 `href`가 현재 탐색 상태와 일치하는지 여부에 따라 자신의 `active` CSS 클래스를 전환하는 것입니다.

`MainLayout`에서 방금 추가한 `<a>` 태그를 다음과 같이 바꾸어 줍니다.(태그 이름을 제외하고는 동일합니다.)

```html
<NavLink href="myorders" class="nav-tab">
    <img src="img/bike.svg" />
    <div>My Orders</div>
</NavLink>
```

이제 탐색 상태에 따라 올바르게 하이라이팅 됩니다.

![My orders nav link](https://user-images.githubusercontent.com/1874516/77241358-412a6380-6bae-11ea-88da-424434d34393.png)

## 주문 목록 표시하기

다시 `MyOrders` 컴포넌트 코드로 돌아옵니다. 데이터를 백엔드에서 조회할 수 있도록 다시 한 번 `HttpClient`를 주입합니다. `@page` 지시어 아래에 다음을 추가합니다.

```html
@inject HttpClient HttpClient
```

그리고 필요한 데이터를 비동기로 요청하는 `@code` 코드 블럭을 추가합니다.

```csharp
@code {
    IEnumerable<OrderWithStatus> ordersWithStatus;

    protected override async Task OnParametersSetAsync()
    {
        ordersWithStatus = await HttpClient.GetFromJsonAsync<List<OrderWithStatus>>("orders");
    }
}
```

세 가지 경우에 대해 다른 UI 출력을 만들어 보겠습니다.

 1. 데이터가 로드되기를 기다리는 동안
 2. 사용자가 주문을 한 적이 없는 경우
 3. 사용자가 하나 이상의 주문을 한 경우

Razor 코드의 `@if/else` 블록을 사용하면 간단히 표현할 수 있습니다. 컴포넌트 내부의 마크업을 다음과 같이 업데이트해 주세요.

```html
<div class="main">
    @if (ordersWithStatus == null)
    {
        <text>Loading...</text>
    }
    else if (!ordersWithStatus.Any())
    {
        <h2>No orders placed</h2>
        <a class="btn btn-success" href="">Order some pizza</a>
    }
    else
    {
        <text>TODO: show orders</text>
    }
</div>
```

아마도 이 코드의 일부 부분은 명확하지 않을 수 있으므로 몇 가지 사항을 설명하겠습니다.

### 1. `<text>`요소 란?

`<text>`는 HTML 요소가 아니고 컴포넌트도 아닙니다. 일단 `MyOrders` 컴포넌트가 컴파일되면 `<text>` 태그는 결과물에 전혀 존재하지 않습니다.

`<text>`는 Razor 컴파일러에 C# 소스코드가 아닌 마크업 문자열로 취급하라는 특별한 신호입니다. 구문이 모호할 수 있는 경우에 사용됩니다.

### 2. href=""은 무슨 뜻입니까?

`<a href="">`(빈 문자열 포함)인 경우, 브라우저는 접두사가 없는 모든 URL에 `<base href="/">` 값을 접두사로 붙인다는 점을 기억하세요. 따라서 빈 문자열은 클라이언트 앱의 루트 URL에 연결하는 올바른 방법입니다.

### 3. 어떻게 렌더링 하나요?

위에서 구현한 비동기 플로우는 컴포넌트가 두 번 렌더링된다는 것을 의미합니다. 한 번은 데이터가 로드되기 전에 ("Loading.." 표시), 그리고 한 번은 그후에 (다른 두 출력 중 나머지 하나를 표시)

### 4. OnParametersSetAsync를 사용하는 이유?

매개 변수 및 속성 값을 적용할 때 비동기 작업은 OnParametersSetAsync 라이브사이클 이벤트 중에 발생합니다. 이후 세션에서 매개 변수를 추가할 예정입니다.

### 5. 데이터베이스를 리셋하려면?

"no orders" 상태를 보기 위해 데이터베이스를 리셋하려면 단순히 **BlazingPizza.Server** 프로젝트에 있는 `pizza.db` 파일을 삭제하고 브라우저에서 페이지를 다시 로드해 주세요.

![My orders empty list](https://user-images.githubusercontent.com/1874516/77241390-a4b49100-6bae-11ea-8dd4-e59afdd8f710.png)

## 주문 그리드 렌더링

이제 모든 데이터가 있으니 HTML 그리드를 그리기 위해 Razor 구문을 이용합니다.

`<text>TODO: show orders</text>`코드를 아래와 같이 바꾸어 주세요.

```html
<div class="list-group orders-list">
    @foreach (var item in ordersWithStatus)
    {
        <div class="list-group-item">
            <div class="col">
                <h5>@item.Order.CreatedTime.ToLongDateString()</h5>
                Items:
                <strong>@item.Order.Pizzas.Count()</strong>;
                Total price:
                <strong>£@item.Order.GetFormattedTotalPrice()</strong>
            </div>
            <div class="col">
                Status: <strong>@item.StatusText</strong>
            </div>
            <div class="col flex-grow-0">
                <a href="myorders/@item.Order.OrderId" class="btn btn-success">
                    Track &gt;
                </a>
            </div>
        </div>
    }
</div>
```

코드가 많은 것처럼 보이지만 특별한 내용은 없습니다. 단순히 `@foreach`를 사용하여 `ordersWithStatus`를 반복하여 각각 `<div>`를 출력합니다.
결과는 아래와 같습니다.

![My orders grid](https://user-images.githubusercontent.com/1874516/77241415-feb55680-6bae-11ea-89ba-f8367ef6a96c.png)

## 주문 상세 보기 추가하기

"Track" 버튼을 클릭하면 브라우저에서 `myorders/<id>`(예:`http://localhost:64589/myorders/37`)로 탐색을 시도합니다. 현재는 이 경로와 일치하는 컴포넌트가 없기 때문에 "Sorry, there's nothing at this address." 메시지가 표시됩니다.

다시 한 번 이것을 처리할 컴포넌트를 추가합니다. `Pages` 폴더에 `OrderDetails.razor` 파일을 만들고 아래 내용을 추가합니다.

```html
@page "/myorders/{orderId:int}"

<div class="main">
    TODO: Show details for order @OrderId
</div>

@code {
    [Parameter] public int OrderId { get; set; }
}
```

이 코드는 컴포넌트가 `@page` 지시어에서 토큰으로 선언함으로써 라우터로부터 파라미터를 수신하는 방법을 보여줍니다. `string`을 수신하려면 단순히 `{parameterName}`로 `[Parameter]` 이름과 일치해야 하며 대소문자를 구분하지 않습니다. 숫자 값을 수신하려면 `{parameterName:int}`와 같은 구문을 사용하며 `:int`는 *route constraint*의 예시입니다. bool, datetime, guid 같은 값도 지원합니다.

![Order details empty](https://user-images.githubusercontent.com/1874516/77241434-391ef380-6baf-11ea-9803-9e7e65a4ea2b.png)

라우팅이 실제로 어떻게 작동하는지 궁금하다면 단계별로 살펴보도록 하겠습니다.

1. 앱이 처음 시작되면 `Program.cs`의 코드가 `App`을 루트 컴포넌트로 렌더링하도록 프레임워크에 지시합니다.
2. `App.razor` 파일에 있는 `App` 컴포넌트는 브라우저의 클라이언트측 네비게이션 API와 연동되는 내장된 컴포넌트로, 사용자가 링크를 클릭할 때마다 알림을 받는 네비게이션 이벤트 핸들러를 등록합니다.
3. 사용자가 링크를 클릭할 때마다 `Router`의 코드는 목적지 URL이 동일한 SPA 내에 있는지(즉, `<base href>` 값 아래에 있는지, 일부 컴포넌트의 선언된 경로와 일치하는지)를 확인합니다. 그렇지 않은 경우 기존의 전체 페이지 탐색은 정상적으로 수행되지만, 해당 URL이 SPA 내에 있으면 `Router`가 처리합니다.
4. `Router`는 적합한 `@page` URL 패턴을 가진 컴포넌트를 찾아 처리합니다. 각 `{parameter}` 토큰에는 값이 있어야 하며, 값은 `:int`와 같은 제약 조건과 일치되어야 합니다.
   * 일치하는 컴포넌트가 있다면 `Router`가 렌더링할 것입니다. 이것이 어플리케이션의 모든 페이지가 렌더링되어 온 방식입니다.
   * 일치하는 컴포넌트가 없다면 라우터는 서버의 내용과 일치하는지 전체 페이지 로드를 시도합니다.
   * 서버가 클라이언트측 블레이저앱을 다시 렌더링하기로 한 경우(방문자가 처음 이 URL에 도착하여 서버가 클라이언트와 일치하는 경로일 수 있다고 생각하는 경우에도), 블레이저는 서버 또는 클라이언트에서 일치하는 내용이 없으므로 구성된 `NotFound` 컨텐츠를 표시합니다.

## 주문 세부 정도에 대한 폴링

`OrderDetails`는 `MyOrders`와는 논리가 많이 다릅니다. 컴포넌트가 인스턴스화될 때 한 번만 데이터를 가져오는 것이 아니라 업데이트가 된 데이터가 있는지 몇 초간격으로 서버를 폴링합니다. 이것은 주문 상태를 (거의) 실시간으로 보여주고 나중에 지도에 배달 기사의 위치를 보여주는 것을 가능하게 합니다.

또한 `OrderId`가 잘 못 되었을 가능성도 고려할 것입니다. 다음과 같은 경우에 발생할 수 있습니다

* 존재하지 않는 주문
* 이후 인증 관련 구현이 완료되었을 때, 해당 주문이 다른 사용자를 위한 것이고 사용자가 그 주문을 볼 수 없는 경우

폴링을 구현하기 전에 `OrderDetails.razor`에 다음 지시어를 추가해야 합니다. 일반적으로 `@page` 지시어 바로 아래에 있습니다.

```html
@using System.Threading
@inject HttpClient HttpClient
```

이미 `HttpClient`와 함께 사용되는 `@inject`를 보았으니, 그것이 무엇을 위한 것인지 알 수 있을 것입니다. 또한 일반 `.cs` 파일에서 `@using`과 유사한 것을 볼 수 있으므로 이 역시 크게 어렵지 않을 것입니다. 다만 불행하게도 Visual Studio는 아직 Razor 파일에 `@using` 지시어를 자동으로 추가하지 않기 때문에 필요할 때에는 직접 작성해야 합니다.

이제 폴링을 구현할 수 있습니다. `@code` 코드 블럭을 아래와 같이 수정해 주세요.

```cs
@code {
    [Parameter] public int OrderId { get; set; }

    OrderWithStatus orderWithStatus;
    bool invalidOrder;
    CancellationTokenSource pollingCancellationToken;

    protected override void OnParametersSet()
    {
        // If we were already polling for a different order, stop doing so
        pollingCancellationToken?.Cancel();

        // Start a new poll loop
        PollForUpdates();
    }

    private async void PollForUpdates()
    {
        pollingCancellationToken = new CancellationTokenSource();
        while (!pollingCancellationToken.IsCancellationRequested)
        {
            try
            {
                invalidOrder = false;
                orderWithStatus = await HttpClient.GetFromJsonAsync<OrderWithStatus>($"orders/{OrderId}");
                StateHasChanged();

                if (orderWithStatus.IsDelivered)
                {
                    pollingCancellationToken.Cancel();
                }
                else
                {
                    await Task.Delay(4000);
                }
            }
            catch (Exception ex)
            {
                invalidOrder = true;
                pollingCancellationToken.Cancel();
                Console.Error.WriteLine(ex);
                StateHasChanged();
            }
        }
    }
}
```

코드가 조금 복잡하므로 진행하기 전에 코드의 각 측면을 이해하기 위해 신중하게 살펴야 합니다. 다음은 몇 가지 참고 사항입니다.

* 이 방법은 `OnInitialized` 또는 `OnInitializedAsync` 대신 `OnParametersSet`을 사용합니다. `OnParametersSet`은 컴포넌트의 생명주기 메서드 중 하나로, 컴포넌트가 처음 인스턴스화될 때와 컴포넌트의 매개 변수가 값을 변경할 때마다 실행됩니다. 사용자가 `myorders/2`에서 `myorders/3`으로 직접 링크를 클릭하면 프레임워크는 `OrderDetails` 인스턴스를 유지하고 `OrderId` 매개 변수를 업데이트하기만 합니다.
  * 이와 같이 한 "my orders" 화면에서 다른 주문 화면으로 연결되는 링크를 제공하지 않았기 때문에 이 애플리케이션에서는 시나리오가 발생하지 않지만 향후 탐색 규칙을 변경할 경우 사용하는 것이 올바른 컴포넌트 생명 주기 메서드입니다.
* 폴링을 표현하기 위해 `async void` 메서드를 사용하고 있습니다. 이 메서드는 다른 메서드가 실행되는 동안에도 임의로 길게 실행됩니다. `async void` 메서드는 예외를 호출자에게 예외를 전달할 방법이 없습니다.(일반적으로 호출자가 이미 완료되었기 때문에) 그래서 `try/catch`를 사용하고 일어날 수 있는 모든 예외에 있어 의미있는 작업을 수행하는 것이 중요합니다.
* 폴링을 중단해야 할 때 `CancellationTokenSource`를 사용합니다. 현재는 예외가 발생하거나 주문이 전달되면 중단됩니다.
* `StateHasChanged`를 호출하여 블레이저가 컴포넌트가 가진 데이터가 바뀌었다(혹은 바뀌었을 수 있다)는 것을 알게 해주어야 합니다. 그러면 프레임워크는 컴포넌트를 다시 렌더링 합니다. 그렇지 않으면 프레임워크는 다시 렌더링 할 시점을 알 수 없습니다. 작성된 폴링 로직에 대해서는 알 수 없기 때문입니다.

## 주문 상세 정보 표시

좋습니다. 이제 주문 상제 정보도 가지고 있고 몇 초마다 그 데이터를 폴링하여 업데이트도 하고 있습니다. 하지만 아직 UI에 표시하고 있지는 않습니다. 화면에 표시되도록 수정해보죠. `<div class="main">` 태그를 아래와 같이 수정해 주세요.

```html
<div class="main">
    @if (invalidOrder)
    {
        <h2>Nope</h2>
        <p>Sorry, this order could not be loaded.</p>
    }
    else if (orderWithStatus == null)
    {
        <text>Loading...</text>
    }
    else
    {
        <div class="track-order">
            <div class="track-order-title">
                <h2>
                    Order placed @orderWithStatus.Order.CreatedTime.ToLongDateString()
                </h2>
                <p class="ml-auto mb-0">
                    Status: <strong>@orderWithStatus.StatusText</strong>
                </p>
            </div>
            <div class="track-order-body">
                TODO: show more details
            </div>
        </div>
    }
</div>
```

이 컴포넌트는 세 가지 주요 상태를 설명합니다.

1. `OrderId` 값이 잘못된 경우(예: 전달 받은 데이터로 서버가 오류라고 판단할 때)
2. 아직 데이터를 수신하지 못 한 경우
3. 표시할 데이터가 있는 경우

![Order details status](https://user-images.githubusercontent.com/1874516/77241460-a7fc4c80-6baf-11ea-80c1-3286374e9e29.png)


추가하고자 하는 UI는 주문의 실제 내용입니다. 재사용 가능하게 하기 위해 다른 컴포넌트로 만듭니다.

`Shared` 폴더 안에 `OrderReview.razor` 파일을 생성해 주세요. `Order`를 전달 받아 해당 컨텐츠를 렌더링하도록 아래와 같이 작성해 주세요.

```html
@foreach (var pizza in Order.Pizzas)
{
    <p>
        <strong>
            @(pizza.Size)"
            @pizza.Special.Name
            (£@pizza.GetFormattedTotalPrice())
        </strong>
    </p>

    <ul>
        @foreach (var topping in pizza.Toppings)
        {
            <li>+ @topping.Topping.Name</li>
        }
    </ul>
}

<p>
    <strong>
        Total price:
        £@Order.GetFormattedTotalPrice()
    </strong>
</p>

@code {
    [Parameter] public Order Order { get; set; }
}
```
마지막으로 `OrderDetails.razor`로 돌아가 `TODO: show more details` 부분을 새로 만든 `OrderReview` 컴포넌트로 바꾸어 주세요.

```html
<div class="track-order-body">
    <div class="track-order-details">
        <OrderReview Order="orderWithStatus.Order" />
    </div>
</div>
```

(올바른 스타일링을 위해 필요한 CSS 클래스 'track-order-details'와 함께 추가 'div'를 추가하는 것을 잊지 마세요.)

이제 주문 세부 정보가 표시 됩니다!

![Order details](https://user-images.githubusercontent.com/1874516/77241512-2e189300-6bb0-11ea-9740-fe778e0ce622.png)


## 실시간 업데이트 보기

백엔드 서버가 주문 상태를 업데이트하여 실제 발송 및 배달 프로세스를 시뮬레이션합니다. 이 작업을 수행하려면 새 주문을 시도한 다음 즉시 세부 정보를 확인하십시오.

처음에는 주문 상태가 *Preparing(준비중)*이고 이후 10-15초 후에는 "Out for delivery(배송중)"으로 변경되고, 60초 후에는 *Delivered(배송완료)*로 변경됩니다. `OrderDetails`가 업데이트를 폴링하기 때문에 사용자가 페이지를 새로 고칠 필요 없이 UI가 업데이트됩니다.

## Dispose를 기억하세요!

현재 상태에서 바로 운영으로 앱을 배포하면 곤란한 일이 발생할 것입니다. `OrderDetails`은 폴링을 시작하지만 끝나지 않습니다. 사용자가 수백 개의 다른 주문을 조회한다면(따라서 수백 개의 다른 `OrderDetails` 인스턴스가 만들어집니다.), 결과적으로는 마지막 한 개 주문만 보여지지만 수백 개의 폴링 폴링 과정이 동시에 실행될 것입니다.

실제로 아래 방법으로 이런 현상을 확인할 수 있습니다.

1. "my orders"로 이동합니다.
2. 세부 정보를 보기 위해 "Track" 버튼을 클릭하세요.
3. "my orders"로 돌아가기 위해 "Back" 버튼을 클릭하세요.
4. 2와 3단계를 여러 번 반복합니다. (예: 20번 정도)
5. 이제 브라우저의 디버깅 도구를 열고 네트워크 탭을 살펴보세요. 20개 이상의 폴링 프로세스가 동시에 있으므로 몇 초마다 20개 이상의 HTTP 요청이 실행되는 것을 볼 수 있습니다.

이는 클라이언트 측 메모리 및 CPU 시간, 네트워크 대역폭 및 서버 리소스를 낭비합니다.

이 문제를 해결하기 위해서는 더 이상 표시되지 않으면 폴링을 중지하도록 `OrderDetails`를 만들 필요가 있습니다. `IDisposable` 인터페이스를 사용하면 쉽게 해결할 수 있습니다.

`OrderDetails.razor` 파일을 열고 파일 가장 위의 다른 지시문 밑에 아래 지시문을 추가해 주세요.

```html
@implements IDisposable
```

이제 어플리케이션을 컴파일하려고 하면 컴파일러가 오류를 이야기 합니다.

```
error CS0535: 'OrderDetails' does not implement interface member 'IDisposable.Dispose()'
```

`@code` 코드 블럭에 아래 메서드를 추가하여 이를 해결합니다.

```cs
void IDisposable.Dispose()
{
    pollingCancellationToken?.Cancel();
}
```

컴포넌트 인스턴스를 삭제하고 UI에서 제거되면 프레임워크는 자동으로 `Dispose`를 호출합니다.

이렇게 수정하고 나면 동시에 많은 폴링 프로세스를 다시 시작할 수 있으며 컴포넌트가 없어지고 난 뒤에 더 이상 실행되지 않는 것을 볼 수 있습니다. 이제 폴링을 하고 있는 것은 화면에 표시되는 컴포넌트 뿐입니다.

## 주문 상세 자동 탐색

현재, 사용자가 주문을 하면 `Index` 컴포넌트가 단순히 상태를 리셋하고 주문이 흔적 없이 사라지는 것처럼 보입니다. 이런 표현은 사용자에게 불안감을 줍니다. 주문은 데이터베이스에 저장되었지만 사용자는 알 수 없으니까요.

일단 주문이 들어가면 자동으로 주문 상세 정보를 표시하면 좋을 것 같습니다. 그리고 생각 보다 쉽게 할 수 있습니다.

다시 `Index` 컴포넌트 코드로 돌아와서 가장 상단에 아래 지시문을 추가해 주세요.

```
@inject NavigationManager NavigationManager
```

`NavigationManager`를 사용하면 URI 및 네비게이션 상태를 컨트롤 할 수 있습니다. 현재 URL을 가져오거나 다른 URL로 이동하는 메서드를 가지고 있습니다.

`NavigationManager.NavigateTo`를 호출하도록 `PlaceOrder` 코드를 수정해 주세요.

```csharp
async Task PlaceOrder()
{
    var response = await HttpClient.PostAsJsonAsync("orders", order);
    var newOrderId = await response.Content.ReadFromJsonAsync<int>();
    order = new Order();
    NavigationManager.NavigateTo($"myorders/{newOrderId}");
}
```

이제 서버가 주문을 받아주면 브라우저는 "주문 상세 정보"를 표시하고 폴링을 시작합니다.

다음 세션 - [상태 관리의 리펙터링](04-refactor-state-management.md)

원문 읽기 - [Show order status](https://github.com/dotnet-presentations/blazor-workshop/blob/main/docs/03-show-order-status.md)