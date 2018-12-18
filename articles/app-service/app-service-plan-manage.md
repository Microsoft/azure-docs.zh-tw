---
title: 管理 Azure 中的 App Service 方案 | Microsoft Docs
description: 了解如何執行不同的工作來管理 App Service 方案。
keywords: App Service, Azure App Service, 級別, App Service方案, 變更, 建立, 管理, 管控
services: app-service
documentationcenter: ''
author: cephalin
manager: cfowler
editor: ''
ms.assetid: 4859d0d5-3e3c-40cc-96eb-f318b2c51a3d
ms.service: app-service
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 11/09/2017
ms.author: cephalin
ms.openlocfilehash: 2c08522df598bd5c6313c3f026efe48e1c4a2c56
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39449354"
---
# <a name="manage-an-app-service-plan-in-azure"></a>在 Azure 建立管理 App Service 方案

[Azure App Service 方案](azure-web-sites-web-hosting-plans-in-depth-overview.md)提供 App Service 應用程式執行所需的資源。 本指南說明如何管理 App Service 方案。

## <a name="create-an-app-service-plan"></a>建立應用程式服務方案

> [!TIP]
> 如果您有 App Service 環境，請參閱[在 App Service 環境中建立 App Service 方案](environment/app-service-web-how-to-create-a-web-app-in-an-ase.md#createplan)。

您可建立空白的 App Service 方案，或是在應用程式建立期間建立方案。

1. 在 [Azure 入口網站](https://portal.azure.com)中，選取 [新增] > [Web + 行動]，然後選取 [Web 應用程式] 或其他 App Service 應用程式種類。

1. 選取現有的 App Service 方案或為新應用程式建立方案。

   ![在 Azure 入口網站中建立應用程式。][createWebApp]

   若要建立方案：

   a. 選取[[+] 建立新項目]。

      ![建立 App Service 方案。][createASP] 

   b. 針對 [App Service 方案]，輸入方案的名稱。

   c. 針對 [位置] ，選取適當的位置。

   d. 針對 [定價層]，為服務選取適當的定價層。 選取 [檢視全部] 以檢視其他價格選項，例如 [免費] 和 [共用]。 選取定價層後，請按一下 [選取]  按鈕。

<a name="move"></a>

## <a name="move-an-app-to-another-app-service-plan"></a>將應用程式移到不同的 App Service 方案

您也可以將應用程式移到另一個 App Service 方案，只要來源方案和目標方案位於「相同的資源群組和地理區域」即可。

1. 在 [Azure 入口網站](https://portal.azure.com)中，瀏覽至您要移動的應用程式。

1. 在功能表中，尋找 [App Service 方案] 區段。

1. 選取 [變更 App Service 方案] 以開啟 [App Service 方案] 選取器。

   ![App Service 方案選取器。][change] 

1. 在 [App Service 方案] 選取器中，選取現有的方案以移動此應用程式。   

> [!IMPORTANT]
> [選取 App Service 方案] 頁面會依下列準則進行篩選： 
> - 存在相同的資源群組內 
> - 存在相同的地理區域內 
> - 存在相同的網路空間內  
> 
> 「網路空間」是 App Service 內的邏輯結構，會定義伺服器資源的群組。 地理區域 (例如美國西部) 包含許多網路空間，可配置使用 App Service 的客戶。 目前，您無法在網路空間之間移動 App Service 資源。 
> 

[!INCLUDE [app-service-dev-test-note](../../includes/app-service-dev-test-note.md)]

每個方案都有其專屬定價層。 例如，如果將網站從**免費**層移至**標準**層，則所有指派給它的應用程式都可以使用**標準**層的功能和資源。 不過，將應用程式從較高層的方案移至較低層的方案，表示您無法再存取特定功能。 如果您的應用程式使用目標方案未提供的功能，您會收到顯示哪些使用中功能無法使用的錯誤。 

例如，如果您其中一個應用程式使用 SSL 憑證，您可能會看到此錯誤訊息：

`Cannot update the site with hostname '<app_name>' because its current SSL configuration 'SNI based SSL enabled' is not allowed in the target compute mode. Allowed SSL configuration is 'Disabled'.`

在此情況下，在您將應用程式移至目標方案之前，必須執行下列其中一個動作：
- 將目標方案的定價層規模調升為**基本**或更高。
- 移除應用程式的所有 SSL 連線。

## <a name="move-an-app-to-a-different-region"></a>將應用程式移到不同的區域

您的應用程式執行所在的區域，就是 App Service 方案所在的區域。 不過，您無法變更 App Service 方案區域。 如果您想在不同的區域執行應用程式，有一個替代方法是複製應用程式。 複製會在任何區域中的新或現有 App Service 方案中產生您應用程式的複本。

您可以在功能表的 [開發工具] 區段中找到 [複製應用程式]。

> [!IMPORTANT]
> 複製有一些限制。 您可在 [Azure App Service 應用程式複製](app-service-web-app-cloning.md)中閱讀相關限制。

## <a name="scale-an-app-service-plan"></a>調整 App Service 方案

若要相應增加 App Service 方案的定價層，請參閱[在 Azure 中相應增加應用程式](web-sites-scale.md)。

若要相應縮小方案的執行個體計數，請參閱[手動或自動調整執行個體計數](../monitoring-and-diagnostics/insights-how-to-scale.md)。

<a name="delete"></a>

## <a name="delete-an-app-service-plan"></a>刪除 App Service 方案

為避免非預期費用，當您刪除 App Service 方案中的最後一個應用程式時，App Service 預設也會刪除此方案。 如果您選擇改為保留方案，您應該將方案變更為**免費**層，以避免產生費用。

> [!IMPORTANT]
> 沒有相關聯應用程式的 App Service 方案仍會產生費用，因為它們會繼續保留已設定的 VM 執行個體。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [在 Azure 中相應增加應用程式的規模](web-sites-scale.md)

[change]: ./media/azure-web-sites-web-hosting-plans-in-depth-overview/change-appserviceplan.png
[createASP]: ./media/azure-web-sites-web-hosting-plans-in-depth-overview/create-appserviceplan.png
[createWebApp]: ./media/azure-web-sites-web-hosting-plans-in-depth-overview/create-web-app.png
