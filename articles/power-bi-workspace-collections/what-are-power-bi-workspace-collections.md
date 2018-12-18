---
title: 什麼是 Power BI 工作區集合？
description: Power BI Embedded 可讓您將 Power BI 報告整合到 Web 或行動應用程式中，您就不需要建置自訂解決方案。
services: power-bi-embedded
author: markingmyname
ROBOTS: NOINDEX
ms.assetid: 03649b72-b7d7-40ca-b077-12356d72d4f3
ms.service: power-bi-embedded
ms.topic: article
ms.workload: powerbi
ms.date: 09/20/2017
ms.author: maghan
ms.openlocfilehash: 73b13dcea84ba322072217c3feacebdefb3bf1e6
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43052034"
---
# <a name="what-are-power-bi-workspace-collections"></a>什麼是 Power BI 工作區集合？

運用 **Power BI 工作區集合**，您可以將 Power BI 報表整合至您的 Web 應用程式或行動應用程式。

![應用程式圖表](media/what-are-power-bi-workspace-collections/what-is.png)

> [!IMPORTANT]
> Power BI 工作區集合已被取代，只能使用到 2018 年 6 月或您的合約所指出的時間。 建議您進行規劃以移轉至 Power BI Embedded，以免應用程式發生中斷。 如需如何將資料移轉至 Power BI Embedded 的資訊，請參閱[如何將 Power BI 工作區集合的內容移轉至 Power BI Embedded](https://powerbi.microsoft.com/documentation/powerbi-developer-migrate-from-powerbi-embedded/)。

Power BI 工作區集合是一個 **Azure 服務**，可讓 ISV 和應用程式開發人員在其應用程式中呈現 Power BI 資料體驗。 身為開發人員，您已建置應用程式，這些應用程式有它們自己的使用者和個別的功能集。 這些應用程式也可能已經內建一些資料元素，例如，現在可以由 Microsoft Power BI 工作區集合提供的圖表和報表。 您不需要 Power BI 帳戶就可以使用您的應用程式。 您可以像以前一樣繼續登入您的應用程式，以及在不需要任何額外授權的情況下，檢視 Power BI 記錄體驗並與它們互動。

## <a name="licensing-for-microsoft-power-bi-workspace-collections"></a>針對 Microsoft Power BI 工作區集合進行授權

在 **Microsoft Power BI 工作區集合**使用方式模型中，針對 Power BI 進行授權並不是使用者的責任。  而是由使用視覺效果之應用程式的開發人員購買**工作階段**，並向擁有那些資源的訂用帳戶收費。 

## <a name="microsoft-power-bi-workspace-collections-conceptual-model"></a>Microsoft Power BI 工作區集合概念模型

![應用程式流程與工作區集合](media/what-are-power-bi-workspace-collections/model.png)

就像 Azure 中的任何其他服務，Power BI 工作區集合的資源也會透過 [Azure Resource Manager API](https://msdn.microsoft.com/library/mt712306.aspx) 來佈建。 在此情況下，所佈建的資源是 **Power BI 工作區集合**。

## <a name="workspace-collection"></a>工作區集合

**工作區集合**是包含 0 個或更多**工作區**之資源的最上層 Azure 容器。  **工作區****集合**擁有所有標準 Azure 屬性，以及下列項目：

* **存取金鑰** – 安全呼叫 Power BI API 時使用的金鑰 (會在稍後的小節中說明)。
* **使用者** – 具有管理權現，可透過 Azure 入口網站或 Azure Resource Manager API 管理 Power BI 工作區集合的 Azure Active Directory (AAD) 使用者。
* **區域** – 佈建**工作區集合**時，您可以選取要佈建的區域。 如需詳細資訊，請參閱 [Azure 地區](https://azure.microsoft.com/regions/)。

## <a name="workspace"></a>工作區

**工作區**是 Power BI 內容的容器，可包括資料集和報告。 **工作區** 在第一次建立時是空白的。 您將會使用 Power BI Desktop 編寫內容，且您會使用 [Power BI Import API](https://msdn.microsoft.com/library/mt711504.aspx) 將 PBIX 自動部署至您的工作區中。 您也可以透過程式設計方式建立資料集，然後在應用程式內建立報告，而不是使用 Power BI Desktop 來建立。

## <a name="using-workspace-collections-and-workspaces"></a>使用工作區集合和工作區

**工作區集合**與**工作區**是內容的容器，您可以針對您正在建立之應用程式的設計，採用任何最佳方式來使用及組織內容。 將會有許多不同的方式可供您管理它們內部的內容。 您可以選擇將所有內容放在某一個工作區內，然後之後再使用應用程式權杖進一步在您的客戶之間細分內容。 您也可以選擇將所有客戶放在個別工作區中，這樣一來他們之間就會有一些區隔。 或者，您可選擇依據地區 (而非依據客戶) 來組織使用者。 這個彈性設計可讓您選擇最佳的內容組織方式。

## <a name="cached-datasets"></a>快取的資料集

您可以使用快取的資料集。  但是，將快取的資料載入 **Microsoft Power BI 工作區集合**之後，您就無法重新整理該資料。 快取的資料集表示您已將資料匯入 Power BI Desktop，而不是使用 DirectQuery。

## <a name="authentication-and-authorization-with-app-tokens"></a>應用程式權杖中的驗證與授權

**Microsoft Power BI 工作區集合**會聽從您的應用程式來執行所有必要的使用者驗證和授權。 並沒有明確要求您的使用者必須是 Azure Active Directory (Azure AD) 的客戶。  您的應用程式會改為透過使用**應用程式驗證權杖 (應用程式權杖)**，將轉譯 Power BI 報表的授權出示給 **Microsoft Power BI 工作區集合**。  這些 **應用程式權杖** 會在您的應用程式想要轉譯報表時視需要建立。

![應用程式權杖使用方式圖表](media/what-are-power-bi-workspace-collections/app-tokens.png)

**應用程式驗證權杖 (應用程式權杖)** 是用來針對 **Microsoft Power BI 工作區集合**進行驗證。  應用程式權杖 有三種類型：

1. 佈建權杖 - 將新的「工作區」佈建到「工作區集合」時使用
2. 開發權杖 - 在直接呼叫 **Power BI REST API**
3. 內嵌權杖 - 在進行呼叫以在內嵌的 iframe 中轉譯報表時使用

這些權杖會在您和 **Microsoft Power BI 工作區集合**互動時的各種不同階段使用。  設計權杖的目的是為了讓您從您的應用程式委派權限給 Power BI。 如需詳細資訊，請參閱 [應用程式權杖流程](app-token-flow.md)。

## <a name="create-or-edit-reports-within-your-application"></a>在應用程式內建立或編輯報告

您現在可以直接在應用程式中編輯現有報表或建立新報表，而不必使用 Power BI Desktop。 這需要工作區內已存在資料集。

## <a name="see-also"></a>另請參閱

[常見的 Microsoft Power BI 工作區集合案例](scenarios.md)  
[開始使用 Microsoft Power BI 工作區集合](get-started.md)  
[開始使用範例](get-started-sample.md)  
[內嵌報告](embed-report.md)  
[在 Power BI 工作區集合中驗證和授權](app-token-flow.md)  
[JavaScript 內嵌範例](https://microsoft.github.io/PowerBI-JavaScript/demo/)  
[PowerBI-CSharp Git 存放庫](https://github.com/Microsoft/PowerBI-CSharp)  
[PowerBI-Node Git存放庫](https://github.com/Microsoft/PowerBI-Node)  

有其他疑問？ [試用 Power BI 社群](http://community.powerbi.com/)
