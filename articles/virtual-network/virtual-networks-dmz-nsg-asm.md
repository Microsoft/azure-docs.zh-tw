---
title: Azure DMZ 範例 – 建置具有 NSG 的簡單 DMZ | Microsoft Docs
description: 建置具有網路安全性群組 (NSG) 的 DMZ
services: virtual-network
documentationcenter: na
author: tracsman
manager: rossort
editor: ''
ms.assetid: f8622b1d-c07d-4ea6-b41c-4ae98d998fff
ms.service: virtual-network
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 01/03/2017
ms.author: jonor
ms.openlocfilehash: ed172d552e1e4c9ee27c58abcd7ad2d98df21579
ms.sourcegitcommit: 6fcd9e220b9cd4cb2d4365de0299bf48fbb18c17
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/05/2018
ms.locfileid: "23125496"
---
# <a name="example-1--build-a-simple-dmz-using-nsgs-with-classic-powershell"></a>範例 1 – 使用 NSG 搭配傳統 PowerShell 建立簡單的 DMZ
[返回 [安全性界限最佳作法] 頁面][HOME]

> [!div class="op_single_selector"]
> * [Resource Manager 範本](virtual-networks-dmz-nsg.md)
> * [傳統 - PowerShell](virtual-networks-dmz-nsg-asm.md)
> 
>

此範例會建立包含四個 Windows 伺服器和網路安全性群組的基本 DMZ。 此範例說明每個相關 PowerShell 命令，以讓您更加深入地了解每個步驟。 另外您還會看到＜流量案例＞一節，本節提供深入的逐步說明，讓您知道流量是如何流經 DMZ 內的各個防禦層。 最後則有＜參考＞一節，本節提供完整的程式碼和指示，以供您建置此環境來測試和試驗各種案例。 

![具有 NSG 的輸入 DMZ][1]

## <a name="environment-description"></a>環境描述
此範例中，訂用帳戶包含下列資源：

* 兩個雲端服務：“FrontEnd001” 和 “BackEnd001”
* 一個虛擬網路 “CorpNetwork”，包含兩個子網路：“FrontEnd” 和 “BackEnd”
* 套用至這兩個子網路的單一網路安全性群組
* 一個代表應用程式 Web 伺服器的 Windows Server (“IIS01”)
* 兩個代表應用程式後端伺服器的 Windows Server (“AppVM01”、“AppVM02”)
* 一個代表 DNS 伺服器的 Windows Server ("DNS01")

在＜參考＞一節中有提供 PowerShell 指令碼，其可建置本範例中的大部分環境。 至於 VM 和虛擬網路的建置，雖然也是由此範例指令碼來完成，但本文不會詳加敘述。 

建置環境：

1. 儲存＜參考＞一節中所包含的網路組態 xml 檔 (更新名稱、位置和 IP 位址以符合給定的案例)
2. 更新指令碼中的使用者變數，以符合要用來執行指令碼的環境 (訂用帳戶、服務名稱等)
3. 在 PowerShell 中執行指令碼

>[!Note]
>PowerShell 指令碼中所指的區域必須符合網路組態 xml 檔中所指的區域。
>
>

指令碼順利執行後，即可採取其他選擇性步驟，＜參考＞一節中有兩個指令碼，其會設定 Web 伺服器和具有簡單 Web 應用程式的應用程式伺服器，以便能使用此 DMZ 組態進行測試。

下列各節藉由逐步解說 PowerShell 指令碼的重要程式行，詳細說明此範例的網路安全性群組和其運作方式。

## <a name="network-security-groups-nsg"></a>網路安全性群組 (NSG)
此範例會組建 NSG 群組，然後載入六個規則。 

> [!TIP]
> 一般而言，您應該先建立特定的「允許」規則，最後再建立較一般的「拒絕」規則。 所指定的優先順序會決定要先評估哪些規則。 一旦發現流量適用特定規則，就不會再評估後續規則。 NSG 規則可以套用在輸入或輸出方向 (從子網路的觀點出發)。
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

有一個預設輸出規則可允許流量外流到網際網路。 在此範例中，我們會允許輸出流量，並不會修改任何輸出規則。 如果兩個方向的流量都要鎖定，則需要使用者定義的路由，會在[安全性界限最佳作法頁面][HOME]上的＜範例 3＞中探討。

以下會更為詳細地討論每個規則 (**注意**：下列清單中以貨幣符號開頭的項目 (例如：$NSGName) 皆為來自本文＜參考＞一節之指令碼的使用者定義變數)：

1. 首先必須建置網路安全性群組來保存規則：

    ```PowerShell
    New-AzureNetworkSecurityGroup -Name $NSGName `
        -Location $DeploymentLocation `
        -Label "Security group for $VNetName subnets in $DeploymentLocation"
    ```

2. 此範例中的第一個規則會允許所有內部網路之間的 DNS 流量流往 Backend 子網路上的 DNS 伺服器。 此規則有一些重要參數：
   
   * 「類型」表示此規則會生效的傳輸流量方向。 方向是來自子網路或虛擬機器的角度 (取決於此 NSG 繫結的位置)。 因此，如果 Type 是 “Inbound” 且流量進入子網路，此規則將會適用，而離開子網路的流量則不受此規則所影響。
   * "Priority" 會設定流量的評估順序。 編號愈低，優先順序就愈高。 當規則套用至特定流量時，就不會再處理其他規則。 因此，如果優先順序為 1 的規則允許流量，優先順序為 2 的規則拒絕流量，而這兩個規則皆適用於流量，則會允許流量流動 (規則 1 有更高的優先順序，所以會生效，並且不會再套用其他規則)。
   * “Action” 指出受此規則影響的流量是要封鎖或允許。

    ```PowerShell    
    Get-AzureNetworkSecurityGroup -Name $NSGName | `
        Set-AzureNetworkSecurityRule -Name "Enable Internal DNS" `
        -Type Inbound -Priority 100 -Action Allow `
        -SourceAddressPrefix VIRTUAL_NETWORK -SourcePortRange '*' `
        -DestinationAddressPrefix $VMIP[4] `
        -DestinationPortRange '53' `
        -Protocol *
    ```

3. 此規則會允許 RDP 流量從網際網路流往繫結子網路上任何伺服器的 RDP 連接埠。 此規則使用兩種特殊位址前置詞：“VIRTUAL_NETWORK” 和 “INTERNET”。 這些標記可輕易處理較大類別的位址前置詞。

    ```PowerShell
    Get-AzureNetworkSecurityGroup -Name $NSGName | `
         Set-AzureNetworkSecurityRule -Name "Enable RDP to $VNetName VNet" `
         -Type Inbound -Priority 110 -Action Allow `
         -SourceAddressPrefix INTERNET -SourcePortRange '*' `
         -DestinationAddressPrefix VIRTUAL_NETWORK `
         -DestinationPortRange '3389' `
         -Protocol *
    ```

4. 此規則允許輸入網際網路流量抵達 Web 伺服器。 此規則不會變更路由的行為。 這個規則只會允許指向 IIS01 的流量通過。 因此，如果來自網際網路的流量將 Web 伺服器做為其目的地，此規則會允許流量，並停止再處理其他規則。 (在優先順序為 140 的規則中，其他所有輸入網際網路流量皆會遭到封鎖)。 如果您只要處理 HTTP 流量，則可將此規則進一步限制為只允許目的地連接埠 80。

    ```PowerShell
    Get-AzureNetworkSecurityGroup -Name $NSGName | `
         Set-AzureNetworkSecurityRule -Name "Enable Internet to $VMName[0]" `
         -Type Inbound -Priority 120 -Action Allow `
         -SourceAddressPrefix Internet -SourcePortRange '*' `
         -DestinationAddressPrefix $VMIP[0] `
         -DestinationPortRange '*' `
         -Protocol *
    ```

5. 此規則允許流量從 IIS01 伺服器傳遞到 AppVM01 伺服器，較後面的規則會封鎖其他所有 Frontend 到 Backend 的流量。 如果已知連接埠應該加入，可改善此規則。 例如，如果 IIS 伺服器只會抵達 AppVM01 上的 SQL Server，且 Web 應用程式曾遭到入侵，則目的地連接埠範圍應該從 “*” (任何) 變更為 1433 (SQL 連接埠)，以縮小 AppVM01 上的輸入攻擊面。

    ```PowerShell
    Get-AzureNetworkSecurityGroup -Name $NSGName | `
        Set-AzureNetworkSecurityRule -Name "Enable $VMName[1] to $VMName[2]" `
        -Type Inbound -Priority 130 -Action Allow `
        -SourceAddressPrefix $VMIP[1] -SourcePortRange '*' `
        -DestinationAddressPrefix $VMIP[2] `
        -DestinationPortRange '*' `
        -Protocol *
    ```

6. 此規則會拒絕從網際網路到網路上任何伺服器的流量。 利用優先順序為 110 和 120 的規則，效果將可只允許輸入網際網路流量流往防火牆以及伺服器上的 RDP 連接埠，除此之外的其他流量則予以封鎖。 此規則是「保險」規則，可封鎖所有未預期的流程。
    ```PowerShell
    Get-AzureNetworkSecurityGroup -Name $NSGName | `
        Set-AzureNetworkSecurityRule `
        -Name "Isolate the $VNetName VNet from the Internet" `
        -Type Inbound -Priority 140 -Action Deny `
        -SourceAddressPrefix INTERNET -SourcePortRange '*' `
        -DestinationAddressPrefix VIRTUAL_NETWORK `
        -DestinationPortRange '*' `
        -Protocol *
    ```
7. 最後一個規則會拒絕從 Frontend 子網路到 Backend 子網路的流量。 此規則是僅限輸入的規則，所以允許反向流量 (從 Backend 到 Frontend)。

    ```PowerShell
    Get-AzureNetworkSecurityGroup -Name $NSGName | `
        Set-AzureNetworkSecurityRule `
        -Name "Isolate the $FESubnet subnet from the $BESubnet subnet" `
        -Type Inbound -Priority 150 -Action Deny `
        -SourceAddressPrefix $FEPrefix -SourcePortRange '*' `
        -DestinationAddressPrefix $BEPrefix `
        -DestinationPortRange '*' `
        -Protocol * 
    ```

## <a name="traffic-scenarios"></a>流量案例
#### <a name="allowed-internet-to-web-server"></a>(允許) 網際網路到 Web 伺服器
1. 網際網路使用者從 FrontEnd001.CloudApp.Net (網際網路面向雲端服務) 要求 HTTP 頁面
2. 雲端服務透過連接埠 80 上的開放端點將流量傳遞至 IIS01 (Web 伺服器)
3. Frontend 子網路開始處理輸入規則：
   1. NSG 規則 1 (DNS) 不適用，移至下一個規則
   2. NSG 規則 2 (RDP) 不適用，移至下一個規則
   3. NSG 規則 3 (網際網路到 IIS01) 適用，允許流量，停止處理規則
4. 流量抵達 Web 伺服器 IIS01 的內部 IP 位址 (10.0.1.5)
5. IIS01 正在接聽 Web 流量，接收此要求並開始處理要求
6. IIS01 向 AppVM01 上的 SQL Server 要求資訊
7. Frontend 子網路上沒有輸出規則，所以允許流量
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
12. IIS 伺服器接收 SQL 回應、完成 HTTP 回應並傳送給要求者
13. Frontend 子網路上沒有輸出規則，所以允許回應，網際網路使用者會收到要求的網頁。

#### <a name="allowed-rdp-to-backend"></a>(允許) RDP 到 Backend
1. 網際網路上的伺服器管理員在 BackEnd001.CloudApp.Net:xxxxx 上要求 AppVM01 的 RDP 工作階段，其中 xxxxx 是 RDP 到 AppVM01 的隨機指派連接埠號碼 (在 Azure 入口網站上或透過 PowerShell，即可找到指派的連接埠)
2. Backend 子網路開始處理輸入規則：
   1. NSG 規則 1 (DNS) 不適用，移至下一個規則
   2. NSG 規則 2 (RDP) 適用，允許流量，停止處理規則
3. 由於沒有輸出規則，會套用預設規則並允許傳回的流量
4. 已啟用 RDP 工作階段
5. AppVM01 會提示輸入使用者名稱和密碼

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

#### <a name="denied-web-to-backend-server"></a>(拒絕) Web 到 Backend 伺服器
1. 網際網路使用者嘗試透過 BackEnd001.CloudApp.Net 服務存取 AppVM01 上的檔案
2. 因為沒有用於檔案共用的開放端點，此流量不會通過雲端服務到達伺服器
3. 如果基於某些原因而開放端點，NSG 規則 5 (網際網路到 VNet) 會封鎖此流量

#### <a name="denied-web-dns-look-up-on-dns-server"></a>(拒絕) DNS 伺服器上的 Web DNS 查閱
1. 網際網路使用者嘗試透過 BackEnd001.CloudApp.Net 服務查閱 DNS01 上的內部 DNS 記錄
2. 因為沒有用於 DNS 的開放端點，此流量不會通過雲端服務到達伺服器
3. 如果基於某些原因而開放端點，NSG 規則 5 (網際網路到 VNet) 會封鎖此流量 (注意：有兩個原因導致規則 1 (DNS) 不適用，首先，來源位址是網際網路，此規則只適用於以本機 VNet 做為來源，再者，此規則是允許規則，所以它永遠不會拒絕流量)

#### <a name="denied-web-to-sql-access-through-firewall"></a>(拒絕) Web 透過防火牆對 SQL 進行存取
1. 網際網路使用者從 FrontEnd001.CloudApp.Net (網際網路面向雲端服務) 要求 SQL 資料
2. 因為沒有用於 SQL 的開放端點，此流量不會通過雲端服務到達防火牆
3. 如果基於某些原因而開放端點，Frontend 子網路會開始處理輸入規則：
   1. NSG 規則 1 (DNS) 不適用，移至下一個規則
   2. NSG 規則 2 (RDP) 不適用，移至下一個規則
   3. NSG 規則 3 (網際網路到 IIS01) 適用，允許流量，停止處理規則
4. 流量抵達 IIS01 的內部 IP 位址 (10.0.1.5)
5. IIS01 未接聽連接埠 1433，所以要求沒有回應

## <a name="conclusion"></a>結論
這個範例隔離後端子網路與輸入流量的方式相當直接簡單。

您可以在[這裡][HOME]找到更多範例和網路安全性界限的概觀。

## <a name="references"></a>參考
### <a name="main-script-and-network-config"></a>主要的指令碼和網路組態
將完整指令碼儲存在 PowerShell 指令碼檔案中。 將網路組態儲存到名為 “NetworkConf1.xml” 的檔案。
視需要修改使用者定義的變數並執行指令碼。

#### <a name="full-script"></a>完整指令碼
根據使用者定義的變數，此指令碼會執行下列動作；

1. 連線到 Azure 訂用帳戶
2. 建立儲存體帳戶
3. 依網路組態檔中的定義建立 VNet 和兩個子網路
4. 建立四個 Windows Server VM
5. 設定 NSG，包括：
   * 建立 NSG
   * 在其中填入規則
   * 將 NSG 繫結至適當的子網路

此 PowerShell 指令碼應該在連線到網際網路的電腦或伺服器上本機執行。

> [!IMPORTANT]
> 此指令碼在執行時，PowerShell 中可能會跳出警告或其他參考訊息。 只有紅色字體的錯誤訊息才需要擔心。
> 
>

```PowerShell
<# 
 .SYNOPSIS
  Example of Network Security Groups in an isolated network (Azure only, no hybrid connections)

 .DESCRIPTION
  This script will build out a sample DMZ setup containing:
   - A default storage account for VM disks
   - Two new cloud services
   - Two Subnets (FrontEnd and BackEnd subnets)
   - One server on the FrontEnd Subnet
   - Three Servers on the BackEnd Subnet
   - Network Security Groups to allow/deny traffic patterns as declared

  Before running script, ensure the network configuration file is created in
  the directory referenced by $NetworkConfigFile variable (or update the
  variable to reflect the path and file name of the config file being used).

 .Notes
  Security requirements are different for each use case and can be addressed in a
  myriad of ways. Please be sure that any sensitive data or applications are behind
  the appropriate layer(s) of protection. This script serves as an example of some
  of the techniques that can be used, but should not be used for all scenarios. You
  are responsible to assess your security needs and the appropriate protections
  needed, and then effectively implement those protections.

  FrontEnd Service (FrontEnd subnet 10.0.1.0/24)
   IIS01      - 10.0.1.5

  BackEnd Service (BackEnd subnet 10.0.2.0/24)
   DNS01      - 10.0.2.4
   AppVM01    - 10.0.2.5
   AppVM02    - 10.0.2.6

#>

# Fixed Variables
    $LocalAdminPwd = Read-Host -Prompt "Enter Local Admin Password to be used for all VMs"
    $VMName = @()
    $ServiceName = @()
    $VMFamily = @()
    $img = @()
    $size = @()
    $SubnetName = @()
    $VMIP = @()

# User-Defined Global Variables
  # These should be changes to reflect your subscription and services
  # Invalid options will fail in the validation section

  # Subscription Access Details
    $subID = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"

  # VM Account, Location, and Storage Details
    $LocalAdmin = "theAdmin"
    $DeploymentLocation = "Central US"
    $StorageAccountName = "vmstore02"

  # Service Details
    $FrontEndService = "FrontEnd001"
    $BackEndService = "BackEnd001"

  # Network Details
    $VNetName = "CorpNetwork"
    $FESubnet = "FrontEnd"
    $FEPrefix = "10.0.1.0/24"
    $BESubnet = "BackEnd"
    $BEPrefix = "10.0.2.0/24"
    $NetworkConfigFile = "C:\Scripts\NetworkConf1.xml"

  # VM Base Disk Image Details
    $SrvImg = Get-AzureVMImage | Where {$_.ImageFamily -match 'Windows Server 2012 R2 Datacenter'} | sort PublishedDate -Descending | Select ImageName -First 1 | ForEach {$_.ImageName}

  # NSG Details
    $NSGName = "MyVNetSG"

# User-Defined VM Specific Configuration
    # Note: To ensure proper NSG Rule creation later in this script:
    #       - The Web Server must be VM 0
    #       - The AppVM1 Server must be VM 1
    #       - The DNS server must be VM 3
    #
    #       Otherwise the NSG rules in the last section of this
    #       script will need to be changed to match the modified
    #       VM array numbers ($i) so the NSG Rule IP addresses
    #       are aligned to the associated VM IP addresses.

    # VM 0 - The Web Server
      $VMName += "IIS01"
      $ServiceName += $FrontEndService
      $VMFamily += "Windows"
      $img += $SrvImg
      $size += "Standard_D3"
      $SubnetName += $FESubnet
      $VMIP += "10.0.1.5"

    # VM 1 - The First Application Server
      $VMName += "AppVM01"
      $ServiceName += $BackEndService
      $VMFamily += "Windows"
      $img += $SrvImg
      $size += "Standard_D3"
      $SubnetName += $BESubnet
      $VMIP += "10.0.2.5"

    # VM 2 - The Second Application Server
      $VMName += "AppVM02"
      $ServiceName += $BackEndService
      $VMFamily += "Windows"
      $img += $SrvImg
      $size += "Standard_D3"
      $SubnetName += $BESubnet
      $VMIP += "10.0.2.6"

    # VM 3 - The DNS Server
      $VMName += "DNS01"
      $ServiceName += $BackEndService
      $VMFamily += "Windows"
      $img += $SrvImg
      $size += "Standard_D3"
      $SubnetName += $BESubnet
      $VMIP += "10.0.2.4"

# ----------------------------- #
# No User-Defined Variables or  #
# Configuration past this point #
# ----------------------------- #    

  # Get your Azure accounts
    Add-AzureAccount
    Set-AzureSubscription –SubscriptionId $subID -ErrorAction Stop
    Select-AzureSubscription -SubscriptionId $subID -Current -ErrorAction Stop

  # Create Storage Account
    If (Test-AzureName -Storage -Name $StorageAccountName) { 
        Write-Host "Fatal Error: This storage account name is already in use, please pick a different name." -ForegroundColor Red
        Return}
    Else {Write-Host "Creating Storage Account" -ForegroundColor Cyan 
          New-AzureStorageAccount -Location $DeploymentLocation -StorageAccountName $StorageAccountName}

  # Update Subscription Pointer to New Storage Account
    Write-Host "Updating Subscription Pointer to New Storage Account" -ForegroundColor Cyan 
    Set-AzureSubscription –SubscriptionId $subID -CurrentStorageAccountName $StorageAccountName -ErrorAction Stop

# Validation
$FatalError = $false

If (-Not (Get-AzureLocation | Where {$_.DisplayName -eq $DeploymentLocation})) {
     Write-Host "This Azure Location was not found or available for use" -ForegroundColor Yellow
     $FatalError = $true}

If (Test-AzureName -Service -Name $FrontEndService) { 
    Write-Host "The FrontEndService service name is already in use, please pick a different service name." -ForegroundColor Yellow
    $FatalError = $true}
Else { Write-Host "The FrontEndService service name is valid for use." -ForegroundColor Green}

If (Test-AzureName -Service -Name $BackEndService) { 
    Write-Host "The BackEndService service name is already in use, please pick a different service name." -ForegroundColor Yellow
    $FatalError = $true}
Else { Write-Host "The BackEndService service name is valid for use." -ForegroundColor Green}

If (-Not (Test-Path $NetworkConfigFile)) { 
    Write-Host 'The network config file was not found, please update the $NetworkConfigFile variable to point to the network config xml file.' -ForegroundColor Yellow
    $FatalError = $true}
Else { Write-Host "The network configuration file was found" -ForegroundColor Green
        If (-Not (Select-String -Pattern $DeploymentLocation -Path $NetworkConfigFile)) {
            Write-Host 'The deployment location was not found in the network config file, please check the network config file to ensure the $DeploymentLocation variable is correct and the network config file matches.' -ForegroundColor Yellow
            $FatalError = $true}
        Else { Write-Host "The deployment location was found in the network config file." -ForegroundColor Green}}

If ($FatalError) {
    Write-Host "A fatal error has occurred, please see the above messages for more information." -ForegroundColor Red
    Return}
Else { Write-Host "Validation passed, now building the environment." -ForegroundColor Green}

# Create VNET
    Write-Host "Creating VNET" -ForegroundColor Cyan 
    Set-AzureVNetConfig -ConfigurationPath $NetworkConfigFile -ErrorAction Stop

# Create Services
    Write-Host "Creating Services" -ForegroundColor Cyan
    New-AzureService -Location $DeploymentLocation -ServiceName $FrontEndService -ErrorAction Stop
    New-AzureService -Location $DeploymentLocation -ServiceName $BackEndService -ErrorAction Stop

# Build VMs
    $i=0
    $VMName | Foreach {
        Write-Host "Building $($VMName[$i])" -ForegroundColor Cyan
        New-AzureVMConfig -Name $VMName[$i] -ImageName $img[$i] –InstanceSize $size[$i] | `
            Add-AzureProvisioningConfig -Windows -AdminUsername $LocalAdmin -Password $LocalAdminPwd  | `
            Set-AzureSubnet  –SubnetNames $SubnetName[$i] | `
            Set-AzureStaticVNetIP -IPAddress $VMIP[$i] | `
            Set-AzureVMMicrosoftAntimalwareExtension -AntimalwareConfiguration '{"AntimalwareEnabled" : true}' | `
            Remove-AzureEndpoint -Name "PowerShell" | `
            New-AzureVM –ServiceName $ServiceName[$i] -VNetName $VNetName -Location $DeploymentLocation
            # Note: A Remote Desktop endpoint is automatically created when each VM is created.
        $i++
    }
    # Add HTTP Endpoint for IIS01
    Get-AzureVM -ServiceName $ServiceName[0] -Name $VMName[0] | Add-AzureEndpoint -Name HTTP -Protocol tcp -LocalPort 80 -PublicPort 80 | Update-AzureVM

# Configure NSG
    Write-Host "Configuring the Network Security Group (NSG)" -ForegroundColor Cyan

  # Build the NSG
    Write-Host "Building the NSG" -ForegroundColor Cyan
    New-AzureNetworkSecurityGroup -Name $NSGName -Location $DeploymentLocation -Label "Security group for $VNetName subnets in $DeploymentLocation"

  # Add NSG Rules
    Write-Host "Writing rules into the NSG" -ForegroundColor Cyan
    Get-AzureNetworkSecurityGroup -Name $NSGName | Set-AzureNetworkSecurityRule -Name "Enable Internal DNS" -Type Inbound -Priority 100 -Action Allow `
        -SourceAddressPrefix VIRTUAL_NETWORK -SourcePortRange '*' `
        -DestinationAddressPrefix $VMIP[3] -DestinationPortRange '53' `
        -Protocol *

    Get-AzureNetworkSecurityGroup -Name $NSGName | Set-AzureNetworkSecurityRule -Name "Enable RDP to $VNetName VNet" -Type Inbound -Priority 110 -Action Allow `
        -SourceAddressPrefix INTERNET -SourcePortRange '*' `
        -DestinationAddressPrefix VIRTUAL_NETWORK -DestinationPortRange '3389' `
        -Protocol *

    Get-AzureNetworkSecurityGroup -Name $NSGName | Set-AzureNetworkSecurityRule -Name "Enable Internet to $($VMName[0])" -Type Inbound -Priority 120 -Action Allow `
        -SourceAddressPrefix Internet -SourcePortRange '*' `
        -DestinationAddressPrefix $VMIP[0] -DestinationPortRange '*' `
        -Protocol *

    Get-AzureNetworkSecurityGroup -Name $NSGName | Set-AzureNetworkSecurityRule -Name "Enable $($VMName[0]) to $($VMName[1])" -Type Inbound -Priority 130 -Action Allow `
        -SourceAddressPrefix $VMIP[0] -SourcePortRange '*' `
        -DestinationAddressPrefix $VMIP[1] -DestinationPortRange '*' `
        -Protocol *

    Get-AzureNetworkSecurityGroup -Name $NSGName | Set-AzureNetworkSecurityRule -Name "Isolate the $VNetName VNet from the Internet" -Type Inbound -Priority 140 -Action Deny `
        -SourceAddressPrefix INTERNET -SourcePortRange '*' `
        -DestinationAddressPrefix VIRTUAL_NETWORK -DestinationPortRange '*' `
        -Protocol *

    Get-AzureNetworkSecurityGroup -Name $NSGName | Set-AzureNetworkSecurityRule -Name "Isolate the $FESubnet subnet from the $BESubnet subnet" -Type Inbound -Priority 150 -Action Deny `
        -SourceAddressPrefix $FEPrefix -SourcePortRange '*' `
        -DestinationAddressPrefix $BEPrefix -DestinationPortRange '*' `
        -Protocol *

    # Assign the NSG to the Subnets
        Write-Host "Binding the NSG to both subnets" -ForegroundColor Cyan
        Set-AzureNetworkSecurityGroupToSubnet -Name $NSGName -SubnetName $FESubnet -VirtualNetworkName $VNetName
        Set-AzureNetworkSecurityGroupToSubnet -Name $NSGName -SubnetName $BESubnet -VirtualNetworkName $VNetName

# Optional Post-script Manual Configuration
  # Install Test Web App (Run Post-Build Script on the IIS Server)
  # Install Backend resource (Run Post-Build Script on the AppVM01)
  Write-Host
  Write-Host "Build Complete!" -ForegroundColor Green
  Write-Host
  Write-Host "Optional Post-script Manual Configuration Steps" -ForegroundColor Gray
  Write-Host " - Install Test Web App (Run Post-Build Script on the IIS Server)" -ForegroundColor Gray
  Write-Host " - Install Backend resource (Run Post-Build Script on the AppVM01)" -ForegroundColor Gray
  Write-Host
```

#### <a name="network-config-file"></a>網路組態檔
以更新的位置儲存此 xml 檔案，並將此檔案的連結新增到先前指令碼中的 $NetworkConfigFile 變數。

```XML
<NetworkConfiguration xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://schemas.microsoft.com/ServiceHosting/2011/07/NetworkConfiguration">
  <VirtualNetworkConfiguration>
    <Dns>
      <DnsServers>
        <DnsServer name="DNS01" IPAddress="10.0.2.4" />
        <DnsServer name="Level3" IPAddress="209.244.0.3" />
      </DnsServers>
    </Dns>
    <VirtualNetworkSites>
      <VirtualNetworkSite name="CorpNetwork" Location="Central US">
        <AddressSpace>
          <AddressPrefix>10.0.0.0/16</AddressPrefix>
        </AddressSpace>
        <Subnets>
          <Subnet name="FrontEnd">
            <AddressPrefix>10.0.1.0/24</AddressPrefix>
          </Subnet>
          <Subnet name="BackEnd">
            <AddressPrefix>10.0.2.0/24</AddressPrefix>
          </Subnet>
        </Subnets>
        <DnsServersRef>
          <DnsServerRef name="DNS01" />
          <DnsServerRef name="Level3" />
        </DnsServersRef>
      </VirtualNetworkSite>
    </VirtualNetworkSites>
  </VirtualNetworkConfiguration>
</NetworkConfiguration>
```

#### <a name="sample-application-scripts"></a>範例應用程式指令碼
如果您想要為此範例和其他 DMZ 範例安裝範例應用程式，下列連結中有提供一個：[範例應用程式指令碼][SampleApp]

## <a name="next-steps"></a>後續步驟
* 更新並儲存 XML 檔案
* 執行 PowerShell 指令碼來建置環境
* 安裝範例應用程式
* 測試流經此 DMZ 的不同流量

<!--Image References-->
[1]: ./media/virtual-networks-dmz-nsg-asm/example1design.png "具有 NSG 的輸入 DMZ"

<!--Link References-->
[HOME]: ../best-practices-network-security.md
[SampleApp]: ./virtual-networks-sample-app.md

