---
ms.openlocfilehash: c954903f38d48bcca4c534f4d0cfbe1605cbf7f6
ms.sourcegitcommit: 6341ad07cd5b03269e7fd20cd3212e48baee7c07
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/23/2021
ms.locfileid: "49942150"
---
<!-- markdownlint-disable MD002 MD041 -->

В этом разделе вы включаете Microsoft Graph в приложение. Для этого приложения вы будете использовать клиентскую библиотеку Microsoft Graph для [.NET](https://github.com/microsoftgraph/msgraph-sdk-dotnet) для вызова Microsoft Graph.

## <a name="get-calendar-events-from-outlook"></a>Получить события календаря из Outlook

Начните с создания нового контроллера для представлений календаря.

1. Добавьте новый файл с **именем CalendarController.cs** **в каталог ./Controllers** и добавьте следующий код.

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

1. Добавьте в класс следующие функции, чтобы получить представление `CalendarController` календаря пользователя.

    :::code language="csharp" source="../demo/GraphTutorial/Controllers/CalendarController.cs" id="GetCalendarViewSnippet":::

    Подумайте, что делает `GetUserWeekCalendar` код.

    - Он использует часовой пояс пользователя для получения значений даты и времени начала и окончания в UTC за неделю.
    - Он запрашивает представление [](/graph/api/calendar-list-calendarview?view=graph-rest-1.0) календаря пользователя, чтобы получить все события, которые попадают между датой и временем начала и окончания. Использование представления календаря [](/graph/api/user-list-events?view=graph-rest-1.0) вместо перечисления событий расширяет повторяющиеся события, возвращая все вхождения, которые происходят в указанном окне времени.
    - Он использует `Prefer: outlook.timezone` заголок для получения результатов в часовом поясе пользователя.
    - Он используется для ограничения полей, возвращаемой только теми, `Select` которые используются приложением.
    - Используется для `OrderBy` сортировки результатов в хронологическом порядке.
    - Он использует `PageIterator` страницу [страницы через коллекцию событий.](/graph/sdks/paging) Это обрабатывает тот случай, когда в календаре пользователя больше событий, чем в запрашиваемом размере страницы.

1. Добавьте в класс следующую `CalendarController` функцию, чтобы реализовать временное представление возвращенных данных.

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

1. Запустите приложение, войдите и щелкните ссылку **"Календарь"** на панели nav. Если все работает, в календаре пользователя должен быть дамп событий JSON.

## <a name="display-the-results"></a>Отображение результатов

Теперь можно добавить представление для более удобного отображения результатов.

### <a name="create-view-models"></a>Создание моделей представления

1. Создайте файл с **именем CalendarViewEvent.cs** **в каталоге ./Models** и добавьте следующий код.

    :::code language="csharp" source="../demo/GraphTutorial/Models/CalendarViewEvent.cs" id="CalendarViewEventSnippet":::

1. Создайте файл с **именем DailyViewModel.cs** **в каталоге ./Models** и добавьте следующий код.

    :::code language="csharp" source="../demo/GraphTutorial/Models/DailyViewModel.cs" id="DailyViewModelSnippet":::

1. Создайте файл с **именем CalendarViewModel.cs** в **каталоге ./Models** и добавьте следующий код.

    :::code language="csharp" source="../demo/GraphTutorial/Models/CalendarViewModel.cs" id="CalendarViewModelSnippet":::

### <a name="create-views"></a>Создание представлений

1. Создайте каталог с именем **Calendar** в **каталоге ./Views.**

1. Создайте файл с именем **_DailyEventsPartial.cshtml** в **каталоге ./Views/Calendar** и добавьте следующий код.

    :::code language="cshtml" source="../demo/GraphTutorial/Views/Calendar/_DailyEventsPartial.cshtml" id="DailyEventsPartialSnippet":::

1. Создайте файл **Index.cshtml** в **каталоге ./Views/Calendar** и добавьте следующий код.

    :::code language="cshtml" source="../demo/GraphTutorial/Views/Calendar/Index.cshtml" id="CalendarIndexSnippet":::

### <a name="update-calendar-controller"></a>Обновление контроллера календаря

1. Откройте **./Controllers/CalendarController.cs** и замените существующую `Index` функцию на следующую.

    :::code language="csharp" source="../demo/GraphTutorial/Controllers/CalendarController.cs" id="IndexSnippet":::

1. Запустите приложение, войдите и щелкните ссылку **"Календарь".** Теперь приложение должно отрисовки таблицы событий.

    ![Снимок экрана с таблицей событий](./images/add-msgraph-01.png)
