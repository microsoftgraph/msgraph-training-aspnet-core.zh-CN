---
ms.openlocfilehash: c954903f38d48bcca4c534f4d0cfbe1605cbf7f6
ms.sourcegitcommit: 6341ad07cd5b03269e7fd20cd3212e48baee7c07
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/23/2021
ms.locfileid: "49942146"
---
<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="bd3c1-101">在此部分中，你将 Microsoft Graph 合并到应用程序中。</span><span class="sxs-lookup"><span data-stu-id="bd3c1-101">In this section you will incorporate Microsoft Graph into the application.</span></span> <span data-ttu-id="bd3c1-102">对于此应用程序，你将使用 [适用于 .NET](https://github.com/microsoftgraph/msgraph-sdk-dotnet) 的 Microsoft Graph 客户端库调用 Microsoft Graph。</span><span class="sxs-lookup"><span data-stu-id="bd3c1-102">For this application, you will use the [Microsoft Graph Client Library for .NET](https://github.com/microsoftgraph/msgraph-sdk-dotnet) to make calls to Microsoft Graph.</span></span>

## <a name="get-calendar-events-from-outlook"></a><span data-ttu-id="bd3c1-103">从 Outlook 获取日历事件</span><span class="sxs-lookup"><span data-stu-id="bd3c1-103">Get calendar events from Outlook</span></span>

<span data-ttu-id="bd3c1-104">首先为日历视图创建新的控制器。</span><span class="sxs-lookup"><span data-stu-id="bd3c1-104">Start by creating a new controller for calendar views.</span></span>

1. <span data-ttu-id="bd3c1-105">在 **./Controllers** **CalendarController.cs** 中添加名为 CalendarController.cs 的新文件，并添加以下代码。</span><span class="sxs-lookup"><span data-stu-id="bd3c1-105">Add a new file named **CalendarController.cs** in the **./Controllers** directory and add the following code.</span></span>

    ```csharp
    using GraphTutorial.Models;
    using Microsoft.AspNetCore.Mvc;
    using Microsoft.Extensions.Logging;
    using Microsoft.Identity.Web;
    using Microsoft.Graph;
    using System;
    using System.Collections.Generic;
    using System.Threading.Tasks;
    using TimeZoneConverter;

    namespace GraphTutorial.Controllers
    {
        public class CalendarController : Controller
        {
            private readonly GraphServiceClient _graphClient;
            private readonly ILogger<HomeController> _logger;

            public CalendarController(
                GraphServiceClient graphClient,
                ILogger<HomeController> logger)
            {
                _graphClient = graphClient;
                _logger = logger;
            }
        }
    }
    ```

1. <span data-ttu-id="bd3c1-106">将以下函数 `CalendarController` 添加到类，获取用户的日历视图。</span><span class="sxs-lookup"><span data-stu-id="bd3c1-106">Add the following functions to the `CalendarController` class to get the user's calendar view.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Controllers/CalendarController.cs" id="GetCalendarViewSnippet":::

    <span data-ttu-id="bd3c1-107">考虑代码所 `GetUserWeekCalendar` 执行哪些功能。</span><span class="sxs-lookup"><span data-stu-id="bd3c1-107">Consider what the code in `GetUserWeekCalendar` does.</span></span>

    - <span data-ttu-id="bd3c1-108">它使用用户的时区获取一周的 UTC 开始日期和结束日期/时间值。</span><span class="sxs-lookup"><span data-stu-id="bd3c1-108">It uses the user's time zone to get UTC start and end date/time values for the week.</span></span>
    - <span data-ttu-id="bd3c1-109">它查询用户的日历 [视图](/graph/api/calendar-list-calendarview?view=graph-rest-1.0) ，获取介于开始日期和结束日期/时间之间的所有事件。</span><span class="sxs-lookup"><span data-stu-id="bd3c1-109">It queries the user's [calendar view](/graph/api/calendar-list-calendarview?view=graph-rest-1.0) to get all events that fall between the start and end date/times.</span></span> <span data-ttu-id="bd3c1-110">使用日历视图而不是 [列出事件](/graph/api/user-list-events?view=graph-rest-1.0) 会扩展定期事件，返回指定时间窗口中发生的任何事件。</span><span class="sxs-lookup"><span data-stu-id="bd3c1-110">Using a calendar view instead of [listing events](/graph/api/user-list-events?view=graph-rest-1.0) expands recurring events, returning any occurrences that happen in the specified time window.</span></span>
    - <span data-ttu-id="bd3c1-111">它使用 `Prefer: outlook.timezone` 标头以用户时区返回结果。</span><span class="sxs-lookup"><span data-stu-id="bd3c1-111">It uses the `Prefer: outlook.timezone` header to get results back in the user's timezone.</span></span>
    - <span data-ttu-id="bd3c1-112">它 `Select` 用于将返回的字段限制为仅由应用使用的字段。</span><span class="sxs-lookup"><span data-stu-id="bd3c1-112">It uses `Select` to limit the fields that come back to just those used by the app.</span></span>
    - <span data-ttu-id="bd3c1-113">它 `OrderBy` 用于按时间顺序对结果进行排序。</span><span class="sxs-lookup"><span data-stu-id="bd3c1-113">It uses `OrderBy` to sort the results chronologically.</span></span>
    - <span data-ttu-id="bd3c1-114">它使用 `PageIterator` 对 [事件集合进行分页](/graph/sdks/paging)。</span><span class="sxs-lookup"><span data-stu-id="bd3c1-114">It uses a `PageIterator` to [page through the events collection](/graph/sdks/paging).</span></span> <span data-ttu-id="bd3c1-115">这将处理用户日历上的事件数多于请求的页面大小的情况。</span><span class="sxs-lookup"><span data-stu-id="bd3c1-115">This handles the case where the user has more events on their calendar than the requested page size.</span></span>

1. <span data-ttu-id="bd3c1-116">将以下函数 `CalendarController` 添加到类中，以实施返回的数据的临时视图。</span><span class="sxs-lookup"><span data-stu-id="bd3c1-116">Add the following function to the `CalendarController` class to implement a temporary view of the returned data.</span></span>

    ```csharp
    // Minimum permission scope needed for this view
    [AuthorizeForScopes(Scopes = new[] { "Calendars.Read" })]
    public async Task<IActionResult> Index()
    {
        try
        {
            var userTimeZone = TZConvert.GetTimeZoneInfo(
                User.GetUserGraphTimeZone());
            var startOfWeek = CalendarController.GetUtcStartOfWeekInTimeZone(
                DateTime.Today, userTimeZone);

            var events = await GetUserWeekCalendar(startOfWeek);

            // Return a JSON dump of events
            return new ContentResult {
                Content = _graphClient.HttpProvider.Serializer.SerializeObject(events),
                ContentType = "application/json"
            };
        }
        catch (ServiceException ex)
        {
            if (ex.InnerException is MicrosoftIdentityWebChallengeUserException)
            {
                throw;
            }

            return new ContentResult {
                Content = $"Error getting calendar view: {ex.Message}",
                ContentType = "text/plain"
            };
        }
    }
    ```

1. <span data-ttu-id="bd3c1-117">启动应用，登录，然后单击导航 **栏中** 的"日历"链接。</span><span class="sxs-lookup"><span data-stu-id="bd3c1-117">Start the app, sign in, and click the **Calendar** link in the nav bar.</span></span> <span data-ttu-id="bd3c1-118">如果一切正常，应在用户日历上看到事件的 JSON 转储。</span><span class="sxs-lookup"><span data-stu-id="bd3c1-118">If everything works, you should see a JSON dump of events on the user's calendar.</span></span>

## <a name="display-the-results"></a><span data-ttu-id="bd3c1-119">显示结果</span><span class="sxs-lookup"><span data-stu-id="bd3c1-119">Display the results</span></span>

<span data-ttu-id="bd3c1-120">现在，您可以添加视图以更用户友好的方式显示结果。</span><span class="sxs-lookup"><span data-stu-id="bd3c1-120">Now you can add a view to display the results in a more user-friendly manner.</span></span>

### <a name="create-view-models"></a><span data-ttu-id="bd3c1-121">创建视图模型</span><span class="sxs-lookup"><span data-stu-id="bd3c1-121">Create view models</span></span>

1. <span data-ttu-id="bd3c1-122">在 ./Models **CalendarViewEvent.cs** 创建一个名为 **CalendarViewEvent.cs** 的新文件，并添加以下代码。</span><span class="sxs-lookup"><span data-stu-id="bd3c1-122">Create a new file named **CalendarViewEvent.cs** in the **./Models** directory and add the following code.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Models/CalendarViewEvent.cs" id="CalendarViewEventSnippet":::

1. <span data-ttu-id="bd3c1-123">在 ./Models **DailyViewModel.cs** 创建一个名为 **DailyViewModel.cs** 的新文件，并添加以下代码。</span><span class="sxs-lookup"><span data-stu-id="bd3c1-123">Create a new file named **DailyViewModel.cs** in the **./Models** directory and add the following code.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Models/DailyViewModel.cs" id="DailyViewModelSnippet":::

1. <span data-ttu-id="bd3c1-124">在 ./Models **CalendarViewModel.cs** 创建一个名为 **CalendarViewModel.cs** 的新文件，并添加以下代码。</span><span class="sxs-lookup"><span data-stu-id="bd3c1-124">Create a new file named **CalendarViewModel.cs** in the **./Models** directory and add the following code.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Models/CalendarViewModel.cs" id="CalendarViewModelSnippet":::

### <a name="create-views"></a><span data-ttu-id="bd3c1-125">创建视图</span><span class="sxs-lookup"><span data-stu-id="bd3c1-125">Create views</span></span>

1. <span data-ttu-id="bd3c1-126">在 **./Views** 目录中新建一个名为 **Calendar** 的目录。</span><span class="sxs-lookup"><span data-stu-id="bd3c1-126">Create a new directory named **Calendar** in the **./Views** directory.</span></span>

1. <span data-ttu-id="bd3c1-127">在 **./Views/Calendar** 目录中创建一个名为 **_DailyEventsPartial.cshtml** 的新文件，并添加以下代码。</span><span class="sxs-lookup"><span data-stu-id="bd3c1-127">Create a new file named **_DailyEventsPartial.cshtml** in the **./Views/Calendar** directory and add the following code.</span></span>

    :::code language="cshtml" source="../demo/GraphTutorial/Views/Calendar/_DailyEventsPartial.cshtml" id="DailyEventsPartialSnippet":::

1. <span data-ttu-id="bd3c1-128">在 **./Views/Calendar** 目录中创建一个名为 **Index.cshtml** 的新文件，并添加以下代码。</span><span class="sxs-lookup"><span data-stu-id="bd3c1-128">Create a new file named **Index.cshtml** in the **./Views/Calendar** directory and add the following code.</span></span>

    :::code language="cshtml" source="../demo/GraphTutorial/Views/Calendar/Index.cshtml" id="CalendarIndexSnippet":::

### <a name="update-calendar-controller"></a><span data-ttu-id="bd3c1-129">更新日历控制器</span><span class="sxs-lookup"><span data-stu-id="bd3c1-129">Update calendar controller</span></span>

1. <span data-ttu-id="bd3c1-130">打开 **./Controllers/CalendarController.cs，** 将现有 `Index` 函数替换为以下内容。</span><span class="sxs-lookup"><span data-stu-id="bd3c1-130">Open **./Controllers/CalendarController.cs** and replace the existing `Index` function with the following.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Controllers/CalendarController.cs" id="IndexSnippet":::

1. <span data-ttu-id="bd3c1-131">启动应用，登录，然后单击 **"日历"** 链接。</span><span class="sxs-lookup"><span data-stu-id="bd3c1-131">Start the app, sign in, and click the **Calendar** link.</span></span> <span data-ttu-id="bd3c1-132">应用现在应呈现事件表。</span><span class="sxs-lookup"><span data-stu-id="bd3c1-132">The app should now render a table of events.</span></span>

    ![事件表的屏幕截图](./images/add-msgraph-01.png)
