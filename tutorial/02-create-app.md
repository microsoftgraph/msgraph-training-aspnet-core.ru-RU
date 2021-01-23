---
ms.openlocfilehash: 5b1a776c28b6f9218c713dde68f45e571ebfd999
ms.sourcegitcommit: 6341ad07cd5b03269e7fd20cd3212e48baee7c07
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/23/2021
ms.locfileid: "49942157"
---
<!-- markdownlint-disable MD002 MD041 -->

Для начала создайте ASP.NET Core web app.

1. Откройте интерфейс командной строки (CLI) в каталоге, где нужно создать проект. Выполните следующую команду.

    ```Shell
    dotnet new mvc -o GraphTutorial
    ```

1. После создания проекта убедитесь, что он работает, изменив текущий каталог на **каталог GraphTutorial** и выполнив следующую команду в CLI.

    ```Shell
    dotnet run
    ```

1. Откройте браузер и перейдите к `https://localhost:5001` . Если все работает, вы увидите стандартную ASP.NET основной странице.

> [!IMPORTANT]
> Если вы получите предупреждение о том, что сертификат **для localhost** не является доверенным, можно использовать .NET Core CLI для установки сертификата разработки и доверия ему. Инструкции [по определенным операционным системам см.](/aspnet/core/security/enforcing-ssl?view=aspnetcore-5.0) в ASP.NET "Enforce HTTPS in ASP.NET Core".

## <a name="add-nuget-packages"></a>Добавление пакетов NuGet

Прежде чем двигаться дальше, установите некоторые дополнительные пакеты NuGet, которые вы будете использовать позже.

- [Microsoft.Identity.Web](https://www.nuget.org/packages/Microsoft.Identity.Web/) для запроса маркеров доступа и управления маркерами доступа.
- [Microsoft.Identity.Web.MicrosoftGraph для](https://www.nuget.org/packages/Microsoft.Identity.Web.MicrosoftGraph/) добавления microsoft Graph SDK с помощью встраивания зависимостей.
- [Microsoft.Identity.Web.UI](https://www.nuget.org/packages/Microsoft.Identity.Web.UI/) для пользовательского интерфейса для входов и выходов.
- [TimeZoneConverter](https://github.com/mj1856/TimeZoneConverter) для обработки идентификаторов часового пояса на разных платформах.

1. Для установки зависимостей в CLI запустите следующие команды.

    ```Shell
    dotnet add package Microsoft.Identity.Web --version 1.5.1
    dotnet add package Microsoft.Identity.Web.MicrosoftGraph --version 1.5.1
    dotnet add package Microsoft.Identity.Web.UI --version 1.5.1
    dotnet add package TimeZoneConverter
    ```

## <a name="design-the-app"></a>Проектирование приложения

В этом разделе вы создадим базовую структуру пользовательского интерфейса приложения.

### <a name="implement-alert-extension-methods"></a>Реализация методов расширения оповещений

В этом разделе мы создадим методы расширения для `IActionResult` типа, возвращаемого представлениями контроллера. Это расширение позволяет передать в представление временные сообщения об ошибках или успешном успешном переходе.

> [!TIP]
> Для редактирования исходных файлов для этого руководства можно использовать любой текстовый редактор. Однако Visual Studio [код](https://code.visualstudio.com/) предоставляет дополнительные функции, такие как отладка и Intellisense.

1. Создайте новый каталог в **каталоге GraphTutorial** с **именем Alerts.**

1. Создайте файл с **именем WithAlertResult.cs** **в каталоге ./Alerts** и добавьте следующий код.

    :::code language="csharp" source="../demo/GraphTutorial/Alerts/WithAlertResult.cs" id="WithAlertResultSnippet":::

1. Создайте файл с **именем AlertExtensions.cs** в **каталоге ./Alerts** и добавьте следующий код.

    :::code language="csharp" source="../demo/GraphTutorial/Alerts/AlertExtensions.cs" id="AlertExtensionsSnippet":::

### <a name="implement-user-data-extension-methods"></a>Реализация методов расширения пользовательских данных

В этом разделе мы создадим методы расширения для `ClaimsPrincipal` объекта, создающегося платформой Microsoft Identity. Это позволит расширить существующее удостоверение пользователя данными из Microsoft Graph.

> [!NOTE]
> На данный момент этот код является просто заполнителям, он будет завершен в более позднем разделе.

1. Создайте новый каталог в **каталоге GraphTutorial** с именем **Graph.**

1. Создайте файл с **именем GraphClaimsPrincipalExtensions.cs** и добавьте следующий код.

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

### <a name="create-views"></a>Создание представлений

В этом разделе будут реализованы представления "Секция" для приложения.

1. Добавьте новый файл с **именем _LoginPartial.cshtml** в **каталог ./Views/Shared** и добавьте следующий код.

    :::code language="cshtml" source="../demo/GraphTutorial/Views/Shared/_LoginPartial.cshtml" id="LoginPartialSnippet":::

1. Добавьте новый файл с **именем _AlertPartial.cshtml** в **каталог ./Views/Shared** и добавьте следующий код.

    :::code language="cshtml" source="../demo/GraphTutorial/Views/Shared/_AlertPartial.cshtml" id="AlertPartialSnippet":::

1. Откройте **файл ./Views/Shared/_Layout.cshtml** и замените его все содержимое на следующий код, чтобы обновить глобальный макет приложения.

    :::code language="cshtml" source="../demo/GraphTutorial/Views/Shared/_Layout.cshtml" id="LayoutSnippet":::

1. Откройте **файл ./wwwroot/css/site.css** и добавьте следующий код в нижней части файла.

    :::code language="css" source="../demo/GraphTutorial/wwwroot/css/site.css" id="CssSnippet":::

1. Откройте файл **./Views/Home/index.cshtml** и замените его содержимое на следующее.

    :::code language="cshtml" source="../demo/GraphTutorial/Views/Home/Index.cshtml" id="HomeIndexSnippet":::

1. Создайте новый каталог в **каталоге ./wwwroot** с именем **img.** Добавьте файл изображения с именем **no-profile-photo.png** в этом каталоге. Это изображение будет использоваться в качестве фотографии пользователя, если у пользователя нет фотографии в Microsoft Graph.

    > [!TIP]
    > Вы можете скачать изображение, использованное на этих снимках экрана, с [GitHub.](https://github.com/microsoftgraph/msgraph-training-aspnet-core/blob/master/demo/GraphTutorial/wwwroot/img/no-profile-photo.png)

1. Сохраните все изменения и перезапустите сервер ( `dotnet run` ). Теперь приложение должно выглядеть совершенно иначе.

    ![Снимок экрана с измененной домашней страницей](./images/create-app-01.png)
