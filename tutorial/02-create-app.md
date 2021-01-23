---
ms.openlocfilehash: 5b1a776c28b6f9218c713dde68f45e571ebfd999
ms.sourcegitcommit: 6341ad07cd5b03269e7fd20cd3212e48baee7c07
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/23/2021
ms.locfileid: "49942157"
---
<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="f5069-101">Для начала создайте ASP.NET Core web app.</span><span class="sxs-lookup"><span data-stu-id="f5069-101">Start by creating an ASP.NET Core web app.</span></span>

1. <span data-ttu-id="f5069-102">Откройте интерфейс командной строки (CLI) в каталоге, где нужно создать проект.</span><span class="sxs-lookup"><span data-stu-id="f5069-102">Open your command-line interface (CLI) in a directory where you want to create the project.</span></span> <span data-ttu-id="f5069-103">Выполните следующую команду.</span><span class="sxs-lookup"><span data-stu-id="f5069-103">Run the following command.</span></span>

    ```Shell
    dotnet new mvc -o GraphTutorial
    ```

1. <span data-ttu-id="f5069-104">После создания проекта убедитесь, что он работает, изменив текущий каталог на **каталог GraphTutorial** и выполнив следующую команду в CLI.</span><span class="sxs-lookup"><span data-stu-id="f5069-104">Once the project is created, verify that it works by changing the current directory to the **GraphTutorial** directory and running the following command in your CLI.</span></span>

    ```Shell
    dotnet run
    ```

1. <span data-ttu-id="f5069-105">Откройте браузер и перейдите к `https://localhost:5001` .</span><span class="sxs-lookup"><span data-stu-id="f5069-105">Open your browser and browse to `https://localhost:5001`.</span></span> <span data-ttu-id="f5069-106">Если все работает, вы увидите стандартную ASP.NET основной странице.</span><span class="sxs-lookup"><span data-stu-id="f5069-106">If everything is working, you should see a default ASP.NET Core page.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="f5069-107">Если вы получите предупреждение о том, что сертификат **для localhost** не является доверенным, можно использовать .NET Core CLI для установки сертификата разработки и доверия ему.</span><span class="sxs-lookup"><span data-stu-id="f5069-107">If you receive a warning that the certificate for **localhost** is un-trusted you can use the .NET Core CLI to install and trust the development certificate.</span></span> <span data-ttu-id="f5069-108">Инструкции [по определенным операционным системам см.](/aspnet/core/security/enforcing-ssl?view=aspnetcore-5.0) в ASP.NET "Enforce HTTPS in ASP.NET Core".</span><span class="sxs-lookup"><span data-stu-id="f5069-108">See [Enforce HTTPS in ASP.NET Core](/aspnet/core/security/enforcing-ssl?view=aspnetcore-5.0) for instructions for specific operating systems.</span></span>

## <a name="add-nuget-packages"></a><span data-ttu-id="f5069-109">Добавление пакетов NuGet</span><span class="sxs-lookup"><span data-stu-id="f5069-109">Add NuGet packages</span></span>

<span data-ttu-id="f5069-110">Прежде чем двигаться дальше, установите некоторые дополнительные пакеты NuGet, которые вы будете использовать позже.</span><span class="sxs-lookup"><span data-stu-id="f5069-110">Before moving on, install some additional NuGet packages that you will use later.</span></span>

- <span data-ttu-id="f5069-111">[Microsoft.Identity.Web](https://www.nuget.org/packages/Microsoft.Identity.Web/) для запроса маркеров доступа и управления маркерами доступа.</span><span class="sxs-lookup"><span data-stu-id="f5069-111">[Microsoft.Identity.Web](https://www.nuget.org/packages/Microsoft.Identity.Web/) for requesting and managing access tokens.</span></span>
- <span data-ttu-id="f5069-112">[Microsoft.Identity.Web.MicrosoftGraph для](https://www.nuget.org/packages/Microsoft.Identity.Web.MicrosoftGraph/) добавления microsoft Graph SDK с помощью встраивания зависимостей.</span><span class="sxs-lookup"><span data-stu-id="f5069-112">[Microsoft.Identity.Web.MicrosoftGraph](https://www.nuget.org/packages/Microsoft.Identity.Web.MicrosoftGraph/) for adding the Microsoft Graph SDK via dependency injection.</span></span>
- <span data-ttu-id="f5069-113">[Microsoft.Identity.Web.UI](https://www.nuget.org/packages/Microsoft.Identity.Web.UI/) для пользовательского интерфейса для входов и выходов.</span><span class="sxs-lookup"><span data-stu-id="f5069-113">[Microsoft.Identity.Web.UI](https://www.nuget.org/packages/Microsoft.Identity.Web.UI/) for sign-in and sign-out UI.</span></span>
- <span data-ttu-id="f5069-114">[TimeZoneConverter](https://github.com/mj1856/TimeZoneConverter) для обработки идентификаторов часового пояса на разных платформах.</span><span class="sxs-lookup"><span data-stu-id="f5069-114">[TimeZoneConverter](https://github.com/mj1856/TimeZoneConverter) for handling time zoned identifiers cross-platform.</span></span>

1. <span data-ttu-id="f5069-115">Для установки зависимостей в CLI запустите следующие команды.</span><span class="sxs-lookup"><span data-stu-id="f5069-115">Run the following commands in your CLI to install the dependencies.</span></span>

    ```Shell
    dotnet add package Microsoft.Identity.Web --version 1.5.1
    dotnet add package Microsoft.Identity.Web.MicrosoftGraph --version 1.5.1
    dotnet add package Microsoft.Identity.Web.UI --version 1.5.1
    dotnet add package TimeZoneConverter
    ```

## <a name="design-the-app"></a><span data-ttu-id="f5069-116">Проектирование приложения</span><span class="sxs-lookup"><span data-stu-id="f5069-116">Design the app</span></span>

<span data-ttu-id="f5069-117">В этом разделе вы создадим базовую структуру пользовательского интерфейса приложения.</span><span class="sxs-lookup"><span data-stu-id="f5069-117">In this section you will create the basic UI structure of the application.</span></span>

### <a name="implement-alert-extension-methods"></a><span data-ttu-id="f5069-118">Реализация методов расширения оповещений</span><span class="sxs-lookup"><span data-stu-id="f5069-118">Implement alert extension methods</span></span>

<span data-ttu-id="f5069-119">В этом разделе мы создадим методы расширения для `IActionResult` типа, возвращаемого представлениями контроллера.</span><span class="sxs-lookup"><span data-stu-id="f5069-119">In this section you will create extension methods for the `IActionResult` type returned by controller views.</span></span> <span data-ttu-id="f5069-120">Это расширение позволяет передать в представление временные сообщения об ошибках или успешном успешном переходе.</span><span class="sxs-lookup"><span data-stu-id="f5069-120">This extension will enable passing temporary error or success messages to the view.</span></span>

> [!TIP]
> <span data-ttu-id="f5069-121">Для редактирования исходных файлов для этого руководства можно использовать любой текстовый редактор.</span><span class="sxs-lookup"><span data-stu-id="f5069-121">You can use any text editor to edit the source files for this tutorial.</span></span> <span data-ttu-id="f5069-122">Однако Visual Studio [код](https://code.visualstudio.com/) предоставляет дополнительные функции, такие как отладка и Intellisense.</span><span class="sxs-lookup"><span data-stu-id="f5069-122">However, [Visual Studio Code](https://code.visualstudio.com/) provides additional features, such as debugging and Intellisense.</span></span>

1. <span data-ttu-id="f5069-123">Создайте новый каталог в **каталоге GraphTutorial** с **именем Alerts.**</span><span class="sxs-lookup"><span data-stu-id="f5069-123">Create a new directory in the **GraphTutorial** directory named **Alerts**.</span></span>

1. <span data-ttu-id="f5069-124">Создайте файл с **именем WithAlertResult.cs** **в каталоге ./Alerts** и добавьте следующий код.</span><span class="sxs-lookup"><span data-stu-id="f5069-124">Create a new file named **WithAlertResult.cs** in the **./Alerts** directory and add the following code.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Alerts/WithAlertResult.cs" id="WithAlertResultSnippet":::

1. <span data-ttu-id="f5069-125">Создайте файл с **именем AlertExtensions.cs** в **каталоге ./Alerts** и добавьте следующий код.</span><span class="sxs-lookup"><span data-stu-id="f5069-125">Create a new file named **AlertExtensions.cs** in the **./Alerts** directory and add the following code.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Alerts/AlertExtensions.cs" id="AlertExtensionsSnippet":::

### <a name="implement-user-data-extension-methods"></a><span data-ttu-id="f5069-126">Реализация методов расширения пользовательских данных</span><span class="sxs-lookup"><span data-stu-id="f5069-126">Implement user data extension methods</span></span>

<span data-ttu-id="f5069-127">В этом разделе мы создадим методы расширения для `ClaimsPrincipal` объекта, создающегося платформой Microsoft Identity.</span><span class="sxs-lookup"><span data-stu-id="f5069-127">In this section you will create extension methods for the `ClaimsPrincipal` object generated by the Microsoft Identity platform.</span></span> <span data-ttu-id="f5069-128">Это позволит расширить существующее удостоверение пользователя данными из Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="f5069-128">This will allow you to extend the existing user identity with data from Microsoft Graph.</span></span>

> [!NOTE]
> <span data-ttu-id="f5069-129">На данный момент этот код является просто заполнителям, он будет завершен в более позднем разделе.</span><span class="sxs-lookup"><span data-stu-id="f5069-129">This code is just a placeholder for now, you will complete it in a later section.</span></span>

1. <span data-ttu-id="f5069-130">Создайте новый каталог в **каталоге GraphTutorial** с именем **Graph.**</span><span class="sxs-lookup"><span data-stu-id="f5069-130">Create a new directory in the **GraphTutorial** directory named **Graph**.</span></span>

1. <span data-ttu-id="f5069-131">Создайте файл с **именем GraphClaimsPrincipalExtensions.cs** и добавьте следующий код.</span><span class="sxs-lookup"><span data-stu-id="f5069-131">Create a new file named **GraphClaimsPrincipalExtensions.cs** and add the following code.</span></span>

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

### <a name="create-views"></a><span data-ttu-id="f5069-132">Создание представлений</span><span class="sxs-lookup"><span data-stu-id="f5069-132">Create views</span></span>

<span data-ttu-id="f5069-133">В этом разделе будут реализованы представления "Секция" для приложения.</span><span class="sxs-lookup"><span data-stu-id="f5069-133">In this section you will implement the Razor views for the application.</span></span>

1. <span data-ttu-id="f5069-134">Добавьте новый файл с **именем _LoginPartial.cshtml** в **каталог ./Views/Shared** и добавьте следующий код.</span><span class="sxs-lookup"><span data-stu-id="f5069-134">Add a new file named **_LoginPartial.cshtml** in the **./Views/Shared** directory and add the following code.</span></span>

    :::code language="cshtml" source="../demo/GraphTutorial/Views/Shared/_LoginPartial.cshtml" id="LoginPartialSnippet":::

1. <span data-ttu-id="f5069-135">Добавьте новый файл с **именем _AlertPartial.cshtml** в **каталог ./Views/Shared** и добавьте следующий код.</span><span class="sxs-lookup"><span data-stu-id="f5069-135">Add a new file named **_AlertPartial.cshtml** in the **./Views/Shared** directory and add the following code.</span></span>

    :::code language="cshtml" source="../demo/GraphTutorial/Views/Shared/_AlertPartial.cshtml" id="AlertPartialSnippet":::

1. <span data-ttu-id="f5069-136">Откройте **файл ./Views/Shared/_Layout.cshtml** и замените его все содержимое на следующий код, чтобы обновить глобальный макет приложения.</span><span class="sxs-lookup"><span data-stu-id="f5069-136">Open the **./Views/Shared/_Layout.cshtml** file, and replace its entire contents with the following code to update the global layout of the app.</span></span>

    :::code language="cshtml" source="../demo/GraphTutorial/Views/Shared/_Layout.cshtml" id="LayoutSnippet":::

1. <span data-ttu-id="f5069-137">Откройте **файл ./wwwroot/css/site.css** и добавьте следующий код в нижней части файла.</span><span class="sxs-lookup"><span data-stu-id="f5069-137">Open **./wwwroot/css/site.css** and add the following code at the bottom of the file.</span></span>

    :::code language="css" source="../demo/GraphTutorial/wwwroot/css/site.css" id="CssSnippet":::

1. <span data-ttu-id="f5069-138">Откройте файл **./Views/Home/index.cshtml** и замените его содержимое на следующее.</span><span class="sxs-lookup"><span data-stu-id="f5069-138">Open the **./Views/Home/index.cshtml** file and replace its contents with the following.</span></span>

    :::code language="cshtml" source="../demo/GraphTutorial/Views/Home/Index.cshtml" id="HomeIndexSnippet":::

1. <span data-ttu-id="f5069-139">Создайте новый каталог в **каталоге ./wwwroot** с именем **img.**</span><span class="sxs-lookup"><span data-stu-id="f5069-139">Create a new directory in the **./wwwroot** directory named **img**.</span></span> <span data-ttu-id="f5069-140">Добавьте файл изображения с именем **no-profile-photo.png** в этом каталоге.</span><span class="sxs-lookup"><span data-stu-id="f5069-140">Add an image file of your choosing named **no-profile-photo.png** in this directory.</span></span> <span data-ttu-id="f5069-141">Это изображение будет использоваться в качестве фотографии пользователя, если у пользователя нет фотографии в Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="f5069-141">This image will be used as the user's photo when the user has no photo in Microsoft Graph.</span></span>

    > [!TIP]
    > <span data-ttu-id="f5069-142">Вы можете скачать изображение, использованное на этих снимках экрана, с [GitHub.](https://github.com/microsoftgraph/msgraph-training-aspnet-core/blob/master/demo/GraphTutorial/wwwroot/img/no-profile-photo.png)</span><span class="sxs-lookup"><span data-stu-id="f5069-142">You can download the image used in these screenshots from [GitHub](https://github.com/microsoftgraph/msgraph-training-aspnet-core/blob/master/demo/GraphTutorial/wwwroot/img/no-profile-photo.png).</span></span>

1. <span data-ttu-id="f5069-143">Сохраните все изменения и перезапустите сервер ( `dotnet run` ).</span><span class="sxs-lookup"><span data-stu-id="f5069-143">Save all of your changes and restart the server (`dotnet run`).</span></span> <span data-ttu-id="f5069-144">Теперь приложение должно выглядеть совершенно иначе.</span><span class="sxs-lookup"><span data-stu-id="f5069-144">Now, the app should look very different.</span></span>

    ![Снимок экрана с измененной домашней страницей](./images/create-app-01.png)
