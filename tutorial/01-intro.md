---
ms.openlocfilehash: 73ba9c271c9c6675ffbb22ce4f800ffb652c715d
ms.sourcegitcommit: 6341ad07cd5b03269e7fd20cd3212e48baee7c07
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/23/2021
ms.locfileid: "49942136"
---
<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="476d3-101">В этом руководстве рассказывается, как создать ASP.NET Core, которое использует API Microsoft Graph для получения сведений календаря для пользователя.</span><span class="sxs-lookup"><span data-stu-id="476d3-101">This tutorial teaches you how to build an ASP.NET Core web app that uses the Microsoft Graph API to retrieve calendar information for a user.</span></span>

> [!TIP]
> <span data-ttu-id="476d3-102">Если вы предпочитаете просто скачать завершенный учебник, вы можете скачать или клонировать [репозиторий GitHub.](https://github.com/microsoftgraph/msgraph-training-aspnet-core)</span><span class="sxs-lookup"><span data-stu-id="476d3-102">If you prefer to just download the completed tutorial, you can download or clone the [GitHub repository](https://github.com/microsoftgraph/msgraph-training-aspnet-core).</span></span> <span data-ttu-id="476d3-103">Инструкции по настройке  приложения с помощью ИД приложения и секрета см. в файле README в папке демонстрации.</span><span class="sxs-lookup"><span data-stu-id="476d3-103">See the README file in the **demo** folder for instructions on configuring the app with an app ID and secret.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="476d3-104">Предварительные условия</span><span class="sxs-lookup"><span data-stu-id="476d3-104">Prerequisites</span></span>

<span data-ttu-id="476d3-105">Прежде чем приступить к этому учебнику, на компьютере разработчика должен быть установлен [.NET Core SDK.](https://dotnet.microsoft.com/download)</span><span class="sxs-lookup"><span data-stu-id="476d3-105">Before you start this tutorial, you should have the [.NET Core SDK](https://dotnet.microsoft.com/download) installed on your development machine.</span></span> <span data-ttu-id="476d3-106">Если у вас нет SDK, посетите предыдущую ссылку для скачивания.</span><span class="sxs-lookup"><span data-stu-id="476d3-106">If you do not have the SDK, visit the previous link for download options.</span></span>

<span data-ttu-id="476d3-107">У вас также должна быть личная учетная запись Майкрософт с почтовым ящиком на Outlook.com или учетная запись Майкрософт для работы или учебного заведения.</span><span class="sxs-lookup"><span data-stu-id="476d3-107">You should also have either a personal Microsoft account with a mailbox on Outlook.com, or a Microsoft work or school account.</span></span> <span data-ttu-id="476d3-108">Если у вас нет учетной записи Майкрософт, существует несколько вариантов получения бесплатной учетной записи:</span><span class="sxs-lookup"><span data-stu-id="476d3-108">If you don't have a Microsoft account, there are a couple of options to get a free account:</span></span>

- <span data-ttu-id="476d3-109">Вы можете [зарегистрироваться для новой личной учетной записи Майкрософт.](https://signup.live.com/signup?wa=wsignin1.0&rpsnv=12&ct=1454618383&rver=6.4.6456.0&wp=MBI_SSL_SHARED&wreply=https://mail.live.com/default.aspx&id=64855&cbcxt=mai&bk=1454618383&uiflavor=web&uaid=b213a65b4fdc484382b6622b3ecaa547&mkt=E-US&lc=1033&lic=1)</span><span class="sxs-lookup"><span data-stu-id="476d3-109">You can [sign up for a new personal Microsoft account](https://signup.live.com/signup?wa=wsignin1.0&rpsnv=12&ct=1454618383&rver=6.4.6456.0&wp=MBI_SSL_SHARED&wreply=https://mail.live.com/default.aspx&id=64855&cbcxt=mai&bk=1454618383&uiflavor=web&uaid=b213a65b4fdc484382b6622b3ecaa547&mkt=E-US&lc=1033&lic=1).</span></span>
- <span data-ttu-id="476d3-110">Вы можете зарегистрироваться в программе для разработчиков [Office 365,](https://developer.microsoft.com/office/dev-program) чтобы получить бесплатную подписку на Office 365.</span><span class="sxs-lookup"><span data-stu-id="476d3-110">You can [sign up for the Office 365 Developer Program](https://developer.microsoft.com/office/dev-program) to get a free Office 365 subscription.</span></span>

> [!NOTE]
> <span data-ttu-id="476d3-111">Это руководство было написано с помощью .NET Core SDK версии 5.0.102.</span><span class="sxs-lookup"><span data-stu-id="476d3-111">This tutorial was written with .NET Core SDK version 5.0.102.</span></span> <span data-ttu-id="476d3-112">Действия в этом руководстве могут работать с другими версиями, но не были протестированы.</span><span class="sxs-lookup"><span data-stu-id="476d3-112">The steps in this guide may work with other versions, but that has not been tested.</span></span>

## <a name="feedback"></a><span data-ttu-id="476d3-113">Отзывы</span><span class="sxs-lookup"><span data-stu-id="476d3-113">Feedback</span></span>

<span data-ttu-id="476d3-114">Поделитесь с этим учебником отзывами в [репозитории GitHub.](https://github.com/microsoftgraph/msgraph-training-aspnet-core)</span><span class="sxs-lookup"><span data-stu-id="476d3-114">Please provide any feedback on this tutorial in the [GitHub repository](https://github.com/microsoftgraph/msgraph-training-aspnet-core).</span></span>
