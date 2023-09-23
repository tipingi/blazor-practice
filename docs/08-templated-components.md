# 템플릿 컴포넌트

원래 컴포넌트 중 일부를 리펙토링하여 더 재사용할 수 있도록 해보죠. 그 과정에서 새로운 컴포넌트를 위한 별도의 라이브러리 프로젝트도 만들 것입니다.

Razor 클래스 라이브러리 템플릿을 이용하여 새 프로젝트를 만들겠습니다.

## 컴포넌트 라이브러리 만들기(Visual Studio)

Visual Studio를 사용하여 솔루션 탐색기 맨 위에 있는 `Solution`을 마우스 오른쪽 단추로 클릭하고 `Add->New Project`를 선택합니다.

그런 다음 Razor 클래스 라이브러리 템플릿을 선택합니다.

![image](https://user-images.githubusercontent.com/1430011/65823337-17990c80-e209-11e9-9096-de4cb0d720ba.png)

프로젝트 이름에 `BlazingComponents`를 입력하고 *Create*를 클릭합니다.

## 컴포넌트 라이브러리 만들기(명령줄)
**dotnet**를 사용하여 새 프로젝트를 만들려면 솔루션 파일이 있는 디렉토리에서 다음 명령을 실행합니다.

```
dotnet new razorclasslib -o BlazingComponents
dotnet sln add BlazingComponents
```

이렇게 하면 `BlazingComponents`라는 새로운 프로젝트가 생성되어 솔루션 파일에 추가됩니다.

## 라이브러리 프로젝트의 이해

*솔루션 탐색기*에서 *BlaidingComponents* 프로젝트 이름을 두 번 클릭하여 프로젝트 파일을 엽니다. 여기서는 아무것도 수정하지 않지만 몇 가지를 이해하는 것이 좋습니다.

다음과 같이 보입니다.

```xml
<Project Sdk="Microsoft.NET.Sdk.Razor">

  <PropertyGroup>
    <TargetFramework>netstandard2.0</TargetFramework>
    <RazorLangVersion>3.0</RazorLangVersion>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.AspNetCore.Components" Version="3.1.3" />
    <PackageReference Include="Microsoft.AspNetCore.Components.Web" Version="3.1.3" />
  </ItemGroup>

</Project>
```

이해해 두면 좋을 것들 몇 가지 있습니다.

먼저 패키지 타겟은 `netstandard2.0`입니다. Blazor Server는 `netcoreapp3.1`을 사용하고 블레이져 웹어셈블리는 `netstandard2.1`을 사용하므로 `netstandard2.0`을 대상으로 하면 두 가지 시나리오 모두에서 작동합니다.

추가적으로 `<RazorLangVersion>3.0</RazorLangVersion>`은 Razor 언어 버전을 설정합니다. 컴포넌트와 `.razor` 파일 확장자를 지원하려면 버전 3이 필요합니다.

마지막으로 `<PackageReference />` 요소는 블레이저 컴포넌트 모델에 패키지 참조를 추가합니다.

## 템플릿 대화상자 작성하기

`Index`의 일부인 대화상자를 다시 검토하고 어플리케이션과 분리되도록 할 것입니다.

*재사용 가능한 대화상자*가 어떻게 작동해야 하는지 생각해 보겠습니다. 대화상자 컴포넌트가 스스로 표시하고 숨김을 처리하고 대화상자처럼 시각적으로 나타날 수도 있습니다. 하지만 정말로 재사용 가능하려면 대화상자 내부에 대한 내용을 제공할 수 있어야 합니다. *content*를 매개변수로 받아들이는 컴포넌트를 *템플릿 컴포넌트*라고 합니다.

블레이저는 바로 이런 케이스에서 활용하기 좋은 기능을 가지고 있는데, 이것은 레이아웃이 작동하는 방법과 비슷합니다. 레이아웃에 `Body` 매개 변수가 있고 레이아웃은 `Body` *주위에* 다른 콘텐츠를 배치하게 됩니다. 레이아웃에서 `Body` 매개 변수는 런타임이 특별하게 처리하는 위임자 타입인 `RenderFragment` 유형입니다. 좋은 소식은 이 기능이 레이아웃에만 제한되지 않는다는 것입니다. 모든 컴포넌트가 `RenderFragment` 타입의 매개 변수를 선언할 수 있습니다. `App.razor` 안에서도 이 기능을 광범위하게 사용했습니다. 라우팅 및 권한 부여를 처리하는 데 사용되는 모든 컴포넌트는 템플릿화된 컴포넌트입니다.

이제 새로운 대화 컴포넌트를 만들어보겠습니다. `BlazingComponents` 프로젝트에서 `TemplatedDialog.razor`라는 새 컴포넌트 파일을 만듭니다. `TemplatedDialog.razor` 파일 안에 다음 마크업을 넣어주세요.

```html
<div class="dialog-container">
    <div class="dialog">

    </div>
</div>
```

매개 변수를 추가하지 않았기 때문에 이것은 아직 아무 것도 하지 않습니다. 해야 하는 2 가지 일을 기억해 주세요.

1. 대화상자의 컨텐츠를 매개변수로 받기
2. 조건에 따라 대화상자 표시

먼저 `RenderFragment` 타입의 `ChildContent` 파라미터를 추가합니다. `ChildContent` 이름은 특별한 파라미터 이름으로 하나의 콘텐츠 파라미터를 받고자 할 때 관행적으로 사용됩니다.

```razor
@code {
    [Parameter] public RenderFragment ChildContent { get; set; }
}
```

그런 다음 마크업 중간에 있는 `ChildContent`를 *render*하도록 마크업을 수정합니다. 아래처럼 바꾸면 됩니다.

```html
<div class="dialog-container">
    <div class="dialog">
        @ChildContent
    </div>
</div>
```

만약 이 구조가 이상하게 보인다면 비슷한 패턴을 따르는 레이아웃 파일과 비교하여 확인해 보세요. `RenderFragment`가 위임자 타입이긴 하지만, *render*를 하는 방법은 호출하는 것이 아니라, 런타임에서 호출할 수 있도록 정상적인 표현식에 값을 배치하는 것입니다.

다음으로 이 대화 상자에 조건부 동작을 부여하기 위해 `show`라는 `bool` 타입의 매개 변수를 추가해 보겠습니다. 그렇게 한 뒤에는 기존의 모든 컨텐츠를 `@if (Show) { ... }`로 감싸주세요. 전체 파일은 아래와 같습니다.

```html
@if (Show)
{
    <div class="dialog-container">
        <div class="dialog">
            @ChildContent
        </div>
    </div>
}

@code {
    [Parameter] public RenderFragment ChildContent { get; set; }
    [Parameter] public bool Show { get; set; }
}
```

솔루션을 빌드하고 이 단계에서 컴파일이 모두 잘 되었는지 확인해 주세요. 다음은 이 새 컴포넌트의 사용법으로 넘어가겠습니다.

## 템플릿 라이브러리의 참조 추가

`BlazingPizza.Client` 프로젝트 안에 있는 이 컴포넌트를 사용하기 전에 프로젝트에 참조를 추가해야 합니다. `BlazingPizza.Client`에서 `BlazingComponents` 프로젝트를 참조 추가하면 됩니다.

이것이 완료되면 이제 작은 단계가 하나 더 남아있습니다. `BlazingPizza.Client`의 가장 상위 디렉토리의 `_Imports.razor` 파일을 열고 마지막에 아래 줄을 추가해 주세요.

```html
@using BlazingComponents
```

프로젝트 참조가 추가되었으니 빌드를 다시 수행하여 모든 것이 여전히 컴파일되는지 확인합니다.

## 또 다른 리펙토링

`TemplatedDialog`에는 몇 개의 `div`가 포함되어 있었던 걸 기억해 주세요. 이것은 `ConfigurePizzaDialog`의 일부 구조를 복제한 것입니다. 정리해보죠. `ConfigurePizzaDialog.razor`을 열면 아래와 같이 보일 것입니다.

```html
<div class="dialog-container">
    <div class="dialog">
        <div class="dialog-title">
        ...
        </div>
        <form class="dialog-body">
        ...
        </form>

        <div class="dialog-buttons">
        ...
        </div>
    </div>
</div>
```

가장 바깥쪽에 있는 두 개의 `div` 요소는 이제 `TemplatedDialog`컴포넌트의 일부이므로 제거해야 합니다. 이들을 제거한 후에는 다음과 같을 것입니다.

```html
<div class="dialog-title">
...
</div>
<form class="dialog-body">
...
</form>

<div class="dialog-buttons">
...
</div>
```

## 새 대화상자 사용하기

`Index.razor`에서 이 새로운 템플릿 컴포넌트를 사용할 것입니다. `Index.razor`를 열고 다음과 같은 코드 블록을 찾습니다:

```html
@if (OrderState.ShowingConfigureDialog)
{
    <ConfigurePizzaDialog
        Pizza="OrderState.ConfiguringPizza"
        OnConfirm="OrderState.ConfirmConfigurePizzaDialog"
        OnCancel="OrderState.CancelConfigurePizzaDialog" />
}
```

찾은 코드를 삭제하고 새 컴포넌트의 호출로 바꿀 것입니다. 위의 코드 블록을 아래와 같은 코드 블럭으로 바꾸어 주세요.

```html
<TemplatedDialog Show="OrderState.ShowingConfigureDialog">
    <ConfigurePizzaDialog
        Pizza="OrderState.ConfiguringPizza"
        OnCancel="OrderState.CancelConfigurePizzaDialog"
        OnConfirm="OrderState.ConfirmConfigurePizzaDialog" />
</TemplatedDialog>
```

이는 `OrderState.ShowingConfigureDialog`를 기준으로 자기자신을 표시하거나 숨기는 새로운 `TemplatedDialog` 컴포넌트를 배치합니다. 또한 컨테츠를 `ChildContent` 매개 변수로 전달하고 있습니다. `<TemplatedDialog> </TemplatedDialog>`안에 있는 모든 컨텐츠를 `ChildContent` 매개 변수로 해서 호출하였기 때문에 `RenderFragment` 대리자에 의해 캡쳐되어 `TemplatedDialog`에 전달됩니다.

> NOTE: 템플릿 컴포넌트는 여러 개의 `RenderFragment` 매개 변수를 가질 수 있습니다. 여기에서 보여주는 것은 호출자가 *main* 콘텐츠를 나타내는 하나의 `RenderFragment`를 제공하고자 할 때의 편리한 규칙입니다.

이 시점에서 코드를 실행하고 새 대화 상자가 올바르게 작동하는지 확인해 보세요. 다음 단계로 넘어가기 전에 이 대화 상자가 정상적으로 작동해야 합니다.

## 고급 템플릿 컴포넌트

이제 기본 템플릿 대화 상자는 완성하였으니 좀 더 세련된 작업을 시도해 보겠습니다. `MyOrders.razor` 페이지에는 주문 목록이 표시되지만 3가지 상태 논리(로딩중, 빈 목록, 목록 표시)도 포함되어 있습니다. 이 논리를 재사용 가능한 컴포넌트로 유용할까요? 한 번 시도해보죠.

`BlazingComponents` 프로젝트에서 `TemplatedList.razor` 파일을 새로 만드는 것으로 시작합니다. 이 목록에는 몇 가지 기능이 있습니다:
1. 모든 데이터의 비동기 로드
2. 세 가지 상태에 대한 별도의 렌더링 로직(로딩중, 빈 목록, 목록 표시)

`Func<Task<IEnumerable<?>>>` 타입의 대리인을 받아들이면 비동기 로딩을 해결할 수 있습니다. (**?**자리에 들어갈 타입을 찾아야 합니다.) 어떤 데이터라도 지원해야 하므로 이 컴포넌트를 제네릭 컴포넌트로 선언합니다. `@typeparam` 지시문을 사용하면 제네릭 타입 컴포넌트를 만들 수 있으므로 `TemplatedList.razor` 상단에 추가해 주세요.

```html
@typeparam TItem
```

제네릭 타입의 컴포넌트를 만드는 것은 C#의 다른 제네릭 타입을 만드는 것과 비슷합니다. `@typeparam`는 사실 제네릭 .NET 타입을 위한 Razor 편의 구문입니다.

> Note: 아직 매개 변수 타입에 대해 지원하고 있지 않습니다. 앞으로 추가할 것입니다.

이제 제네릭 타입 매개 변수를 정의했으므로 매개 변수 선언에 사용할 수 있습니다. 데이터를 로드하는데 사용할 대리자를 매개 변수로 축하고 다음 컴포넌트로 유사한 방식으로 데이터를 로드하세요.

```html
@code {
    IEnumerable<TItem> items;

    [Parameter] public Func<Task<IEnumerable<TItem>>> Loader { get; set; }

    protected override async Task OnParametersSetAsync()
    {
        items = await Loader();
    }
}
```

데이터가 있으므로 각 상태에 대한 구조를 추가할 수 있씁니다. `TemplatedList.razor`에 다음 마크업을 추가합니다:

```html
@if (items == null)
{

}
else if (!items.Any())
{
}
else
{
    <div class="list-group">
        @foreach (var item in items)
        {
            <div class="list-group-item">

            </div>
        }
    </div>
}
```

대화 상자에는 세 가지 상태가 있습니다. 상태에 따라 필요한 컨텐츠 파를 받도록 합니다. 그럴려면 3개의 `RenderFragment` 매개 변수가 정의되어야 합니다. 여러 개의 매개 변수가 있으므로 `ChildContent`로 부르지 않고 고유한 이름으로 선언합니다. 항목을 표시하기 위한 매개변수는 `RenderFragment<T>`를 이용하면 됩니다.

3개의 매개변수를 추가한 예시입니다.

```C#
    [Parameter] public RenderFragment Loading{ get; set; }
    [Parameter] public RenderFragment Empty { get; set; }
    [Parameter] public RenderFragment<TItem> Item { get; set; }
```

`RenderFragment` 매개 변수를 가지고 각 위치에 맞게 마크업을 수정해 주세요. 아래와 같이 하면 됩니다.

```html
@if (items == null)
{
    @Loading
}
else if (!items.Any())
{
    @Empty
}
else
{
    <div class="list-group">
        @foreach (var item in items)
        {
            <div class="list-group-item">
                @Item(item)
            </div>
        }
    </div>
}
```

`Item`은 매개 변수를 받습니다. 이는 함수를 호출하는 방법으로 처리하며 파라미터를 받아들이며 `RenderFragment<T>`를 호출한 결과는 또 다른 `RenderFragment`를 렌더링하는 것입니다.

이 시점에서 새 컴포넌트는 컴파일 되지만 아직 한 가지를 더 해야 합니다. `MyOrders.razor`가 하고 있는 `<div class="list-group">`의 스타일링을 하게 하고 싶습니다. CSS 클래스를 추가하는 아 작은 확장성은 재사용성에 큰 도움이 됩니다.

또 다른 `string` 매개변수를 추가면 `TemplatedList.razor`의 코드 블럭은 아래처럼 보이게 될 것입니다.

```html
@code {
    IEnumerable<TItem> items;

    [Parameter] public Func<Task<IEnumerable<TItem>>> Loader { get; set; }
    [Parameter] public RenderFragment Loading { get; set; }
    [Parameter] public RenderFragment Empty { get; set; }
    [Parameter] public RenderFragment<TItem> Item { get; set; }
    [Parameter] public string ListGroupClass { get; set; }

    protected override async Task OnParametersSetAsync()
    {
        items = await Loader();
    }
}
```

마지막으로 `<div class="list-group @ListGroupClass">`에서 `<div class="list-group">`를 포함하도록 수정합니다. 이제 `TemplatedList.razor`는 아래와 같이 바뀝니다.

```html
@typeparam TItem

@if (items == null)
{
    @Loading
}
else if (!items.Any())
{
    @Empty
}
else
{
    <div class="list-group @ListGroupClass">
        @foreach (var item in items)
        {
            <div class="list-group-item">
                @Item(item)
            </div>
        }
    </div>
}

@code {
    IEnumerable<TItem> items;

    [Parameter] public Func<Task<IEnumerable<TItem>>> Loader { get; set; }
    [Parameter] public RenderFragment Loading { get; set; }
    [Parameter] public RenderFragment Empty { get; set; }
    [Parameter] public RenderFragment<TItem> Item { get; set; }
    [Parameter] public string ListGroupClass { get; set; }

    protected override async Task OnParametersSetAsync()
    {
        items = await Loader();
    }
}
```

## 템플릿 목록 사용하기

새로운 `TemplatedList` 컴포넌트를 사용하기 위해 `MyOrders.razor`를 수정합니다.

먼저 주문 데이터를 로드할 `TemplatedList`에 전달할 대리자를 만들어야 합니다. `MyOrders.OnParametersSetAsync`에 변경이 필요합니다. `@code` 코드 블럭은 다음과 같습니다.

```html
@code {
    async Task<IEnumerable<OrderWithStatus>> LoadOrders()
    {
        var ordersWithStatus = Enumerable.Empty<OrderWithStatus>();
        try
        {
            ordersWithStatus = await OrdersClient.GetOrders();
        }
        catch (AccessTokenNotAvailableException ex)
        {
            ex.Redirect();
        }
        return ordersWithStatus;
    }
}
```

`TemplatedList`의 `Loader` 매개변수가 기대하는 메서드 시그니처와 일치합니다. `OrderWithStatus`가 **?**로 바뀐 것이 `Func<Task<IEnumerable<?>>>`입니다. 잘 하고 있어요.

`TemplatedList`를 사용하면 아래와 같이 됩니다.

```html
<div class="main">
    <TemplatedList>
    </TemplatedList>
</div>
```

컴파일러는 `TemplatedList`의 제네릭 타입을 알 수 없다고 할 것입니다. 일반 C#처럼 타입 추론을 수행할 만큼 컴파일러는 충분히 똑똑하지만 우린 아직 충분한 정도를 전달하진 않았습니다.

이 이슈를 수정하기 위해 `Loader` 속성을 추가해 주세요.

```html
<div class="main">
    <TemplatedList Loader="LoadOrders">
    </TemplatedList>
</div>
```

> Note: 제네릭 타입 컴포넌트는 같은 이름 속성으로 타입 매개 변수를 지정할 수 있습니다. (이 경우 `TItem`으로 지정된 것입니다.) 유용한 경우가 있을 수 있으니 알아두세요.

```html
<div class="main">
    <TemplatedList TItem="OrderWithStatus">
    </TemplatedList>
</div>
```

`Loader`에서 타입을 추론할 수 있으니 지금은 이 작업이 필요 없습니다.

-----

다음으로, 여러 콘텐츠(`RenderFragment`) 매개변수를 컴포넌트에 전달하는 방법에 대해 생각해야 합니다. `[Parameter] RenderFragment ChildContent`를 설정하여 컴포넌트내의 컨텐츠를 중첩하여 `TemratedDialog`를 사용하수 있다는 것을 알고 있습니다. 하지만 가장 간단한 경우 편리한 구문일 뿐입니다. 여러 콘텐츠 매개변수를 전달하려는 경우 컴포넌트 내에 매개변수 이름과 일치하는 요소를 중첩하여 이를 처리할 수 있습니다.

`TemplatedList`에 각 매개변수를 더미 콘텐츠로 설정하는 예는 다음과 같습니다.

```html
<div class="main">
    <TemplatedList Loader="LoadOrders">
        <Loading>Hi there!</Loading>
        <Empty>
            How are you?
        </Empty>
        <Item>
            Are you enjoying Blazor?
        </Item>
    </TemplatedList>
</div>
```

`Item`은 `RenderFragment<T>`의 매개변수입니다. 기본적으로 이 매개변수는 `context`입니다. `<Item>  </Item>` 사이에 입력하면 `@context`가 `OrderStatus` 타입의 변수에 바인딩되어 있음을 볼 수 있습니다. `Context` 속성을 사용하여 매개변수 이름을 이름을 바꿀 수 있습니다.

```html
<div class="main">
    <TemplatedList Loader="LoadOrders">
        <Loading>Hi there!</Loading>
        <Empty>
            How are you?
        </Empty>
        <Item Context="item">
            Are you enjoying Blazor?
        </Item>
    </TemplatedList>
</div>
```

이제 `MyOrders.razor`의 기존 콘텐츠를 모두 포함하려고 하므로 이를 모두 합치면 다음과 같아야 합니다.

```html
<div class="main">
    <TemplatedList Loader="LoadOrders" ListGroupClass="orders-list">
        <Loading>Loading...</Loading>
        <Empty>
            <h2>No orders placed</h2>
            <a class="btn btn-success" href="">Order some pizza</a>
        </Empty>
        <Item Context="item">
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
        </Item>
    </TemplatedList>
</div>
```

기존 `MyOrders.razor`에 있었던 스타일을 추가하기 위해 `ListGroupClass` 매개변수도 설정하고 있습니다.

여기에 소개할 여러 단계와 새로운 기능이 있습니다. 실행해 보고 템플릿 목록을 사용하여 정상 동작하는지 확인하세요.

아래 작업을 통해 목록이 정상 동작하는지 확인할 수 있습니다.
1. 주문이 없는 경우를 테스트하기 위해 `Blazor.Server` 프로젝트에서 `pizza.db`를 삭제합니다.
2. 아직 로드중인 경우를 테스트하기 위해 `LoadOrders`(해당 메소드를 `async`로 표시)에 `await Task.Delay(3000);`를 추가합니다.

## 요약

이번 세션에서 알아본 것은 무엇인가요?

1. *content*를 매개변수를 받는 컴포넌트를 작성할 수 있습니다. 심지어 여러 콘텐츠 매개변수도 가능합니다.
2. 템플릿 컴포넌트는 대화 상자 표시 또는 데이터 비동기 로딩과 같은 작업을 추상화하는 데 사용할 수 있습니다.
3. 제네릭 타입을 사용하는 컴포넌트이므로 재사용이 더 쉽습니다.

다음 세션 - [프로그레시브 웹앱(PWA)](09-progressive-web-app.md)

원문 읽기 - [Templated components](https://github.com/dotnet-presentations/blazor-workshop/blob/main/docs/08-templated-components.md)