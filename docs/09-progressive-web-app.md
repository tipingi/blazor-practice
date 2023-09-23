# 프로그레시브 웹앱(PWA) 기능

*프로그레시브 웹앱(PWA)*라는 용어는 특정 최신 브라우저 API를 사용하여 사용자의 데스크톱 또는 모바일 OS와 통합되는 네이티브와 유사한 앱 환경을 만드는 웹 애플리케이션을 말합니다.

아래와 같은 특징이 있습니다.

 * OS의 작업 표시줄 또는 홈 화면에 설치
 * 오프라인 작업
 * 푸시 알림 수신

블레이저는 표준 웹 기술을 사용하므로 다른 최신 웹 프레임워크와 마찬가지로 이러한 브라우저 API를 활용할 수 있습니다.

## 서비스 워커 추가하기

대부분의 PWA 타입 API를 사용하려면 애플리케이션은 *서비스 워커*가 있어야 합니다. 이것은 보통 꽤 작은 자바스크립트 파일입니다. 예를 들어 도메인에서 리소스를 가져올 때, 또는 푸시 알림이 도착할 때 브라우저가 실행 중인 애플리케이션의 컨텍스트 밖에서 호출할 수 있는 이벤트 핸들러를 제공합니다. 서비스 워커에 대한 자세한 내용은 Google [Web Fundamentals guide](https://developers.google.com/web/fundamentals/primers/service-workers) 에서 확인할 수 있습니다.

블레이저 어플리케이션이 .NET에 내장되어 있더라도 어플리케이션의 컨텍스트 밖에서 실행되므로 서비스 워커는 여전히 JavaScript입니다. 기술적으로는 Mono WebAssembly 런타임을 시작한 다음 서비스 워커 컨텍스트 내에서 .NET 코드를 실행하는 서비스 워커를 만들 수 있지만 자바스크립트 코드의 몇 줄만 필요할 수 있다는 점을 고려하면 불필요한 작업이 많을 수 있습니다.

서비스 작업자를 추가하려면 클라이언트 앱의 `wwwroot` 디렉토리에 아래 코드를 포함하는 `service-worker.js`라는 파일을 생성합니다.

```js
self.addEventListener('install', async event => {
    console.log('Installing service worker...');
    self.skipWaiting();
});

self.addEventListener('fetch', event => {
    // You can add custom logic here for controlling whether to use cached data if offline, etc.
    // The following line opts out, so requests go directly to the network as usual.
    return null;
});
```

이 서비스 워커는 아직 아무 일도 하지 않습니다. 자체 설치만 한 다음 `fetch` 이벤트(브라우저가 origin과 HTTP 요청을 수행하고 있음을 의미함)가 발생할 때마다 브라우저가 요청을 정상적으로 처리하도록 선택합니다. 이 파일에 오프라인 지원과 같은 고급 기능을 이곳에 추가할 수 있지만 아직은 필요하지 않습니다.

`index.html`파일의 기존에 있는 `<script>` 요소 밑에 아래 `<script>` 요소를 추가하여 서비스 워커를 사용하도록 설정합니다.

```html
<script>navigator.serviceWorker.register('service-worker.js');</script>
```

지금 앱을 실행하면 브라우저의 dev 도구 콘솔에 다음 메시지가 기록되는 것을 볼 수 있습니다.

```
Installing service worker...
```

> 이는 `service-worker.js`를 수정할 때마다 첫 페이지 로드 중에만 발생합니다. 해당 파일의 내용(바이트 단위 비교)이 변경되지 않은 경우 다시 설치되지 않습니다.

파일에 약간의 사소한 변경(댓글 추가 또는 공백 변경 등)에 따라 서비스 워커가 다시 설치 되는지 그렇지 않은지를 확인해 보세요.

아직 아무 것도 된 것이 없어보이지만 PWA 기능을 위한 토대가 마련된 것입니다.

## 앱 설치 기능 만들기

다음으로 OS에 블레이저 피자 가게 앱을 설치할 수 있도록 해보겠습니다. 이 기능은 Windows/Mac/Linux용 Chrome/Edge-beta 또는 iOS/Android용 Safari/Chrome의  브라우저 기능을 사용합니다. Firefox와 같은 다른 브라우저에서는 아직 구현되지 않을 수 있습니다.

먼저 클라이언트 앱의 `wwwroot`에 아래와 같은 내용을 포함하는 `manifest.json`이라는 파일을 추가합니다.

```js
{
  "short_name": "Blazing Pizza",
  "name": "Blazing Pizza",
  "icons": [
    {
      "src": "img/icon-512.png",
      "type": "image/png",
      "sizes": "512x512"
    }
  ],
  "start_url": "/",
  "background_color": "#860000",
  "display": "standalone",
  "scope": "/",
  "theme_color": "#860000"
}
```

내용을 보면 이 정보가 무엇에 사용되는지 감이 오실 겁니다. OS에 설치될 때 앱이 사용자에게 표시되는 방법을 결정됩니다. 필요에 따라 텍스트나 색상을 자유롭게 변경해 주세요.

다음으로 이 파일을 찾을 위치를 브라우저에게 알려줘야 합니다. `index.html` 파일의 `<head>` 섹션에 다음 요소를 추가해 주세요.

```html
<link rel="manifest" href="manifest.json" />
```

다 되었습니다!. Chrome 또는 Edge beta에서 사이트를 로드하면 주소 표시줄에 사용자에게 앱을 설치하라는 메시지를 표시하는 새 아이콘이 표시됩니다

![image](https://user-images.githubusercontent.com/1101362/66352975-d1eee900-e958-11e9-9042-85ea4ac0c56b.png)

모바일 사용자는 *홈 화면에 추가* 또는 유사한 옵션을 통해 동일한 기능에 도달합니다.

일단 설치되면, 더 이상 브라우저 UI가 없는 독립 실행형 앱으로 나타납니다.

![image](https://user-images.githubusercontent.com/1101362/66356174-0024f680-e962-11e9-9218-3f1ca657a7a7.png)

Windows 사용자도 시작 메뉴에서 찾을 수 있으며 작업 표시줄에 고정할 수도 있습니다. macOS에도 유사한 옵션을 지원합니다.

## 푸시 알림 전송

또 다른 주요 PWA 기능은 사용자가 사이트를 보고 있지 않거나 설치된 앱에 없을 때에도 백엔드 서버에서 *푸시 알림*을 수신하고 표시할 수 있다는 것입니다. 이 기능을 사용하여 다음을 수행할 수 있습니다.

 * 정말 중요한 일이 발생했음을 사용자에게 알려 사이트 혹은 앱을 확인하게 합니다.
 * 앱(뉴스 피드 등)에 저장된 데이터를 업데이트하여 사용자가 오프라인이더라도 데이터를 확인할 수 있게 합니다.
 * 원치 않는 광고 또는 "*Hey we miss you, 다시 방문해주세요!*"라는 메시지를 보냅니다. (장난이에요! 그렇게 하면 바로 차단됩니다.)

  블레이징 피자 가게 앱의 경우 매우 좋은 활용 방법이 있습니다. 많은 사용자가 주문 발송 또는 배송 상태 업데이트를 제공하는 푸시 알림을 받기를 진심으로 원할 것입니다.

### 구독 처리

사용자에게 푸시 알림을 보내기 전에 사용자에게 권한을 요청해야 합니다. 사용자가 동의하면 브라우저가 이 사용자에게 알림을 라우팅하는 데 사용할 수 있는 토큰 세트인 "구독"을 생성합니다.

언제든 이 권한을 요청할 수 있지만 허락해 주실 바란다면 사용자가 구독하려는 이유가 생겼을 때 요청하는 게 좋겠습니다. *업데이트* 버튼을 만들고 싶을 수도 있지만 간편하게 하기 위해 사용자가 체크아웃 페이지에 도착하면 요청합니다. 이 때 사용자가 업데이트에 대해서 호의적인 생각을 가질 것이 분명하니까요.

`Checkout.razor` 파일에서 아래 `OnInitialized` 메서드를 추가합니다.

```cs
protected override void OnInitialized()
{
    // In the background, ask if they want to be notified about order updates
    _ = RequestNotificationSubscriptionAsync();
}
```

그리고 `RequestNotificationSubscriptionAsync` 메서드를 정의합니다. `@code` 코드 블럭 어디에든 추가해 주세요.

```cs
async Task RequestNotificationSubscriptionAsync()
{
    var subscription = await JSRuntime.InvokeAsync<NotificationSubscription>("blazorPushNotifications.requestSubscription");
    if (subscription != null)
    {
        try
        {
            await OrdersClient.SubscribeToNotifications(subscription);
        }
        catch (AccessTokenNotAvailableException ex)
        {
            ex.Redirect();
        }
    }
}
```

또한 `OrdersClient`에 `SubscribeToNotifications` 메서드도 추가해 주세요.

```csharp
public async Task SubscribeToNotifications(NotificationSubscription subscription)
{
    var response = await httpClient.PutAsJsonAsync("notifications/subscribe", subscription);
    response.EnsureSuccessStatusCode();
}
```

그리고 `IJSRuntime` 서비스를 `Checkout` 컴포넌트에 주입해야 합니다.

```razor
@inject IJSRuntime JSRuntime
```

`RequestNotificationSubscriptionAsync` 코드는 `BlazingPizza.ComponentsLibrary/wwwroot/pushNotifications.js`에 있는 자바스크립트 함수를 찾아 실행합니다. 자바스크립트 코드는 `pushManager.subscribe` API를 호출하고 그 결과를 .NET에 반환합니다.

사용자가 알림 수신에 동의하는 경우 이 코드는 나중에 사용할 수 있도록 토큰이 데이터베이스에 저장된 서버로 데이터를 전송합니다.

잘 동작하는지 확인하려면 주문을 시작하고 체크아웃 화면으로 이동해 보세요. 다음과 같은 요청이 표시됩니다.

![image](https://user-images.githubusercontent.com/1101362/66354176-eed8eb80-e95b-11e9-9799-b4eba6410971.png)

*Allow*를 선택하고 브라우저 개발 콘솔에서 오류가 발생하지 않았는지 해주세요. 필요하면 `NotificationsController`의 `Subscribe` 메서드에 중단점을 설정한 후 디버깅을 진행해 보세요.. 브라우저에서 들어오는 데이터를 볼 수 있어야 합니다. 여기에는 엔드포인트 URL과 일부 암호 토큰이 포함되어 있습니다.

지정된 사이트에 대한 알림을 허용하거나 차단한 후에는 브라우저에서 다시 묻지 않습니다. 추가 테스트를 위해 재설정해야 하는 경우 Chrome 또는 Edge beta를 사용하는 경우 주소 표시줄 왼쪽의 "information" 아이콘을 클릭하고 다음 스크린샷과 같이 *Notification*을 다시 *Ask(기본값)*로 변경할 수 있습니다:

![image](https://user-images.githubusercontent.com/1101362/66354317-58f19080-e95c-11e9-8c24-dfa2d19b45f6.png)

### 알림 전송

이제 구독이 있고 알림을 보낼 수 있습니다. 이는 전송 중인 데이터를 보호하기 위해 서버에서 몇 가지 복잡한 암호화 작업을 수행하는 것을 포함합니다. 다행히도 대부분의 복잡성은 타사 NuGet 패키지에서 처리됩니다.

시작하려면 `BlazingPizza.Server` 프로젝트에서 `WebPush` NuGet package를 참조하세요. 아래 실습은 버전 `1.0.11`를 사용한 것으로 가정하겠습니다.

다음으로 `OrdersController`를 열어 주세요. `TrackAndSendNotificationsAsync` 메서드를 살펴보겠습니다. 이 메서드는 주문이 접수될 때마다 일련의 배송 단계를 시뮬레이션하며 아직 구현되지 않은 메서드 `SendNotificationAsync`를 호출합니다.

이제 주문자에게 앞서 처리한 구독을 사용하여 알림을 실제로 발송할 수 있도록 `SendNotificationAsync`를 수정합니다. 다음 코드는 `WebPush` API를 사용하여 알림을 발송합니다.

```cs
private static async Task SendNotificationAsync(Order order, NotificationSubscription subscription, string message)
{
    // For a real application, generate your own
    var publicKey = "BLC8GOevpcpjQiLkO7JmVClQjycvTCYWm6Cq_a7wJZlstGTVZvwGFFHMYfXt6Njyvgx_GlXJeo5cSiZ1y4JOx1o";
    var privateKey = "OrubzSz3yWACscZXjFQrrtDwCKg-TGFuWhluQ2wLXDo";

    var pushSubscription = new PushSubscription(subscription.Url, subscription.P256dh, subscription.Auth);
    var vapidDetails = new VapidDetails("mailto:<someone@example.com>", publicKey, privateKey);
    var webPushClient = new WebPushClient();
    try
    {
        var payload = JsonSerializer.Serialize(new
        {
            message,
            url = $"myorders/{order.OrderId}",
        });
        await webPushClient.SendNotificationAsync(pushSubscription, payload, vapidDetails);
    }
    catch (Exception ex)
    {
        Console.Error.WriteLine("Error sending push notification: " + ex.Message);
    }
}
```

암호화 키는 워크스테이션에서 로컬로 생성하거나 https://tools.reactpwa.com/vapid 과 같은 도구를 사용하여 온라인으로 생성할 수 있습니다. 위 코드에서 데모 키를 변경할 경우 `pushNotification.js`에서 공개 키를 업데이트해야 합니다. C# 코드의 `someone@example.com`주소도 사용자 지정 키 쌍과 일치하도록 업데이트해야 합니다.

현재 상황에서 테스트해보면 서버가 알림을 보내지만 브라우저에 알림이 표시되지 않습니다. 이는 서비스 워커에게 수신된 알림을 처리하는 방법을 알려주지 않았기 때문입니다.

브라우저의 개발 도구를 사용하여 주문 후 10초 후에 알림이 도착하는지 확인해 보세요. 개발 도구 *Application* 탭을 사용하여 *Push Messaging* 섹션을 연 다음 *Start recording*을 클릭하세요.

![image](https://user-images.githubusercontent.com/1101362/66354962-690a6f80-e95e-11e9-9b2c-c254c36e49b4.png)

### 알림 표시

거의 다 왔어요! 이제 `service-worker.js`를 수정하여 수신된 알림을 어떻게 해야 하는지 알려주는 것만 남았습니다. 다음 이벤트 핸들러 함수를 추가하세요.

```js
self.addEventListener('push', event => {
    const payload = event.data.json();
    event.waitUntil(
        self.registration.showNotification('Blazing Pizza', {
            body: payload.message,
            icon: 'img/icon-512.png',
            vibrate: [100, 50, 100],
            data: { url: payload.url }
        })
    );
});
```

브라우저 로그에서 `Installing service worker...`가 표시되고 다음 페이지 로드될 때까지 적용되지 않는다는 것을 기억해 주세요. 서비스 워커의 업데이트가 잘 처리되지 않으면 개발 도구 *Application* 탭을 사용하고 *Service Workers*에서 *Update*(또는 *등록 취소*)를 선택할 수 있습니다.

이 상태에서 주문을 하면 주문이 *Out for delivery*(10초 후) 상태로 전환되는 즉시 푸시 알림을 받을 수 있습니다.

![image](https://user-images.githubusercontent.com/1101362/66355395-0bc2ee00-e95f-11e9-898d-23be0a17829f.png)

크롬 또는 최신 엣지 브라우저를 사용하는 경우 블레이저 피자 가게 앱을 사용하지 않더라도 브라우저가 실행중이라면 알림이 표시 됩니다.(혹은 다음 브라우저를 열었을 때) PWA를 설치했다면 앱을 실행하고 있지 않아도 알림이 표시될 것입니다.

## 알림 클릭 처리

현재 사용자가 알림을 클릭하면 아무 일도 일어나지 않습니다. 하지만 알림에 표시된 주문의 주문 상태 페이지로 이동하면 훨씬 더 좋을 것 같습니다.

서버측 코드에서 이미 이런 목적에 맞게 알림과 `url` 데이터 매개 변수를 보냅니다. 이를 활용하려면 `service-worker.js`에 아래 코드를 추가해 주세요.

```js
self.addEventListener('notificationclick', event => {
    event.notification.close();
    event.waitUntil(clients.openWindow(event.notification.data.url));
});
```

이제 서비스 워커가 업데이트되면 다음 번에 수신된 알림을 클릭하면 관련 주문 상태 정보로 이동합니다. 블레이져 피자 가게 PWA가 설치되어 있으면 PWA로 이동하는 반면 그렇지 않으면 브라우저의 페이지로 이동합니다.

## 요약

이 세션에서는 블레이져 어플리케이션이 .NET으로 작성되더라도 최신 브라우저/자바스크립트 기능의 이점에 완전히 액세스할 수 있는 방법을 보여주었습니다. 웹앱의 항상 업데이트되는 이점을 가지면서 네이티브와 같은 느낌을 주는 OS 설치형 앱을 만들 수 있습니다.

PWA와 관련된 내용을 더 진행하고 싶다면 좀 더 발전된 과제로 오프라인 지원을 추가하는 것을 생각해 보세요. 기본적인 작업을 수행하는 것은 비교적 쉽습니다 - 다른 오프라인 전략을 설명하고 다양한 서비스 워커 예제를 [The Offline Cookbook](https://developers.google.com/web/fundamentals/instant-and-offline/offline-cookbook)에서 확인해 보세요. 블레이저 피자 가게 앱은 보기나 주문과 같은 흥미로운 작업을 수행하려면 서버 API가 필요하기 때문에 네트워크에 연결할 수 없을 때 합리적인 동작을 제공하도록 컴포넌트를 수정해야 합니다. (예를 들면 캐시된 데이터를 사용하거나 오프라인 상태에서 네트워크 액세스가 필요한 작업을 시도하는 경우 표시되는 UI를 제공하는 것 등입니다.)

다음 세션 - [게시 및 배포](10-publish-and-deploy.md)

원문 읽기 - [Progressive web app](https://github.com/dotnet-presentations/blazor-workshop/blob/main/docs/09-progressive-web-app.md)