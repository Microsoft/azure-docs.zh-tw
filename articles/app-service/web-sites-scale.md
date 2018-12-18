---
title: 在 Azure 中相應增加應用程式的規模 | Microsoft Docs
description: 了解如何在 Azure App Service 中相應增加應用程式的規模，以增加容量和功能。
services: app-service
documentationcenter: ''
author: cephalin
manager: erikre
editor: mollybos
ms.assetid: f7091b25-b2b6-48da-8d4a-dcf9b7baccab
ms.service: app-service
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/05/2016
ms.author: cephalin
ms.openlocfilehash: 79450cdd0928304c3b98cf13f8aaca7a1bf11d33
ms.sourcegitcommit: a3a0f42a166e2e71fa2ffe081f38a8bd8b1aeb7b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/01/2018
ms.locfileid: "43381930"
---
# <a name="scale-up-an-app-in-azure"></a>在 Azure 中相應增加應用程式的規模

> [!NOTE]
> 新 **PremiumV2** 層提供更快速的 CPU、SSD 儲存體，而且記憶體與核心的比例是現有定價層的兩倍。 有此效能優勢，您可在較少的執行個體上執行應用程式來節省成本。 若要相應增加 **PremiumV2** 層，請參閱[設定 App Service 的 PremiumV2 層](app-service-configure-premium-tier.md)。
>

本文將說明如何在 Azure App Service 中相應增加應用程式的規模。 有兩個工作流程適合用來相應增加和相應放大規模，而本文說明相應增加工作流程。

* [相應增加](https://en.wikipedia.org/wiki/Scalability#Horizontal_and_vertical_scaling)︰取得更多的 CPU、記憶體、磁碟空間和額外的功能，例如專用虛擬機器 (VM)、自訂網域和憑證、預備位置，以及自動調整等等。 您可以藉由變更應用程式所屬的 App Service 方案定價層來相應增加。
* [相應放大](https://en.wikipedia.org/wiki/Scalability#Horizontal_and_vertical_scaling)︰增加執行您的應用程式的 VM 執行個體數目。
  視您的定價層而定，最多可以相應放大至 20 個執行個體。 **隔離**層中的 [App Service 環境](environment/intro.md)，進一步將您的相應放大計數增加到 100 個執行個體。 如需相應放大的詳細資訊，請參閱[手動或自動調整執行個體計數](../monitoring-and-diagnostics/insights-how-to-scale.md)。 您可以在該文章中了解如何使用自動調整，也就是根據預先定義的規則與排程，自動調整執行個體計數。

這些調整設定只需幾秒鐘便能套用，且影響範圍遍及 [App Service 方案](../app-service/azure-web-sites-web-hosting-plans-in-depth-overview.md)內的所有應用程式。
在此過程中，您不需要變更程式碼或重新部署應用程式。

如需各 App Service 方案價格資訊及功能的詳細資訊，請參閱 [App Service 價格詳細資料](https://azure.microsoft.com/pricing/details/web-sites/)。  

> [!NOTE]
> 從 **免費** 層切換 App Service 方案之前，您必須先適當地移除 Azure 訂用帳戶的 [消費限制](https://azure.microsoft.com/pricing/spending-limits/) 。 若要檢視或變更 Microsoft Azure App Service 訂用帳戶的選項，請參閱 [Microsoft Azure 訂訂用帳戶][azuresubscriptions]。
> 
> 

<a name="scalingsharedorbasic"></a>
<a name="scalingstandard"></a>

## <a name="scale-up-your-pricing-tier"></a>相應增加您的定價層
1. 在瀏覽器中，開啟 [Azure 入口網站][portal]。
2. 在 App Service 應用程式頁面中，按一下 [所有設定]，然後按一下 [Scale Up]\(相應增加\)。
   
    ![瀏覽以相應增加您的 Azure 應用程式規模。][ChooseWHP]
3. 選擇您的定價層，然後按一下 [套用]。
   
    當操作完成時，[通知] 索引標籤會有綠色的「成功」字樣閃爍顯示。

<a name="ScalingSQLServer"></a>

## <a name="scale-related-resources"></a>調整相關資源
如果您的應用程式相依於其他服務 (如 Azure SQL Database 或 Azure 儲存體)，您可以單獨相應增加這些資源。 這些資源不受 App Service 方案管理。

1. 在 [基本功能] 中，按一下 [資源群組] 連結。
   
    ![相應增加 Azure 應用程式的相關資源](./media/web-sites-scale/RGEssentialsLink.png)
2. 在 [資源群組] 頁面的 [摘要] 組件中，按一下您想要調整的資源。 以下螢幕擷取畫面顯示 SQL Database 資源和 Azure 儲存體資源。
   
    ![巡覽至資源群組頁面以相應增加 Azure 應用程式的規模](./media/web-sites-scale/ResourceGroup.png)
3. 針對 SQL Database 資源，按一下 [設定]  >  [定價層] 來調整定價層。
   
    ![相應增加 Azure 應用程式的 SQL Database 後端](./media/web-sites-scale/ScaleDatabase.png)
   
    您也可以針對 SQL 資料庫執行個體開啟 [異地複寫](../sql-database/sql-database-geo-replication-overview.md) 。
   
    針對 Azure 儲存體資源，按一下 [設定]  >  [組態]，以便相應增加您的儲存體選項。
   
    ![相應增加您 Azure 應用程式所使用的 Azure 儲存體帳戶](./media/web-sites-scale/ScaleStorage.png)

<a name="OtherFeatures"></a>
<a name="devfeatures"></a>

## <a name="compare-pricing-tiers"></a>比較定價層
如需詳細資訊，例如每個定價層的 VM 大小，請參閱 [App Service 定價詳細資料](https://azure.microsoft.com/pricing/details/web-sites/)。
如需服務限制、配額及條件約束的表格，以及每層支援的功能，請參閱 [App Service 限制](../azure-subscription-service-limits.md#app-service-limits)。

<a name="Next Steps"></a>

## <a name="next-steps"></a>後續步驟
* 如需價格、支援及 SLA 的相關資訊，請參閱以下連結：
  
    [資料傳輸定價詳細資料](https://azure.microsoft.com/pricing/details/data-transfers/)
  
    [Microsoft Azure 支援方案](https://azure.microsoft.com/support/plans/)
  
    [服務等級協定](https://azure.microsoft.com/support/legal/sla/)
  
    [SQL Database 定價詳細資料](https://azure.microsoft.com/pricing/details/sql-database/)
  
    [Microsoft Azure 的虛擬機器和雲端服務大小][vmsizes]
  
* 如需 Azure App Service 最佳作法 (包括建置可調整且具彈性的架構) 的詳細資訊，請參閱 [最佳作法：Azure App Service Web Apps](https://azure.microsoft.com/blog/best-practices-windows-azure-websites-waws/)。
* 如需調整 App Service 應用程式的相關影片，請參閱下列資源：
  
  * [何時該調整 Azure 網站 - Stefan Schackow](https://azure.microsoft.com/resources/videos/azure-web-sites-free-vs-standard-scaling/)
  * [自動調整 Azure 網站、CPU 或排程 - Stefan Schackow](https://azure.microsoft.com/resources/videos/auto-scaling-azure-web-sites/)
  * [如何調整 Azure 網站 - Stefan Schackow](https://azure.microsoft.com/resources/videos/how-azure-web-sites-scale/)

<!-- LINKS -->
[vmsizes]:https://azure.microsoft.com/pricing/details/app-service/
[SQLaccountsbilling]:http://go.microsoft.com/fwlink/?LinkId=234930
[azuresubscriptions]:https://account.windowsazure.com/subscriptions
[portal]: https://portal.azure.com/

<!-- IMAGES -->
[ChooseWHP]: ./media/web-sites-scale/scale1ChooseWHP.png
[ResourceGroup]: ./media/web-sites-scale/scale10ResourceGroup.png
[ScaleDatabase]: ./media/web-sites-scale/scale11SQLScale.png
[GeoReplication]: ./media/web-sites-scale/scale12SQLGeoReplication.png
