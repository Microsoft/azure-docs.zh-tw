---
title: Azure Cosmos DB 防火牆支援和 IP 存取控制 | Microsoft Docs
description: 了解如何使用 IP 存取控制原則進行 Azure Cosmos DB 資料庫帳戶上的防火牆支援。
keywords: IP 存取控制，防火牆支援
services: cosmos-db
author: kanshiG
manager: kfile
tags: azure-resource-manager
ms.service: cosmos-db
ms.devlang: na
ms.topic: conceptual
ms.date: 03/30/2018
ms.author: govindk
ms.openlocfilehash: b21debdd6baa0a6587318ad861a821840ec6879c
ms.sourcegitcommit: 31241b7ef35c37749b4261644adf1f5a029b2b8e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/04/2018
ms.locfileid: "43666692"
---
# <a name="azure-cosmos-db-firewall-support"></a>Azure Cosmos DB 防火牆支援
為了保護 Azure Cosmos DB 資料庫帳戶中所儲存的資料，Azure Cosmos DB 已支援利用強式雜湊式訊息驗證碼 (HMAC) 的密碼型[授權模型](https://msdn.microsoft.com/library/azure/dn783368.aspx)。 現在，除了密碼型授權模型之外，Azure Cosmos DB 還支援使用原則驅動的 IP 型存取控制來進行輸入防火牆支援。 此模型與傳統資料庫系統的防火牆規則相類似，且可為 Azure Cosmos DB 資料庫帳戶提供額外的安全性層級。 您現在可以使用這個模型，設定只能從一組核准的電腦和 (或) 雲端服務存取 Azure Cosmos DB 資料庫帳戶。 透過這些核准的電腦和服務組合來存取 Azure Cosmos DB 資源，仍然需要呼叫者呈現有效的授權權杖。

> [!NOTE]
> 防火牆支援目前適用於 Azure Cosmos DB SQL API 和 Mongo API 帳戶。 為其他 API 和 Azure 德國或 Azure Government 等主權雲端設定防火牆的功能即將推出。 如果您打算針對已設定現有 IP 防火牆的 Azure Cosmos DB 帳戶設定服務端點 ACL，請注意防火牆組態、移除 IP 防火牆，然後設定服務端點 ACL。 服務端點設定好之後，您可以重新啟用 IP 防火牆 (如有需要)。

## <a name="ip-access-control-overview"></a>IP 存取控制概觀
只要要求伴隨有效的授權權杖，預設就可以從公用網際網路存取 Azure Cosmos DB 資料庫帳戶。 若要設定 IP 原則型存取控制，使用者必須以 CIDR 形式提供這組 IP 位址或 IP 位址範圍，以作為指定資料庫帳戶的允許用戶端 IP 清單。 套用這個組態之後，伺服器將會封鎖源自此允許清單外部之電腦的所有要求。  下圖說明 IP 型存取控制的連線處理流程：

![顯示 IP 型存取控制之連接程序的圖表](./media/firewall-support/firewall-support-flow.png)

## <a id="configure-ip-policy"></a> 設定 IP 存取控制原則
您可以在 Azure 入口網站中設定 IP 存取控制原則，也可以透過 [Azure CLI](cli-samples.md)、[Azure Powershell](powershell-samples.md) 或 [REST API](/rest/api/cosmos-db/)，以程式設計方式更新 **ipRangeFilter** 屬性來設定。 

若要在 Azure 入口網站中設定 IP 存取控制原則，請瀏覽至 [Azure Cosmos DB 帳戶] 頁面、按一下導覽功能表中的 [防火牆和虛擬網路]，接著將 [允許從下項存取] 值變更為 [選取的網路]，然後按一下 [儲存]。 

![顯示如何在 Azure 入口網站中開啟 [防火牆] 頁面的螢幕擷取畫面](./media/firewall-support/azure-portal-firewall.png)

一旦開啟 IP 存取控制之後，入口網站就能指定 IP 位址和範圍，以及提供參數來允許存取其他 Azure 服務和 Azure 入口網站。 下列各節會提供有關這些參數的額外資訊。

> [!NOTE]
> 啟用 Azure Cosmos DB 資料庫帳戶的 IP 存取控制原則，即會封鎖所設定之允許 IP 位址範圍清單外部的電腦對您 Azure Cosmos DB 資料庫帳戶的所有存取。 透過這個模型，也會封鎖從入口網站瀏覽資料平面作業，確保存取控制的完整性。

## <a name="connections-from-the-azure-portal"></a>來自 Azure 入口網站的連線 

當您以程式設計方式啟用 IP 存取控制原則時，您必須將 Azure 入口網站的 IP 位址加入到 **ipRangeFilter** 屬性以維持存取權。 入口網站 IP 位址是：

|區域|IP 位址|
|------|----------|
|所有區域 (下面指定的區域除外)|104.42.195.92、40.76.54.131、52.176.6.30、52.169.50.45、52.187.184.26|
|德國|51.4.229.218|
|中國|139.217.8.252|
|US Gov|52.244.48.71|

當您在 Azure 入口網站中將防火牆設定變更為 [選取的網路] 時，預設允許存取 Azure 入口網站。 

![顯示如何允許存取 Azure 入口網站的螢幕擷取畫面](./media/firewall-support/enable-azure-portal.png)

## <a name="connections-from-global-azure-datacenters-or-azure-paas-services"></a>來自全域 Azure 資料中心或 Azure PaaS 服務的連線

如 Azure 串流分析、Azure Functions 之類的 Azure PaaS 服務會與 Azure Cosmos DB 搭配使用。 若要允許應用程式從其他的 Azure PaaS 服務連線到您的 Azure Cosmos DB 資源，則必須啟用防火牆設定。 若要啟用此防火牆設定，請將 IP 位址- 0.0.0.0 新增至允許的 IP 位址清單。 0.0.0.0 會限制只能連線到 Azure 資料中心 IP 範圍內的 Azure Cosmos DB 帳戶。 此設定不允許任何其他 IP 範圍存取 Azure Cosmos DB 帳戶。

> [!IMPORTANT]
> 這個選項會設定防火牆，以允許所有來自 Azure 的連線，包括來自其他客戶訂用帳戶的連線。 選取這個選項時，請確定您的登入和使用者權限會限制為只有授權的使用者才能存取。
> 

當您在 Azure 入口網站中將防火牆設定變更為 [選取的網路] 時，預設會啟用從全域 Azure 資料中心內存取連線的功能。 

![顯示如何在 Azure 入口網站中開啟 [防火牆] 頁面的螢幕擷取畫面](./media/firewall-support/enable-azure-services.png)

## <a name="connections-from-your-current-ip"></a>來自目前 IP 的連線

為了簡化開發工作，Azure 入口網站可協助您識別用戶端電腦的 IP 並新增至允許清單，讓您電腦上執行的應用程式可以存取 Azure Cosmos DB 帳戶。 此處的用戶端 IP 位址是入口網站所偵測到的。 它可能是您電腦的用戶端 IP 位址，但也可能是網路閘道的 IP 位址。 移至生產環境之前，別忘記移除它。

若要啟用目前的 IP，選取 [Add my current IP] \(新增我目前的 IP\)，將您目前的 IP 新增至 IP 清單，然後按一下 [儲存]。

![顯示如何為目前的 IP 進行防火牆設定的螢幕擷取畫面](./media/firewall-support/enable-current-ip.png)

## <a name="connections-from-cloud-services"></a>從雲端服務的連接
在 Azure 中，雲端服務是使用 Azure Cosmos DB 裝載中介層服務邏輯的常見方式。 若要從雲端服務存取 Azure Cosmos DB 資料庫帳戶，必須[設定 IP 存取控制原則](#configure-ip-policy)，以將雲端服務的公用 IP 位址新增至與 Azure Cosmos DB 資料庫帳戶相關聯的允許 IP 位址清單。 這確保雲端服務的所有角色執行個體都能存取您的 Azure Cosmos DB 資料庫帳戶。 您可以在 Azure 入口網站中擷取雲端服務的 IP 位址，如下列螢幕擷取畫面所示：

![這個螢幕擷取畫面顯示 Azure 入口網站中所顯示雲端服務的公用 IP 位址](./media/firewall-support/public-ip-addresses.png)

當您新增其他角色執行個體來相應放大雲端服務時，那些新的執行個體將可自動存取 Azure Cosmos DB 資料庫帳戶，因為它們是相同雲端服務的一部分。

## <a name="connections-from-virtual-machines"></a>從虛擬機器的連接
您也可以使用[虛擬機器](https://azure.microsoft.com/services/virtual-machines/)或[虛擬機器擴展集](../virtual-machine-scale-sets/virtual-machine-scale-sets-overview.md)，透過 Azure Cosmos DB 裝載中介層服務。  若要設定允許從虛擬機器存取 Azure Cosmos DB 資料庫帳戶，必須[設定 IP 存取控制原則](#configure-ip-policy)，以將虛擬機器和 (或) 虛擬機器擴展集的公用 IP 位址設定為 Azure Cosmos DB 資料庫帳戶的其中一個允許 IP 位址。 您可以在 Azure 入口網站中擷取虛擬機器的 IP 位址，如下列螢幕擷取畫面所示。

![這個螢幕擷取畫面顯示 Azure 入口網站中所顯示虛擬機器的公用 IP 位址](./media/firewall-support/public-ip-addresses-dns.png)

當您將其他虛擬機器執行個體新增至群組時，它們即可自動存取您的 Azure Cosmos DB 資料庫帳戶。

## <a name="connections-from-the-internet"></a>從網際網路的連接
從網際網路上的電腦存取 Azure Cosmos DB 資料庫帳戶時，必須將電腦的用戶端 IP 位址或 IP 位址範圍新增至 Azure Cosmos DB 資料庫帳戶的允許 IP 位址清單。 

## <a name="using-azure-resource-manager-template-to-set-up-the-ip-access-control"></a>使用 Azure Resource Manager 範本設定 IP 存取控制

將下列 JSON 新增至您的範本，以設定 IP 存取控制。 帳戶的 Resource Manager 範本將具有 ipRangeFilter 屬性，而該屬性為應列入白名單的 IP 範圍清單。

```json
   {
     "apiVersion": "2015-04-08",
     "type": "Microsoft.DocumentDB/databaseAccounts",
     "kind": "GlobalDocumentDB",
     "name": "[parameters('databaseAccountName')]",
     "location": "[resourceGroup().location]",
     "properties": {
     "databaseAccountOfferType": "Standard",
     "name": "[parameters('databaseAccountName')]",
     "ipRangeFilter":"10.0.0.1,10.0.0.2,183.240.196.255"
   }
   }
```

## <a name="troubleshooting-the-ip-access-control-policy"></a>針對 IP 存取控制原則進行疑難排解
### <a name="portal-operations"></a>入口網站作業
啟用 Azure Cosmos DB 資料庫帳戶的 IP 存取控制原則，即會封鎖所設定之允許 IP 位址範圍清單外部的電腦對您 Azure Cosmos DB 資料庫帳戶的所有存取。 因此，如果您想要啟用入口網站資料層面作業 (例如瀏覽容器和查詢文件)，則需要在入口網站使用 [防火牆] 頁面，明確允許存取 Azure 入口網站。 

### <a name="sdk--rest-api"></a>SDK & Rest API
基於安全性考量，如果從電腦透過 SDK 或 REST API 的存取不在允許清單上，則會傳回沒有其他詳細資料的一般「404 找不到」回應。 請確認您針對 Azure Cosmos DB 資料庫帳戶設定的 IP 允許清單，以確保會將正確的原則組態套用至您的 Azure Cosmos DB 資料庫帳戶。

## <a name="next-steps"></a>後續步驟
如需網路相關效能秘訣的相關資訊，請參閱[效能秘訣](performance-tips.md)。

