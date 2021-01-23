---
ms.openlocfilehash: a024fb533c552563da6c9179301e16a2e1d09d5f
ms.sourcegitcommit: 6341ad07cd5b03269e7fd20cd3212e48baee7c07
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/23/2021
ms.locfileid: "49942160"
---
<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="735e8-101">在此练习中，你将扩展上一练习中的应用程序，以支持使用 Azure AD 进行身份验证。</span><span class="sxs-lookup"><span data-stu-id="735e8-101">In this exercise you will extend the application from the previous exercise to support authentication with Azure AD.</span></span> <span data-ttu-id="735e8-102">这是获取调用 Microsoft Graph API 所需的 OAuth 访问令牌所必需的。</span><span class="sxs-lookup"><span data-stu-id="735e8-102">This is required to obtain the necessary OAuth access token to call the Microsoft Graph API.</span></span> <span data-ttu-id="735e8-103">在此步骤中，您将配置 [Microsoft.Identity.Web](https://www.nuget.org/packages/Microsoft.Identity.Web/) 库。</span><span class="sxs-lookup"><span data-stu-id="735e8-103">In this step you will configure the [Microsoft.Identity.Web](https://www.nuget.org/packages/Microsoft.Identity.Web/) library.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="735e8-104">为了避免将应用程序 ID 和密码存储在源中，将使用 [.NET 密码管理器](/aspnet/core/security/app-secrets) 存储这些值。</span><span class="sxs-lookup"><span data-stu-id="735e8-104">To avoid storing the application ID and secret in source, you will use the [.NET Secret Manager](/aspnet/core/security/app-secrets) to store these values.</span></span> <span data-ttu-id="735e8-105">密码管理器仅供开发使用，生产应用应使用受信任的密码管理器来存储密码。</span><span class="sxs-lookup"><span data-stu-id="735e8-105">The Secret Manager is for development purposes only, production apps should use a trusted secret manager for storing secrets.</span></span>

1. <span data-ttu-id="735e8-106">打开 **./appsettings.js，** 并将其内容替换为以下内容。</span><span class="sxs-lookup"><span data-stu-id="735e8-106">Open **./appsettings.json** and replace its contents with the following.</span></span>

    :::code language="json" source="../demo/GraphTutorial/appsettings.json" highlight="2-6":::

1. <span data-ttu-id="735e8-107">在 **GraphTu一l.csproj** 所在的目录中打开 CLI，然后运行以下命令，使用 Azure 门户中的应用程序 ID 和应用程序密码进行代用。 `YOUR_APP_ID` `YOUR_APP_SECRET`</span><span class="sxs-lookup"><span data-stu-id="735e8-107">Open your CLI in the directory where **GraphTutorial.csproj** is located, and run the following commands, substituting `YOUR_APP_ID` with your application ID from the Azure portal, and `YOUR_APP_SECRET` with your application secret.</span></span>

    ```Shell
    dotnet user-secrets init
    dotnet user-secrets set "AzureAd:ClientId" "YOUR_APP_ID"
    dotnet user-secrets set "AzureAd:ClientSecret" "YOUR_APP_SECRET"
    ```

## <a name="implement-sign-in"></a><span data-ttu-id="735e8-108">实现登录</span><span class="sxs-lookup"><span data-stu-id="735e8-108">Implement sign-in</span></span>

<span data-ttu-id="735e8-109">首先，将 Microsoft Identity 平台服务添加到应用程序。</span><span class="sxs-lookup"><span data-stu-id="735e8-109">Start by adding the Microsoft Identity platform services to the application.</span></span>

1. <span data-ttu-id="735e8-110">在 **./Graph** **GraphConstants.cs** 创建一个名为 GraphConstants.cs 的新文件，并添加以下代码。</span><span class="sxs-lookup"><span data-stu-id="735e8-110">Create a new file named **GraphConstants.cs** in the **./Graph** directory and add the following code.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Graph/GraphConstants.cs" id="GraphConstantsSnippet":::

1. <span data-ttu-id="735e8-111">打开 **./Startup.cs** 文件，将以下语句 `using` 添加到文件顶部。</span><span class="sxs-lookup"><span data-stu-id="735e8-111">Open the **./Startup.cs** file and add the following `using` statements to the top of the file.</span></span>

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

1. <span data-ttu-id="735e8-112">将现有的 `ConfigureServices` 函数替换为以下内容。</span><span class="sxs-lookup"><span data-stu-id="735e8-112">Replace the existing `ConfigureServices` function with the following.</span></span>

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

1. <span data-ttu-id="735e8-113">在 `Configure` 函数中，在行上方添加以下 `app.UseAuthorization();` 行。</span><span class="sxs-lookup"><span data-stu-id="735e8-113">In the `Configure` function, add the following line above the `app.UseAuthorization();` line.</span></span>

    ```csharp
    app.UseAuthentication();
    ```

1. <span data-ttu-id="735e8-114">打开 **./Controllers/HomeController.cs，** 并将其内容替换为以下内容。</span><span class="sxs-lookup"><span data-stu-id="735e8-114">Open **./Controllers/HomeController.cs** and replace its contents with the following.</span></span>

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

1. <span data-ttu-id="735e8-115">保存更改并启动项目。</span><span class="sxs-lookup"><span data-stu-id="735e8-115">Save your changes and start the project.</span></span> <span data-ttu-id="735e8-116">使用 Microsoft 帐户登录。</span><span class="sxs-lookup"><span data-stu-id="735e8-116">Login with your Microsoft account.</span></span>

1. <span data-ttu-id="735e8-117">检查同意提示。</span><span class="sxs-lookup"><span data-stu-id="735e8-117">Examine the consent prompt.</span></span> <span data-ttu-id="735e8-118">权限列表对应于 **./Graph/GraphConstants.cs** 中配置的权限范围列表。</span><span class="sxs-lookup"><span data-stu-id="735e8-118">The list of permissions correspond to list of permissions scopes configured in **./Graph/GraphConstants.cs**.</span></span>

    - <span data-ttu-id="735e8-119">**保持对已授予** 其访问权限的数据的访问权限： () MSAL 请求获取此权限， `offline_access` 以便检索刷新令牌。</span><span class="sxs-lookup"><span data-stu-id="735e8-119">**Maintain access to data you have given it access to:** (`offline_access`) This permission is requested by MSAL in order to retrieve refresh tokens.</span></span>
    - <span data-ttu-id="735e8-120">**登录并读取个人资料： ()** 此权限允许应用获取登录用户的 `User.Read` 配置文件和个人资料照片。</span><span class="sxs-lookup"><span data-stu-id="735e8-120">**Sign you in and read your profile:** (`User.Read`) This permission allows the app to get the logged-in user's profile and profile photo.</span></span>
    - <span data-ttu-id="735e8-121">**读取邮箱设置： ()** 此权限允许应用读取用户的邮箱设置，包括时区 `MailboxSettings.Read` 和时间格式。</span><span class="sxs-lookup"><span data-stu-id="735e8-121">**Read your mailbox settings:** (`MailboxSettings.Read`) This permission allows the app to read the user's mailbox settings, including time zone and time format.</span></span>
    - <span data-ttu-id="735e8-122">**具有对** 日历的完全访问权限： () 此权限允许应用读取用户日历上的事件、添加新事件和修改现有 `Calendars.ReadWrite` 事件。</span><span class="sxs-lookup"><span data-stu-id="735e8-122">**Have full access to your calendars:** (`Calendars.ReadWrite`) This permission allows the app to read events on the user's calendar, add new events, and modify existing ones.</span></span>

    ![Microsoft 标识平台同意提示的屏幕截图](./images/add-aad-auth-03.png)

    <span data-ttu-id="735e8-124">有关同意详细信息，请参阅了解 [Azure AD 应用程序同意体验](/azure/active-directory/develop/application-consent-experience)。</span><span class="sxs-lookup"><span data-stu-id="735e8-124">For more information regarding consent, see [Understanding Azure AD application consent experiences](/azure/active-directory/develop/application-consent-experience).</span></span>

1. <span data-ttu-id="735e8-125">同意请求的权限。</span><span class="sxs-lookup"><span data-stu-id="735e8-125">Consent to the requested permissions.</span></span> <span data-ttu-id="735e8-126">浏览器重定向到应用，显示令牌。</span><span class="sxs-lookup"><span data-stu-id="735e8-126">The browser redirects to the app, showing the token.</span></span>

### <a name="get-user-details"></a><span data-ttu-id="735e8-127">获取用户详细信息</span><span class="sxs-lookup"><span data-stu-id="735e8-127">Get user details</span></span>

<span data-ttu-id="735e8-128">用户登录后，可以从 Microsoft Graph 获取其信息。</span><span class="sxs-lookup"><span data-stu-id="735e8-128">Once the user is logged in, you can get their information from Microsoft Graph.</span></span>

1. <span data-ttu-id="735e8-129">打开 **./Graph/GraphClaimsPrincipalExtensions.cs，** 并将其全部内容替换为以下内容。</span><span class="sxs-lookup"><span data-stu-id="735e8-129">Open **./Graph/GraphClaimsPrincipalExtensions.cs** and replace its entire contents with the following.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Graph/GraphClaimsPrincipalExtensions.cs" id="GraphClaimsExtensionsSnippet":::

1. <span data-ttu-id="735e8-130">打开 **./Startup.cs，** 将现有 `.AddMicrosoftIdentityWebApp(Configuration)` 行替换为以下代码。</span><span class="sxs-lookup"><span data-stu-id="735e8-130">Open **./Startup.cs** and replace the existing `.AddMicrosoftIdentityWebApp(Configuration)` line with the following code.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Startup.cs" id="AddSignInSnippet":::

    <span data-ttu-id="735e8-131">考虑此代码执行哪些功能。</span><span class="sxs-lookup"><span data-stu-id="735e8-131">Consider what this code does.</span></span>

    - <span data-ttu-id="735e8-132">它添加事件的事件 `OnTokenValidated` 处理程序。</span><span class="sxs-lookup"><span data-stu-id="735e8-132">It adds an event handler for the `OnTokenValidated` event.</span></span>
        - <span data-ttu-id="735e8-133">它使用 `ITokenAcquisition` 接口获取访问令牌。</span><span class="sxs-lookup"><span data-stu-id="735e8-133">It uses the `ITokenAcquisition` interface to get an access token.</span></span>
        - <span data-ttu-id="735e8-134">它调用 Microsoft Graph 获取用户配置文件和照片。</span><span class="sxs-lookup"><span data-stu-id="735e8-134">It calls Microsoft Graph to get the user's profile and photo.</span></span>
        - <span data-ttu-id="735e8-135">它将 Graph 信息添加到用户标识中。</span><span class="sxs-lookup"><span data-stu-id="735e8-135">It adds the Graph information to the user's identity.</span></span>

1. <span data-ttu-id="735e8-136">在调用之后和调用之前 `EnableTokenAcquisitionToCallDownstreamApi` 添加以下函数 `AddInMemoryTokenCaches` 调用。</span><span class="sxs-lookup"><span data-stu-id="735e8-136">Add the following function call after the `EnableTokenAcquisitionToCallDownstreamApi` call and before the `AddInMemoryTokenCaches` call.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Startup.cs" id="AddGraphClientSnippet":::

    <span data-ttu-id="735e8-137">这将通过依赖关系注入向控制器提供经过身份验证的 **GraphServiceClient。**</span><span class="sxs-lookup"><span data-stu-id="735e8-137">This will make an authenticated **GraphServiceClient** available to controllers via dependency injection.</span></span>

1. <span data-ttu-id="735e8-138">打开 **./Controllers/HomeController.cs，** 将 `Index` 函数替换为以下内容。</span><span class="sxs-lookup"><span data-stu-id="735e8-138">Open **./Controllers/HomeController.cs** and replace the `Index` function with the following.</span></span>

    ```csharp
    public IActionResult Index()
    {
        return View();
    }
    ```

1. <span data-ttu-id="735e8-139">删除对 `ITokenAcquisition` **HomeController** 类的所有引用。</span><span class="sxs-lookup"><span data-stu-id="735e8-139">Remove all references to `ITokenAcquisition` in the **HomeController** class.</span></span>

1. <span data-ttu-id="735e8-140">保存更改、启动应用并完成登录过程。</span><span class="sxs-lookup"><span data-stu-id="735e8-140">Save your changes, start the app, and go through the sign-in process.</span></span> <span data-ttu-id="735e8-141">你最终应返回到主页，但 UI 应更改以指示你已登录。</span><span class="sxs-lookup"><span data-stu-id="735e8-141">You should end up back on the home page, but the UI should change to indicate that you are signed-in.</span></span>

    ![登录后主页的屏幕截图](./images/add-aad-auth-01.png)

1. <span data-ttu-id="735e8-143">单击右上角的用户头像以访问 **"注销"** 链接。</span><span class="sxs-lookup"><span data-stu-id="735e8-143">Click the user avatar in the top right corner to access the **Sign Out** link.</span></span> <span data-ttu-id="735e8-144">单击 **"注销** "可重置会话，并返回到主页。</span><span class="sxs-lookup"><span data-stu-id="735e8-144">Clicking **Sign Out** resets the session and returns you to the home page.</span></span>

    ![包含"注销"链接的下拉菜单屏幕截图](./images/add-aad-auth-02.png)

> [!TIP]
> <span data-ttu-id="735e8-146">如果在主页上看不到用户名，并且"使用头像"下拉列表在进行更改后缺少名称和电子邮件，请注销并重新登录。</span><span class="sxs-lookup"><span data-stu-id="735e8-146">If you do not see your user name on the home page and the use avatar dropdown is missing name and email after making these changes, sign out and sign back in.</span></span>

## <a name="storing-and-refreshing-tokens"></a><span data-ttu-id="735e8-147">存储和刷新令牌</span><span class="sxs-lookup"><span data-stu-id="735e8-147">Storing and refreshing tokens</span></span>

<span data-ttu-id="735e8-148">此时，应用程序具有访问令牌，该令牌在 API 调用 `Authorization` 标头中发送。</span><span class="sxs-lookup"><span data-stu-id="735e8-148">At this point your application has an access token, which is sent in the `Authorization` header of API calls.</span></span> <span data-ttu-id="735e8-149">这是允许应用代表用户访问 Microsoft Graph 的令牌。</span><span class="sxs-lookup"><span data-stu-id="735e8-149">This is the token that allows the app to access Microsoft Graph on the user's behalf.</span></span>

<span data-ttu-id="735e8-150">但是，此令牌是短期的。</span><span class="sxs-lookup"><span data-stu-id="735e8-150">However, this token is short-lived.</span></span> <span data-ttu-id="735e8-151">令牌在颁发后一小时过期。</span><span class="sxs-lookup"><span data-stu-id="735e8-151">The token expires an hour after it is issued.</span></span> <span data-ttu-id="735e8-152">此时刷新令牌将变得有用。</span><span class="sxs-lookup"><span data-stu-id="735e8-152">This is where the refresh token becomes useful.</span></span> <span data-ttu-id="735e8-153">刷新令牌允许应用请求新的访问令牌，而无需用户重新登录。</span><span class="sxs-lookup"><span data-stu-id="735e8-153">The refresh token allows the app to request a new access token without requiring the user to sign in again.</span></span>

<span data-ttu-id="735e8-154">由于应用使用的是 Microsoft.Identity.Web 库，因此不需要实现任何令牌存储或刷新逻辑。</span><span class="sxs-lookup"><span data-stu-id="735e8-154">Because the app is using the Microsoft.Identity.Web library, you do not have to implement any token storage or refresh logic.</span></span>

<span data-ttu-id="735e8-155">应用使用内存中令牌缓存，这足以满足应用重启时无需保留令牌的应用。</span><span class="sxs-lookup"><span data-stu-id="735e8-155">The app uses the in-memory token cache, which is sufficient for apps that do not need to persist tokens when the app restarts.</span></span> <span data-ttu-id="735e8-156">生产应用可能会改为使用 Microsoft.Identity.Web [库中](https://github.com/AzureAD/microsoft-identity-web/wiki/token-cache-serialization) 的分布式缓存选项。</span><span class="sxs-lookup"><span data-stu-id="735e8-156">Production apps may instead use the [distributed cache options](https://github.com/AzureAD/microsoft-identity-web/wiki/token-cache-serialization) in the Microsoft.Identity.Web library.</span></span>

<span data-ttu-id="735e8-157">`GetAccessTokenForUserAsync`此方法会处理令牌过期并刷新。</span><span class="sxs-lookup"><span data-stu-id="735e8-157">The `GetAccessTokenForUserAsync` method handles token expiration and refresh for you.</span></span> <span data-ttu-id="735e8-158">它首先检查缓存的令牌，如果它未过期，它将返回它。</span><span class="sxs-lookup"><span data-stu-id="735e8-158">It first checks the cached token, and if it is not expired, it returns it.</span></span> <span data-ttu-id="735e8-159">如果已过期，它将使用缓存的刷新令牌获取新的刷新令牌。</span><span class="sxs-lookup"><span data-stu-id="735e8-159">If it is expired, it uses the cached refresh token to obtain a new one.</span></span>

<span data-ttu-id="735e8-160">控制器通过依赖关系注入获取的 **GraphServiceClient** 将使用您使用的身份验证提供程序 `GetAccessTokenForUserAsync` 进行预配置。</span><span class="sxs-lookup"><span data-stu-id="735e8-160">The **GraphServiceClient** that controllers get via dependency injection will be pre-configured with an authentication provider that uses `GetAccessTokenForUserAsync` for you.</span></span>
