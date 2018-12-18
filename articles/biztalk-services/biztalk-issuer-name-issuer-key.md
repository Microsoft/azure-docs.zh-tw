---
title: BizTalk 服務中的簽發者名稱和簽發者金鑰 | Microsoft Docs
description: 了解如何在 BizTalk 服務中擷取服務匯流排或存取控制 (ACS) 的簽發者名稱和簽發者金鑰。 MABS，WABS
services: biztalk-services
documentationcenter: ''
author: MandiOhlinger
manager: anneta
editor: ''
ms.assetid: 067fe356-d1aa-420f-b2f2-1a418686470a
ms.service: biztalk-services
ms.workload: integration
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 11/07/2016
ms.author: mandia
ms.openlocfilehash: 78796b5dc62cb573f149c24d90205d26fb139cf7
ms.sourcegitcommit: 4de6a8671c445fae31f760385710f17d504228f8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/08/2018
ms.locfileid: "39628642"
---
# <a name="biztalk-services-issuer-name-and-issuer-key"></a>BizTalk 服務：簽發者名稱和簽發者金鑰

> [!INCLUDE [BizTalk Services is being retired, and replaced with Azure Logic Apps](../../includes/biztalk-services-retirement.md)]

Azure BizTalk 服務使用服務匯流排簽發者名稱和簽發者金鑰，以及存取控制簽發者名稱和簽發者金鑰。 具體而言：

| Task | 什麼簽發者名稱和簽發者金鑰 |
| --- | --- |
| 從 Visual Studio 部署應用程式 |存取控制簽發者名稱和簽發者金鑰 |
| 設定 Azure BizTalk 服務入口網站 |存取控制簽發者名稱和簽發者金鑰 |
| 在 Visual Studio 中使用 BizTalk 配接器服務建立 LOB 轉送 |服務匯流排簽發者名稱和簽發者金鑰 |

本主題列出擷取簽發者名稱和簽發者金鑰的步驟。 

## <a name="access-control-issuer-name-and-issuer-key"></a>存取控制簽發者名稱和簽發者金鑰
下列項目會使用存取控制簽發者名稱和簽發者金鑰：

* 在 Visual Studio 中建立的 Azure BizTalk 服務應用程式：若要在 Visual Studio 中成功將您的 BizTalk 服務應用程式部署至 Azure，請輸入存取控制簽發者名稱和簽發者金鑰。 
* Azure BizTalk 服務入口網站：在您建立 BizTalk 服務且開啟 BizTalk 服務入口網站時，系統會使用相同的存取控制值，自動為您的部署註冊您的存取控制簽發者名稱和簽發者金鑰。

### <a name="get-the-access-control-issuer-name-and-issuer-key"></a>取得存取控制簽發者名稱和簽發者金鑰

若要使用 ACS 進行驗證，並取得簽發者名稱和簽發者金鑰值，整體步驟包括：

1. 安裝 [Azure PowerShell Cmdlet](https://azure.microsoft.com/documentation/articles/powershell-install-configure/)。
2. 新增您的 Azure 帳戶：`Add-AzureAccount`
3. 傳回您的訂用帳戶名稱：`get-azuresubscription`
4. 選取您的訂用帳戶：`select-azuresubscription <name of your subscription>` 
5. 建立新的命名空間：`new-azuresbnamespace <name for the service bus> "Location" -CreateACSNamespace $true -NamespaceType Messaging`

    範例：`new-azuresbnamespace biztalksbnamespace "South Central US" -CreateACSNamespace $true -NamespaceType Messaging`
      
5. 建立新的 ACS 命名空間時 (這可能要花幾分鐘的時間)，會在連接字串中列出簽發者名稱和簽發者金鑰值： 

    ```
    Name                  : biztalksbnamespace
    Region                : South Central US
    DefaultKey            : abcdefghijklmnopqrstuvwxyz
    Status                : Active
    CreatedAt             : 10/18/2016 9:36:30 PM
    AcsManagementEndpoint : https://biztalksbnamespace-sb.accesscontrol.windows.net/
    ServiceBusEndpoint    : https://biztalksbnamespace.servicebus.windows.net/
    ConnectionString      : Endpoint=sb://biztalksbnamespace.servicebus.windows.net/;SharedSecretIssuer=owner;SharedSecretValue=abcdefghijklmnopqrstuvwxyz
    NamespaceType         : Messaging
    ```

總結：  
簽發者名稱 = SharedSecretIssuer  
簽發者金鑰 = SharedSecretKey

如需更多資訊，請參閱 [New-AzureSBNamespace](https://docs.microsoft.com/powershell/module/servicemanagement/azure/new-azuresbnamespace) Cmdlet。 

## <a name="service-bus-issuer-name-and-issuer-key"></a>服務匯流排簽發者名稱和簽發者金鑰
BizTalk 配接器服務會使用服務匯流排簽發者名稱和簽發者金鑰。 在 Visual Studio 中，您在 BizTalk 服務專案中使用 BizTalk 配接器服務來連線至內部部署企業營運 (LOB) 系統。 若要連線，您需要建立 LOB 轉送並輸入 LOB 系統詳細資料。 如果這樣做，則您也需要輸入服務匯流排簽發者名稱和簽發者金鑰。

### <a name="to-retrieve-the-service-bus-issuer-name-and-issuer-key"></a>擷取服務匯流排簽發者名稱和簽發者金鑰
1. 登入 [Azure 入口網站](http://portal.azure.com)。
2. 搜尋**服務匯流排**，並選取您的命名空間。 
3. 開啟 [共用存取原則] 屬性，選取您的原則，並檢視 [連接字串] 的名稱和金鑰值。  

## <a name="next"></a>下一頁
其他 Azure BizTalk 服務主題：

* [安裝 Azure BizTalk 服務 SDK](http://go.microsoft.com/fwlink/p/?LinkID=241589)<br/>
* [教學課程：Azure BizTalk 服務](http://go.microsoft.com/fwlink/p/?LinkID=236944)<br/>
* [如何開始使用 Azure BizTalk 服務 SDK](http://go.microsoft.com/fwlink/p/?LinkID=302335)<br/>
* [Azure BizTalk 服務](http://go.microsoft.com/fwlink/p/?LinkID=303664)<br/>

## <a name="see-also"></a>另請參閱
* [做法：使用 ACS 管理服務來設定服務身分識別](http://go.microsoft.com/fwlink/p/?LinkID=303942)<br/>
* [BizTalk 服務：開發人員、基本、標準和高級版本圖表](http://go.microsoft.com/fwlink/p/?LinkID=302279)<br/>
* [BizTalk 服務：佈建](http://go.microsoft.com/fwlink/p/?LinkID=302280)<br/>
* [BizTalk 服務：佈建狀態圖](http://go.microsoft.com/fwlink/p/?LinkID=329870)<br/>
* [BizTalk 服務：儀表板、監視和調整索引標籤](http://go.microsoft.com/fwlink/p/?LinkID=302281)<br/>
* [BizTalk 服務：備份與還原](http://go.microsoft.com/fwlink/p/?LinkID=329873)<br/>
* [BizTalk 服務：節流](http://go.microsoft.com/fwlink/p/?LinkID=302282)<br/>

