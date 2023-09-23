# 인증

애플리케이션은 잘 동작합니다. 사용자는 주문을 하고 주문 상태를 추적할 수 있습니다. 하지만 한 가지 작은 문제가 있습니다. 현재 사용자를 전혀 구분하지 않고 있습니다. "내 주문" 페이지에는 *모든* 사용자가 주문한 *모든* 주문이 나열되어 있으며, 누구나 다른 사용자의 주문 상태를 볼 수 있습니다. 사용자의 고객과 개인 정보 보호 규정에 문제가 있을 수 있습니다.

해결책은 *인증*입니다. 사용자가 로그인하기 시작하면 누가 누구인지 알 수 있습니다. 그런 다음 누가 무엇을 할 수 있는지에 대한 규칙을 시행하는 *권한 부여*를 구현할 수 있습니다.

## 서버에서 실행

무엇보다 가장 중요한 원칙은 모든 *실제* 보안 규칙이 백엔드 서버에 적용되어야 한다는 것입니다. 클라이언트(UI)는 선량한 사용자에 대한 배려로 옵션을 표시하거나 숨길 뿐이지만, 악의적인 사용자는 언제든지 클라이언트 측 코드의 동작을 변경할 수 있습니다.

따라서 클라이언트 코드에 무엇인가 하기 전에 백엔드 서버에 일부 액세스 규칙을 적용하는 것부터 시작하겠습니다.

`BlazingPizza.Server` 프로젝트 내에서 `OrdersController.cs` 파일이 있습니다. 이 컨트롤러는 `/orders` 및 `/orders/{orderId}`에 대한 HTTP 요청을 처리하는 컨트롤러 클래스입니다. 이러한 엔드포인트에 대한 모든 요청이 인증된 사용자(로그인한 사람)로부터 오도록 요구하려면 `OrdersController` 클래스에 `[Authorize]` 속성을 추가하세요.

```csharp
[Route("orders")]
[ApiController]
[Authorize]
public class OrdersController : Controller
{
}
```

`AuthorizeAttribute` 클래스는 `Microsoft.AspNetCore.Authorization` 네임스페이스에 있습니다.

지금 애플리케이션을 실행해서 확인해 보면 더 이상 주문을 할 수 없으며 이미 주문한 세부 정보를 검색할 수도 없다는 것을 알게 될 것입니다. 이러한 엔드포인트에 대한 요청은 HTTP 401 "승인되지 않음" 응답을 반환하고 UI에 오류 메시지를 트리거합니다. 좋습니다. 서버에서 규칙이 시행되고 있음을 보여주기 때문입니다!

![Secure orders](https://user-images.githubusercontent.com/1101362/83876158-49ffef80-a730-11ea-8c86-f1fb2b51755b.png)

## 인증 상태 추적하기

클라이언트 코드에는 사용자가 로그인했는지 여부와 로그인한 경우 *어떤* 사용자가 로그인했는지 확인하여 UI 동작에 영향을 줄 수 있는 방법이 필요합니다. 블레이저에는 이 작업을 수행하기 위한 내장 DI 서비스인 `AuthenticationStateProvider`가 있습니다. 블레이저 사용자가 누구인지 확인하는 모든 세부 정보를 처리하는 [OpenID Connect](https://openid.net/connect/)를 기반으로 `AuthenticationStateProvider` 서비스와 기타 관련 서비스 및 컴포넌트의 구현을 제공합니다. 이러한 서비스 및 컴포넌트는 클라이언트 프로젝트에 이미 추가된 Microsoft.AspNetCore.Components.WebAssembly.Authentication 패키지에서 제공됩니다.

넓은 의미에서 이러한 서비스가 구현하는 인증 프로세스는 다음과 같습니다.

* 사용자가 로그인을 시도하거나 보호된 리소스에 액세스하려고 하면 사용자는 앱의 로그인 페이지(`/authentication/login`)로 리디렉션됩니다.
* 로그인 페이지에서 앱은 구성된 ID 공급자의 인증 엔드포인트으로 리디렉션할 준비를 합니다. 엔드포인트는 사용자가 인증되었는지 여부를 결정하고 이에 대한 응답으로 하나 이상의 토큰을 발행하는 일을 담당합니다. 앱은 인증 응답을 수신하기 위해 로그인 콜백을 제공합니다.
  * 사용자가 인증되지 않으면 사용자는 먼저 기본 인증 시스템(일반적으로 ASP.NET Core ID)으로 리디렉션됩니다.
  * 사용자가 인증되면 승인 엔드포인트는 적절한 토큰을 생성하고 브라우저를 로그인 콜백 엔드포인트(`/authentication/login-callback`)로 다시 리디렉션합니다.
* 블레이저 웹어셈플리 앱이 로그인 콜백 엔드포인트(`/authentication/login-callback`)를 로드하면 인증 응답이 처리됩니다.
  * 인증 프로세스가 성공적으로 완료되면 사용자가 인증되고 선택적으로 사용자가 요청한 원래 보호된 URL로 다시 전송됩니다.
  * 어떠한 이유로든 인증 과정에 실패할 경우 로그인 실패 페이지(`/authentication/login-failed`)로 이동되며 오류가 표시됩니다.

자세한 내용은 [ASP.NET Core Blazor WebAssembly 보호](https://docs.microsoft.com/aspnet/core/security/blazor/webassembly/)를 참조하세요.

인증 서비스를 활성화하려면 클라이언트 프로젝트의 *Program.cs*에 `AddApiAuthorization`에 대한 호출을 추가하세요.

```csharp
using BlazingPizza.Client;
using Microsoft.AspNetCore.Components.Web;
using Microsoft.AspNetCore.Components.WebAssembly.Hosting;

var builder = WebAssemblyHostBuilder.CreateDefault(args);
builder.RootComponents.Add<App>("#app");
builder.RootComponents.Add<HeadOutlet>("head::after");

builder.Services.AddScoped(sp => new HttpClient { BaseAddress = new Uri(builder.HostEnvironment.BaseAddress) });
builder.Services.AddScoped<OrderState>();

// Add auth services
builder.Services.AddApiAuthorization();

await builder.Build().RunAsync();
```

추가된 서비스는 기본적으로 앱과 동일한 출처에서 ID 공급자를 사용하도록 구성됩니다. 블레이저 피자 가게 앱의 서버 프로젝트는 이미 [IdentityServer](https://identityserver.io/)를 ID 공급자로 사용하고 ASP.NET Core ID를 인증 시스템으로 사용하도록 설정되어 있습니다.

*BlazingPizza.Server/Program.cs*

```csharp
using BlazingPizza.Server;
using Microsoft.AspNetCore.Authentication;
using Microsoft.EntityFrameworkCore;

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllersWithViews()
    .AddJsonOptions(options => {
        options.JsonSerializerOptions.AddContext<BlazingPizza.OrderContext>();
    });
builder.Services.AddRazorPages();

builder.Services.AddDbContext<PizzaStoreContext>(options =>
        options.UseSqlite("Data Source=pizza.db")
            .UseModel(BlazingPizza.Server.Models.PizzaStoreContextModel.Instance));

builder.Services.AddDefaultIdentity<PizzaStoreUser>(options => options.SignIn.RequireConfirmedAccount = true)
        .AddEntityFrameworkStores<PizzaStoreContext>();

builder.Services.AddIdentityServer()
        .AddApiAuthorization<PizzaStoreUser, PizzaStoreContext>();

builder.Services.AddAuthentication()
        .AddIdentityServerJwt();

var app = builder.Build();

// 아래 더 많은 코드가 있습니다.
```

서버는 클라이언트 앱에 토큰을 발행하도록 이미 구성되어 있습니다.

*BlazingPizza.Server/appsettings.json*

```json
"IdentityServer": {
  "Clients": {
    "BlazingPizza.Client": {
      "Profile": "IdentityServerSPA"
    }
  }
}
```

인증 흐름을 조정하려면 클라이언트 프로젝트의 *Pages* 디렉터리에 `Authentication` 컴포넌트를 추가해 주세요.

*BlazingPizza.Client/Pages/Authentication.razor*

```razor
@page "/authentication/{action}"

<RemoteAuthenticatorView Action="@Action" />

@code{
    [Parameter]
    public string Action { get; set; }
}
```

`Authentication` 컴포넌트는 내장된 `RemoteAuthenticatorView` 컴포넌트를 사용하여 다양한 인증 작업을 처리하도록 설정됩니다. `Action` 매개변수는 `{action}` 경로 값에 바인딩된 다음 이를 처리하기 위해 `RemoteAuthenticatorView` 컴포넌트에 전달됩니다. `RemoteAuthenticatorView`는 원격 인증의 일부로 사용되는 모든 작업을 처리합니다. 유효한 작업에는 등록, 로그인, 프로필 및 로그아웃이 포함됩니다. 자세한 내용은 [앱 경로 사용자 지정](https://learn.microsoft.com/ko-kr/aspnet/core/blazor/security/webassembly/additional-scenarios?view=aspnetcore-7.0#customize-app-routes)을 참조하세요.

앱을 통해 인증 상태 정보를 전달하려면 컴포넌트를 하나 더 추가해야 합니다. `App.razor`에서 `<Router>` 전체를 `<CascadingAuthenticationState>`로 묶습니다.

```html
<CascadingAuthenticationState>
    <Router AppAssembly="typeof(Program).Assembly" Context="routeData">
        ...
    </Router>
</CascadingAuthenticationState>
```

처음에는 아무 작업도 수행하지 않는 것처럼 보이지만 실제로는 모든 하위 컴포넌트에서 *계단식 매개변수*를 사용할 수 있게 되었습니다. 계단식 매개 변수는 계층 구조의 한 수준으로만 전달되는 것이 아니라 여러 수준을 통해 전달되는 매개 변수입니다.

이제 UI에 무언가를 표시할 준비가 되었습니다!

## 로그인 상태 표시하기

클라이언트 프로젝트의 `Shared` 폴더에 `LoginDisplay`라는 새 컴포넌트를 만들고 아래 내용을 추가해 주세요.

```html
@inject NavigationManager Navigation
@inject SignOutSessionStateManager SignOutManager

<div class="user-info">
    <AuthorizeView>
        <Authorizing>
            <text>...</text>
        </Authorizing>
        <Authorized>
            <img src="img/user.svg" />
            <div>
                <a href="authentication/profile" class="username">@context.User.Identity.Name</a>
                <button class="btn btn-link sign-out" @onclick="BeginSignOut">Sign out</button>
            </div>
        </Authorized>
        <NotAuthorized>
            <a class="sign-in" href="authentication/register">Register</a>
            <a class="sign-in" href="authentication/login">Log in</a>
        </NotAuthorized>
    </AuthorizeView>
</div>

@code{
    async Task BeginSignOut()
    {
        await SignOutManager.SetSignOutState();
        Navigation.NavigateTo("authentication/logout");
    }
}
```

`AuthorizeView`는 사용자가 지정된 인증 조건을 충족하는지 여부에 따라 다른 콘텐츠를 표시하는 내장 구성 요소입니다. 인증 조건을 지정하지 않았으므로 기본적으로 사용자가 인증(로그인)되면 인증된 것으로 간주하고, 그렇지 않으면 인증되지 않은 것으로 간주합니다.

사용자 역할에 따라 메뉴 항목의 가시성을 제어하는 등 인증 상태에 따라 UI 콘텐츠를 변경해야 하는 경우 어디에서나 `AuthorizeView`를 사용할 수 있습니다. 이 경우 이를 사용하여 사용자에게 자신이 누구인지 알려주고 조건에 따라 "로그인" 또는 "로그아웃" 링크를 표시합니다.

등록, 로그인, 사용자 프로필 보기 링크는 `인증` 컴포넌트로 이동하는 일반 링크입니다. 로그아웃 링크는 버튼이며 위조된 요청으로 인해 사용자가 로그아웃되는 것을 방지하는 추가 논리가 있습니다. 버튼을 사용하면 로그아웃이 사용자 작업에 의해서만 트리거될 수 있으며 `SignOutSessionStateManager` 서비스는 로그아웃 흐름 전체에서 상태를 유지하여 전체 흐름이 사용자 작업으로 시작되었는지 확인합니다.

UI 어딘가에 'LoginDisplay'를 넣어 보겠습니다. `MainLayout`을 열고 다음과 같이 `<div class="top-bar">`를 수정합니다.

```html
<div class="top-bar">
    (... leave existing content in place ...)

    <LoginDisplay />
</div>
```

## 사용자 등록 및 로그인

확인해 보세요. 앱을 실행하고 새로운 사용자를 등록합니다.

홈페이지에서 등록을 선택하세요.

![Select register](https://user-images.githubusercontent.com/1874516/78322144-b25d0580-7522-11ea-863d-59083c2bf111.png)

신규 사용자의 이메일 주소와 비밀번호를 입력하세요.

![Register a new user](https://user-images.githubusercontent.com/1874516/78322197-e6d0c180-7522-11ea-8728-2bd9cbd3c8f8.png)

사용자 등록을 완료하려면 사용자는 이메일 주소를 확인해야 합니다. 개발 중에는 링크를 바로 클릭하여 계정을 확인할 수 있습니다.

![Email confirmation](https://user-images.githubusercontent.com/1874516/78389880-62208a80-7598-11ea-945a-d2ced76133d9.png)

사용자의 이메일이 확인되면 로그인을 선택하고 사용자의 이메일 주소와 비밀번호를 입력하세요.

![Select login](https://user-images.githubusercontent.com/1874516/78389922-7bc1d200-7598-11ea-8a10-e8bf8efa512e.png)

![Login](https://user-images.githubusercontent.com/1874516/78390092-cc392f80-7598-11ea-9d8e-562c2be1aad6.png)

사용자가 로그인되어 홈 페이지로 다시 리디렉션됩니다.

![Logged in](https://user-images.githubusercontent.com/1874516/78390115-d9561e80-7598-11ea-912b-e9dd71f787f2.png)

## 액세스 토큰 요청

아직 로그인했더라도 주문을 하기 위한 HTTP 요청에 유효한 액세스 토큰이 필요하기 때문에 여전히 주문에 실패합니다. 액세스 토큰을 요청하고 이를 아웃바운드 요청에 첨부하려면 요청을 만드는 데 사용하는 `HttpClient`와 함께 `BaseAddressAuthorizationMessageHandler`를 사용하세요. 이 메시지 핸들러는 내장된 `IAccessTokenProvider` 서비스를 사용하여 액세스 토큰을 획득하고 표준 Authorization 헤더를 사용하여 각 요청에 첨부합니다. 액세스 토큰을 획득할 수 없는 경우 `AccessTokenNotAvailableException`이 발생하며, 이를 통해 사용자를 로그인 페이지로 리디렉션하여 새 토큰을 승인할 수 있습니다.

앱의 `HttpClient`에 `BaseAddressAuthorizationMessageHandler`를 추가하기 위해 ASP.NET Core의 [ASP.NET Core에서 IHttpClientFactory를 사용하여 HTTP 요청 만들기](https://docs.microsoft.com/aspnet/core/fundamentals/http-requests ) 강력한 유형의 클라이언트를 사용합니다.

강력한 형식의 클라이언트를 만들려면 클라이언트 프로젝트에 새 `OrdersClient` 클래스를 추가하세요. 클래스는 생성자에서 `HttpClient`를 가져와야 하며 주문을 받고 발주하는 메서드를 제공해야 합니다.

*BlazingPizza.Client/OrdersClient.cs*

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Net.Http;
using System.Net.Http.Json;
using System.Threading.Tasks;

namespace BlazingPizza.Client
{
    public class OrdersClient
    {
        private readonly HttpClient httpClient;

        public OrdersClient(HttpClient httpClient)
        {
            this.httpClient = httpClient;
        }

        public async Task<IEnumerable<OrderWithStatus>> GetOrders() =>
            await httpClient.GetFromJsonAsync<IEnumerable<OrderWithStatus>>("orders");


        public async Task<OrderWithStatus> GetOrder(int orderId) =>
            await httpClient.GetFromJsonAsync<OrderWithStatus>($"orders/{orderId}");


        public async Task<int> PlaceOrder(Order order)
        {
            var response = await httpClient.PostAsJsonAsync("orders", order);
            response.EnsureSuccessStatusCode();
            var orderId = await response.Content.ReadFromJsonAsync<int>();
            return orderId;
        }
    }
}
```

올바른 기본 주소와 `BaseAddressAuthorizationMessageHandler`로 구성된 기본 `HttpClient`를 사용하여 `OrdersClient`를 형식화된 클라이언트로 등록합니다.

```csharp
builder.Services.AddHttpClient<OrdersClient>(client => client.BaseAddress = new Uri(builder.HostEnvironment.BaseAddress))
    .AddHttpMessageHandler<BaseAddressAuthorizationMessageHandler>();
```

### 선택 사항: .NET 6 JSON CodeGeneration과 JSON 상호 작용 최적화

.NET 6부터 System.Text.Json.JsonSerializer는 JSON 페이로드 직렬화 및 역직렬화를 위해 생성된 최적화된 코드 작업을 지원합니다. 코드는 빌드시 생성되므로 JSON 데이터의 직렬화 및 역직렬화 성능이 크게 향상됩니다. 이는 다음 단계를 수행하여 구성됩니다.

1. `System.Text.Json.Serialization.JsonSerializerContext`에서 상속되는 부분 컨텍스트 클래스를 만듭니다.
2. `System.Text.Json.JsonSourceGenerationOptions` 속성으로 클래스를 장식합니다.
3. 코드를 생성하려는 각 유형의 클래스 정의에 `JsonSerialized` 속성을 추가합니다.

`BlazingPizza.Shared.Order.cs` 파일에 이미 작성되어 있으므로 확인해 보세요.

```csharp
[JsonSourceGenerationOptions(GenerationMode = JsonSourceGenerationMode.Default, PropertyNamingPolicy = JsonKnownNamingPolicy.CamelCase)]
[JsonSerializable(typeof(Order))]
[JsonSerializable(typeof(OrderWithStatus))]
[JsonSerializable(typeof(List<OrderWithStatus>))]
[JsonSerializable(typeof(Pizza))]
[JsonSerializable(typeof(List<PizzaSpecial>))]
[JsonSerializable(typeof(List<Topping>))]
[JsonSerializable(typeof(Topping))]
public partial class OrderContext : JsonSerializerContext {}
```

이제 두 번째 매개변수로 찾는 유형을 가리키는 `OrderContext.Default` 매개변수를 전달하여 `OrdersClient` 클래스에서 HttpClient에 대한 호출을 최적화할 수 있습니다. `OrdersClient` 클래스의 메서드를 아래처럼 수정하세요.

```csharp
public async Task<IEnumerable<OrderWithStatus>> GetOrders() =>
  await httpClient.GetFromJsonAsync("orders", OrderContext.Default.ListOrderWithStatus);

public async Task<OrderWithStatus> GetOrder(int orderId) =>
  await httpClient.GetFromJsonAsync($"orders/{orderId}", OrderContext.Default.OrderWithStatus);

public async Task<int> PlaceOrder(Order order)
{
  var response = await httpClient.PostAsJsonAsync("orders", order, OrderContext.Default.Order);
  response.EnsureSuccessStatusCode();
  var orderId = await response.Content.ReadFromJsonAsync<int>();
  return orderId;
}
```

### 페이지에 OrdersClient 배포

새로운 유형의 `OrdersClient`를 사용하도록 주문을 관리하는 데 `HttpClient`가 사용되는 각 페이지를 수정해 주세요. `HttpClient` 대신 `OrdersClient`를 삽입하고 새 클라이언트를 사용하여 API 호출합니다. 제공된 `Redirect()` 메서드를 호출하여 `AccessTokenNotAvailableException` 유형의 예외를 처리하는 `try-catch`로 각 호출을 래핑합니다.

*Checkout.razor*

```csharp
async Task PlaceOrder()
{
    isSubmitting = true;

    try
    {
        var newOrderId = await OrdersClient.PlaceOrder(OrderState.Order);
        OrderState.ResetOrder();
        NavigationManager.NavigateTo($"myorders/{newOrderId}");
    }
    catch (AccessTokenNotAvailableException ex)
    {
        ex.Redirect();
    }
}
```

*MyOrders.razor*

```csharp
protected override async Task OnParametersSetAsync()
{
    try
    {
        ordersWithStatus = await OrdersClient.GetOrders();
    }
    catch (AccessTokenNotAvailableException ex)
    {
        ex.Redirect();
    }
}
```

*OrderDetails.razor*

```csharp
private async void PollForUpdates()
{
    invalidOrder = false;
    pollingCancellationToken = new CancellationTokenSource();
    while (!pollingCancellationToken.IsCancellationRequested)
    {
        try
        {
            orderWithStatus = await OrdersClient.GetOrder(OrderId);
            StateHasChanged();
            await Task.Delay(4000);
        }
        catch (AccessTokenNotAvailableException ex)
        {
            pollingCancellationToken.Cancel();
            ex.Redirect();
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
```

## 특정 주문 세부정보에 대한 액세스 승인

서버는 주문 정보에 대한 쿼리를 수락하기 전에 인증을 요구하지만 여전히 사용자를 구분하지 않습니다. 로그인한 모든 사용자는 로그인한 다른 모든 사용자의 주문을 볼 수 있습니다. 인증은 있지만 승인이 없습니다!

이를 확인하려면 하나의 계정으로 로그인한 상태에서 주문하세요. 그런 다음 로그아웃했다가 다른 계정을 사용하여 다시 로그인하세요. 동일한 주문 세부정보를 계속 볼 수 있습니다.

이것은 쉽게 고칠수 있습니다. `OrdersController` 코드로 돌아가서 `PlaceOrder`에서 주석 처리된 줄을 찾아 주석 처리를 제거합니다.

```cs
order.UserId = GetUserId();
```

이제 각 주문에는 해당 주문을 소유한 사용자의 ID가 찍혀 있습니다.

다음으로 `GetOrders` 및 `GetOrderWithStatus`에서 주석 처리된 `.Where` 행을 찾아 둘 다 주석 처리를 제거합니다. 이 줄은 사용자가 자신의 주문에 대한 세부 정보만 검색할 수 있도록 합니다.

```csharp
.Where(o => o.UserId == GetUserId())
```

이제 앱을 다시 실행하면 기존 주문 세부정보가 사용자 ID와 연결되어 있지 않기 때문에 더 이상 볼 수 없습니다. 하나의 계정으로 새로 주문하면 다른 계정에서 해당 주문을 볼 수 없습니다. 이는 어플리케이션을 훨씬 더 유용하게 만듭니다.

## 특정 페이지에 로그인 강제 적용

이제 로그인하시면 주문을 하고 주문상태를 보실 수 있습니다. 하지만 로그인하지 않고 주문을 시도하면 흐름이 이상적이지 않습니다. 체크아웃 양식을 *제출*할 때까지 로그인하라는 메시지가 표시되지 않습니다(이때 서버가 401 Not Authorized로 응답하기 때문입니다). 서버로부터 401 Not Authorized 응답을 받기 전에도 특정 페이지에 인증이 필요하도록 하려면 어떻게 해야 할까요?

이것도 아주 쉽게 할 수 있습니다. 서버 측 코드에서 `[Authorize]` 속성을 사용하는 것과 동일한 방식으로 클라이언트측 블레이저 페이지에서 해당 속성을 사용할 수 있습니다. 양식을 제출할 때뿐만 아니라 도착하자마자 로그인해야 하도록 결제 페이지를 수정해 보겠습니다.

기본적으로 모든 페이지는 익명 액세스를 허용하지만 `Checkout.razor` 상단에 `[Authorize]` 속성을 추가하여 체크아웃 페이지에 액세스하려면 사용자가 로그인해야 함을 지정할 수 있습니다.

```razor
@attribute [Authorize]
```

다음으로, 라우터가 이러한 속성을 존중하도록 하려면 경로가 발견되면 `RouteView` 대신 `AuthorizeRouteView`를 렌더링하도록 *App.razor*를 수정하세요.

```razor
<CascadingAuthenticationState>
    <Router AppAssembly="typeof(Program).Assembly" Context="routeData">
        <Found>
            <AuthorizeRouteView RouteData="routeData" DefaultLayout="typeof(MainLayout)">
                <NotAuthorized>
                    <p>You are not authorized to access this resource.</p>
                </NotAuthorized>
                <Authorizing>
                    <div class="main">Please wait...</div>
                </Authorizing>
            </AuthorizeRouteView>
        </Found>
        <NotFound>
            <LayoutView Layout="typeof(MainLayout)">
                <div class="main">Sorry, there's nothing at this address.</div>
            </LayoutView>
        </NotFound>
    </Router>
</CascadingAuthenticationState>
```

`AuthorizeRouteView`는 탐색을 올바른 컴포넌트로 라우팅하지만 사용자가 인증된 경우에만 가능합니다. 사용자가 승인되지 않은 경우 `NotAuthorized` 내용이 표시됩니다. `AuthorizeRouteView`가 사용자에게 권한이 있는지 확인하는 동안 표시할 콘텐츠를 지정할 수도 있습니다.

이제 로그아웃한 상태에서 결제 페이지로 이동하려고 하면 *App.razor*에 설정한 'UnAuthorized' 콘텐츠가 표시됩니다.

![Not authorized](https://user-images.githubusercontent.com/1874516/78410504-63b27880-75c1-11ea-8c2c-ab62c1c24596.png)

사용자에게 권한이 없다고 알리는 것보다는 로그인 페이지로 리디렉션하는 것이 더 좋습니다. 그렇게 하려면 다음 `RedirectToLogin` 컴포넌트를 추가하세요.

*BlazingPizza.Client/Shared/RedirectToLogin.razor*

```razor
@inject NavigationManager Navigation
@code {
    protected override void OnInitialized()
    {
        Navigation.NavigateTo($"authentication/login?returnUrl={Navigation.Uri}");
    }
}
```

그런 다음 *App.razor*의 `NotAuthorized` 콘텐츠를 `RedirectToLogin` 컴포넌트로 바꾸어 주세요.

```razor
<CascadingAuthenticationState>
    <Router AppAssembly="typeof(Program).Assembly" Context="routeData">
        <Found>
            <AuthorizeRouteView RouteData="routeData" DefaultLayout="typeof(MainLayout)">
                <NotAuthorized>
                    <RedirectToLogin />
                </NotAuthorized>
                <Authorizing>
                    <div class="main">Please wait...</div>
                </Authorizing>
            </AuthorizeRouteView>
        </Found>
        <NotFound>
            <LayoutView Layout="typeof(MainLayout)">
                <div class="main">Sorry, there's nothing at this address.</div>
            </LayoutView>
        </NotFound>
    </Router>
</CascadingAuthenticationState>
```

이제 로그아웃한 상태에서 결제 페이지에 액세스하려고 하면 로그인 페이지로 리디렉션됩니다. 그리고 사용자가 로그인하면 `returnUrl` 매개변수 덕분에 액세스하려고 했던 페이지로 다시 리디렉션됩니다.

## 인증 상태에 따라 탐색 옵션 숨기기

사용자가 로그인하지 않은 상태에서도 내 주문 탭을 볼 수 있다는 점은 다소 아쉽습니다. `AuthorizeView` 컴포넌트를 사용하여 인증되지 않은 사용자를 위해 내 주문 탭을 숨길 수 있습니다.

내 주문 탭을 가리키는 `NavLink`를 `AuthorizeView`로 감싸도록 `MainLayout`를 수정해 주세요.

```razor
<AuthorizeView>
    <NavLink href="myorders" class="nav-tab">
        <img src="img/bike.svg" />
        <div>My Orders</div>
    </NavLink>
</AuthorizeView>
```

이제 내 주문 탭은 사용자가 로그인한 경우에만 표시됩니다.

이제 컴포넌트 내부의 인증/권한 부여 시스템과 상호 작용하는 두 가지 방법을 살펴보았습니다.

 * `AuthorizeView`에 콘텐츠를 래핑합니다. 이는 인증 상태에 따라 일부 UI 콘텐츠를 변경해야 할 때 유용합니다.
 * 라우팅 가능한 구성 요소에 `[Authorize]` 속성을 배치합니다. 이는 인증 조건에 따라 전체 페이지의 연결 가능성을 제어하려는 경우 유용합니다.

## 리디렉션 흐름 전체에서 주문 상태 보존

방금 어플리케이션에 매우 심각한 결함이 있다는 걸 알게 되었습니다. 클라이언트측 SPA를 구축하고 있으므로 애플리케이션 상태(예를 들면 현재 주문)는 브라우저의 메모리에 보관됩니다. 로그인을 위해 리디렉션하면 해당 상태가 삭제됩니다. 사용자가 다시 리디렉션되면 이제 주문이 비어 있게 됩니다!

이 버그를 재현해 보세요. 로그아웃한 후 주문을 생성하고 주문을 시도하면 로그인 페이지로 리디렉션됩니다. 로그인 후 결제 페이지로 리디렉션되지만 주문한 피자가 사라졌습니다! 이는 브라우저 기반 싱글 페이지 애플리케이션(SPA)의 일반적인 문제이지만 다행히도 간단한 해결 방법이 있습니다.

주문 상태를 유지하여 버그를 수정하겠습니다. 블레이저의 인증 라이브러리를 사용하면 이 작업을 간단하게 수행할 수 있습니다.

지속되기를 원하는 상태를 정의하려면 `RemoteAuthenticationState`에서 상속되는 `PizzaAuthenticationState` 클래스를 추가해 주세요. `RemoteAuthenticationState`는 인증 시스템에서 반환 URL과 같은 리디렉션 전반에 걸쳐 상태를 유지하는 데 사용됩니다. 이 타입에서 파생되면 모든 공용 속성은 지속형 상태의 일부로 JSON으로 직렬화됩니다. 현재 주문을 유지하려면 `Order` 속성을 추가해 주세요.

```csharp
public class PizzaAuthenticationState : RemoteAuthenticationState
{
    public Order Order { get; set; }
}
```

기본 `RemoteAuthenticationState` 대신 `PizzaAuthenticationState`를 사용하도록 인증 시스템을 구성하려면 *Program.cs*를 다음과 같이 수정해 주세요.

```csharp
// Add auth services
builder.Services.AddApiAuthorization<PizzaAuthenticationState>();
```

이제 현재 주문을 유지하는 논리를 추가한 다음 사용자가 성공적으로 로그인한 후 지속된 상태에서 현재 주문을 다시 설정해야 합니다. 그렇게 하려면 `RemoteAuthenticatorView` 대신 `RemoteAuthenticatorViewCore`를 사용하도록 `Authentication` 컴포넌트를 수정해 주세요. 주문 상태가 유지되도록 설정하려면 `OnInitialized`를 재정의하고, 주문 상태를 다시 설정하려면 `OnLogInSucceeded` 콜백을 구현해 주세요.

*BlazingPizza.Client/Pages/Authentication.razor*

```razor
@page "/authentication/{action}"
@inject OrderState OrderState
@inject NavigationManager NavigationManager

<RemoteAuthenticatorViewCore
    TAuthenticationState="PizzaAuthenticationState"
    AuthenticationState="RemoteAuthenticationState"
    OnLogInSucceeded="RestorePizza"
    Action="@Action" />

@code{
    [Parameter] public string Action { get; set; }

    public PizzaAuthenticationState RemoteAuthenticationState { get; set; } = new PizzaAuthenticationState();

    protected override void OnInitialized()
    {
        if (RemoteAuthenticationActions.IsAction(RemoteAuthenticationActions.LogIn, Action))
        {
            // Preserve the current order so that we don't loose it
            RemoteAuthenticationState.Order = OrderState.Order;
        }
    }

    private void RestorePizza(PizzaAuthenticationState pizzaState)
    {
        if (pizzaState.Order != null)
        {
            OrderState.ReplaceOrder(pizzaState.Order);
        }
    }
}
```

이제 로그아웃한 상태에서 주문을 시도하면 인증 프로세스 중에 로컬 저장소에 유지된 주문을 볼 수 있습니다.

![Persisted order state](https://user-images.githubusercontent.com/1874516/78414685-30c4b080-75d2-11ea-98df-d1ac73548774.png)

## 로그아웃 경험 수정하기

현재 사용자가 로그아웃하면 일반 로그아웃 페이지로 이동됩니다.

![Logged out](https://user-images.githubusercontent.com/1874516/78414080-4684a680-75cf-11ea-808d-8d44a5f3941e.png)

`RemoteAuthenticatorViewCore`에서 `LogOutSucceeded` 속성을 설정하여 `Authentication` 컴포넌트에서 이 페이지를 사용자 정의할 수 있습니다.

하지만 사용자가 로그아웃한 후 홈 페이지로 다시 리디렉션되도록 하려면 어떻게 해야 할까요? 이를 위해 *Program.cs*에서 사용자가 성공적으로 로그아웃할 때 사용자를 안내할 경로를 구성할 수 있습니다.

```csharp
// Add auth services
builder.Services.AddApiAuthorization<PizzaAuthenticationState>(options =>
{
    options.AuthenticationPaths.LogOutSucceededPath = "";
});
```

이제 로그아웃하면 사용자가 홈 페이지로 다시 돌아오게 됩니다.

다음 세션 - [자바스크립트 상호운용성](07-javascript-interop.md)

원문 읽기 - [Authentication and authorization](https://github.com/dotnet-presentations/blazor-workshop/blob/main/docs/06-authentication-and-authorization.md)