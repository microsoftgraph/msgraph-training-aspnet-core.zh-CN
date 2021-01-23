---
ms.openlocfilehash: 5b1a776c28b6f9218c713dde68f45e571ebfd999
ms.sourcegitcommit: 6341ad07cd5b03269e7fd20cd3212e48baee7c07
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/23/2021
ms.locfileid: "49942153"
---
<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="0aa27-101">首先创建一个ASP.NET核心 Web 应用。</span><span class="sxs-lookup"><span data-stu-id="0aa27-101">Start by creating an ASP.NET Core web app.</span></span>

1. <span data-ttu-id="0aa27-102">在要创建项目的目录中 (CLI) 打开命令行界面。</span><span class="sxs-lookup"><span data-stu-id="0aa27-102">Open your command-line interface (CLI) in a directory where you want to create the project.</span></span> <span data-ttu-id="0aa27-103">运行以下命令。</span><span class="sxs-lookup"><span data-stu-id="0aa27-103">Run the following command.</span></span>

    ```Shell
    dotnet new mvc -o GraphTutorial
    ```

1. <span data-ttu-id="0aa27-104">创建项目后，通过更改当前目录到 **GraphTu且在** CLI 中运行以下命令来验证项目是否有效。</span><span class="sxs-lookup"><span data-stu-id="0aa27-104">Once the project is created, verify that it works by changing the current directory to the **GraphTutorial** directory and running the following command in your CLI.</span></span>

    ```Shell
    dotnet run
    ```

1. <span data-ttu-id="0aa27-105">打开浏览器并浏览到 `https://localhost:5001` 。</span><span class="sxs-lookup"><span data-stu-id="0aa27-105">Open your browser and browse to `https://localhost:5001`.</span></span> <span data-ttu-id="0aa27-106">如果一切正常，则应该会看到默认ASP.NET核心页面。</span><span class="sxs-lookup"><span data-stu-id="0aa27-106">If everything is working, you should see a default ASP.NET Core page.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="0aa27-107">如果您收到一条警告，指出 **localhost** 的证书不可信，您可以使用 .NET Core CLI 安装和信任开发证书。</span><span class="sxs-lookup"><span data-stu-id="0aa27-107">If you receive a warning that the certificate for **localhost** is un-trusted you can use the .NET Core CLI to install and trust the development certificate.</span></span> <span data-ttu-id="0aa27-108">有关 [特定操作系统的说明，请参阅 ASP.NET Core](/aspnet/core/security/enforcing-ssl?view=aspnetcore-5.0) 中的强制 HTTPS。</span><span class="sxs-lookup"><span data-stu-id="0aa27-108">See [Enforce HTTPS in ASP.NET Core](/aspnet/core/security/enforcing-ssl?view=aspnetcore-5.0) for instructions for specific operating systems.</span></span>

## <a name="add-nuget-packages"></a><span data-ttu-id="0aa27-109">添加 NuGet 程序包</span><span class="sxs-lookup"><span data-stu-id="0aa27-109">Add NuGet packages</span></span>

<span data-ttu-id="0aa27-110">在继续之前，请安装一些稍后将使用的其他 NuGet 程序包。</span><span class="sxs-lookup"><span data-stu-id="0aa27-110">Before moving on, install some additional NuGet packages that you will use later.</span></span>

- <span data-ttu-id="0aa27-111">[用于请求和管理访问令牌的 Microsoft.Identity.Web。](https://www.nuget.org/packages/Microsoft.Identity.Web/)</span><span class="sxs-lookup"><span data-stu-id="0aa27-111">[Microsoft.Identity.Web](https://www.nuget.org/packages/Microsoft.Identity.Web/) for requesting and managing access tokens.</span></span>
- <span data-ttu-id="0aa27-112">[Microsoft.Identity.Web.MicrosoftGraph，](https://www.nuget.org/packages/Microsoft.Identity.Web.MicrosoftGraph/) 用于通过依赖关系注入添加 Microsoft Graph SDK。</span><span class="sxs-lookup"><span data-stu-id="0aa27-112">[Microsoft.Identity.Web.MicrosoftGraph](https://www.nuget.org/packages/Microsoft.Identity.Web.MicrosoftGraph/) for adding the Microsoft Graph SDK via dependency injection.</span></span>
- <span data-ttu-id="0aa27-113">用于登录和注销 UI 的[Microsoft.Identity.Web.UI。](https://www.nuget.org/packages/Microsoft.Identity.Web.UI/)</span><span class="sxs-lookup"><span data-stu-id="0aa27-113">[Microsoft.Identity.Web.UI](https://www.nuget.org/packages/Microsoft.Identity.Web.UI/) for sign-in and sign-out UI.</span></span>
- <span data-ttu-id="0aa27-114">[用于跨平台处理时区标识符的 TimeZoneConverter。](https://github.com/mj1856/TimeZoneConverter)</span><span class="sxs-lookup"><span data-stu-id="0aa27-114">[TimeZoneConverter](https://github.com/mj1856/TimeZoneConverter) for handling time zoned identifiers cross-platform.</span></span>

1. <span data-ttu-id="0aa27-115">在 CLI 中运行以下命令以安装依赖项。</span><span class="sxs-lookup"><span data-stu-id="0aa27-115">Run the following commands in your CLI to install the dependencies.</span></span>

    ```Shell
    dotnet add package Microsoft.Identity.Web --version 1.5.1
    dotnet add package Microsoft.Identity.Web.MicrosoftGraph --version 1.5.1
    dotnet add package Microsoft.Identity.Web.UI --version 1.5.1
    dotnet add package TimeZoneConverter
    ```

## <a name="design-the-app"></a><span data-ttu-id="0aa27-116">设计应用</span><span class="sxs-lookup"><span data-stu-id="0aa27-116">Design the app</span></span>

<span data-ttu-id="0aa27-117">在此部分中，你将创建应用程序的基本 UI 结构。</span><span class="sxs-lookup"><span data-stu-id="0aa27-117">In this section you will create the basic UI structure of the application.</span></span>

### <a name="implement-alert-extension-methods"></a><span data-ttu-id="0aa27-118">实现警报扩展方法</span><span class="sxs-lookup"><span data-stu-id="0aa27-118">Implement alert extension methods</span></span>

<span data-ttu-id="0aa27-119">在此部分中，你将为控制器视图返回 `IActionResult` 的类型创建扩展方法。</span><span class="sxs-lookup"><span data-stu-id="0aa27-119">In this section you will create extension methods for the `IActionResult` type returned by controller views.</span></span> <span data-ttu-id="0aa27-120">此扩展将允许向视图传递临时错误或成功消息。</span><span class="sxs-lookup"><span data-stu-id="0aa27-120">This extension will enable passing temporary error or success messages to the view.</span></span>

> [!TIP]
> <span data-ttu-id="0aa27-121">可以使用任何文本编辑器编辑本教程的源文件。</span><span class="sxs-lookup"><span data-stu-id="0aa27-121">You can use any text editor to edit the source files for this tutorial.</span></span> <span data-ttu-id="0aa27-122">但是 [，Visual Studio代码](https://code.visualstudio.com/) 提供了其他功能，如调试和 Intellisense。</span><span class="sxs-lookup"><span data-stu-id="0aa27-122">However, [Visual Studio Code](https://code.visualstudio.com/) provides additional features, such as debugging and Intellisense.</span></span>

1. <span data-ttu-id="0aa27-123">在 **GraphTu一** l 目录中新建一个名为 **Alerts 的目录**。</span><span class="sxs-lookup"><span data-stu-id="0aa27-123">Create a new directory in the **GraphTutorial** directory named **Alerts**.</span></span>

1. <span data-ttu-id="0aa27-124">在 **./Alerts** **WithAlertResult.cs** 创建一个名为 WithAlertResult.cs 的新文件，并添加以下代码。</span><span class="sxs-lookup"><span data-stu-id="0aa27-124">Create a new file named **WithAlertResult.cs** in the **./Alerts** directory and add the following code.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Alerts/WithAlertResult.cs" id="WithAlertResultSnippet":::

1. <span data-ttu-id="0aa27-125">在 ./Alerts **AlertExtensions.cs** 创建一个名为 **AlertExtensions.cs** 的新文件，并添加以下代码。</span><span class="sxs-lookup"><span data-stu-id="0aa27-125">Create a new file named **AlertExtensions.cs** in the **./Alerts** directory and add the following code.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Alerts/AlertExtensions.cs" id="AlertExtensionsSnippet":::

### <a name="implement-user-data-extension-methods"></a><span data-ttu-id="0aa27-126">实现用户数据扩展方法</span><span class="sxs-lookup"><span data-stu-id="0aa27-126">Implement user data extension methods</span></span>

<span data-ttu-id="0aa27-127">在此部分中，你将为 Microsoft Identity 平台生成的对象创建 `ClaimsPrincipal` 扩展方法。</span><span class="sxs-lookup"><span data-stu-id="0aa27-127">In this section you will create extension methods for the `ClaimsPrincipal` object generated by the Microsoft Identity platform.</span></span> <span data-ttu-id="0aa27-128">这将允许你使用 Microsoft Graph 数据扩展现有用户标识。</span><span class="sxs-lookup"><span data-stu-id="0aa27-128">This will allow you to extend the existing user identity with data from Microsoft Graph.</span></span>

> [!NOTE]
> <span data-ttu-id="0aa27-129">此代码目前只是一个占位符，你将在稍后部分中完成它。</span><span class="sxs-lookup"><span data-stu-id="0aa27-129">This code is just a placeholder for now, you will complete it in a later section.</span></span>

1. <span data-ttu-id="0aa27-130">在 **GraphTu一** l 目录中新建一个名为 Graph 的 **目录**。</span><span class="sxs-lookup"><span data-stu-id="0aa27-130">Create a new directory in the **GraphTutorial** directory named **Graph**.</span></span>

1. <span data-ttu-id="0aa27-131">创建一个名为 **GraphClaimsPrincipalExtensions.cs** 的新文件，并添加以下代码。</span><span class="sxs-lookup"><span data-stu-id="0aa27-131">Create a new file named **GraphClaimsPrincipalExtensions.cs** and add the following code.</span></span>

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

### <a name="create-views"></a><span data-ttu-id="0aa27-132">创建视图</span><span class="sxs-lookup"><span data-stu-id="0aa27-132">Create views</span></span>

<span data-ttu-id="0aa27-133">在此部分中，你将实现应用程序的"一览"视图。</span><span class="sxs-lookup"><span data-stu-id="0aa27-133">In this section you will implement the Razor views for the application.</span></span>

1. <span data-ttu-id="0aa27-134">在 **./Views/Shared** 目录中添加 **一个名为 _LoginPartial.cshtml** 的新文件，并添加以下代码。</span><span class="sxs-lookup"><span data-stu-id="0aa27-134">Add a new file named **_LoginPartial.cshtml** in the **./Views/Shared** directory and add the following code.</span></span>

    :::code language="cshtml" source="../demo/GraphTutorial/Views/Shared/_LoginPartial.cshtml" id="LoginPartialSnippet":::

1. <span data-ttu-id="0aa27-135">在 **./Views/Shared** 目录中添加一个名为 **_AlertPartial.cshtml** 的新文件，并添加以下代码。</span><span class="sxs-lookup"><span data-stu-id="0aa27-135">Add a new file named **_AlertPartial.cshtml** in the **./Views/Shared** directory and add the following code.</span></span>

    :::code language="cshtml" source="../demo/GraphTutorial/Views/Shared/_AlertPartial.cshtml" id="AlertPartialSnippet":::

1. <span data-ttu-id="0aa27-136">打开 **./Views/Shared/_Layout.cshtml** 文件，并将其全部内容替换为以下代码以更新应用程序的全局布局。</span><span class="sxs-lookup"><span data-stu-id="0aa27-136">Open the **./Views/Shared/_Layout.cshtml** file, and replace its entire contents with the following code to update the global layout of the app.</span></span>

    :::code language="cshtml" source="../demo/GraphTutorial/Views/Shared/_Layout.cshtml" id="LayoutSnippet":::

1. <span data-ttu-id="0aa27-137">打开 **./wwwroot/css/site.css，** 在文件底部添加以下代码。</span><span class="sxs-lookup"><span data-stu-id="0aa27-137">Open **./wwwroot/css/site.css** and add the following code at the bottom of the file.</span></span>

    :::code language="css" source="../demo/GraphTutorial/wwwroot/css/site.css" id="CssSnippet":::

1. <span data-ttu-id="0aa27-138">打开 **./Views/Home/index.cshtml** 文件，并将其内容替换为以下内容。</span><span class="sxs-lookup"><span data-stu-id="0aa27-138">Open the **./Views/Home/index.cshtml** file and replace its contents with the following.</span></span>

    :::code language="cshtml" source="../demo/GraphTutorial/Views/Home/Index.cshtml" id="HomeIndexSnippet":::

1. <span data-ttu-id="0aa27-139">在名为 **img** 的 **./wwwroot** 目录中创建新目录。</span><span class="sxs-lookup"><span data-stu-id="0aa27-139">Create a new directory in the **./wwwroot** directory named **img**.</span></span> <span data-ttu-id="0aa27-140">在此目录中添加你选择的名为 **no-profile-photo.png的图像** 文件。</span><span class="sxs-lookup"><span data-stu-id="0aa27-140">Add an image file of your choosing named **no-profile-photo.png** in this directory.</span></span> <span data-ttu-id="0aa27-141">当用户在 Microsoft Graph 中没有照片时，此图像将用作用户的照片。</span><span class="sxs-lookup"><span data-stu-id="0aa27-141">This image will be used as the user's photo when the user has no photo in Microsoft Graph.</span></span>

    > [!TIP]
    > <span data-ttu-id="0aa27-142">可以从 [GitHub](https://github.com/microsoftgraph/msgraph-training-aspnet-core/blob/master/demo/GraphTutorial/wwwroot/img/no-profile-photo.png)下载这些屏幕截图中使用的图像。</span><span class="sxs-lookup"><span data-stu-id="0aa27-142">You can download the image used in these screenshots from [GitHub](https://github.com/microsoftgraph/msgraph-training-aspnet-core/blob/master/demo/GraphTutorial/wwwroot/img/no-profile-photo.png).</span></span>

1. <span data-ttu-id="0aa27-143">保存所有更改并重新启动服务器 `dotnet run` () 。</span><span class="sxs-lookup"><span data-stu-id="0aa27-143">Save all of your changes and restart the server (`dotnet run`).</span></span> <span data-ttu-id="0aa27-144">现在，应用看起来应该非常不同。</span><span class="sxs-lookup"><span data-stu-id="0aa27-144">Now, the app should look very different.</span></span>

    ![重新设计主页的屏幕截图](./images/create-app-01.png)
