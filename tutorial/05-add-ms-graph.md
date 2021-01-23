---
ms.openlocfilehash: c954903f38d48bcca4c534f4d0cfbe1605cbf7f6
ms.sourcegitcommit: 6341ad07cd5b03269e7fd20cd3212e48baee7c07
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/23/2021
ms.locfileid: "49942150"
---
<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="f0cd2-101">В этом разделе вы включаете Microsoft Graph в приложение.</span><span class="sxs-lookup"><span data-stu-id="f0cd2-101">In this section you will incorporate Microsoft Graph into the application.</span></span> <span data-ttu-id="f0cd2-102">Для этого приложения вы будете использовать клиентскую библиотеку Microsoft Graph для [.NET](https://github.com/microsoftgraph/msgraph-sdk-dotnet) для вызова Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="f0cd2-102">For this application, you will use the [Microsoft Graph Client Library for .NET](https://github.com/microsoftgraph/msgraph-sdk-dotnet) to make calls to Microsoft Graph.</span></span>

## <a name="get-calendar-events-from-outlook"></a><span data-ttu-id="f0cd2-103">Получить события календаря из Outlook</span><span class="sxs-lookup"><span data-stu-id="f0cd2-103">Get calendar events from Outlook</span></span>

<span data-ttu-id="f0cd2-104">Начните с создания нового контроллера для представлений календаря.</span><span class="sxs-lookup"><span data-stu-id="f0cd2-104">Start by creating a new controller for calendar views.</span></span>

1. <span data-ttu-id="f0cd2-105">Добавьте новый файл с **именем CalendarController.cs** **в каталог ./Controllers** и добавьте следующий код.</span><span class="sxs-lookup"><span data-stu-id="f0cd2-105">Add a new file named **CalendarController.cs** in the **./Controllers** directory and add the following code.</span></span>

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

1. <span data-ttu-id="f0cd2-106">Добавьте в класс следующие функции, чтобы получить представление `CalendarController` календаря пользователя.</span><span class="sxs-lookup"><span data-stu-id="f0cd2-106">Add the following functions to the `CalendarController` class to get the user's calendar view.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Controllers/CalendarController.cs" id="GetCalendarViewSnippet":::

    <span data-ttu-id="f0cd2-107">Подумайте, что делает `GetUserWeekCalendar` код.</span><span class="sxs-lookup"><span data-stu-id="f0cd2-107">Consider what the code in `GetUserWeekCalendar` does.</span></span>

    - <span data-ttu-id="f0cd2-108">Он использует часовой пояс пользователя для получения значений даты и времени начала и окончания в UTC за неделю.</span><span class="sxs-lookup"><span data-stu-id="f0cd2-108">It uses the user's time zone to get UTC start and end date/time values for the week.</span></span>
    - <span data-ttu-id="f0cd2-109">Он запрашивает представление [](/graph/api/calendar-list-calendarview?view=graph-rest-1.0) календаря пользователя, чтобы получить все события, которые попадают между датой и временем начала и окончания.</span><span class="sxs-lookup"><span data-stu-id="f0cd2-109">It queries the user's [calendar view](/graph/api/calendar-list-calendarview?view=graph-rest-1.0) to get all events that fall between the start and end date/times.</span></span> <span data-ttu-id="f0cd2-110">Использование представления календаря [](/graph/api/user-list-events?view=graph-rest-1.0) вместо перечисления событий расширяет повторяющиеся события, возвращая все вхождения, которые происходят в указанном окне времени.</span><span class="sxs-lookup"><span data-stu-id="f0cd2-110">Using a calendar view instead of [listing events](/graph/api/user-list-events?view=graph-rest-1.0) expands recurring events, returning any occurrences that happen in the specified time window.</span></span>
    - <span data-ttu-id="f0cd2-111">Он использует `Prefer: outlook.timezone` заголок для получения результатов в часовом поясе пользователя.</span><span class="sxs-lookup"><span data-stu-id="f0cd2-111">It uses the `Prefer: outlook.timezone` header to get results back in the user's timezone.</span></span>
    - <span data-ttu-id="f0cd2-112">Он используется для ограничения полей, возвращаемой только теми, `Select` которые используются приложением.</span><span class="sxs-lookup"><span data-stu-id="f0cd2-112">It uses `Select` to limit the fields that come back to just those used by the app.</span></span>
    - <span data-ttu-id="f0cd2-113">Используется для `OrderBy` сортировки результатов в хронологическом порядке.</span><span class="sxs-lookup"><span data-stu-id="f0cd2-113">It uses `OrderBy` to sort the results chronologically.</span></span>
    - <span data-ttu-id="f0cd2-114">Он использует `PageIterator` страницу [страницы через коллекцию событий.](/graph/sdks/paging)</span><span class="sxs-lookup"><span data-stu-id="f0cd2-114">It uses a `PageIterator` to [page through the events collection](/graph/sdks/paging).</span></span> <span data-ttu-id="f0cd2-115">Это обрабатывает тот случай, когда в календаре пользователя больше событий, чем в запрашиваемом размере страницы.</span><span class="sxs-lookup"><span data-stu-id="f0cd2-115">This handles the case where the user has more events on their calendar than the requested page size.</span></span>

1. <span data-ttu-id="f0cd2-116">Добавьте в класс следующую `CalendarController` функцию, чтобы реализовать временное представление возвращенных данных.</span><span class="sxs-lookup"><span data-stu-id="f0cd2-116">Add the following function to the `CalendarController` class to implement a temporary view of the returned data.</span></span>

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

1. <span data-ttu-id="f0cd2-117">Запустите приложение, войдите и щелкните ссылку **"Календарь"** на панели nav.</span><span class="sxs-lookup"><span data-stu-id="f0cd2-117">Start the app, sign in, and click the **Calendar** link in the nav bar.</span></span> <span data-ttu-id="f0cd2-118">Если все работает, в календаре пользователя должен быть дамп событий JSON.</span><span class="sxs-lookup"><span data-stu-id="f0cd2-118">If everything works, you should see a JSON dump of events on the user's calendar.</span></span>

## <a name="display-the-results"></a><span data-ttu-id="f0cd2-119">Отображение результатов</span><span class="sxs-lookup"><span data-stu-id="f0cd2-119">Display the results</span></span>

<span data-ttu-id="f0cd2-120">Теперь можно добавить представление для более удобного отображения результатов.</span><span class="sxs-lookup"><span data-stu-id="f0cd2-120">Now you can add a view to display the results in a more user-friendly manner.</span></span>

### <a name="create-view-models"></a><span data-ttu-id="f0cd2-121">Создание моделей представления</span><span class="sxs-lookup"><span data-stu-id="f0cd2-121">Create view models</span></span>

1. <span data-ttu-id="f0cd2-122">Создайте файл с **именем CalendarViewEvent.cs** **в каталоге ./Models** и добавьте следующий код.</span><span class="sxs-lookup"><span data-stu-id="f0cd2-122">Create a new file named **CalendarViewEvent.cs** in the **./Models** directory and add the following code.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Models/CalendarViewEvent.cs" id="CalendarViewEventSnippet":::

1. <span data-ttu-id="f0cd2-123">Создайте файл с **именем DailyViewModel.cs** **в каталоге ./Models** и добавьте следующий код.</span><span class="sxs-lookup"><span data-stu-id="f0cd2-123">Create a new file named **DailyViewModel.cs** in the **./Models** directory and add the following code.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Models/DailyViewModel.cs" id="DailyViewModelSnippet":::

1. <span data-ttu-id="f0cd2-124">Создайте файл с **именем CalendarViewModel.cs** в **каталоге ./Models** и добавьте следующий код.</span><span class="sxs-lookup"><span data-stu-id="f0cd2-124">Create a new file named **CalendarViewModel.cs** in the **./Models** directory and add the following code.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Models/CalendarViewModel.cs" id="CalendarViewModelSnippet":::

### <a name="create-views"></a><span data-ttu-id="f0cd2-125">Создание представлений</span><span class="sxs-lookup"><span data-stu-id="f0cd2-125">Create views</span></span>

1. <span data-ttu-id="f0cd2-126">Создайте каталог с именем **Calendar** в **каталоге ./Views.**</span><span class="sxs-lookup"><span data-stu-id="f0cd2-126">Create a new directory named **Calendar** in the **./Views** directory.</span></span>

1. <span data-ttu-id="f0cd2-127">Создайте файл с именем **_DailyEventsPartial.cshtml** в **каталоге ./Views/Calendar** и добавьте следующий код.</span><span class="sxs-lookup"><span data-stu-id="f0cd2-127">Create a new file named **_DailyEventsPartial.cshtml** in the **./Views/Calendar** directory and add the following code.</span></span>

    :::code language="cshtml" source="../demo/GraphTutorial/Views/Calendar/_DailyEventsPartial.cshtml" id="DailyEventsPartialSnippet":::

1. <span data-ttu-id="f0cd2-128">Создайте файл **Index.cshtml** в **каталоге ./Views/Calendar** и добавьте следующий код.</span><span class="sxs-lookup"><span data-stu-id="f0cd2-128">Create a new file named **Index.cshtml** in the **./Views/Calendar** directory and add the following code.</span></span>

    :::code language="cshtml" source="../demo/GraphTutorial/Views/Calendar/Index.cshtml" id="CalendarIndexSnippet":::

### <a name="update-calendar-controller"></a><span data-ttu-id="f0cd2-129">Обновление контроллера календаря</span><span class="sxs-lookup"><span data-stu-id="f0cd2-129">Update calendar controller</span></span>

1. <span data-ttu-id="f0cd2-130">Откройте **./Controllers/CalendarController.cs** и замените существующую `Index` функцию на следующую.</span><span class="sxs-lookup"><span data-stu-id="f0cd2-130">Open **./Controllers/CalendarController.cs** and replace the existing `Index` function with the following.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Controllers/CalendarController.cs" id="IndexSnippet":::

1. <span data-ttu-id="f0cd2-131">Запустите приложение, войдите и щелкните ссылку **"Календарь".**</span><span class="sxs-lookup"><span data-stu-id="f0cd2-131">Start the app, sign in, and click the **Calendar** link.</span></span> <span data-ttu-id="f0cd2-132">Теперь приложение должно отрисовки таблицы событий.</span><span class="sxs-lookup"><span data-stu-id="f0cd2-132">The app should now render a table of events.</span></span>

    ![Снимок экрана с таблицей событий](./images/add-msgraph-01.png)
