# 컴포넌트와 레이아웃

이 세션에서는 블레이저를 이용하여 피자 가게 앱을 만들 것입니다. 이 앱은 사용자가 피자를 주문하고 토핑을 변경할 수 있으며 주문 배달을 추적할 수 있습니다.

## 피자 가게 앱의 첫 걸음

이 레포지토리에 피자 가게 앱을 위한 초기 솔루션을 준비하였습니다.
이제 레포지토리를 복사하세요. "save-points" 폴더에 각 세션별로 준비가 되어 있습니다.
[starting point](https://github.com/dotnet-presentations/blazor-workshop/tree/master/save-points/00-get-started)를 찾아보세요.

> Note: 이 워크샵 코드를 시스템의 다른 위치에 복사하는 경우, 레포지토리 root에 있는 *Directory.Build.props* 파일도 함께 복사해야 합니다.

이 솔루션은 4개의 프로젝트를 이미 포함하고 있습니다.
![image](https://user-images.githubusercontent.com/1874516/77238114-e2072780-6b8a-11ea-8e44-de6d7910183e.png)


- **BlazingPizza.Client**: 블레이저 프로젝트 입니다. 앱 UI 컴포넌트가 포함되어 있습니다.

- **BlazingPizza.Server**: 블레이저 앱을 호스팅하는 ASP.NET Core 프로젝트이며 이 앱의 백엔드 서비스입니다.

- **BlazingPizza.Shared**: 이 앱에서 공유하는 코드가 포함된 프로젝트입니다.

- **BlazingPizza.ComponentsLibrary**: 이후 세션에서 사용될 컴포넌트와 헬퍼코드 라이브러리입니다.

**BlazingPizza.Server** 프로젝트가 시작 프로젝트로 설정되어야 합니다.

앱을 실행하면 현재는 단순한 홈페이지가 표시될 것입니다.

![image](https://user-images.githubusercontent.com/1874516/77238160-25fa2c80-6b8b-11ea-8145-e163a9f743fe.png)

이 홈 페이지의 코드를 보기 위해 **BlazingPizza.Client** 프로젝트에 있는 *Pages/Index.razor* 파일을 열어보세요.

```
@page "/"

<h1>Blazing Pizzas</h1>
```

이 홈페이지는 단일 컴포넌트로 구현되어 있습니다. `@page` 지시문은 `Index` 컴포넌트가 특정 경로의 라우팅 가능한 페이지임을 지정합니다.

## 피자 메뉴 목록 표시

첫 번째로 홈페이지에 주문 가능한 피자 메뉴를 표시하는 것부터 진행할 것입니다. 이 목록은 `Index` 컴포넌트 상태에 포함됩니다.

*Index.razor* 페이지에 메뉴 목록을 저장할 목록 필드를 저장할 코드를 `@code` 블럭으로 추가해 주세요.

```csharp
@code {
    List<PizzaSpecial> specials;
}
```

`@code` 블럭안의 코드는 컴포넌트를 위해 생성된 클래스에 추가됩니다. `PizzaSpecial` 타입은 **BlazingPizza.Shared**에 이미 정의되어 있습니다.

메뉴 목록을 얻을 려면 백엔드의 API를 호출해야 합니다. 블레이저는 기본값으로 이미 설정된 `HttpClient`를 의존성 주입을 통해 제공합니다. `@inject` 지시어를 사용하여 `Index` 컴포넌트에 `HttpClient`를 주입합니다.

```
@page "/"
@inject HttpClient HttpClient
```

`@inject` 지시문은 기본적으로 속성 유형과 속성 이름을 지정하는 것으로 컴포넌트에 새 속성을 정의합니다. 이 속성은 종속성 주입을 통해 제공 됩니다.

피자 메뉴 목록을 받기 위해 `@code` 블럭에 `OnInitializedAsync` 메서드를 재정의합니다. 이 메서드는 컴포넌트 생명주기의 일부로 컴포넌트가 초기화되고 나서 호출됩니다. 응답으로 온 JSON을 역직렬화하기 위해 `GetFromJsonAsync<T>()` 메서드를 이용합니다.

```csharp
@code {
    List<PizzaSpecial> specials;

    protected override async Task OnInitializedAsync()
    {
        specials = await HttpClient.GetFromJsonAsync<List<PizzaSpecial>>("specials", BlazingPizza.OrderContext.Default.ListPizzaSpecial);
    }
}
```

`/specials` API는 minimal API로 정의되어 있으며 **BlazingPizza.Server** 프로젝트의 `PizzaApiExtensions.cs`파일에서 확인하실 수 있습니다.

minimal API와 관련한 상세 내용은 [ASP.NET Core를 사용하여 최소 API 만들기](https://docs.microsoft.com/ko-kr/aspnet/core/tutorials/min-web-api?view=aspnetcore-7.0)를 참고하세요.

> Note: `BlazingPizza.OrderContext.Default.ListPizzaSpecial`는 [source generators](https://docs.microsoft.com/en-us/dotnet/standard/serialization/system-text-json-source-generation)로 JSON을 직렬화한 것입니다.

컴포넌트가 초기화되면 마크업이 렌더링됩니다. `Index` 컴포넌트의 마크업을 아래와 같이 피자 메뉴 목록을 표시하도록 수정해 주세요.

```html
<div class="main">
    <ul class="pizza-cards">
        @if (specials != null)
        {
            @foreach (var special in specials)
            {
                <li style="background-image: url('@special.ImageUrl')">
                    <div class="pizza-info">
                        <span class="title">@special.Name</span>
                        @special.Description
                        <span class="price">@special.GetFormattedBasePrice()</span>
                    </div>
                </li>
            }
        }
    </ul>
</div>
```

`Ctrl-F5`를 눌러 앱을 실행해 주세요. 이제 주문 가능한 메뉴 목록이 보일 것입니다.

![image](https://user-images.githubusercontent.com/1874516/77239386-6c558880-6b97-11ea-9a14-83933146ba68.png)


## 레이아웃 만들기

다음으로 앱의 레이아웃을 만듭니다.

블레이저의 레이아웃 역시 컴포넌트입니다. 레이아웃의 본문을 렌더링할 수 있는 `Body`를 정의하는 `LayoutComponentBase`로부터 상속됩니다. 우리가 사용할 피자 가게 앱의 레이아웃 컴포넌트는 *Shared/MainLayout.razor* 파일에 정의되어 있습니다.

```html
@inherits LayoutComponentBase

<div class="content">
    @Body
</div>
```

레이아웃이 페이지와 어떻게 연관되어 있는지 보려면 `App.razor` 파일의 `<Router>` 컴포넌트를 살펴보세요. `DefaultLayout` 매개 변수는 레이아웃을 지정하지 않은 페이지에 사용되는 레이아웃을 결정합니다.

페이지 단위로 `DefaultLayout`를 재정의할 수 있습니다. 이를 위해서는 `.razor` 페이지 컴포넌트 맨 위에 `@layout SomeOtherLayout` 같은 지시문을 추가할 수 있습니다.

`MainLayout` 컴포넌트를 수정하여 홈페이지의 탐색 링크와 브랜드 로고가 있는 상단 메뉴를 추가해 주세요.

```html
@inherits LayoutComponentBase

<div class="top-bar">
    <a class="logo" href="">
        <img src="img/logo.svg" />
    </a>

    <NavLink href="" class="nav-tab" Match="NavLinkMatch.All">
        <img src="img/pizza-slice.svg" />
        <div>Get Pizza</div>
    </NavLink>
</div>

<div class="content">
    @Body
</div>
```

블레이저는 `NavLink` 컴포넌트를 제공합니다. 컴포넌트는 컴포넌트의 타입 이름과 매개 변수를 속성으로 지정하여 사용할 수 있습니다.

`NavLink` 컴포넌트는 현재 URL이 링크의 주소와 일치할 때 `active` 클래스를 추가하는 것을 제외하면 앵커 태그와 동일합니다. `NavLinkMatch.All`은 전체 URL(URL Prefix만이 아닌)이 일치하였을 때만 링크가 활성화되어야 하는 것을 의미합니다. `NavLink`의 자세한 내용은 이후 세션에서 더 자세히 다루겠습니다.

`Ctrl-F5`를 눌러 앱을 실행해 주세요. 새로운 레이아웃이 적용되어 피자 가게 앱이 아래 이미지와 같이 보일 것입니다.

![image](https://user-images.githubusercontent.com/1874516/77239419-aa52ac80-6b97-11ea-84ae-f880db776f5c.png)


다음 세션 - [내 피자 만들기](02-customize-a-pizza.md)

원문 읽기 - [Components and Layout](https://github.com/dotnet-presentations/blazor-workshop/blob/main/docs/01-components-and-layout.md)