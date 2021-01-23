---
ms.openlocfilehash: 5b1a776c28b6f9218c713dde68f45e571ebfd999
ms.sourcegitcommit: 6341ad07cd5b03269e7fd20cd3212e48baee7c07
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/23/2021
ms.locfileid: "49942153"
---
<!-- markdownlint-disable MD002 MD041 -->

首先创建一个ASP.NET核心 Web 应用。

1. 在要创建项目的目录中 (CLI) 打开命令行界面。 运行以下命令。

    ```Shell
    dotnet new mvc -o GraphTutorial
    ```

1. 创建项目后，通过更改当前目录到 **GraphTu且在** CLI 中运行以下命令来验证项目是否有效。

    ```Shell
    dotnet run
    ```

1. 打开浏览器并浏览到 `https://localhost:5001` 。 如果一切正常，则应该会看到默认ASP.NET核心页面。

> [!IMPORTANT]
> 如果您收到一条警告，指出 **localhost** 的证书不可信，您可以使用 .NET Core CLI 安装和信任开发证书。 有关 [特定操作系统的说明，请参阅 ASP.NET Core](/aspnet/core/security/enforcing-ssl?view=aspnetcore-5.0) 中的强制 HTTPS。

## <a name="add-nuget-packages"></a>添加 NuGet 程序包

在继续之前，请安装一些稍后将使用的其他 NuGet 程序包。

- [用于请求和管理访问令牌的 Microsoft.Identity.Web。](https://www.nuget.org/packages/Microsoft.Identity.Web/)
- [Microsoft.Identity.Web.MicrosoftGraph，](https://www.nuget.org/packages/Microsoft.Identity.Web.MicrosoftGraph/) 用于通过依赖关系注入添加 Microsoft Graph SDK。
- 用于登录和注销 UI 的[Microsoft.Identity.Web.UI。](https://www.nuget.org/packages/Microsoft.Identity.Web.UI/)
- [用于跨平台处理时区标识符的 TimeZoneConverter。](https://github.com/mj1856/TimeZoneConverter)

1. 在 CLI 中运行以下命令以安装依赖项。

    ```Shell
    dotnet add package Microsoft.Identity.Web --version 1.5.1
    dotnet add package Microsoft.Identity.Web.MicrosoftGraph --version 1.5.1
    dotnet add package Microsoft.Identity.Web.UI --version 1.5.1
    dotnet add package TimeZoneConverter
    ```

## <a name="design-the-app"></a>设计应用

在此部分中，你将创建应用程序的基本 UI 结构。

### <a name="implement-alert-extension-methods"></a>实现警报扩展方法

在此部分中，你将为控制器视图返回 `IActionResult` 的类型创建扩展方法。 此扩展将允许向视图传递临时错误或成功消息。

> [!TIP]
> 可以使用任何文本编辑器编辑本教程的源文件。 但是 [，Visual Studio代码](https://code.visualstudio.com/) 提供了其他功能，如调试和 Intellisense。

1. 在 **GraphTu一** l 目录中新建一个名为 **Alerts 的目录**。

1. 在 **./Alerts** **WithAlertResult.cs** 创建一个名为 WithAlertResult.cs 的新文件，并添加以下代码。

    :::code language="csharp" source="../demo/GraphTutorial/Alerts/WithAlertResult.cs" id="WithAlertResultSnippet":::

1. 在 ./Alerts **AlertExtensions.cs** 创建一个名为 **AlertExtensions.cs** 的新文件，并添加以下代码。

    :::code language="csharp" source="../demo/GraphTutorial/Alerts/AlertExtensions.cs" id="AlertExtensionsSnippet":::

### <a name="implement-user-data-extension-methods"></a>实现用户数据扩展方法

在此部分中，你将为 Microsoft Identity 平台生成的对象创建 `ClaimsPrincipal` 扩展方法。 这将允许你使用 Microsoft Graph 数据扩展现有用户标识。

> [!NOTE]
> 此代码目前只是一个占位符，你将在稍后部分中完成它。

1. 在 **GraphTu一** l 目录中新建一个名为 Graph 的 **目录**。

1. 创建一个名为 **GraphClaimsPrincipalExtensions.cs** 的新文件，并添加以下代码。

    ```csharp
    using System.Security.Claims;

    namespace GraphTutorial
    {
        public static class GraphClaimTypes {
            public const string DisplayName ="graph_name";
            public const string Email = "graph_email";
            public const string Photo = "graph_photo";
            public const string TimeZone = "graph_timezone";
            public const string DateTimeFormat = "graph_datetimeformat";
        }

        // Helper methods to access Graph user data stored in
        // the claims principal
        public static class GraphClaimsPrincipalExtensions
        {
            public static string GetUserGraphDisplayName(this ClaimsPrincipal claimsPrincipal)
            {
                return "Adele Vance";
            }

            public static string GetUserGraphEmail(this ClaimsPrincipal claimsPrincipal)
            {
                return "adelev@contoso.com";
            }

            public static string GetUserGraphPhoto(this ClaimsPrincipal claimsPrincipal)
            {
                return "/img/no-profile-photo.png";
            }
        }
    }
    ```

### <a name="create-views"></a>创建视图

在此部分中，你将实现应用程序的"一览"视图。

1. 在 **./Views/Shared** 目录中添加 **一个名为 _LoginPartial.cshtml** 的新文件，并添加以下代码。

    :::code language="cshtml" source="../demo/GraphTutorial/Views/Shared/_LoginPartial.cshtml" id="LoginPartialSnippet":::

1. 在 **./Views/Shared** 目录中添加一个名为 **_AlertPartial.cshtml** 的新文件，并添加以下代码。

    :::code language="cshtml" source="../demo/GraphTutorial/Views/Shared/_AlertPartial.cshtml" id="AlertPartialSnippet":::

1. 打开 **./Views/Shared/_Layout.cshtml** 文件，并将其全部内容替换为以下代码以更新应用程序的全局布局。

    :::code language="cshtml" source="../demo/GraphTutorial/Views/Shared/_Layout.cshtml" id="LayoutSnippet":::

1. 打开 **./wwwroot/css/site.css，** 在文件底部添加以下代码。

    :::code language="css" source="../demo/GraphTutorial/wwwroot/css/site.css" id="CssSnippet":::

1. 打开 **./Views/Home/index.cshtml** 文件，并将其内容替换为以下内容。

    :::code language="cshtml" source="../demo/GraphTutorial/Views/Home/Index.cshtml" id="HomeIndexSnippet":::

1. 在名为 **img** 的 **./wwwroot** 目录中创建新目录。 在此目录中添加你选择的名为 **no-profile-photo.png的图像** 文件。 当用户在 Microsoft Graph 中没有照片时，此图像将用作用户的照片。

    > [!TIP]
    > 可以从 [GitHub](https://github.com/microsoftgraph/msgraph-training-aspnet-core/blob/master/demo/GraphTutorial/wwwroot/img/no-profile-photo.png)下载这些屏幕截图中使用的图像。

1. 保存所有更改并重新启动服务器 `dotnet run` () 。 现在，应用看起来应该非常不同。

    ![重新设计主页的屏幕截图](./images/create-app-01.png)
