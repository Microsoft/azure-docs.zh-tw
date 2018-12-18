---
title: Azure DMZ 範例 – 建置具有 NSG 的簡單 DMZ | Microsoft Docs
description: 建置具有網路安全性群組 (NSG) 的 DMZ
services: virtual-network
documentationcenter: na
author: tracsman
manager: rossort
editor: ''
ms.assetid: ''
ms.service: virtual-network
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 01/03/2017
ms.author: jonor
ms.openlocfilehash: ec29e6b250f927a3a4a94ffdf83d6c7c0e325722
ms.sourcegitcommit: 6fcd9e220b9cd4cb2d4365de0299bf48fbb18c17
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/05/2018
ms.locfileid: "23126676"
---
# <a name="example-1--build-a-simple-dmz-using-nsgs-with-an-azure-resource-manager-template"></a>範例 1 – 使用 NSG 搭配 Azure Resource Manager 範本建立簡單的 DMZ
[返回 [安全性界限最佳作法] 頁面][HOME]

> [!div class="op_single_selector"]
> * [Resource Manager 範本](virtual-networks-dmz-nsg.md)
> * [傳統 - PowerShell](virtual-networks-dmz-nsg-asm.md)
> 
>

此範例會建立包含四個 Windows 伺服器和網路安全性群組的基本 DMZ。 此範例說明每個相關範本區段，以讓您更加深入地了解每個步驟。 另外您還會看到＜流量案例＞一節，本節提供深入的逐步說明，讓您知道流量是如何流經 DMZ 內的各個防禦層。 最後則有＜參考＞一節，本節提供完整的範本程式碼和指示，以供您建置此環境來測試和試驗各種案例。 

[!INCLUDE [azure-arm-classic-important-include](../../includes/azure-arm-classic-important-include.md)] 

![具有 NSG 的輸入 DMZ][1]

## <a name="environment-description"></a>環境描述
此範例中，訂用帳戶包含下列資源：

* 單一資源群組
* 含有兩個子網路的虛擬網路；“FrontEnd” 和 “BackEnd”
* 套用至這兩個子網路的單一網路安全性群組
* 一個代表應用程式 Web 伺服器的 Windows Server (“IIS01”)
* 兩個代表應用程式後端伺服器的 Windows Server (“AppVM01”、“AppVM02”)
* 一個代表 DNS 伺服器的 Windows Server ("DNS01")
* 與應用程式 web 伺服器相關聯的公用 IP 位址

在 [參考] 區段中，有連結至 Azure Resource Manager 範本，可建置這個範例中所述的環境。 至於 VM 和虛擬網路的建置，雖然也是由此範例範本來完成，但本文不會詳加敘述。 

**若要建置此環境**(這份文件的參考一節中有詳細指示)。

1. 將 Azure Resource Manager 範本部署在：[Azure 快速入門範本][Template]
2. 將範例應用程式安裝在︰[範例應用程式指令碼][SampleApp]

>[!NOTE]
>在這個執行個體中 RDP 至任何後端伺服器，IIS 伺服器會作為「跳躍箱」。 首先 RDP 到 IIS 伺服器，然後從 IIS 伺服器 RDP 到後端伺服器。 或者，可以將公用 IP 與每個伺服器 NIC 相關聯以便更容易 RDP。
> 
>

下列各節藉由逐步解說 Azure Resource Manager 範本的重要程式行，詳細說明此範例的網路安全性群組和其運作方式。

## <a name="network-security-groups-nsg"></a>網路安全性群組 (NSG)
此範例會組建 NSG 群組，然後載入六個規則。 

>[!TIP]
>一般而言，您應該先建立特定的「允許」規則，最後再建立較一般的「拒絕」規則。 所指定的優先順序會決定要先評估哪些規則。 一旦發現流量適用特定規則，就不會再評估後續規則。 NSG 規則可以套用在輸入或輸出方向 (從子網路的觀點出發)。
>
>

指令碼會以宣告方式為輸入流量建置下列規則：

1. 允許內部 DNS 流量 (連接埠 53)
2. 允許從網際網路到任何 VM 的 RDP 流量 (連接埠 3389)
3. 允許從網際網路到 Web 伺服器 (IIS01) 的 HTTP 流量 (連接埠 80)
4. 允許從 IIS01 到 AppVM1 的任何流量 (所有連接埠)
5. 拒絕從網際網路到整個 VNet (兩個子網路) 的任何流量 (所有連接埠)
6. 拒絕從 Frontend 子網路到 Backend 子網路的任何流量 (所有連接埠)

這些規則繫結至每個子網路後，如果有從網際網路到 Web 伺服器的輸入 HTTP 要求，規則 3 (允許) 和規則 5 (拒絕) 皆適用，但由於規則 3 具有較高的優先順序，所以只會適用規則 3，規則 5 則不會派上用場。 因此會允許 HTTP 要求送往 Web 伺服器。 如果相同的流量嘗試抵達 DNS01 伺服器，規則 5 (拒絕) 會先適用，因此不會允許流量傳遞給伺服器。 規則 6 (拒絕) 會阻止 Frontend 子網路與 Backend 子網路交談 (規則 1 和 4 允許的流量除外)，此規則集可在攻擊者入侵 Frontend 上的 Web 應用程式時保護 Backend 網路，攻擊者只能對 Backend 的「受保護」網路進行有限度的存取 (只能存取 AppVM01 伺服器上公開的資源)。

有一個預設輸出規則可允許流量外流到網際網路。 在此範例中，我們會允許輸出流量，並不會修改任何輸出規則。 如果兩個方向的流量都要套用安全性原則，則需要使用者定義的路由，會在[安全性界限最佳作法頁面][HOME]上的＜範例 3＞中探討。

會詳細討論每個規則，如下所示︰

1. 必須具現化網路安全性群組資源，以保存規則︰

    ```JSON
    "resources": [
      {
        "apiVersion": "2015-05-01-preview",
        "type": "Microsoft.Network/networkSecurityGroups",
        "name": "[variables('NSGName')]",
        "location": "[resourceGroup().location]",
        "properties": { }
      }
    ]
    ``` 

2. 此範例中的第一個規則會允許所有內部網路之間的 DNS 流量流往 Backend 子網路上的 DNS 伺服器。 此規則有一些重要參數：
  * "destinationAddressPrefix" - 規則可以使用稱為「預設標記」的位址首碼特殊類型，這些標記是系統提供的識別項，可使用簡單的方法來解決較大的位址前置詞類別。 此規則會使用預設標記「網際網路」來表示 VNet 之外的任何位址。 其他前置詞標籤為 VirtualNetwork 和 AzureLoadBalancer。
  * 「方向」表示此規則會生效的傳輸流量方向。 方向是來自子網路或虛擬機器的角度 (取決於此 NSG 繫結的位置)。 因此，如果 Direction 是 “Inbound” 且流量進入子網路，此規則將會適用，而離開子網路的流量則不受此規則所影響。
  * "Priority" 會設定流量的評估順序。 編號愈低，優先順序就愈高。 當規則套用至特定流量時，就不會再處理其他規則。 因此，如果優先順序為 1 的規則允許流量，優先順序為 2 的規則拒絕流量，而這兩個規則皆適用於流量，則會允許流量流動 (規則 1 有更高的優先順序，所以會生效，並且不會再套用其他規則)。
  * “Access” 指出受此規則影響的流量是要封鎖 ("Deny") 或允許 ("Allow")。

    ```JSON
    "properties": {
    "securityRules": [
      {
        "name": "enable_dns_rule",
        "properties": {
          "description": "Enable Internal DNS",
          "protocol": "*",
          "sourcePortRange": "*",
          "destinationPortRange": "53",
          "sourceAddressPrefix": "VirtualNetwork",
          "destinationAddressPrefix": "10.0.2.4",
          "access": "Allow",
          "priority": 100,
          "direction": "Inbound"
        }
      },
    ```

3. 此規則會允許 RDP 流量從網際網路流往繫結子網路上任何伺服器的 RDP 連接埠。 

    ```JSON
    {
      "name": "enable_rdp_rule",
      "properties": {
        "description": "Allow RDP",
        "protocol": "Tcp",
        "sourcePortRange": "*",
        "destinationPortRange": "3389",
        "sourceAddressPrefix": "*",
        "destinationAddressPrefix": "*",
        "access": "Allow",
        "priority": 110,
        "direction": "Inbound"
      }
    },
    ```

4. 此規則允許輸入網際網路流量抵達 Web 伺服器。 此規則不會變更路由的行為。 這個規則只會允許指向 IIS01 的流量通過。 因此，如果來自網際網路的流量將 Web 伺服器做為其目的地，此規則會允許流量，並停止再處理其他規則。 (在優先順序為 140 的規則中，其他所有輸入網際網路流量皆會遭到封鎖)。 如果您只要處理 HTTP 流量，則可將此規則進一步限制為只允許目的地連接埠 80。

    ```JSON
    {
      "name": "enable_web_rule",
      "properties": {
        "description": "Enable Internet to [variables('VM01Name')]",
        "protocol": "Tcp",
        "sourcePortRange": "*",
        "destinationPortRange": "80",
        "sourceAddressPrefix": "Internet",
        "destinationAddressPrefix": "10.0.1.5",
        "access": "Allow",
        "priority": 120,
        "direction": "Inbound"
        }
      },
    ```

5. 此規則允許流量從 IIS01 伺服器傳遞到 AppVM01 伺服器，較後面的規則會封鎖其他所有 Frontend 到 Backend 的流量。 如果已知連接埠應該加入，可改善此規則。 例如，如果 IIS 伺服器只會抵達 AppVM01 上的 SQL Server，且 Web 應用程式曾遭到入侵，則目的地連接埠範圍應該從 “*” (任何) 變更為 1433 (SQL 連接埠)，以縮小 AppVM01 上的輸入攻擊面。

    ```JSON
    {
      "name": "enable_app_rule",
      "properties": {
        "description": "Enable [variables('VM01Name')] to [variables('VM02Name')]",
        "protocol": "*",
        "sourcePortRange": "*",
        "destinationPortRange": "*",
        "sourceAddressPrefix": "10.0.1.5",
        "destinationAddressPrefix": "10.0.2.5",
        "access": "Allow",
        "priority": 130,
        "direction": "Inbound"
      }
    },
     ```

6. 此規則會拒絕從網際網路到網路上任何伺服器的流量。 利用優先順序為 110 和 120 的規則，效果將可只允許輸入網際網路流量流往防火牆以及伺服器上的 RDP 連接埠，除此之外的其他流量則予以封鎖。 此規則是「保險」規則，可封鎖所有未預期的流程。

    ```JSON
    {
      "name": "deny_internet_rule",
      "properties": {
        "description": "Isolate the [variables('VNetName')] VNet from the Internet",
        "protocol": "*",
        "sourcePortRange": "*",
        "destinationPortRange": "*",
        "sourceAddressPrefix": "Internet",
        "destinationAddressPrefix": "VirtualNetwork",
        "access": "Deny",
        "priority": 140,
        "direction": "Inbound"
      }
    },
     ```

7. 最後一個規則會拒絕從 Frontend 子網路到 Backend 子網路的流量。 此規則是僅限輸入的規則，所以允許反向流量 (從 Backend 到 Frontend)。

    ```JSON
    {
      "name": "deny_frontend_rule",
      "properties": {
        "description": "Isolate the [variables('Subnet1Name')] subnet from the [variables('Subnet2Name')] subnet",
        "protocol": "*",
        "sourcePortRange": "*",
        "destinationPortRange": "*",
        "sourceAddressPrefix": "[variables('Subnet1Prefix')]",
        "destinationAddressPrefix": "[variables('Subnet2Prefix')]",
        "access": "Deny",
        "priority": 150,
        "direction": "Inbound"
      }
    }
    ```

## <a name="traffic-scenarios"></a>流量案例
#### <a name="allowed-internet-to-web-server"></a>(允許) 網際網路到 Web 伺服器
1. 網際網路使用者向 IIS01 NIC 相關聯的 NIC 公用 IP 位址要求 HTTP 頁面
2. 公用 IP 位址會將對 VNet 的流量傳遞至 IIS01 (web 伺服器)
3. Frontend 子網路開始處理輸入規則：
  1. NSG 規則 1 (DNS) 不適用，移至下一個規則
  2. NSG 規則 2 (RDP) 不適用，移至下一個規則
  3. NSG 規則 3 (網際網路到 IIS01) 適用，允許流量，停止處理規則
4. 流量抵達 Web 伺服器 IIS01 的內部 IP 位址 (10.0.1.5)
5. IIS01 正在接聽 Web 流量，接收此要求並開始處理要求
6. IIS01 向 AppVM01 上的 SQL Server 要求資訊
7. Frontend 子網路上沒有輸出規則，允許流量
8. Backend 子網路開始處理輸入規則：
  1. NSG 規則 1 (DNS) 不適用，移至下一個規則
  2. NSG 規則 2 (RDP) 不適用，移至下一個規則
  3. NSG 規則 3 (網際網路到防火牆) 不適用，移至下一個規則
  4. NSG 規則 4 (IIS01 到 AppVM01) 適用，允許流量，停止處理規則
9. AppVM01 接收 SQL 查詢並回應
10. Backend 子網路上沒有輸出規則，所以允許回應
11. Frontend 子網路開始處理輸入規則：
  1. Backend 子網路到 Frontend 子網路的輸入流量沒有適用的 NSG 規則，因此不會套用任何 NSG 規則
  2. 允許子網路間流量的預設系統規則會允許此流量，因此允許流量。
12. IIS 伺服器會接收 SQL 回應、完成 HTTP 回應並傳送給要求者
13. Frontend 子網路上沒有輸出規則，所以允許回應，網際網路使用者會收到要求的網頁。

#### <a name="allowed-rdp-to-iis-server"></a>(允許) RDP 到 IIS 伺服器
1. 網際網路上的伺服器系統管理員會要求 RDP 工作階段至與 IIS01 NIC 相關聯的 NIC 之公用 IP 位址上的 IIS01 (可以透過入口網站或 PowerShell 找到這個公用 IP 位址)
2. 公用 IP 位址會將對 VNet 的流量傳遞至 IIS01 (web 伺服器)
3. Frontend 子網路開始處理輸入規則：
  1. NSG 規則 1 (DNS) 不適用，移至下一個規則
  2. NSG 規則 2 (RDP) 適用，允許流量，停止處理規則
4. 由於沒有輸出規則，會套用預設規則並允許傳回的流量
5. 已啟用 RDP 工作階段
6. IIS01 會提示輸入使用者名稱和密碼

>[!NOTE]
>在這個執行個體中 RDP 至任何後端伺服器，IIS 伺服器會作為「跳躍箱」。 首先 RDP 到 IIS 伺服器，然後從 IIS 伺服器 RDP 到後端伺服器。
>
>

#### <a name="allowed-web-server-dns-look-up-on-dns-server"></a>(允許) DNS 伺服器上的 Web 伺服器 DNS 查閱
1. Web 伺服器 IIS01 需要 www.data.gov 的資料摘要，但需要解析位址。
2. VNet 的網路組態將 DNS01 (Backend 子網路上的 10.0.2.4) 列為主要 DNS 伺服器，IIS01 將 DNS 要求傳送至 DNS01
3. Frontend 子網路上沒有輸出規則，允許流量
4. Backend 子網路開始處理輸入規則：
  * NSG 規則 1 (DNS) 適用，允許流量，停止處理規則
5. DNS 伺服器收到要求
6. DNS 伺服器沒有快取的位址，並要求網際網路上的根 DNS 伺服器
7. Backend 子網路上沒有輸出規則，允許流量
8. 網際網路 DNS 伺服器回應，因為此工作階段是從內部起始，所以允許回應
9. DNS 伺服器快取回應，然後將初始要求回應到 IIS01
10. Backend 子網路上沒有輸出規則，允許流量
11. Frontend 子網路開始處理輸入規則：
  1. Backend 子網路到 Frontend 子網路的輸入流量沒有適用的 NSG 規則，因此不會套用任何 NSG 規則
  2. 允許子網路間流量的預設系統規則會允許此流量，因此允許流量
12. IIS01 從 DNS01 接收回應

#### <a name="allowed-web-server-access-file-on-appvm01"></a>(允許) Web 伺服器存取 AppVM01 上的檔案
1. IIS01 要求 AppVM01 上的檔案
2. Frontend 子網路上沒有輸出規則，允許流量
3. Backend 子網路開始處理輸入規則：
  1. NSG 規則 1 (DNS) 不適用，移至下一個規則
  2. NSG 規則 2 (RDP) 不適用，移至下一個規則
  3. NSG 規則 3 (網際網路到 IIS01) 不適用，移至下一個規則
  4. NSG 規則 4 (IIS01 到 AppVM01) 適用，允許流量，停止處理規則
4. AppVM01 接收要求並以檔案回應 (假設已獲得存取授權)
5. Backend 子網路上沒有輸出規則，所以允許回應
6. Frontend 子網路開始處理輸入規則：
  1. Backend 子網路到 Frontend 子網路的輸入流量沒有適用的 NSG 規則，因此不會套用任何 NSG 規則
  2. 允許子網路間流量的預設系統規則會允許此流量，因此允許流量。
7. IIS 伺服器接收檔案

#### <a name="denied-rdp-to-backend"></a>(拒絕) RDP 到 Backend
1. 網際網路使用者嘗試 RDP 伺服器 AppVM01
2. 沒有與此伺服器 NIC 相關聯的公用 IP 位址，因此這個流量永遠不會輸入 VNet，而且不會連線到伺服器
3. 不過，如果因為某些原因已啟用公用 IP 位址，NSG 規則 2 (RDP) 會允許此流量

>[!NOTE]
>在這個執行個體中 RDP 至任何後端伺服器，IIS 伺服器會作為「跳躍箱」。 首先 RDP 到 IIS 伺服器，然後從 IIS 伺服器 RDP 到後端伺服器。
>
>

#### <a name="denied-web-to-backend-server"></a>(拒絕) Web 到 Backend 伺服器
1. 網際網路使用者嘗試存取 AppVM01 上的檔案
2. 沒有與此伺服器 NIC 相關聯的公用 IP 位址，因此這個流量永遠不會輸入 VNet，而且不會連線到伺服器
3. 如果因為某些原因已啟用公用 IP 位址，NSG 規則 5 (網際網路到 VNet) 會封鎖此流量

#### <a name="denied-web-dns-look-up-on-dns-server"></a>(拒絕) DNS 伺服器上的 Web DNS 查閱
1. 網際網路使用者嘗試查閱 DNS01 上的內部 DNS 記錄
2. 沒有與此伺服器 NIC 相關聯的公用 IP 位址，因此這個流量永遠不會輸入 VNet，而且不會連線到伺服器
3. 如果因為某些原因已啟用公用 IP 位址，NSG 規則 5 (網際網路到 VNet) 會封鎖此流量 (請注意︰因為要求來源位址是網際網路且規則 1 僅適用於做為來源的本機 VNet，因此不會套用規則 1 (DNS))

#### <a name="denied-sql-access-on-the-web-server"></a>(拒絕) 網頁伺服器上的 SQL 存取
1. 網際網路使用者向 IIS01 要求 SQL 資料
2. 沒有與此伺服器 NIC 相關聯的公用 IP 位址，因此這個流量永遠不會輸入 VNet，而且不會連線到伺服器
3. 如果基於某些原因而啟用公用 IP 位址，Frontend 子網路會開始處理輸入規則：
  1. NSG 規則 1 (DNS) 不適用，移至下一個規則
  2. NSG 規則 2 (RDP) 不適用，移至下一個規則
  3. NSG 規則 3 (網際網路到 IIS01) 適用，允許流量，停止處理規則
4. 流量抵達 IIS01 的內部 IP 位址 (10.0.1.5)
5. IIS01 未接聽連接埠 1433，所以要求沒有回應

## <a name="conclusion"></a>結論
這個範例隔離後端子網路與輸入流量的方式相當直接簡單。

您可以在[這裡][HOME]找到更多範例和網路安全性界限的概觀。

## <a name="references"></a>參考
### <a name="azure-resource-manager-template"></a>Azure Resource Manager 範本
此範例使用預先定義的 Azure Resource Manager 範本 (範本位於由 Microsoft 維護且開放社群使用的 GitHub 存放庫中)。 這個範本可以直接從 GitHub 部署，或下載並修改以符合您的需求。 

主要範本位於名為 "azuredeploy.json" 的檔案中。 此範本可以透過 PowerShell 或 CLI (具有相關聯的 "azuredeploy.parameters.json" 檔案) 提交來部署此範本。 我發現最簡單的方式是在 GitHub 使用 README.md 頁面上的 [部署至 Azure] 按鈕。

若要部署從 GitHub 和 Azure 入口網站建立此範例的範本，請依照下列步驟執行︰

1. 從瀏覽器，瀏覽至[範本][Template]
2. 按一下 [部署至 Azure] 按鈕 (或 [視覺化] 按鈕以查看這個範本的圖形表示法)
3. 在 [參數] 刀鋒視窗中，輸入儲存體帳戶、使用者名稱和密碼，然後按一下 [確定]
5. 建立此部署的資源群組 (您可以使用現有的資源群組，但建議您建立新的以獲得最佳結果)
6. 如有必要，變更 VNet 的訂用帳戶和位置設定。
7. 按一下 [檢閱法律條款] 並閱讀條款，然後按一下 [購買] 表示同意條款。
8. 按一下 [建立] 開始部署此範本。
9. 部署成功完成後，瀏覽至針對此部署建立的資源群組，以查看內部設定的資源。

>[!NOTE]
>此範本僅對 IIS01 伺服器啟用 RDP (在入口網站上尋找 IIS01 的公用 IP)。 在這個執行個體中 RDP 至任何後端伺服器，IIS 伺服器會作為「跳躍箱」。 首先 RDP 到 IIS 伺服器，然後從 IIS 伺服器 RDP 到後端伺服器。
>
>

若要移除此部署，請刪除資源群組，也會刪除所有子資源。

#### <a name="sample-application-scripts"></a>範例應用程式指令碼
一旦範本順利執行後，您可以設定 Web 伺服器和具有簡單 Web 應用程式的應用程式伺服器，以便能使用此 DMZ 組態進行測試。 如需為此範例和其他 DMZ 範例安裝範例應用程式，下列連結中有提供一個：[範例應用程式指令碼][SampleApp]

## <a name="next-steps"></a>後續步驟

* 部署此範例
* 建置範例應用程式
* 測試流經此 DMZ 的不同流量

<!--Image References-->
[1]: ./media/virtual-networks-dmz-nsg-arm/example1design.png "具有 NSG 的輸入 DMZ"

<!--Link References-->
[HOME]: ../best-practices-network-security.md
[Template]: https://github.com/Azure/azure-quickstart-templates/tree/master/301-dmz-nsg
[SampleApp]: ./virtual-networks-sample-app.md