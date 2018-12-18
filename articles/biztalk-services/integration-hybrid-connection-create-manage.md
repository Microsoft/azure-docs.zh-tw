---
title: 建立和管理混合式連接 | Microsoft Docs
description: 了解如何建立混合式連線、管理連線和安裝 Hybrid Connection Manager。 MABS，WABS
services: biztalk-services
documentationcenter: ''
author: MandiOhlinger
manager: erikre
editor: ''
ms.assetid: aac0546b-3bae-41e0-b874-583491a395ea
ms.service: biztalk-services
ms.workload: integration
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 10/18/2016
ms.author: ccompy
ms.openlocfilehash: 1751d33b5f6f6a506654daedd15bbd75ae271483
ms.sourcegitcommit: e266df9f97d04acfc4a843770fadfd8edf4fa2b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/11/2017
ms.locfileid: "26736960"
---
# <a name="create-and-manage-hybrid-connections"></a>建立和管理混合式連線

> [!IMPORTANT]
> BizTalk 混合式連線已停用，並以 App Service 混合式連線取代。 如需詳細資訊，包括如何管理您現有的 BizTalk 混合式連線，請參閱 [Azure App Service 混合式連線](../app-service/app-service-hybrid-connections.md)。

>[!INCLUDE [Use APIs to manage MABS](../../includes/biztalk-services-retirement-azure-classic-portal.md)]

## <a name="overview-of-the-steps"></a>步驟概觀
1. 輸入私人網路中內部部署資源的 **host name** 或 **FQDN** of the on-premises resource in your private netw或k.
2. 將 Azure Web Apps 或 Azure Mobile Apps 連結至混合式連線。
3. 安裝混合式連線管理員在內部部署資源上，並連接至特定的混合式連線。 Azure 入口網站提供按一下即可安裝和連接的體驗。
4. 管理混合式連線及其連接金鑰。

本主題將列出這些步驟。 

> [!IMPORTANT]
> 可以將「混合式連線」端點設定為 IP 位址。 如果您使用 IP 位址，視您的用戶端而言，您不一定會觸達內部部署資源。 「混合式連線」取決於執行 DNS 查閱的用戶端。 在大部分情況下， **用戶端** 是您的應用程式程式碼。 如果用戶端不執行 DNS 查閱 (不會嘗試解析 IP 位址，彷彿它是網域名稱 (x.x.x.x) 一樣)，則不會透過「混合式連線」傳送流量。
> 
> 例如 (虛擬程式碼)，您會定義 **10.4.5.6** 做為內部部署主機︰
> 
> **下列案例可運作︰**  
> `Application code -> GetHostByName("10.4.5.6") -> Resolves to 127.0.0.3 -> Connect("127.0.0.3") -> Hybrid Connection -> on-prem host`
> 
> **下列案例無法運作︰**  
> `Application code -> Connect("10.4.5.6") -> ?? -> No route to host`
> 
> 

## <a name="CreateHybridConnection"></a>建立混合式連線
您可以 [Azure App Service 混合式連線](../app-service/app-service-hybrid-connections.md)中建立混合式連線**或**使用 [BizTalk 服務 REST API](https://msdn.microsoft.com/library/azure/dn232347.aspx)。 

<!-- **To create Hybrid Connections using Web Apps**, see [Connect Azure Web Apps to an On-Premises Resource](../app-service-web/web-sites-hybrid-connection-get-started.md). You can also install the Hybrid Connection Manager (HCM) from your web app, which is the preferred method.  -->

#### <a name="additional"></a>其他
* 可建立多個混合式連線。 請參閱「 [BizTalk 服務：版本圖表](biztalk-editions-feature-chart.md) 」(英文) 瞭解允許的連接數量。 
* 每個「混合式連線」都是由一對連接字串建立而成：分別是負責「傳送」的應用程式金鑰，以及負責「接聽」的內部部署金鑰。 每一對都有「主要」和「次要」金鑰。 

## <a name="LinkWebSite"></a>連結 Azure App Service Web 應用程式或行動應用程式
若要將 Azure App Service 中的 Web 應用程式或行動應用程式連結至現有的「混合式連線」，請在 [混合式連線] 刀鋒視窗中選取 [使用現有的混合式連線]。 
<!-- See [Access on-premises resources using hybrid connections in Azure App Service](../app-service-web/web-sites-hybrid-connection-get-started.md). -->

## <a name="InstallHCM"></a>在內部部署安裝混合式連線管理員
建立混合式連線之後，請在內部部署資源上安裝混合式連線管理員。 可從 Azure Web Apps 或 BizTalk 服務下載此程式。 

[!INCLUDE [Use APIs to manage MABS](../../includes/biztalk-services-retirement-azure-classic-portal.md)]
 
[Azure App Service 混合式連線](../app-service/app-service-hybrid-connections.md)也是很好的資源。

<!--
You can also download the Hybrid Connection Manager MSI file and copy the file to your on-premises resource. Specific steps:

1. Copy the on-premises primary Connection String. See [Manage Hybrid Connections](#ManageHybridConnection) in this topic for the specific steps.
2. Download the Hybrid Connection Manager MSI file. 
3. On the on-premises resource, install the Hybrid Connection Manager from the MSI file. 
4. Using Windows PowerShell, type: 
> Add-HybridConnection -ConnectionString “*Your On-Premises Connection String that you copied*” 
--> 

#### <a name="additional"></a>其他
* 混合式連線管理員可以安裝在以下作業系統上︰
  
  * Windows Server 2008 R2 (須有 .NET Framework 4.5+ 和 Windows Management Framework 4.0+)
  * Windows Server 2012 (須有 Windows Management Framework 4.0+)
  * Windows Server 2012 R2
* 安裝混合式連線管理員之後的情況如下： 
  
  * 裝載於 Azure 的混合式連線會自動設定為使用「主要應用程式連接字串」。 
  * 內部部署資源會自動設定為使用「主要內部部署連接字串」。
* 混合式連線管理員必須使用有效的內部部署連接字串，才能進行授權。 Azure Web Apps 或 Azure Mobile Apps 必須使用有效的應用程式連接字串，才能進行授權。
* 您可以在另一部伺服器上安裝另一個混合式連線管理員的執行個體，藉此調整混合式連線。 設定內部部署接聽程式，以使用與第一個內部部署接聽程式相同的位址。 在此情況下，流量會隨機分散 (循環配置資源) 到作用中的內部部署接聽程式之間。 

## <a name="ManageHybridConnection"></a>管理混合式連線

[!INCLUDE [Use APIs to manage MABS](../../includes/biztalk-services-retirement-azure-classic-portal.md)] 

[Azure App Service 混合式連線](../app-service/app-service-hybrid-connections.md)也是很好的資源。

#### <a name="copyregenerate-the-hybrid-connection-strings"></a>複製/重新產生混合式連線字串

[!INCLUDE [Use APIs to manage MABS](../../includes/biztalk-services-retirement-azure-classic-portal.md)] 

[Azure App Service 混合式連線](../app-service/app-service-hybrid-connections.md)也是很好的資源。

#### <a name="use-group-policy-to-control-the-on-premises-resources-used-by-a-hybrid-connection"></a>使用群組原則來控制混合式連線使用的內部部署資源
1. 下載 [混合式連線管理員的系統管理範本](http://www.microsoft.com/download/details.aspx?id=42963)。
2. 將檔案解壓縮。
3. 在修改群組原則的電腦上，執行下列動作：  
   
   * 將 .ADMX 檔案複製到 %WINROOT%\PolicyDefinitions 資料夾。
   * 將 .ADML 檔案複製到 %WINROOT%\PolicyDefinitions\en-us 資料夾。

複製之後，您可以使用群組原則編輯器來變更原則。

## <a name="next"></a>下一頁
[混合式連線概觀](integration-hybrid-connection-overview.md)

## <a name="see-also"></a>另請參閱
[用於管理 Microsoft Azure 上之 BizTalk 服務的 REST API](http://msdn.microsoft.com/library/azure/dn232347.aspx)  
[BizTalk 服務：版本圖表](biztalk-editions-feature-chart.md)  
[建立 BizTalk 服務](biztalk-provision-services.md)  
[BizTalk 服務：儀表板、監視和調整索引標籤](biztalk-dashboard-monitor-scale-tabs.md)

[HybridConnectionTab]: ./media/integration-hybrid-connection-create-manage/WABS_HybridConnectionTab.png
[HCOnPremSetup]: ./media/integration-hybrid-connection-create-manage/WABS_HybridConnectionOnPremSetup.png
[HCManageConnection]: ./media/integration-hybrid-connection-create-manage/WABS_HybridConnectionManageConn.png 
