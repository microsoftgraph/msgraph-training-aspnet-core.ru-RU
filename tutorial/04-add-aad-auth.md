---
ms.openlocfilehash: a024fb533c552563da6c9179301e16a2e1d09d5f
ms.sourcegitcommit: 6341ad07cd5b03269e7fd20cd3212e48baee7c07
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/23/2021
ms.locfileid: "49942164"
---
<!-- markdownlint-disable MD002 MD041 -->

В этом упражнении вы расширим приложение из предыдущего упражнения, чтобы поддерживать проверку подлинности с помощью Azure AD. Это необходимо для получения необходимого маркера доступа OAuth для вызова API Microsoft Graph. На этом этапе вы настроим [библиотеку Microsoft.Identity.Web.](https://www.nuget.org/packages/Microsoft.Identity.Web/)

> [!IMPORTANT]
> Чтобы не хранить код и секрет приложения в источнике, для хранения этих значений используется диспетчер секретов [.NET.](/aspnet/core/security/app-secrets) Диспетчер секретов предназначен только для разработки, и в производственных приложениях для хранения секретов следует использовать доверенного диспетчера секретов.

1. Откройте **./appsettings.jsи** замените его содержимое на следующее.

    :::code language="json" source="../demo/GraphTutorial/appsettings.json" highlight="2-6":::

1. Откройте CLI в каталоге, в котором расположен **GraphTutorial.csproj,** и запустите следующие команды, заменив его на ИД приложения на портале Azure и с помощью секрета `YOUR_APP_ID` `YOUR_APP_SECRET` приложения.

    ```Shell
    dotnet user-secrets init
    dotnet user-secrets set "AzureAd:ClientId" "YOUR_APP_ID"
    dotnet user-secrets set "AzureAd:ClientSecret" "YOUR_APP_SECRET"
    ```

## <a name="implement-sign-in"></a>Реализация входов

Для начала добавьте службы платформы удостоверений Майкрософт в приложение.

1. Создайте файл с **именем GraphConstants.cs** **в каталоге ./Graph** и добавьте следующий код.

    :::code language="csharp" source="../demo/GraphTutorial/Graph/GraphConstants.cs" id="GraphConstantsSnippet":::

1. Откройте файл **./Startup.cs** и добавьте следующие утверждения в `using` верхнюю часть файла.

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

1. Замените имеющуюся функцию `ConfigureServices` указанным ниже кодом.

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

1. В `Configure` функции добавьте следующую строку над `app.UseAuthorization();` строкой.

    ```csharp
    app.UseAuthentication();
    ```

1. Откройте **./Controllers/HomeController.cs** и замените его содержимое следующим:

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

1. Сохраните изменения и запустите проект. Войдите с помощью учетной записи Майкрософт.

1. Проверьте запрос согласия. Список разрешений соответствует списку областей разрешений, настроенных в **./Graph/GraphConstants.cs.**

    - **Поддержив доступ** к данным, к ним предоставлен доступ: ( ) Это разрешение запрашивается MSAL для получения `offline_access` маркеров обновления.
    - **Во sign you in and read your profile:** ( `User.Read` ) This permission allows the app to get the logged-in user's profile and profile photo.
    - **Прочитайте параметры** почтового ящика: ( ) Это разрешение позволяет приложению читать параметры почтового ящика пользователя, включая часовой пояс `MailboxSettings.Read` и формат времени.
    - **Полный доступ к** календарям: ( ) Это разрешение позволяет приложению читать события в календаре пользователя, добавлять новые события и изменять `Calendars.ReadWrite` существующие.

    ![Снимок экрана: запрос на согласие платформы удостоверений Майкрософт](./images/add-aad-auth-03.png)

    Дополнительные сведения о согласии см. в сведениях о предоставлении согласия [приложениям Azure AD.](/azure/active-directory/develop/application-consent-experience)

1. Согласие на запрашиваемую разрешения. Браузер перенаправляет пользователя в приложение с отображением маркера.

### <a name="get-user-details"></a>Получить сведения о пользователе

После входа пользователя вы можете получить его сведения из Microsoft Graph.

1. Откройте **./Graph/GraphClaimsPrincipalExtensions.cs** и замените все его содержимое следующим:

    :::code language="csharp" source="../demo/GraphTutorial/Graph/GraphClaimsPrincipalExtensions.cs" id="GraphClaimsExtensionsSnippet":::

1. Откройте **./Startup.cs** и замените существующую `.AddMicrosoftIdentityWebApp(Configuration)` строку следующим кодом.

    :::code language="csharp" source="../demo/GraphTutorial/Startup.cs" id="AddSignInSnippet":::

    Подумайте, что делает этот код.

    - Он добавляет обработок `OnTokenValidated` события.
        - Он использует `ITokenAcquisition` интерфейс для получения маркера доступа.
        - Он вызывает Microsoft Graph, чтобы получить профиль и фотографию пользователя.
        - Он добавляет данные Graph в удостоверение пользователя.

1. Добавьте следующий вызов функции после `EnableTokenAcquisitionToCallDownstreamApi` вызова и перед `AddInMemoryTokenCaches` вызовом.

    :::code language="csharp" source="../demo/GraphTutorial/Startup.cs" id="AddGraphClientSnippet":::

    Это сделает **graphServiceClient** с проверкой подлинности доступным для контроллеров через введение зависимостей.

1. Откройте **./Controllers/HomeController.cs** и замените `Index` функцию на следующую.

    ```csharp
    public IActionResult Index()
    {
        return View();
    }
    ```

1. Удалите все ссылки `ITokenAcquisition` на класс **HomeController.**

1. Сохраните изменения, запустите приложение и войдите в нее. Вы должны вернуться на home-страницу, но пользовательский интерфейс должен измениться, чтобы указать, что вы вписались.

    ![Снимок экрана с домашней страницей после входов](./images/add-aad-auth-01.png)

1. Щелкните аватар пользователя в правом верхнем углу, чтобы получить доступ к ссылке **"Выйти".** При **нажатии** кнопки "Выйти" сеанс сбрасывается и возвращается на домашней странице.

    ![Снимок экрана с выпадающим меню со ссылкой "Выйти"](./images/add-aad-auth-02.png)

> [!TIP]
> Если после внесения этих изменений имя пользователя на домашней странице отсутствует, а в расстановке "Использовать аватар" отсутствует имя и электронная почта, вы можете выйти из системы и снова войти.

## <a name="storing-and-refreshing-tokens"></a>Хранение и обновление маркеров

На этом этапе приложение имеет маркер доступа, который отправляется в `Authorization` заголовке вызовов API. Это маркер, который позволяет приложению получать доступ к Microsoft Graph от имени пользователя.

Однако этот маркер является кратковременной. Срок действия маркера истекает через час после его выпуска. В этом случае маркер обновления становится полезным. Маркер обновления позволяет приложению запрашивать новый маркер доступа, не требуя от пользователя повторного входить.

Так как приложение использует библиотеку Microsoft.Identity.Web, вам не нужно реализовывать логику хранения маркеров или обновления.

Приложение использует кэш маркеров в памяти, которого достаточно для приложений, которым не нужно сохранять маркеры при перезапуске приложения. Вместо этого производственные приложения могут использовать [параметры распределенного кэша](https://github.com/AzureAD/microsoft-identity-web/wiki/token-cache-serialization) в библиотеке Microsoft.Identity.Web.

Метод `GetAccessTokenForUserAsync` обрабатывает истечение срока действия маркера и обновление. Сначала он проверяет кэшный маркер и, если срок его действия еще не истек, возвращает его. Если срок действия истек, для получения нового маркера используется кэшный маркер обновления.

**GraphServiceClient,** получаемый контроллерами через введение зависимостей, будет предварительно настроен с использованием поставщика проверки `GetAccessTokenForUserAsync` подлинности, который используется для вас.
