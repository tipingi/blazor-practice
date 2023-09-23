# 게시 및 배포

이 선택 세션에서는 피자 가게 앱을 애저 앱 서비스에 배포합니다.

## Azure 계정 만들기

이 세션을 완료하려면 애저 계정 또는 구독이 필요합니다. 아직 애저 계정이 없는 경우 [지금 Azure 계정 만들기](https://azure.microsoft.com/Free)를 통해 만들 수 있습니다.

계정을 만든 후에는 이 계정으로 Visual Studio에 로그인해야 Azure 리소스에 액세스할 수 있습니다.

## 새로운 Azure 앱 서비스에 게시하기

Azure 앱 서비스를 이용하면 ASP.NET Core 웹 앱을 클라우드에 쉽게 배포할 수 있습니다.

솔루션에서 서버 프로젝트를 마우스 오른쪽 버튼으로 클릭하고 게시를 선택합니다. ASP.NET Core Server 프로젝트는 클라이언트 블레이저 프로젝트를 참조하므로 서버 프로젝트를 게시하면 블레이저 부분과 그 종속성이 포함됩니다.

![Publish from VS](https://user-images.githubusercontent.com/1874516/51885818-2501ac80-2385-11e9-8025-4d1477083a8d.png)

게시 마법사에서 "Azure"를 선택한 후 다음을 선택합니다.

![Pick a publish target](https://user-images.githubusercontent.com/1874516/78459197-31118a00-766c-11ea-9d41-470ea772e34f.png)

배포 대상에 대해 "Azure App Service(Windows)"를 선택한 다음 다음 다음을 선택합니다.

![Publish to App Service](https://user-images.githubusercontent.com/1874516/78459246-8baae600-766c-11ea-9600-b01e168bf71a.png)

구독이 로드될 때까지 기다렸다가 Azure 앱 서비스에 사용할 구독을 선택하십시오. 구독을 선택한 후 대화 상자 하단에서 "새 Azure 앱 서비스 만들기..."를 선택하십시오.

![Select existing or create new](https://user-images.githubusercontent.com/1874516/78459794-7041da00-7670-11ea-96ab-d103b8f21739.png)

"App Service: Create New" 대화상자에서 다음을 수행합니다.

- 새 Azure 앱 서비스에 사용할 올바른 계정이 오른쪽 상단의 계정 드롭다운에서 선택되었는지 확인합니다.
- 앱의 고유한 이름을 선택합니다(앱의 기본 URL의 일부가 됩니다.)
- 리소스 그룹 및 호스팅 계획과 함께 사용할 Azure 서브스크립션을 선택합니다.
    - 리소스 그룹은 Azure에서 관련 리소스를 그룹화하는 편리한 방법이므로 피자 가게 앱에 맞게 새로 만드는 것을 검토해 보세요.
    - 호스팅 계획의 경우 기본 계층 호스팅 계획 이상을 선택해야 합니다.

![Create new App Service](https://user-images.githubusercontent.com/1874516/78463095-e0ab2400-768d-11ea-8ec3-f8885368118d.png)

앱 서비스를 만들려면 만들기를 클릭합니다. 이 작업은 몇 분 정도 걸릴 수 있습니다.

앱 서비스가 생성되면 해당 서비스가 선택되었는지 확인한 다음 게시 대화상자에서 마침을 클릭합니다

![Finish Publish](https://user-images.githubusercontent.com/1874516/78459868-0d047780-7671-11ea-87d5-0a72ca9e5d36.png)

앱 서비스가 생성되면 게시 페이지에 게시 프로필이 표시됩니다.

![Publish profile](https://user-images.githubusercontent.com/1874516/78460244-0e836f00-7674-11ea-975a-f582d6af9942.png)

이 때, 앱에 대한 프로덕션 데이터베이스를 만들 수 있습니다. 앱이 SQLite를 사용하고 자체 데이터베이스를 배포하기 때문에 데이터베이스를 만들 필요는 없지만 실제 앱의 경우 만들게 될 것입니다.

앱은 게시하였지만, 서버가 시작에 실패하고 오류가 반환됩니다. 이는 먼저 IdentityServer에 대한 서명 키를 구성해야 하기 때문입니다. 개발하는 동안 개발 키(*BlazingPizza.Server/appsettings.Development.json* 참조)를 사용했지만 실제 운영에서는 토큰을 발행하기 위한 인증서를 구성해야 합니다. 이 작업은 Azure Key Vault를 사용하여 수행합니다.

## Azure Key Vault를 사용하여 서명 인증서 설정

기존 키 볼트를 사용하여 서명 인증서를 생성하거나 새 인증서를 생성할 수 있습니다.

새롭게 키 볼트를 만들려면 아래와 같이 진행하세요.

<span style="color:red">리뷰 요망</span>
- [Azure 포탈](https://portal.azure.com)에 로그인 해주세요.
- 검색 상자에 **Key Vault**를 입력합니다.
- 결과 목록에서 **Key vaults**를 선택합니다.
- 키 볼트 섹션에서 **추가**를 선택합니다.
- **키 볼트 만들기** 섹션에서 다음 정보를 제공합니다:
    - **Subscription**: 구독을 선택합니다.
    - **Resource group**: 키 볼트의 리소스 그룹을 선택합니다.
    - **Key vault name**: 고유한 이름이 필요합니다.
    - **Region** 풀다운 메뉴에서 위치를 선택합니다.
    - 다른 옵션은 기본값으로 유지합니다.
- 위의 정보를 제공한 후 **Review + create**를 선택하여 키 볼트를 만듭니다.

Azure 포털에서 키 볼트를 찾아 **Certificates**를 선택하고 **Generate/Import**를 선택하여 새 인증서를 만듭니다.

![Generate key vault certificate](https://user-images.githubusercontent.com/1874516/78463378-ba3ab800-7690-11ea-9744-6850c2d1a7e6.png)

선택한 이름과 일치하는 제목 이름("CN="으로 구분)을 사용하여 자체 서명된 인증서를 생성하고 **Create**를 선택합니다.

![Create certificate](https://user-images.githubusercontent.com/1874516/78463413-17cf0480-7691-11ea-91dc-343cdea5aa79.png)

Azure 포털에서 앱 서비스를 찾아 **TLS/SSL Settings**을 선택하고 **Private Key Certificates (.pfx)** 탭을 선택한 다음 **Import Key Vault Certificate**를 선택합니다.

![Import key vault certificate](https://user-images.githubusercontent.com/1874516/78463445-890eb780-7691-11ea-949a-d7dd38b43550.png)

조금 전에 만든 인증서를 선택하여 앱 서비스로 가져옵니다.

![Select certificate](https://user-images.githubusercontent.com/1874516/78463454-ae9bc100-7691-11ea-9ca4-64d27582f699.png)

가져온 인증서를 선택하고 지문(thumbprint)을 복사합니다.

![Copy certificate thumbprint](https://user-images.githubusercontent.com/1874516/78463487-1520df00-7692-11ea-93ae-697406bfdd86.png)

앱 서비스의 왼쪽 메뉴에서 **Configuration**을 선택합니다. 방금 복사한 인증서 지문(thumbprint)에 값이 설정된 `WEBSITE_LOAD_CERTIFICATES` 어플리케이션 설정을 추가합니다. 이 설정을 사용하면 Windows 인증서 저장소를 사용하여 인증서를 사용할 수 있습니다.

![Load certificates setting](https://user-images.githubusercontent.com/1874516/78463547-e8b99280-7692-11ea-9d02-394b20c653cd.png)

이제 서버 프로젝트에서 **appsettings.json**를 업데이트하여 인증서를 프로덕션에서 사용하도록 앱을 구성합니다.

```json
"IdentityServer": {
  "Key": {
    "Type": "Store",
    "StoreName": "My",
    "StoreLocation": "CurrentUser",
    "Name": "CN=BlazingPizzaCertificate"
  },
  "Clients": {
    "BlazingPizza.Client": {
      "Profile": "IdentityServerSPA"
    }
  }
}
```

게시할 준비가 되었습니다! 게시를 클릭합니다.

앱을 게시하는 데 몇 분 정도 걸릴 수 있습니다. 앱이 배포를 마치면 브라우저에 자동으로 로드됩니다.

![Published app](https://user-images.githubusercontent.com/1874516/78463636-09ceb300-7694-11ea-9d3c-57b52b982186.png)

축하합니다!

완료된 블레이저 앱을 친구에게 자랑한 후에는 더 이상 유지 관리하고 싶지 않은 Azure 리소스를 정리해야 합니다.

원문 읽기 - [Publish and deploy](https://github.com/dotnet-presentations/blazor-workshop/blob/main/docs/10-publish-and-deploy.md)