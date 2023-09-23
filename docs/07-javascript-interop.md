# 자바스크립트 상호운용성(JavaScript interop)

이제 피자 가게 사용자는 주문 상태를 실시간으로 추적할 수 있습니다. 이 세션에서는 자바스크립트 상호운용성을 사용하여 "내 피자는 어디 있지?!?"라는 오래된 질문에 답하는 주문 상태 페이지에 실시간 지도를 추가합니다.

## 지도 컴포넌트

ComponentsLibrary 프로젝트에는 마커 세트의 위치를 표시하고 시간 경과에 따른 움직임에 애니메이션을 위해 이미 작성된 `Map` 컴포넌트가 포함되어 있습니다. 이 컴포넌트를 사용하여 사용자의 피자 주문이 배달 되는 위치를 표시하지만 `Map` 컴포넌트가 어떻게 구현되는지 살펴보겠습니다.

*Map.razor* 파일을 열고 아래 코드를 살펴보세요.

```csharp
@using Microsoft.JSInterop
@inject IJSRuntime JSRuntime

<div id="@elementId" style="height: 100%; width: 100%;"></div>

@code {
    string elementId = $"map-{Guid.NewGuid().ToString("D")}";

    [Parameter] double Zoom { get; set; }
    [Parameter] List<Marker> Markers { get; set; }

    protected async override Task OnAfterRenderAsync(bool firstRender)
    {
        await JSRuntime.InvokeVoidAsync(
            "deliveryMap.showOrUpdate",
            elementId,
            Markers);
    }
}
```

`Map` 구성 요소는 종속성 주입을 사용하여 `IJSRuntime` 인스턴스를 가져옵니다. 이 서비스를 사용하면 `InvokeVoidAsync` 또는 `InvokeAsync<TResult>` 메서드를 호출하여 브라우저 API 또는 기존 JavaScript 라이브러리에 대한 JavaScript 호출을 수행할 수 있습니다. 이 메소드의 첫 번째 매개변수는 루트 `window` 객체를 기준으로 호출할 JavaScript 함수의 경로를 지정합니다. 나머지 매개변수는 JavaScript 함수에 전달할 인수입니다. 인수는 JavaScript로 처리될 수 있도록 JSON으로 직렬화됩니다.

`Map` 컴포넌트는 먼저 지도의 고유 ID로 `div`를 렌더링한 다음 `deliveryMap.showOrUpdate` 함수를 호출하여 지정된 마커가 `Map` 컴포넌트에 전달해 지도에 표시합니다. 이 작업은 `OnAfterRenderAsync` 함수에서 완료되어 컴포넌트 수명 주기 이벤트에서 수행이 완료됩니다. `deliveryMap.showOrUpdate` 함수는 *wwwroot/deliveryMap.js* 파일에 정의되어 있으며 [leaflet.js](http://leafletjs.com)과 [OpenStreetMap](https://www.openstreetmap.org/)을 이용하여 지도에 표시합니다. 이 코드의 작동 방식에 대한 세부 사항은 실제로 중요하지 않습니다. 중요한 점은 모든 JavaScript 함수를 이런 방식으로 호출할 수 있다는 것입니다.

이런 파일은 어떻게 블레이저 앱에 추가하나요? 블레이저 라이브러리 프로젝트(`Sdk="Microsoft.NET.Sdk.Razor"`)를 사용하면 `wwwroot/` 폴더에 있는 모든 파일은 라이브러리와 함께 번들로 제공됩니다. 서버 프로젝트는 정적 파일 미들웨어를 사용하여 이러한 파일을 자동으로 제공합니다.

블레이저 클라이언트 앱을 호스팅하는 페이지에 원하는 파일(이 경우 `.js` 및 `.css`)을 포함시키는 링크가 마지막에 있습니다. `index.html`에는 `_content/BlazingPizza.ComponentsLibrary/localStorage.js`와 같은 상대 URI를 사용하여 이러한 파일이 포함됩니다. 이는 블레이저 클래스 라이브러리(`_content/<라이브러리 이름>/<파일 경로>`)와 함께 번들로 제공되는 참조 파일의 일반적인 패턴입니다.

---

`Map`에 입력을 시작하면 편집기에서 완성 기능을 제공하지 않는다는 것을 알 수 있습니다. 이는 요소와 컴포넌트 간의 바인딩이 C#의 네임스페이스 바인딩 규칙에 의해 제어되기 때문입니다. `Map` 구성 요소는 `BlazingPizza.ComponentsLibrary.Map` 네임스페이스에 정의되어 있지만 이에 대한 `@using`이 없습니다.

이 네임스페이스에 대한 `@using`을 루트 `_Imports.razor`에 추가하여 이 컴포넌트 범위로 가져옵니다.

```razor
@using System.Net.Http
@using Microsoft.AspNetCore.Authorization
@using Microsoft.AspNetCore.Components.Authorization
@using Microsoft.AspNetCore.Components.Forms
@using Microsoft.AspNetCore.Components.Routing
@using Microsoft.JSInterop
@using BlazingPizza.Client
@using BlazingPizza.Client.Shared
@using BlazingPizza.ComponentsLibrary
@using BlazingPizza.ComponentsLibrary.Map
```

`track-order-details` `div` 바로 아래에 다음을 추가하여 `OrderDetails` 페이지에 `Map` 구성 요소를 추가합니다.

```html
<div class="track-order-map">
    <Map Zoom="13" Markers="orderWithStatus.MapMarkers" />
</div>
```

*이전에는 컴포넌트에 `@using`을 추가할 필요가 없었던 이유는 루트 `_Imports.razor`에 우리가 작성한 재사용 가능한 컴포넌트와 일치하는 `@using BlazingPizza.Shared`가 이미 포함되어 있기 때문입니다.*

`OrderDetails` 컴포넌트가 주문 상태 업데이트를 위해 폴링할 때 지도에 피자의 최신 위치가 반영된 마커가 반영됩니다.

![Real-time pizza map](https://user-images.githubusercontent.com/1874516/51807322-6018b880-227d-11e9-89e5-ef75f03466b9.gif)

## 피자 삭제 확인 메시지 추가

`Map` 컴포넌트에 대한 자바스크립트 상호운용성 코드는 제공되었습니다. 다음으로는 자바스크립트 상호운용성 코드를 추가해 보겠습니다.

사용자가 실수로 주문에서 피자를 삭제하고 결국 피자를 구매하지 않게 된다면 안타까운 일이 될 것입니다. 사용자가 피자를 삭제하려고 할 때 확인 메시지를 추가해 보겠습니다. 자바스크립트 상호운용성을 사용하여 확인 프롬프트를 표시하겠습니다.

`IJSRuntime`의 `Confirm` 확장 메서드를 사용하여 클라이언트 프로젝트에 정적 `JSRuntimeExtensions` 클래스를 추가합니다. 내장된 JavaScript `confirm` 함수를 호출하려면 `Confirm` 메소드를 구현하십시오.

```csharp
public static class JSRuntimeExtensions
{
    public static ValueTask<bool> Confirm(this IJSRuntime jsRuntime, string message)
    {
        return jsRuntime.InvokeAsync<bool>("confirm", message);
    }
}
```

자바스크립트 상호운용성 호출을 수행하는 데 사용할 수 있도록 `Index` 구성 요소에 `IJSRuntime` 서비스를 삽입합니다.

```razor
@page "/"
@inject HttpClient HttpClient
@inject OrderState OrderState
@inject NavigationManager NavigationManager
@inject IJSRuntime JS
```

사용자가 실제로 주문에서 피자를 제거하기를 원하는지 확인하기 위해 `Confirm` 메서드를 호출하는 `Index` 구성 요소에 비동기 `RemovePizza` 메서드를 추가합니다.

```csharp
async Task RemovePizza(Pizza configuredPizza)
{
    if (await JS.Confirm($"Remove {configuredPizza.Special.Name} pizza from the order?"))
    {
        OrderState.RemoveConfiguredPizza(configuredPizza);
    }
}
```

`Index` 컴포넌트에서 `ConfiguredPizzaItems`에 대한 이벤트 핸들러를 수정하여 새로운 `RemovePizza` 메서드를 호출합니다.

```csharp
@foreach (var configuredPizza in OrderState.Order.Pizzas)
{
    <ConfiguredPizzaItem Pizza="configuredPizza" OnRemoved="@(() => RemovePizza(configuredPizza))" />
}
```

앱을 실행하고 주문에서 피자를 삭제해 보세요.

![Confirm pizza removal](https://user-images.githubusercontent.com/1874516/77243688-34b40400-6bca-11ea-9d1c-331fecc8e307.png)

`ConfiguredPizzaItem.OnRemoved`를 비동기로 실행하기 위해 메서드 시그니처를 수정해야 할 필요가 없습니다. `EventCallback`의 다른 특수 속성으로 동기, 비동기 이벤트 핸들러를 모두 지원합니다.

다음 세션 - [템플릿 컴포넌트](08-templated-components.md)

원문 읽기 - [Javascript interop](https://github.com/dotnet-presentations/blazor-workshop/blob/main/docs/07-javascript-interop.md)