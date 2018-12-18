---
title: 混合式連線概觀 | Microsoft Docs
description: 了解混合式連線、安全性、TCP 連接埠和支援的組態。 MABS，WABS。
services: biztalk-services
documentationcenter: ''
author: MandiOhlinger
manager: erikre
editor: ''
ms.assetid: 216e4927-6863-46e7-aa7c-77fec575c8a6
ms.service: biztalk-services
ms.workload: integration
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: get-started-article
ms.date: 10/18/2016
ms.author: ccompy
ms.openlocfilehash: 819af52bb10c9ffcb7e1133437f6d0afbe6105ae
ms.sourcegitcommit: e266df9f97d04acfc4a843770fadfd8edf4fa2b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/11/2017
ms.locfileid: "26629681"
---
# <a name="hybrid-connections-overview"></a>混合式連線概觀

> [!IMPORTANT]
> BizTalk 混合式連線已停用，並以 App Service 混合式連線取代。 如需詳細資訊，包括如何管理您現有的 BizTalk 混合式連線，請參閱 [Azure App Service 混合式連線](../app-service/app-service-hybrid-connections.md)。

簡介混合式連線、列出支援的組態，並列出需要的 TCP 連接埠。

## <a name="what-is-a-hybrid-connection"></a>什麼是混合式連線
混合式連線是 Azure BizTalk 服務的功能。 混合式連線可讓您簡單又方便地將 Azure App Service 中的 Web Apps 功能 (前身為網站) 和 Azure App Service 中的 Mobile Apps 功能 (前身為行動服務)，連線到防火牆後的內部部署資源。

![混合式連線][HCImage]

混合式連線的優點包括：

* Web Apps 和 Mobile Apps 可以安全地存取現有的內部部署資料和服務。
* 多個 Web Apps 或 Mobile Apps 可共用一個混合式連線存取內部部署資源。
* 只需要最少的 TCP 連接埠就能存取您的網路。
* 使用混合式連線的應用程式只存取透過混合式連線發佈的特定內部部署資源。
* 可以連接到任何使用靜態 TCP 連接埠的內部部署資源，例如 SQL Server、MySQL、HTTP Web API 和大部分的自訂 Web 服務。
  
  > [!NOTE]
  > 目前不支援使用動態連接埠的 TCP 服務 (例如 FTP 被動模式或延伸被動模式)。 也不支援 LDAP。 LDAP 使用靜態 TCP 連接埠，但它也可以使用 UDP。 因此不受支援。
  > 
  > 
* 可以與 Web Apps (.NET、PHP、Java、Python、Node.js) 和 Mobile Apps (Node.js、.NET) 支援的所有架構搭配使用。
* Web Apps 和 Mobile Apps 能夠以完全相同的方式存取內部部署資源，就像是該 Web 或 Mobile Apps 位於本機網路一樣。 例如，內部部署使用的相同連接字串也可以在 Azure 上使用。

混合式連線也可讓企業系統管理員控制和檢視混合式應用程式所存取的公司資源，包括：

* 系統管理員可以使用群組原則設定，以允許在網站上使用混合式連線，也能指定混合式應用程式可存取的資源。
* 公司網路上的事件和稽核記錄可讓您查看混合式連線所存取的資源。

## <a name="example-scenarios"></a>範例案例
混合式連線支援下列架構和應用程式的組合：

* .NET 架構存取 SQL Server
* .NET 架構搭配 WebClient 存取 HTTP/HTTPS 服務
* PHP 存取 SQL Server、MySQL
* Java 存取 SQL Server、MySQL 和 Oracle
* Java 存取 HTTP/HTTPS 服務

使用混合式連線來存取內部部署 SQL Server 時，請注意下列事項：

* SQL Express 具名執行個體必須設定為使用靜態連接埠。 依預設，SQL Express 具名執行個體是使用動態連接埠。
* SQL Express 預設執行個體使用靜態連接埠，但必須啟用 TCP。 預設不會啟用 TCP。
* 使用叢集或可用性群組時，暫不支援 `MultiSubnetFailover=true` 模式。
* 目前不支援 `ApplicationIntent=ReadOnly` 。
* 可能需要 SQL 驗證當做 Azure 應用程式和內部部署 SQL Server 所支援的端對端授權方法。

## <a name="security-and-ports"></a>安全性和連接埠
混合式連線採用共用存取簽章 (SAS) 授權，保護從 Azure 應用程式和內部部署混合式連線管理員至混合式連線的連接安全。 針對應用程式和內部部署混合式連線管理員，系統會建立個別的連線金鑰。 這些連線金鑰可以個別地更換和撤銷。

混合式連線能夠以順暢且安全的方式，將金鑰發佈給應用程式和內部部署混合式連線管理員。

請參閱 [建立和管理混合式連線](integration-hybrid-connection-create-manage.md)(英文)。

*應用程式授權與混合式連線分開*。 任何適當的授權方法都可使用。 授權方法視 Azure 雲端和內部部署元件之間支援的端對端授權方法而定。 例如，您的 Azure 應用程式存取內部部署 SQL Server。 在此情況下，SQL 授權可能是端對端支援的授權方法。

#### <a name="tcp-ports"></a>TCP 連接埠
混合式連線只需要您私人網路的輸出 TCP 或 HTTP 連線。 您不需要開啟任何防火牆連接埠，或變更您的網路周邊組態，即可允許任何輸入連線進入您的網路。

混合式連線會使用下列 TCP 通訊埠：

| Port | 您為何需要它 |
| --- | --- |
| 9350 - 9354 |這些連接埠用於資料傳輸。 服務匯流排轉送管理員會探查連接埠 9350，以判斷 TCP 連線是否可用。 如果可用的話，則會假設連接埠 9352 也是可用。 資料流量會經過連接埠 9352。 <br/><br/>允許對這些連接埠的輸出連線。 |
| 5671 |當連接埠 9352 用於資料流量時，連接埠 5671 就做為控制通道。 <br/><br/>允許對此連接埠的輸出連線。 |
| 80、443 |這些連接埠用於傳送部分資料要求給 Azure。 而且，如果連接埠 9352 和 5671 無法使用，則連接埠 80 和 443 是用於資料傳輸和控制通道的後援連接埠。允許對這些連接埠的輸出連線。<br/><br/>允許對這些連接埠的輸出連線。 <br/><br/>**注意** 不建議使用這些做為後援連接埠來代替其他 TCP 連接埠。 HTTP/WebSocket 是做為通訊協定使用，而非資料通道的原生 TCP。 這可能會導致效能變低。 |

## <a name="next-steps"></a>後續步驟
[Create and manage Hybrid Connections](integration-hybrid-connection-create-manage.md)

## <a name="see-also"></a>另請參閱
[用於管理 Microsoft Azure 上之 BizTalk 服務的 REST API](http://msdn.microsoft.com/library/azure/dn232347.aspx)  
[BizTalk 服務：版本圖表](biztalk-editions-feature-chart.md)  
[建立 BizTalk 服務](biztalk-provision-services.md)  
[BizTalk 服務：儀表板、監視和調整索引標籤](biztalk-dashboard-monitor-scale-tabs.md)  

[HCImage]: ./media/integration-hybrid-connection-overview/WABS_HybridConnectionImage.png
