---
ms.openlocfilehash: a024fb533c552563da6c9179301e16a2e1d09d5f
ms.sourcegitcommit: 6341ad07cd5b03269e7fd20cd3212e48baee7c07
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/23/2021
ms.locfileid: "49942164"
---
<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="d5c4e-101">В этом упражнении вы расширим приложение из предыдущего упражнения, чтобы поддерживать проверку подлинности с помощью Azure AD.</span><span class="sxs-lookup"><span data-stu-id="d5c4e-101">In this exercise you will extend the application from the previous exercise to support authentication with Azure AD.</span></span> <span data-ttu-id="d5c4e-102">Это необходимо для получения необходимого маркера доступа OAuth для вызова API Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="d5c4e-102">This is required to obtain the necessary OAuth access token to call the Microsoft Graph API.</span></span> <span data-ttu-id="d5c4e-103">На этом этапе вы настроим [библиотеку Microsoft.Identity.Web.](https://www.nuget.org/packages/Microsoft.Identity.Web/)</span><span class="sxs-lookup"><span data-stu-id="d5c4e-103">In this step you will configure the [Microsoft.Identity.Web](https://www.nuget.org/packages/Microsoft.Identity.Web/) library.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="d5c4e-104">Чтобы не хранить код и секрет приложения в источнике, для хранения этих значений используется диспетчер секретов [.NET.](/aspnet/core/security/app-secrets)</span><span class="sxs-lookup"><span data-stu-id="d5c4e-104">To avoid storing the application ID and secret in source, you will use the [.NET Secret Manager](/aspnet/core/security/app-secrets) to store these values.</span></span> <span data-ttu-id="d5c4e-105">Диспетчер секретов предназначен только для разработки, и в производственных приложениях для хранения секретов следует использовать доверенного диспетчера секретов.</span><span class="sxs-lookup"><span data-stu-id="d5c4e-105">The Secret Manager is for development purposes only, production apps should use a trusted secret manager for storing secrets.</span></span>

1. <span data-ttu-id="d5c4e-106">Откройте **./appsettings.jsи** замените его содержимое на следующее.</span><span class="sxs-lookup"><span data-stu-id="d5c4e-106">Open **./appsettings.json** and replace its contents with the following.</span></span>

    :::code language="json" source="../demo/GraphTutorial/appsettings.json" highlight="2-6":::

1. <span data-ttu-id="d5c4e-107">Откройте CLI в каталоге, в котором расположен **GraphTutorial.csproj,** и запустите следующие команды, заменив его на ИД приложения на портале Azure и с помощью секрета `YOUR_APP_ID` `YOUR_APP_SECRET` приложения.</span><span class="sxs-lookup"><span data-stu-id="d5c4e-107">Open your CLI in the directory where **GraphTutorial.csproj** is located, and run the following commands, substituting `YOUR_APP_ID` with your application ID from the Azure portal, and `YOUR_APP_SECRET` with your application secret.</span></span>

    ```Shell
    dotnet user-secrets init
    dotnet user-secrets set "AzureAd:ClientId" "YOUR_APP_ID"
    dotnet user-secrets set "AzureAd:ClientSecret" "YOUR_APP_SECRET"
    ```

## <a name="implement-sign-in"></a><span data-ttu-id="d5c4e-108">Реализация входов</span><span class="sxs-lookup"><span data-stu-id="d5c4e-108">Implement sign-in</span></span>

<span data-ttu-id="d5c4e-109">Для начала добавьте службы платформы удостоверений Майкрософт в приложение.</span><span class="sxs-lookup"><span data-stu-id="d5c4e-109">Start by adding the Microsoft Identity platform services to the application.</span></span>

1. <span data-ttu-id="d5c4e-110">Создайте файл с **именем GraphConstants.cs** **в каталоге ./Graph** и добавьте следующий код.</span><span class="sxs-lookup"><span data-stu-id="d5c4e-110">Create a new file named **GraphConstants.cs** in the **./Graph** directory and add the following code.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Graph/GraphConstants.cs" id="GraphConstantsSnippet":::

1. <span data-ttu-id="d5c4e-111">Откройте файл **./Startup.cs** и добавьте следующие утверждения в `using` верхнюю часть файла.</span><span class="sxs-lookup"><span data-stu-id="d5c4e-111">Open the **./Startup.cs** file and add the following `using` statements to the top of the file.</span></span>

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

1. <span data-ttu-id="d5c4e-112">Замените имеющуюся функцию `ConfigureServices` указанным ниже кодом.</span><span class="sxs-lookup"><span data-stu-id="d5c4e-112">Replace the existing `ConfigureServices` function with the following.</span></span>

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

1. <span data-ttu-id="d5c4e-113">В `Configure` функции добавьте следующую строку над `app.UseAuthorization();` строкой.</span><span class="sxs-lookup"><span data-stu-id="d5c4e-113">In the `Configure` function, add the following line above the `app.UseAuthorization();` line.</span></span>

    ```csharp
    app.UseAuthentication();
    ```

1. <span data-ttu-id="d5c4e-114">Откройте **./Controllers/HomeController.cs** и замените его содержимое следующим:</span><span class="sxs-lookup"><span data-stu-id="d5c4e-114">Open **./Controllers/HomeController.cs** and replace its contents with the following.</span></span>

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

1. <span data-ttu-id="d5c4e-115">Сохраните изменения и запустите проект.</span><span class="sxs-lookup"><span data-stu-id="d5c4e-115">Save your changes and start the project.</span></span> <span data-ttu-id="d5c4e-116">Войдите с помощью учетной записи Майкрософт.</span><span class="sxs-lookup"><span data-stu-id="d5c4e-116">Login with your Microsoft account.</span></span>

1. <span data-ttu-id="d5c4e-117">Проверьте запрос согласия.</span><span class="sxs-lookup"><span data-stu-id="d5c4e-117">Examine the consent prompt.</span></span> <span data-ttu-id="d5c4e-118">Список разрешений соответствует списку областей разрешений, настроенных в **./Graph/GraphConstants.cs.**</span><span class="sxs-lookup"><span data-stu-id="d5c4e-118">The list of permissions correspond to list of permissions scopes configured in **./Graph/GraphConstants.cs**.</span></span>

    - <span data-ttu-id="d5c4e-119">**Поддержив доступ** к данным, к ним предоставлен доступ: ( ) Это разрешение запрашивается MSAL для получения `offline_access` маркеров обновления.</span><span class="sxs-lookup"><span data-stu-id="d5c4e-119">**Maintain access to data you have given it access to:** (`offline_access`) This permission is requested by MSAL in order to retrieve refresh tokens.</span></span>
    - <span data-ttu-id="d5c4e-120">**Во sign you in and read your profile:** ( `User.Read` ) This permission allows the app to get the logged-in user's profile and profile photo.</span><span class="sxs-lookup"><span data-stu-id="d5c4e-120">**Sign you in and read your profile:** (`User.Read`) This permission allows the app to get the logged-in user's profile and profile photo.</span></span>
    - <span data-ttu-id="d5c4e-121">**Прочитайте параметры** почтового ящика: ( ) Это разрешение позволяет приложению читать параметры почтового ящика пользователя, включая часовой пояс `MailboxSettings.Read` и формат времени.</span><span class="sxs-lookup"><span data-stu-id="d5c4e-121">**Read your mailbox settings:** (`MailboxSettings.Read`) This permission allows the app to read the user's mailbox settings, including time zone and time format.</span></span>
    - <span data-ttu-id="d5c4e-122">**Полный доступ к** календарям: ( ) Это разрешение позволяет приложению читать события в календаре пользователя, добавлять новые события и изменять `Calendars.ReadWrite` существующие.</span><span class="sxs-lookup"><span data-stu-id="d5c4e-122">**Have full access to your calendars:** (`Calendars.ReadWrite`) This permission allows the app to read events on the user's calendar, add new events, and modify existing ones.</span></span>

    ![Снимок экрана: запрос на согласие платформы удостоверений Майкрософт](./images/add-aad-auth-03.png)

    <span data-ttu-id="d5c4e-124">Дополнительные сведения о согласии см. в сведениях о предоставлении согласия [приложениям Azure AD.](/azure/active-directory/develop/application-consent-experience)</span><span class="sxs-lookup"><span data-stu-id="d5c4e-124">For more information regarding consent, see [Understanding Azure AD application consent experiences](/azure/active-directory/develop/application-consent-experience).</span></span>

1. <span data-ttu-id="d5c4e-125">Согласие на запрашиваемую разрешения.</span><span class="sxs-lookup"><span data-stu-id="d5c4e-125">Consent to the requested permissions.</span></span> <span data-ttu-id="d5c4e-126">Браузер перенаправляет пользователя в приложение с отображением маркера.</span><span class="sxs-lookup"><span data-stu-id="d5c4e-126">The browser redirects to the app, showing the token.</span></span>

### <a name="get-user-details"></a><span data-ttu-id="d5c4e-127">Получить сведения о пользователе</span><span class="sxs-lookup"><span data-stu-id="d5c4e-127">Get user details</span></span>

<span data-ttu-id="d5c4e-128">После входа пользователя вы можете получить его сведения из Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="d5c4e-128">Once the user is logged in, you can get their information from Microsoft Graph.</span></span>

1. <span data-ttu-id="d5c4e-129">Откройте **./Graph/GraphClaimsPrincipalExtensions.cs** и замените все его содержимое следующим:</span><span class="sxs-lookup"><span data-stu-id="d5c4e-129">Open **./Graph/GraphClaimsPrincipalExtensions.cs** and replace its entire contents with the following.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Graph/GraphClaimsPrincipalExtensions.cs" id="GraphClaimsExtensionsSnippet":::

1. <span data-ttu-id="d5c4e-130">Откройте **./Startup.cs** и замените существующую `.AddMicrosoftIdentityWebApp(Configuration)` строку следующим кодом.</span><span class="sxs-lookup"><span data-stu-id="d5c4e-130">Open **./Startup.cs** and replace the existing `.AddMicrosoftIdentityWebApp(Configuration)` line with the following code.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Startup.cs" id="AddSignInSnippet":::

    <span data-ttu-id="d5c4e-131">Подумайте, что делает этот код.</span><span class="sxs-lookup"><span data-stu-id="d5c4e-131">Consider what this code does.</span></span>

    - <span data-ttu-id="d5c4e-132">Он добавляет обработок `OnTokenValidated` события.</span><span class="sxs-lookup"><span data-stu-id="d5c4e-132">It adds an event handler for the `OnTokenValidated` event.</span></span>
        - <span data-ttu-id="d5c4e-133">Он использует `ITokenAcquisition` интерфейс для получения маркера доступа.</span><span class="sxs-lookup"><span data-stu-id="d5c4e-133">It uses the `ITokenAcquisition` interface to get an access token.</span></span>
        - <span data-ttu-id="d5c4e-134">Он вызывает Microsoft Graph, чтобы получить профиль и фотографию пользователя.</span><span class="sxs-lookup"><span data-stu-id="d5c4e-134">It calls Microsoft Graph to get the user's profile and photo.</span></span>
        - <span data-ttu-id="d5c4e-135">Он добавляет данные Graph в удостоверение пользователя.</span><span class="sxs-lookup"><span data-stu-id="d5c4e-135">It adds the Graph information to the user's identity.</span></span>

1. <span data-ttu-id="d5c4e-136">Добавьте следующий вызов функции после `EnableTokenAcquisitionToCallDownstreamApi` вызова и перед `AddInMemoryTokenCaches` вызовом.</span><span class="sxs-lookup"><span data-stu-id="d5c4e-136">Add the following function call after the `EnableTokenAcquisitionToCallDownstreamApi` call and before the `AddInMemoryTokenCaches` call.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Startup.cs" id="AddGraphClientSnippet":::

    <span data-ttu-id="d5c4e-137">Это сделает **graphServiceClient** с проверкой подлинности доступным для контроллеров через введение зависимостей.</span><span class="sxs-lookup"><span data-stu-id="d5c4e-137">This will make an authenticated **GraphServiceClient** available to controllers via dependency injection.</span></span>

1. <span data-ttu-id="d5c4e-138">Откройте **./Controllers/HomeController.cs** и замените `Index` функцию на следующую.</span><span class="sxs-lookup"><span data-stu-id="d5c4e-138">Open **./Controllers/HomeController.cs** and replace the `Index` function with the following.</span></span>

    ```csharp
    public IActionResult Index()
    {
        return View();
    }
    ```

1. <span data-ttu-id="d5c4e-139">Удалите все ссылки `ITokenAcquisition` на класс **HomeController.**</span><span class="sxs-lookup"><span data-stu-id="d5c4e-139">Remove all references to `ITokenAcquisition` in the **HomeController** class.</span></span>

1. <span data-ttu-id="d5c4e-140">Сохраните изменения, запустите приложение и войдите в нее.</span><span class="sxs-lookup"><span data-stu-id="d5c4e-140">Save your changes, start the app, and go through the sign-in process.</span></span> <span data-ttu-id="d5c4e-141">Вы должны вернуться на home-страницу, но пользовательский интерфейс должен измениться, чтобы указать, что вы вписались.</span><span class="sxs-lookup"><span data-stu-id="d5c4e-141">You should end up back on the home page, but the UI should change to indicate that you are signed-in.</span></span>

    ![Снимок экрана с домашней страницей после входов](./images/add-aad-auth-01.png)

1. <span data-ttu-id="d5c4e-143">Щелкните аватар пользователя в правом верхнем углу, чтобы получить доступ к ссылке **"Выйти".**</span><span class="sxs-lookup"><span data-stu-id="d5c4e-143">Click the user avatar in the top right corner to access the **Sign Out** link.</span></span> <span data-ttu-id="d5c4e-144">При **нажатии** кнопки "Выйти" сеанс сбрасывается и возвращается на домашней странице.</span><span class="sxs-lookup"><span data-stu-id="d5c4e-144">Clicking **Sign Out** resets the session and returns you to the home page.</span></span>

    ![Снимок экрана с выпадающим меню со ссылкой "Выйти"](./images/add-aad-auth-02.png)

> [!TIP]
> <span data-ttu-id="d5c4e-146">Если после внесения этих изменений имя пользователя на домашней странице отсутствует, а в расстановке "Использовать аватар" отсутствует имя и электронная почта, вы можете выйти из системы и снова войти.</span><span class="sxs-lookup"><span data-stu-id="d5c4e-146">If you do not see your user name on the home page and the use avatar dropdown is missing name and email after making these changes, sign out and sign back in.</span></span>

## <a name="storing-and-refreshing-tokens"></a><span data-ttu-id="d5c4e-147">Хранение и обновление маркеров</span><span class="sxs-lookup"><span data-stu-id="d5c4e-147">Storing and refreshing tokens</span></span>

<span data-ttu-id="d5c4e-148">На этом этапе приложение имеет маркер доступа, который отправляется в `Authorization` заголовке вызовов API.</span><span class="sxs-lookup"><span data-stu-id="d5c4e-148">At this point your application has an access token, which is sent in the `Authorization` header of API calls.</span></span> <span data-ttu-id="d5c4e-149">Это маркер, который позволяет приложению получать доступ к Microsoft Graph от имени пользователя.</span><span class="sxs-lookup"><span data-stu-id="d5c4e-149">This is the token that allows the app to access Microsoft Graph on the user's behalf.</span></span>

<span data-ttu-id="d5c4e-150">Однако этот маркер является кратковременной.</span><span class="sxs-lookup"><span data-stu-id="d5c4e-150">However, this token is short-lived.</span></span> <span data-ttu-id="d5c4e-151">Срок действия маркера истекает через час после его выпуска.</span><span class="sxs-lookup"><span data-stu-id="d5c4e-151">The token expires an hour after it is issued.</span></span> <span data-ttu-id="d5c4e-152">В этом случае маркер обновления становится полезным.</span><span class="sxs-lookup"><span data-stu-id="d5c4e-152">This is where the refresh token becomes useful.</span></span> <span data-ttu-id="d5c4e-153">Маркер обновления позволяет приложению запрашивать новый маркер доступа, не требуя от пользователя повторного входить.</span><span class="sxs-lookup"><span data-stu-id="d5c4e-153">The refresh token allows the app to request a new access token without requiring the user to sign in again.</span></span>

<span data-ttu-id="d5c4e-154">Так как приложение использует библиотеку Microsoft.Identity.Web, вам не нужно реализовывать логику хранения маркеров или обновления.</span><span class="sxs-lookup"><span data-stu-id="d5c4e-154">Because the app is using the Microsoft.Identity.Web library, you do not have to implement any token storage or refresh logic.</span></span>

<span data-ttu-id="d5c4e-155">Приложение использует кэш маркеров в памяти, которого достаточно для приложений, которым не нужно сохранять маркеры при перезапуске приложения.</span><span class="sxs-lookup"><span data-stu-id="d5c4e-155">The app uses the in-memory token cache, which is sufficient for apps that do not need to persist tokens when the app restarts.</span></span> <span data-ttu-id="d5c4e-156">Вместо этого производственные приложения могут использовать [параметры распределенного кэша](https://github.com/AzureAD/microsoft-identity-web/wiki/token-cache-serialization) в библиотеке Microsoft.Identity.Web.</span><span class="sxs-lookup"><span data-stu-id="d5c4e-156">Production apps may instead use the [distributed cache options](https://github.com/AzureAD/microsoft-identity-web/wiki/token-cache-serialization) in the Microsoft.Identity.Web library.</span></span>

<span data-ttu-id="d5c4e-157">Метод `GetAccessTokenForUserAsync` обрабатывает истечение срока действия маркера и обновление.</span><span class="sxs-lookup"><span data-stu-id="d5c4e-157">The `GetAccessTokenForUserAsync` method handles token expiration and refresh for you.</span></span> <span data-ttu-id="d5c4e-158">Сначала он проверяет кэшный маркер и, если срок его действия еще не истек, возвращает его.</span><span class="sxs-lookup"><span data-stu-id="d5c4e-158">It first checks the cached token, and if it is not expired, it returns it.</span></span> <span data-ttu-id="d5c4e-159">Если срок действия истек, для получения нового маркера используется кэшный маркер обновления.</span><span class="sxs-lookup"><span data-stu-id="d5c4e-159">If it is expired, it uses the cached refresh token to obtain a new one.</span></span>

<span data-ttu-id="d5c4e-160">**GraphServiceClient,** получаемый контроллерами через введение зависимостей, будет предварительно настроен с использованием поставщика проверки `GetAccessTokenForUserAsync` подлинности, который используется для вас.</span><span class="sxs-lookup"><span data-stu-id="d5c4e-160">The **GraphServiceClient** that controllers get via dependency injection will be pre-configured with an authentication provider that uses `GetAccessTokenForUserAsync` for you.</span></span>
