---
title: 開始使用 Microsoft Power BI 工作區集合
description: Power BI 工作區集合是一項 Azure 服務，可讓應用程式開發人員將互動式 Power BI 報告加入至自己的應用程式。
services: power-bi-embedded
author: markingmyname
ROBOTS: NOINDEX
ms.assetid: 4787cf44-5d1c-4bc3-b3fd-bf396e5c1176
ms.service: power-bi-embedded
ms.topic: conceptual
ms.workload: powerbi
ms.date: 09/25/2017
ms.author: maghan
ms.openlocfilehash: b565955abaae44efe20291a6e46b57a5fec941d1
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43046553"
---
# <a name="get-started-with-microsoft-power-bi-workspace-collections"></a>開始使用 Microsoft Power BI 工作區集合

**Power BI 工作區集合**是一項 Azure 服務，可讓應用程式開發人員將互動式 Power BI 報告加入至自己的應用程式。 **Power BI 工作區集合**會與現有的應用程式一同運作，而不需要重新設計或變更使用者登入的方式。

> [!IMPORTANT]
> Power BI 工作區集合已被取代，只能使用到 2018 年 6 月或您的合約所指出的時間。 建議您進行規劃以移轉至 Power BI Embedded，以免應用程式發生中斷。 如需如何將資料移轉至 Power BI Embedded 的資訊，請參閱[如何將 Power BI 工作區集合的內容移轉至 Power BI Embedded](https://powerbi.microsoft.com/documentation/powerbi-developer-migrate-from-powerbi-embedded/)。

**Microsoft Power BI 工作區集合**的資源可透過 [Azure Resource Manager API](https://msdn.microsoft.com/library/mt712306.aspx) 來佈建。 在此案例中，所佈建的資源是 **Power BI 工作區集合**。

![Power BI 工作區集合的一般流程](media/get-started/introduction.png)

## <a name="create-a-workspace-collection"></a>建立工作區集合

**工作區集合** 是最上層的 Azure 資源，而內容的容器將會內嵌在您的應用程式中。 建立**工作區集合**的方式有兩種︰

* 以手動方式使用 Azure 入口網站
* 以程式設計方式使用 Azure Resource Manager API

讓我們逐步解說使用 Azure 入口網站建立**工作區集合**的步驟。

1. 開啟 **Azure 入口網站**並登入：[http://portal.azure.com](http://portal.azure.com)。
2. 選取頂端面板上的 [+ 新增]。
   
   ![Azure 入口網站內的 [+ 新增]](media/get-started/create-workspace-1.png)
3. 在 [資料 + 分析] 底下，選取 [Power BI 工作區集合]。
4. 在開始使用訊息中，如果您已有現有的 Power BI 工作區集合訂用帳戶，請選取底部的 [建立工作區集合]。

5. 在 [工作區集合] 上輸入必要資訊。
   
   ![工作區集合建立](media/get-started/create-workspace-2.png)
1. 選取 [建立] 。

**工作區集合**會花費一些時間來佈建。 完成後，您將會進入 [工作區集合]。

   ![Azure 入口網站中的工作區集合](media/get-started/create-workspace-3.png)

[建立] 結果包含呼叫 API 所需的資訊，以建立工作區並將內容部署到這些工作區。

<a name="view-access-keys"/>

## <a name="view-power-bi-api-access-keys"></a>檢視 Power BI API 存取金鑰

呼叫 Power BI REST API 所需的其中一項最重要資訊是**存取金鑰**。 這些存取金鑰用來產生**應用程式權杖**，而這些權杖用來驗證 API 要求。 若要檢視您的**存取金鑰**，請按一下 [設定] 上的 [存取金鑰]。 若要深入了解**應用程式權杖**，請參閱[使用 Power BI 工作區集合進行驗證和授權](app-token-flow.md)。

   ![Azure 入口網站之工作區集合設定內的存取金鑰](media/get-started/access-keys.png)

您會發現您有兩個金鑰。

   ![存取金鑰內的兩個金鑰](media/get-started/access-keys-2.png)

複製這些金鑰並將它們安全地儲存在您的應用程式中。 請務必如同密碼一樣處理這些金鑰，因為這些金鑰可供存取您的**工作區集合**中的所有內容。

雖已列出兩個金鑰，但特定時間只需要一個金鑰。 系統會提供第二個金鑰，以便您定期重新產生金鑰，而不需中斷對服務的存取。

您現在已有應用程式的 Power BI 執行個體以及 **存取金鑰**，您可以將報告匯入自己的應用程式中。 在了解如何匯入報告之前，下一節說明如何建立要內嵌到應用程式中的 Power BI 資料集和報告。

## <a name="working-with-workspaces"></a>使用工作區

建立工作區集合之後，您必須建立將存放報告和資料集的工作區。 若要建立工作區，您必須使用 [Post Workspace REST API](https://msdn.microsoft.com/library/azure/mt711503.aspx)。

## <a name="create-power-bi-datasets-and-reports-to-embed-into-an-app-using-power-bi-desktop"></a>使用 Power BI Desktop 建立要內嵌到應用程式中的 Power BI 資料集和報告

您現已為您的應用程式建立 Power BI 執行個體，而且有**存取金鑰**，您必須建立想要內嵌的 Power BI 資料集和報告。 使用 **Power BI Desktop**可以建立資料集和報告。 您可以下載 [免費的 Power BI Desktop](https://go.microsoft.com/fwlink/?LinkId=521662)。 或者，若要快速開始，您可以下載 [零售分析範例 PBIX](http://go.microsoft.com/fwlink/?LinkID=780547)。

> [!NOTE]
> 若要深入了解如何使用 **Power BI Desktop**，請參閱[開始使用 Power BI Desktop](https://powerbi.microsoft.com/guided-learning/powerbi-learning-0-2-get-started-power-bi-desktop)。

透過 **Power BI Desktop**，將資料複本匯入 **Power BI Desktop** 或使用 **DirectQuery** 直接連接到資料來源，即可連接到資料來源。

以下是使用 **Import** 和 **DirectQuery** 之間的差異。

| Import | DirectQuery |
| --- | --- |
| 資料表、資料行和資料  會匯入或複製到 **Power BI Desktop**中。 當您使用視覺效果時， **Power BI Desktop** 會查詢資料的複本。 若要查看基礎資料所發生的任何變更，您必須重新整理或再次匯入完整的目前資料集。 |只有資料表和資料行  會匯入或複製到 **Power BI Desktop**中。 當您使用視覺效果時， **Power BI Desktop** 會查詢基礎資料來源，這表示您一直在檢視目前的資料。 |

如需有關連接到資料來源的詳細資訊，請參閱 [連接到資料來源](connect-datasource.md)。

在 **Power BI Desktop**中儲存您的工作之後，會建立一個 PBIX 檔案。 此檔案包含您的報告。 此外，如果您匯入資料，則 PBIX 會包含完整的資料集，或如果您使用 **DirectQuery**，則 PBIX 只包含資料集結構描述。 您會使用 [Power BI Import API](https://msdn.microsoft.com/library/mt711504.aspx)，以程式設計方式將 PBIX 部署到您工作區。

> [!NOTE]
> **Power BI 工作區集合**有其他 API 可變更您的資料集所指向的伺服器和資料庫，以及設定資料集將用來連接到您的資料庫的服務帳戶認證。 請參閱 [Post SetAllConnections](https://msdn.microsoft.com/library/mt711505.aspx) 和 [Patch 閘道資料來源](https://msdn.microsoft.com/library/mt711498.aspx)。

## <a name="create-power-bi-datasets-and-reports-using-apis"></a>使用 API 建立 Power BI 資料集和報告

### <a name="datasets"></a>資料集

您可以使用 REST API 在 Power BI 工作區集合內建立資料集。 接著，將資料推送至資料集。 這可讓您不需要 Power BI Desktop 即可處理資料。 如需詳細資訊，請參閱[公佈資料集](https://msdn.microsoft.com/library/azure/mt778875.aspx)。

### <a name="reports"></a>報告

您可以使用 JavaScript API，直接在應用程式中從資料集建立報告。 如需詳細資訊，請參閱[在 Power BI 工作區集合中從資料集建立新的報告](create-report-from-dataset.md)。

## <a name="see-also"></a>另請參閱

[開始使用範例](get-started-sample.md)  
[在 Power BI 工作區集合中驗證和授權](app-token-flow.md)  
[內嵌報告](embed-report.md)  
[在 Power BI 工作區集合中從資料集建立新的報告](create-report-from-dataset.md)
[儲存報告](save-reports.md)  
[Power BI Desktop](https://powerbi.microsoft.com/documentation/powerbi-desktop-get-the-desktop/)  
[JavaScript 內嵌範例](https://microsoft.github.io/PowerBI-JavaScript/demo/)  

有其他疑問？ [試用 Power BI 社群](http://community.powerbi.com/)

