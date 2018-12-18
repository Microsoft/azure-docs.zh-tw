---
title: 設定 Azure App Service Environment 使用強制通道
description: 輸出流量使用強制通道處理時，讓 App Service Environment 可以運作
services: app-service
documentationcenter: na
author: ccompy
manager: stefsch
ms.assetid: 384cf393-5c63-4ffb-9eb2-bfd990bc7af1
ms.service: app-service
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: quickstart
ms.date: 05/29/2018
ms.author: ccompy
ms.custom: mvc
ms.openlocfilehash: ba93aab14c8eaccf9e3ed9ae9db0d169f41dddea
ms.sourcegitcommit: d211f1d24c669b459a3910761b5cacb4b4f46ac9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/06/2018
ms.locfileid: "44024040"
---
# <a name="configure-your-app-service-environment-with-forced-tunneling"></a>設定 App Service Environment 搭配強制通道

App Service Environment (ASE) 是在客戶的 Azure 虛擬網路之中的 Azure App Service 部署。 許多客戶將其 Azure 虛擬網路設定為內部部署網路 (具備 VPN 或 Azure ExpressRoute 連線) 的擴充。 當您將網際網路繫結流量重新導向至您的 VPN 或虛擬設備時，就會使用強制通道。 這通常是安全性需求的一部分，用以檢查和稽核所有的輸出流量。 

ASE 有一些外部相依性，[App Service Environment 網路架構][network]文件會予以說明。 通常所有 ASE 輸出相依性流量都必須都通過與 ASE 一起佈建的 VIP。 如果您變更往返 ASE 的流量路由，但未遵循以下的資訊，您的 ASE 將會停止運作。

在 Azure 虛擬網路中，路由是根據 最長首碼比對 (LPM) 而進行。 如果有多個符合相同 LPM 的路由，則會根據其來源，以下列順序選取路由：

* 使用者定義的路由 (UDR)
* BGP 路由 (使用 ExpressRoute 時)
* 系統路由

若要深入了解虛擬網路中的路由，請閱讀[使用者定義的路由和 IP 轉送][routes]。 

如果您想要將 ASE 輸出流量路由傳送至某處，而非直接傳送至網際網路，您有下列選項：

* 讓 ASE 可以直接存取網際網路
* 將您的 ASE 子網路設定為略過 BGP 路由
* 將您的 ASE 子網路設定為使用 Azure SQL 和 Azure 儲存體的服務端點
* 將您自己的 IP 新增至 ASE Azure SQL 防火牆

## <a name="enable-your-app-service-environment-to-have-direct-internet-access"></a>讓 App Service Environment 具備網際網路直接存取權

若要讓您的 ASE 能夠直接移至網際網路，即使已使用 ExpressRoute 設定您的 Azure 虛擬網路，您仍可：

* 將 ExpressRoute 設定為公告 0.0.0.0/0。 根據預設，它會將所有輸出流量路由傳送至內部部署網路。
* 建立位址首碼為 0.0.0.0/0、下一個躍點類型為網際網路的 UDR，並將它套用至 ASE 子網路。

如果您做了這兩項變更，則源自 App Service Environment 子網路且目的地為網際網路的流量，將不會被強制經由 ExpressRoute 連線傳輸。

如果網路已在內部部署路由流量，則您必須建立子網路以裝載您 ASE，並為其設定 UDR，然後再嘗試部署 ASE。  

> [!IMPORTANT]
> UDR 中定義的路由必須足夠明確，以優先於 ExpressRoute 組態所通告的任何路由。 前面的範例使用廣泛的 0.0.0.0/0 位址範圍。 因此有可能會不小心由使用更明確位址範圍的路由通告所覆寫。
>
> 針對從公用對等互連路徑至私人對等互連路徑的路由進行交叉通告的 ExpressRoute 設定，不支援 App Service Environment。 已設定公用對等互連的 ExpressRoute 設定，會收到來自 Microsoft 的路由通告。 通告中會包含一大組 Microsoft Azure 位址範圍。 如果位址範圍在私人對等互連路徑上交叉通告，來自 App Service Environment 子網路的所有連出網路封包都會路由傳送至客戶的內部部署網路基礎結構。 App Service Environment 預設不支援這種網路流量。 此問題的一個解決方案是停止從公用對等互連路徑至私人對等互連路徑的交叉通告路由。 另一個解決方式是讓 App Service Environment 可以在強制通道設定中運作。

![直接存取網際網路][1]

## <a name="configure-your-ase-subnet-to-ignore-bgp-routes"></a>將您的 ASE 子網路設定為略過 BGP 路由 ## 

您可以將您的 ASE 子網路設定為略過所有 BGP 路由。  若已經設定此項目，ASE 將能夠存取其相依性，而不發生任何問題。  不過，您必須建立 UDR，才能讓您的應用程式存取內部部署資源。

若要將您的 ASE 子網路設定為略過 BGP 路由：

* 如果您還沒有 UDR，請加以建立並將它指派給您的 ASE 子網路。
* 在 Azure 入口網站中，開啟指派給 ASE 子網路的路由表 UI。  選取組態。  將 BCP 路由傳播設定為 [停用]。  按一下 [儲存]。 將它關閉的相關文件位於[建立路由表][routetable]文件中。

執行這項操作之後，您的應用程式就不再能夠觸達內部部署環境。 若要解決此問題，請編輯指派給 ASE 子網路的 UDR，並新增內部部署位址範圍的路由。 下一個躍點類型應該設定為虛擬網路閘道。 


## <a name="configure-your-ase-with-service-endpoints"></a>使用服務端點設定您的 ASE ##

 > [!NOTE]
   > 使用 SQL 的服務端點不適用於美國政府區域中的 ASE。  下列資訊僅適用於 Azure 公用區域。  

若要從您的 ASE 路由傳送所有輸出流量 (移至 Azure SQL 和 Azure 儲存體的流量除外)，請執行下列步驟：

1. 您可以建立路由表並將它指派給 ASC 子網路。 請在 [App Service Environment 管理位址][management]尋找符合您區域的位址。 為下一個躍點為網際網路的位址建立路由。 這是必要作業，因為 App Service Environment 輸入環境流量必須從它傳送至的相同位址回覆。   

2. 在您的 ASE 子網路中使用 Azure SQL 和 Azure 儲存體啟用服務端點。  完成此步驟後，您可以使用強制通道設定您的 VNet。

若要在已設定成在內部部署路由所有流量的虛擬網路中建立您的 ASE，您必須使用 Resource Manager 範本建立 ASE。  您無法使用入口網站在已存在的子網路中建立 ASE。  將您的 ASE 部署到已設定成在內部部署路由輸出流量的 VNet 時，您必須使用可讓您指定已存在的子網路的 Resource Manager 範本來建立 ASE。 如需關於使用範本部署 ASE 的詳細資訊，請參閱[使用範本建立 App Service 環境][template]。

服務端點可讓您將多租用戶服務的存取權限制於一組 Azure 虛擬網路和子網路。 您可以在[虛擬網路服務端點][serviceendpoints]文件中深入了解服務端點。 

當您在資源上啟用服務端點時，所建立的路由具有高於所有其他路由的優先順序。 如果您使用服務端點搭配強制通道 ASE，Azure SQL 和 Azure 儲存體管理流量不會使用強制通道。 其他 ASE 相依性流量會使用強制通道且不能遺失，否則 ASE 會無法正常運作。

透過 Azure SQL 執行個體在子網路上啟用服務端點時，從該子網路連線的所有 Azure SQL 執行個體都必須啟用服務端點。 如果您想要從相同的子網路存取多個 Azure SQL 執行個體，您就無法在一個 Azure SQL 執行個體啟用服務端點，而不要在另一個執行個體上啟用。  Azure 儲存體與 Azure SQL 的運作方式不同。  當您使用 Azure 儲存體啟用服務端點時，您會封鎖您的子網路存取該資源，，但仍可存取其他 Azure 儲存體帳戶 (即使它們未啟用服務端點)。  

如果您設定對網路篩選設備使用強制通道，請記住，除了 Azure SQL 和 Azure 儲存體以外，ASE 還具有一些相依性。 您必須允許對這些相依項目的流量，否則 ASE 無法正常運作。

![服務端點使用強制通道][2]

## <a name="add-your-own-ips-to-the-ase-azure-sql-firewall"></a>將您自己的 IP 新增至 ASE Azure SQL 防火牆 ##

若要從您的 ASE 傳輸所有輸出流量 (移至 Azure 儲存體的流量除外)，請執行下列步驟：

1. 您可以建立路由表並將它指派給 ASC 子網路。 請在 [App Service Environment 管理位址][management]尋找符合您區域的位址。 為下一個躍點為網際網路的位址建立路由。 這是必要作業，因為 App Service Environment 輸入環境流量必須從它傳送至的相同位址回覆。 

2. 在您的 ASE 子網路中使用 Azure 儲存體啟用服務端點

3. 取得位址，該位址將使用於從您的 App Service Environment 至網際網路的所有輸出流量。 如果您將流量路由傳送至內部部署網路，則這些位址就是您的 NAT 或閘道 IP。 如果您想要透過 NVA 路由傳送 App Service Environment 連出流量，則輸出位址為 NVA 的公用 IP。

4. _若要設定現有 App Service Environment 中的輸出位址：_ 移至 resources.azure.com，而後移至 Subscription/\<subscription id>/resourceGroups/\<ase resource group>/providers/Microsoft.Web/hostingEnvironments/\<ase name>。 接著，您就可以看到描述您 App Service Environment 的 JSON。 確定最上方寫的是「讀取/寫入」。 選取 [編輯]。 向下捲動至底部。 將 **userWhitelistedIpRanges** 值從 **null** 變更為類似以下這樣。 使用您要設為輸出位址範圍的位址。 

        "userWhitelistedIpRanges": ["11.22.33.44/32", "55.66.77.0/24"] 

   選取頂端的 [PUT]。 此選項會觸發 App Service Environment 上的規模調整作業，並調整防火牆。

_若要以輸出位址建立您的 ASE_ ：請依照[使用範本建立 App Service Environment][template]中的指示操作並提取適當的範本。  編輯 azuredeploy.json 檔案中的 "resources" 區段 (但不在 "properties" 區塊中)，而且納入包含您的值的 **userWhitelistedIpRanges** 行。

    "resources": [
      {
        "apiVersion": "2015-08-01",
        "type": "Microsoft.Web/hostingEnvironments",
        "name": "[parameters('aseName')]",
        "kind": "ASEV2",
        "location": "[parameters('aseLocation')]",
        "properties": {
          "name": "[parameters('aseName')]",
          "location": "[parameters('aseLocation')]",
          "ipSslAddressCount": 0,
          "internalLoadBalancingMode": "[parameters('internalLoadBalancingMode')]",
          "dnsSuffix" : "[parameters('dnsSuffix')]",
          "virtualNetwork": {
            "Id": "[parameters('existingVnetResourceId')]",
            "Subnet": "[parameters('subnetName')]"
          },
        "userWhitelistedIpRanges":  ["11.22.33.44/32", "55.66.77.0/30"]
        }
      }
    ]

這些變更會將流量直接從 ASE 傳送至 Azure 儲存體，並允許從 ASE 的 VIP 以外的其他位址存取 Azure SQL。

   ![搭配 SQL 白名單使用強制通道][3]

## <a name="preventing-issues"></a>防止問題發生 ##

如果 ASE 與其相依性之間的通訊已中斷，則 ASE 會變成不良狀態。  如果維持不良狀態太久，則 ASE 會變成暫停狀態。 若要使 ASE 恢復權限，請遵循 ASE 入口網站中的指示操作。

除了中斷通訊以外，您可能會因為造成過久延遲而對 ASE 產生負面影響。 如果您的 ASE 離您的內部部署網路太遠，可能會發生很久的延遲。  舉例來說，跨越海洋或大陸連到您的內部部署網路，都屬於太遠的距離。 內部網路擁塞或輸出頻寬限制，也可能造成延遲。


<!--IMAGES-->
[1]: ./media/forced-tunnel-support/asedependencies.png
[2]: ./media/forced-tunnel-support/forcedtunnelserviceendpoint.png
[3]: ./media/forced-tunnel-support/forcedtunnelexceptstorage.png

<!--Links-->
[management]: ./management-addresses.md
[network]: ./network-info.md
[routes]: ../../virtual-network/virtual-networks-udr-overview.md
[template]: ./create-from-template.md
[serviceendpoints]: ../../virtual-network/virtual-network-service-endpoints-overview.md
[routetable]: ../../virtual-network/manage-route-table.md#create-a-route-table
