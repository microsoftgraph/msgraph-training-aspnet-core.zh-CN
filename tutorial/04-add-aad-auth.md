---
ms.openlocfilehash: a024fb533c552563da6c9179301e16a2e1d09d5f
ms.sourcegitcommit: 6341ad07cd5b03269e7fd20cd3212e48baee7c07
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/23/2021
ms.locfileid: "49942160"
---
<!-- markdownlint-disable MD002 MD041 -->

在此练习中，你将扩展上一练习中的应用程序，以支持使用 Azure AD 进行身份验证。 这是获取调用 Microsoft Graph API 所需的 OAuth 访问令牌所必需的。 在此步骤中，您将配置 [Microsoft.Identity.Web](https://www.nuget.org/packages/Microsoft.Identity.Web/) 库。

> [!IMPORTANT]
> 为了避免将应用程序 ID 和密码存储在源中，将使用 [.NET 密码管理器](/aspnet/core/security/app-secrets) 存储这些值。 密码管理器仅供开发使用，生产应用应使用受信任的密码管理器来存储密码。

1. 打开 **./appsettings.js，** 并将其内容替换为以下内容。

    :::code language="json" source="../demo/GraphTutorial/appsettings.json" highlight="2-6":::

1. 在 **GraphTu一l.csproj** 所在的目录中打开 CLI，然后运行以下命令，使用 Azure 门户中的应用程序 ID 和应用程序密码进行代用。 `YOUR_APP_ID` `YOUR_APP_SECRET`

    ```Shell
    dotnet user-secrets init
    dotnet user-secrets set "AzureAd:ClientId" "YOUR_APP_ID"
    dotnet user-secrets set "AzureAd:ClientSecret" "YOUR_APP_SECRET"
    ```

## <a name="implement-sign-in"></a>实现登录

首先，将 Microsoft Identity 平台服务添加到应用程序。

1. 在 **./Graph** **GraphConstants.cs** 创建一个名为 GraphConstants.cs 的新文件，并添加以下代码。

    :::code language="csharp" source="../demo/GraphTutorial/Graph/GraphConstants.cs" id="GraphConstantsSnippet":::

1. 打开 **./Startup.cs** 文件，将以下语句 `using` 添加到文件顶部。

    ```csharp
    using Microsoft.AspNetCore.Authentication.OpenIdConnect;
    using Microsoft.AspNetCore.Authorization;
    using Microsoft.AspNetCore.Mvc.Authorization;
    using Microsoft.Identity.Web;
    using Microsoft.Identity.Web.UI;
    using Microsoft.IdentityModel.Protocols.OpenIdConnect;
    using Microsoft.Graph;
    using System.Net;
    using System.Net.Http.Headers;
    ```

1. 将现有的 `ConfigureServices` 函数替换为以下内容。

    ```csharp
    public void ConfigureServices(IServiceCollection services)
    {
        services
            // Use OpenId authentication
            .AddAuthentication(OpenIdConnectDefaults.AuthenticationScheme)
            // Specify this is a web app and needs auth code flow
            .AddMicrosoftIdentityWebApp(Configuration)
            // Add ability to call web API (Graph)
            // and get access tokens
            .EnableTokenAcquisitionToCallDownstreamApi(options => {
                Configuration.Bind("AzureAd", options);
            }, GraphConstants.Scopes)
            // Use in-memory token cache
            // See https://github.com/AzureAD/microsoft-identity-web/wiki/token-cache-serialization
            .AddInMemoryTokenCaches();

        // Require authentication
        services.AddControllersWithViews(options =>
        {
            var policy = new AuthorizationPolicyBuilder()
                .RequireAuthenticatedUser()
                .Build();
            options.Filters.Add(new AuthorizeFilter(policy));
        })
        // Add the Microsoft Identity UI pages for signin/out
        .AddMicrosoftIdentityUI();
    }
    ```

1. 在 `Configure` 函数中，在行上方添加以下 `app.UseAuthorization();` 行。

    ```csharp
    app.UseAuthentication();
    ```

1. 打开 **./Controllers/HomeController.cs，** 并将其内容替换为以下内容。

    ```csharp
    using GraphTutorial.Models;
    using Microsoft.AspNetCore.Authorization;
    using Microsoft.AspNetCore.Mvc;
    using Microsoft.Extensions.Logging;
    using Microsoft.Identity.Web;
    using System.Diagnostics;
    using System.Threading.Tasks;

    namespace GraphTutorial.Controllers
    {
        public class HomeController : Controller
        {
            ITokenAcquisition _tokenAcquisition;
            private readonly ILogger<HomeController> _logger;

            // Get the ITokenAcquisition interface via
            // dependency injection
            public HomeController(
                ITokenAcquisition tokenAcquisition,
                ILogger<HomeController> logger)
            {
                _tokenAcquisition = tokenAcquisition;
                _logger = logger;
            }

            public async Task<IActionResult> Index()
            {
                // TEMPORARY
                // Get the token and display it
                try
                {
                    string token = await _tokenAcquisition
                        .GetAccessTokenForUserAsync(GraphConstants.Scopes);
                    return View().WithInfo("Token acquired", token);
                }
                catch (MicrosoftIdentityWebChallengeUserException)
                {
                    return Challenge();
                }
            }

            public IActionResult Privacy()
            {
                return View();
            }

            [ResponseCache(Duration = 0, Location = ResponseCacheLocation.None, NoStore = true)]
            public IActionResult Error()
            {
                return View(new ErrorViewModel { RequestId = Activity.Current?.Id ?? HttpContext.TraceIdentifier });
            }

            [ResponseCache(Duration = 0, Location = ResponseCacheLocation.None, NoStore = true)]
            [AllowAnonymous]
            public IActionResult ErrorWithMessage(string message, string debug)
            {
                return View("Index").WithError(message, debug);
            }
        }
    }
    ```

1. 保存更改并启动项目。 使用 Microsoft 帐户登录。

1. 检查同意提示。 权限列表对应于 **./Graph/GraphConstants.cs** 中配置的权限范围列表。

    - **保持对已授予** 其访问权限的数据的访问权限： () MSAL 请求获取此权限， `offline_access` 以便检索刷新令牌。
    - **登录并读取个人资料： ()** 此权限允许应用获取登录用户的 `User.Read` 配置文件和个人资料照片。
    - **读取邮箱设置： ()** 此权限允许应用读取用户的邮箱设置，包括时区 `MailboxSettings.Read` 和时间格式。
    - **具有对** 日历的完全访问权限： () 此权限允许应用读取用户日历上的事件、添加新事件和修改现有 `Calendars.ReadWrite` 事件。

    ![Microsoft 标识平台同意提示的屏幕截图](./images/add-aad-auth-03.png)

    有关同意详细信息，请参阅了解 [Azure AD 应用程序同意体验](/azure/active-directory/develop/application-consent-experience)。

1. 同意请求的权限。 浏览器重定向到应用，显示令牌。

### <a name="get-user-details"></a>获取用户详细信息

用户登录后，可以从 Microsoft Graph 获取其信息。

1. 打开 **./Graph/GraphClaimsPrincipalExtensions.cs，** 并将其全部内容替换为以下内容。

    :::code language="csharp" source="../demo/GraphTutorial/Graph/GraphClaimsPrincipalExtensions.cs" id="GraphClaimsExtensionsSnippet":::

1. 打开 **./Startup.cs，** 将现有 `.AddMicrosoftIdentityWebApp(Configuration)` 行替换为以下代码。

    :::code language="csharp" source="../demo/GraphTutorial/Startup.cs" id="AddSignInSnippet":::

    考虑此代码执行哪些功能。

    - 它添加事件的事件 `OnTokenValidated` 处理程序。
        - 它使用 `ITokenAcquisition` 接口获取访问令牌。
        - 它调用 Microsoft Graph 获取用户配置文件和照片。
        - 它将 Graph 信息添加到用户标识中。

1. 在调用之后和调用之前 `EnableTokenAcquisitionToCallDownstreamApi` 添加以下函数 `AddInMemoryTokenCaches` 调用。

    :::code language="csharp" source="../demo/GraphTutorial/Startup.cs" id="AddGraphClientSnippet":::

    这将通过依赖关系注入向控制器提供经过身份验证的 **GraphServiceClient。**

1. 打开 **./Controllers/HomeController.cs，** 将 `Index` 函数替换为以下内容。

    ```csharp
    public IActionResult Index()
    {
        return View();
    }
    ```

1. 删除对 `ITokenAcquisition` **HomeController** 类的所有引用。

1. 保存更改、启动应用并完成登录过程。 你最终应返回到主页，但 UI 应更改以指示你已登录。

    ![登录后主页的屏幕截图](./images/add-aad-auth-01.png)

1. 单击右上角的用户头像以访问 **"注销"** 链接。 单击 **"注销** "可重置会话，并返回到主页。

    ![包含"注销"链接的下拉菜单屏幕截图](./images/add-aad-auth-02.png)

> [!TIP]
> 如果在主页上看不到用户名，并且"使用头像"下拉列表在进行更改后缺少名称和电子邮件，请注销并重新登录。

## <a name="storing-and-refreshing-tokens"></a>存储和刷新令牌

此时，应用程序具有访问令牌，该令牌在 API 调用 `Authorization` 标头中发送。 这是允许应用代表用户访问 Microsoft Graph 的令牌。

但是，此令牌是短期的。 令牌在颁发后一小时过期。 此时刷新令牌将变得有用。 刷新令牌允许应用请求新的访问令牌，而无需用户重新登录。

由于应用使用的是 Microsoft.Identity.Web 库，因此不需要实现任何令牌存储或刷新逻辑。

应用使用内存中令牌缓存，这足以满足应用重启时无需保留令牌的应用。 生产应用可能会改为使用 Microsoft.Identity.Web [库中](https://github.com/AzureAD/microsoft-identity-web/wiki/token-cache-serialization) 的分布式缓存选项。

`GetAccessTokenForUserAsync`此方法会处理令牌过期并刷新。 它首先检查缓存的令牌，如果它未过期，它将返回它。 如果已过期，它将使用缓存的刷新令牌获取新的刷新令牌。

控制器通过依赖关系注入获取的 **GraphServiceClient** 将使用您使用的身份验证提供程序 `GetAccessTokenForUserAsync` 进行预配置。
