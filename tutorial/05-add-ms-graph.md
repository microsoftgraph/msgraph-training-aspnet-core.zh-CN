---
ms.openlocfilehash: c954903f38d48bcca4c534f4d0cfbe1605cbf7f6
ms.sourcegitcommit: 6341ad07cd5b03269e7fd20cd3212e48baee7c07
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/23/2021
ms.locfileid: "49942146"
---
<!-- markdownlint-disable MD002 MD041 -->

在此部分中，你将 Microsoft Graph 合并到应用程序中。 对于此应用程序，你将使用 [适用于 .NET](https://github.com/microsoftgraph/msgraph-sdk-dotnet) 的 Microsoft Graph 客户端库调用 Microsoft Graph。

## <a name="get-calendar-events-from-outlook"></a>从 Outlook 获取日历事件

首先为日历视图创建新的控制器。

1. 在 **./Controllers** **CalendarController.cs** 中添加名为 CalendarController.cs 的新文件，并添加以下代码。

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

1. 将以下函数 `CalendarController` 添加到类，获取用户的日历视图。

    :::code language="csharp" source="../demo/GraphTutorial/Controllers/CalendarController.cs" id="GetCalendarViewSnippet":::

    考虑代码所 `GetUserWeekCalendar` 执行哪些功能。

    - 它使用用户的时区获取一周的 UTC 开始日期和结束日期/时间值。
    - 它查询用户的日历 [视图](/graph/api/calendar-list-calendarview?view=graph-rest-1.0) ，获取介于开始日期和结束日期/时间之间的所有事件。 使用日历视图而不是 [列出事件](/graph/api/user-list-events?view=graph-rest-1.0) 会扩展定期事件，返回指定时间窗口中发生的任何事件。
    - 它使用 `Prefer: outlook.timezone` 标头以用户时区返回结果。
    - 它 `Select` 用于将返回的字段限制为仅由应用使用的字段。
    - 它 `OrderBy` 用于按时间顺序对结果进行排序。
    - 它使用 `PageIterator` 对 [事件集合进行分页](/graph/sdks/paging)。 这将处理用户日历上的事件数多于请求的页面大小的情况。

1. 将以下函数 `CalendarController` 添加到类中，以实施返回的数据的临时视图。

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

1. 启动应用，登录，然后单击导航 **栏中** 的"日历"链接。 如果一切正常，应在用户日历上看到事件的 JSON 转储。

## <a name="display-the-results"></a>显示结果

现在，您可以添加视图以更用户友好的方式显示结果。

### <a name="create-view-models"></a>创建视图模型

1. 在 ./Models **CalendarViewEvent.cs** 创建一个名为 **CalendarViewEvent.cs** 的新文件，并添加以下代码。

    :::code language="csharp" source="../demo/GraphTutorial/Models/CalendarViewEvent.cs" id="CalendarViewEventSnippet":::

1. 在 ./Models **DailyViewModel.cs** 创建一个名为 **DailyViewModel.cs** 的新文件，并添加以下代码。

    :::code language="csharp" source="../demo/GraphTutorial/Models/DailyViewModel.cs" id="DailyViewModelSnippet":::

1. 在 ./Models **CalendarViewModel.cs** 创建一个名为 **CalendarViewModel.cs** 的新文件，并添加以下代码。

    :::code language="csharp" source="../demo/GraphTutorial/Models/CalendarViewModel.cs" id="CalendarViewModelSnippet":::

### <a name="create-views"></a>创建视图

1. 在 **./Views** 目录中新建一个名为 **Calendar** 的目录。

1. 在 **./Views/Calendar** 目录中创建一个名为 **_DailyEventsPartial.cshtml** 的新文件，并添加以下代码。

    :::code language="cshtml" source="../demo/GraphTutorial/Views/Calendar/_DailyEventsPartial.cshtml" id="DailyEventsPartialSnippet":::

1. 在 **./Views/Calendar** 目录中创建一个名为 **Index.cshtml** 的新文件，并添加以下代码。

    :::code language="cshtml" source="../demo/GraphTutorial/Views/Calendar/Index.cshtml" id="CalendarIndexSnippet":::

### <a name="update-calendar-controller"></a>更新日历控制器

1. 打开 **./Controllers/CalendarController.cs，** 将现有 `Index` 函数替换为以下内容。

    :::code language="csharp" source="../demo/GraphTutorial/Controllers/CalendarController.cs" id="IndexSnippet":::

1. 启动应用，登录，然后单击 **"日历"** 链接。 应用现在应呈现事件表。

    ![事件表的屏幕截图](./images/add-msgraph-01.png)
