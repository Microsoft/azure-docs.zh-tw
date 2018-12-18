---
title: DMZ 範例 – 建置 DMZ 以透過防火牆和 NSG 保護應用程式 | Microsoft Docs
description: 建置具有防火牆和網路安全性群組 (NSG) 的 DMZ
services: virtual-network
documentationcenter: na
author: tracsman
manager: rossort
editor: ''
ms.assetid: c78491c7-54ac-4469-851c-b35bfed0f528
ms.service: virtual-network
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/01/2016
ms.author: jonor;sivae
ms.openlocfilehash: cc0e8a3fa749eb2e6f65ef92c2d3cb404cfc8bc0
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "23126926"
---
# <a name="example-2--build-a-dmz-to-protect-applications-with-a-firewall-and-nsgs"></a>範例 2 – 建置 DMZ 以透過防火牆和 NSG 保護應用程式
[返回 [安全性界限最佳作法] 頁面][HOME]

此範例會建立 DMZ，其內含防火牆、四個 Windows 伺服器和網路安全性群組。 此範例也會逐步解說每個相關命令，以讓您更加深入地了解每個步驟。 另外您還會看到＜流量案例＞一節，本節提供深入的逐步說明，讓您知道流量是如何流經 DMZ 內的各個防禦層。 最後則有＜參考＞一節，本節提供完整的程式碼和指示，以供您建置此環境來測試和試驗各種案例。 

![具有 NVA 和 NSG 的輸入 DMZ][1]

## <a name="environment-description"></a>環境描述
此範例中，有一個訂用帳戶包含下列項目：

* 兩個雲端服務：“FrontEnd001” 和 “BackEnd001”
* 一個虛擬網路 “CorpNetwork”，包含兩個子網路：“FrontEnd” 和 “BackEnd”
* 套用至這兩個子網路的單一網路安全性群組
* 一個網路虛擬應用裝置 (在此範例中為 Barracuda NextGen 防火牆)，且已連線到 Frontend 子網路
* 一個代表應用程式 Web 伺服器的 Windows Server (“IIS01”)
* 兩個代表應用程式後端伺服器的 Windows Server (“AppVM01”、“AppVM02”)
* 一個代表 DNS 伺服器的 Windows Server ("DNS01")

> [!NOTE]
> 雖然此範例使用 Barracuda NextGen 防火牆，但還有許多不同的網路虛擬應用裝置能夠用於此範例。
> 
> 

在下面的＜參考＞一節中有提供 PowerShell 指令碼，其可建置上述的大部分環境。 至於 VM 和虛擬網路的建置，雖然也是由此範例指令碼來完成，但本文不會詳加敘述。

若要建置環境：

1. 儲存＜參考＞一節中所包含的網路組態 xml 檔 (更新名稱、位置和 IP 位址以符合給定的案例)
2. 更新指令碼中的使用者變數，以符合要用來執行指令碼的環境 (訂用帳戶、服務名稱等)
3. 在 PowerShell 中執行指令碼

**注意**：PowerShell 指令碼中所指的區域必須符合網路組態 xml 檔中所指的區域。

指令碼順利執行後，即可採取下列指令碼後續步驟：

1. 設定防火牆規則，下面的＜防火牆規則＞一節會有這部分的說明。
2. (選擇性)＜參考＞一節中有兩個指令碼，其會設定 Web 伺服器和具有簡單 Web 應用程式的應用程式伺服器，以便能使用此 DMZ 組態進行測試。

下一節說明大多數與網路安全性群組相關的指令碼陳述式。

## <a name="network-security-groups-nsg"></a>網路安全性群組 (NSG)
此範例會建置 NSG 群組，然後在其中載入六個規則。 

> [!TIP]
> 一般而言，您應該先建立特定的「允許」規則，最後再建立較一般的「拒絕」規則。 所指定的優先順序會決定要先評估哪些規則。 一旦發現流量適用特定規則，就不會再評估後續規則。 NSG 規則可以套用在輸入或輸出方向 (從子網路的觀點出發)。
> 
> 

指令碼會以宣告方式為輸入流量建置下列規則：

1. 允許內部 DNS 流量 (連接埠 53)
2. 允許從網際網路到任何 VM 的 RDP 流量 (連接埠 3389)
3. 允許從網際網路到 NVA (防火牆) 的 HTTP 流量 (連接埠 80)
4. 允許從 IIS01 到 AppVM1 的任何流量 (所有連接埠)
5. 拒絕從網際網路到整個 VNet (兩個子網路) 的任何流量 (所有連接埠)
6. 拒絕從 Frontend 子網路到 Backend 子網路的任何流量 (所有連接埠)

這些規則繫結至每個子網路後，如果有從網際網路到 Web 伺服器的輸入 HTTP 要求，規則 3 (允許) 和規則 5 (拒絕) 皆適用，但由於規則 3 具有較高的優先順序，所以只會適用規則 3，規則 5 則不會派上用場。 因此會允許 HTTP 要求送往防火牆。 如果相同的流量嘗試抵達 DNS01 伺服器，規則 5 (拒絕) 會先適用，因此不會允許流量傳遞給伺服器。 規則 6 (拒絕) 會阻止 Frontend 子網路與 Backend 子網路交談 (規則 1 和 4 允許的流量除外)，這可在攻擊者入侵 Frontend 上的 Web 應用程式時保護 Backend 網路，攻擊者只能對 Backend 的「受保護」網路進行有限度的存取 (只能存取 AppVM01 伺服器上公開的資源)。

有一個預設輸出規則可允許流量外流到網際網路。 在此範例中，我們會允許輸出流量，並不會修改任何輸出規則。 如果兩個方向的流量都要鎖定，則需要使用者定義的路由，這個部分是在不同範例中探討，您可以在[主要安全性界限文件][HOME]中找到相關資訊。

上面討論的 NSG 規則非常類似[範例 1 - 建置具有 NSG 的簡單 DMZ][Example1] 中的 NSG 規則。 請檢閱該文件中的 NSG 說明，以清楚了解每個 NSG 規則和它的屬性。

## <a name="firewall-rules"></a>防火牆規則
電腦上必須安裝管理用戶端，才能管理防火牆和建立所需的組態。 請參閱防火牆 (或其他 NVA) 廠商提供的說明文件，以了解如何管理裝置。 本節剩餘部分將說明如何透過廠商的管理用戶端 (亦即不是使用 Azure 入口網站或 PowerShell) 設定防火牆本身。

適用於下載用戶端和連線到此範例所用 Barracuda 的指示，可以在這裡找到： [Barracuda NG Admin](https://techlib.barracuda.com/NG61/NGAdmin)

防火牆上必須建立轉送規則。 此範例只會將網際網路流量往內路由傳送到防火牆，再傳送到 Web 伺服器，因此只需要一個轉送 NAT 規則。 在此範例所使用的 Barracuda NextGen 防火牆上，傳送此流量的這個規則是目的地 NAT 規則 (“Dst NAT”)。

若要建立下列規則 (或驗證現有的預設規則)，請先從 Barracuda NG Admin 用戶端儀表板瀏覽至 [設定] 索引標籤，在 [作業組態] 區段中按一下 [規則集]。 此時會出現「主要規則」方格，顯示此防火牆現有的作用中規則和已停用規則。 此方格右上角有一個小型的綠色 "+" 按鈕，按一下此按鈕即可建立新規則 (注意：您的防火牆可能會遭「鎖定」不準變更，如果您看到標示為 [鎖定] 的按鈕且無法建立或編輯規則，請按一下此按鈕以「解除鎖定」規則集並允許編輯)。 如果您想要編輯現有規則，請選取該規則，以滑鼠右鍵按一下並選取 [編輯規則]。

建立新規則並提供名稱，例如 "WebTraffic"。 

目的地 NAT 規則圖示看起來像這樣：![目的地 NAT 圖示][2]

規則本身則看起來像這樣：

![防火牆規則][3]

在這邊，任何到達嘗試連線到 HTTP (連接埠 80，若為 HTTPS 則為 443) 的輸入位址，都會被傳送出防火牆的「DHCP1 本機 IP」介面，並重新導向至 IP 位址為 10.0.1.5 的 Web 伺服器。 流量是在連接埠 80 上進入，並在連接埠 80 上前往 Web 伺服器，所以不需要變更連接埠。 不過，如果我們的 Web 伺服器是在連接埠 8080 上接聽，因此將防火牆上的輸入連接埠 80 轉譯為 Web 伺服器上的輸入連接埠 8080，則 [目標清單] 可以是 [10.0.1.5:8080]。

另外，也應該針對「來自網際網路的目的地規則」表明連線方法，其中「動態 SNAT」是最合適的。 

雖然只建立了一個規則，但請務必正確設定其優先順序。 如果防火牆上所有規則的方格中，此新規則位於底部 (在「BLOCKALL」規則之下)，它將永遠不會派上用場。 請確定針對 Web 流量新建立的規則在 BLOCKALL 規則之上。

規則一經建立，就必須推送至防火牆並啟用，如果沒有這麼做，規則的變更就不會生效。 下一節會說明推送和啟用程序。

## <a name="rule-activation"></a>啟用規則
修改規則集以新增此規則後，必須將規則集上傳至防火牆並加以啟用。

![防火牆規則啟用][4]

管理用戶端右上角是按鈕叢集。 按一下 [傳送變更] 按鈕將修改過的規則傳送到防火牆，然後按一下 [啟用] 按鈕。

在啟用防火牆規則集後，這個範例環境的建置便已完成。 (選擇性) 您可以執行＜參考＞一節中的後續建置指令碼，將應用程式新增至此環境來測試下方的流量案例。

> [!IMPORTANT]
> 請務必了解您不會直接到達 Web 伺服器。 當瀏覽器從 FrontEnd001.CloudApp.Net 要求 HTTP 頁面時，HTTP 端點 (連接埠 80) 會將此流量傳遞至防火牆而非 Web 伺服器。 然後，防火牆會根據上面建立的規則，將該要求 NAT 處理到 Web 伺服器。
> 
> 

## <a name="traffic-scenarios"></a>流量案例
#### <a name="allowed-web-to-web-server-through-firewall"></a>(允許) 透過防火牆從 Web 到 Web 伺服器
1. 網際網路使用者從 FrontEnd001.CloudApp.Net (網際網路面向雲端服務) 要求 HTTP 頁面
2. 雲端服務透過連接埠 80 上的開放端點將流量傳遞至 10.0.1.4:80 上的防火牆本機介面
3. Frontend 子網路開始處理輸入規則：
   1. NSG 規則 1 (DNS) 不適用，移至下一個規則
   2. NSG 規則 2 (RDP) 不適用，移至下一個規則
   3. NSG 規則 3 (網際網路到防火牆) 適用，允許流量，停止處理規則
4. 流量抵達防火牆的內部 IP 位址 (10.0.1.4)
5. 防火牆轉送規則會認為此流量是連接埠 80 流量，將它重新導向至 Web 伺服器 IIS01
6. IIS01 正在接聽 Web 流量，接收此要求並開始處理要求
7. IIS01 向 AppVM01 上的 SQL Server 要求資訊
8. Frontend 子網路上沒有輸出規則，允許流量
9. Backend 子網路開始處理輸入規則：
   1. NSG 規則 1 (DNS) 不適用，移至下一個規則
   2. NSG 規則 2 (RDP) 不適用，移至下一個規則
   3. NSG 規則 3 (網際網路到防火牆) 不適用，移至下一個規則
   4. NSG 規則 4 (IIS01 到 AppVM01) 適用，允許流量，停止處理規則
10. AppVM01 接收 SQL 查詢並回應
11. Backend 子網路上沒有輸出規則，所以允許回應
12. Frontend 子網路開始處理輸入規則：
    1. Backend 子網路到 Frontend 子網路的輸入流量沒有適用的 NSG 規則，因此不會套用任何 NSG 規則
    2. 允許子網路間流量的預設系統規則會允許此流量，因此允許流量。
13. IIS 伺服器接收 SQL 回應、完成 HTTP 回應並傳送給要求者
14. 這是來自防火牆的 NAT 工作階段，所以回應目的地 (一開始) 是針對防火牆
15. 防火牆接收來自 Web 伺服器的回應，並往回轉送給網際網路使用者
16. Frontend 子網路上沒有輸出規則，所以允許回應，網際網路使用者會收到要求的網頁。

#### <a name="allowed-rdp-to-backend"></a>(允許) RDP 到 Backend
1. 網際網路上的伺服器管理員在 BackEnd001.CloudApp.Net:xxxxx 上要求 AppVM01 的 RDP 工作階段，其中 xxxxx 是 RDP 到 AppVM01 的隨機指派連接埠號碼 (在 Azure 入口網站上或透過 PowerShell，即可找到指派的連接埠)
2. 防火牆只在 FrontEnd001.CloudApp.Net 位址上接聽，因此不會與此流量有關聯
3. Backend 子網路開始處理輸入規則：
   1. NSG 規則 1 (DNS) 不適用，移至下一個規則
   2. NSG 規則 2 (RDP) 適用，允許流量，停止處理規則
4. 由於沒有輸出規則，會套用預設規則並允許傳回的流量
5. 已啟用 RDP 工作階段
6. AppVM01 會提示輸入使用者名稱和密碼

#### <a name="allowed-web-server-dns-lookup-on-dns-server"></a>(允許) DNS 伺服器上的 Web 伺服器 DNS 查閱
1. Web 伺服器 IIS01 需要 www.data.gov 的資料摘要，但需要解析位址。
2. VNet 的網路組態將 DNS01 (Backend 子網路上的 10.0.2.4) 列為主要 DNS 伺服器，IIS01 將 DNS 要求傳送至 DNS01
3. Frontend 子網路上沒有輸出規則，允許流量
4. Backend 子網路開始處理輸入規則：
   1. NSG 規則 1 (DNS) 適用，允許流量，停止處理規則
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
   3. NSG 規則 3 (網際網路到防火牆) 不適用，移至下一個規則
   4. NSG 規則 4 (IIS01 到 AppVM01) 適用，允許流量，停止處理規則
4. AppVM01 接收要求並以檔案回應 (假設已獲得存取授權)
5. Backend 子網路上沒有輸出規則，所以允許回應
6. Frontend 子網路開始處理輸入規則：
   1. Backend 子網路到 Frontend 子網路的輸入流量沒有適用的 NSG 規則，因此不會套用任何 NSG 規則
   2. 允許子網路間流量的預設系統規則會允許此流量，因此允許流量。
7. IIS 伺服器接收檔案

#### <a name="denied-web-direct-to-web-server"></a>(拒絕) Web 直接到 Web 伺服器
Web 伺服器、IIS01 和防火牆都在相同的雲端服務中，因此共用相同的公開 IP 位址。 因此，HTTP 流量皆會導向至防火牆。 雖然可成功服務要求，但要求不能直接前往 Web 伺服器，依設計它會先傳遞通過防火牆。 請參閱本節第一個案例中的流量。

#### <a name="denied-web-to-backend-server"></a>(拒絕) Web 到 Backend 伺服器
1. 網際網路使用者嘗試透過 BackEnd001.CloudApp.Net 服務存取 AppVM01 上的檔案
2. 因為沒有用於檔案共用的開放端點，此流量不會通過雲端服務到達伺服器
3. 如果基於某些原因而開放端點，NSG 規則 5 (網際網路到 VNet) 會封鎖此流量

#### <a name="denied-web-dns-lookup-on-dns-server"></a>(拒絕) DNS 伺服器上的 Web DNS 查閱
1. 網際網路使用者嘗試透過 BackEnd001.CloudApp.Net 服務查閱 DNS01 上的內部 DNS 記錄
2. 因為沒有用於 DNS 的開放端點，此流量不會通過雲端服務到達伺服器
3. 如果基於某些原因而開放端點，NSG 規則 5 (網際網路到 VNet) 會封鎖此流量 (注意：有兩個原因導致規則 1 (DNS) 不適用，首先，來源位址是網際網路，此規則只適用於以本機 VNet 做為來源，再者，這是允許規則，所以它永遠不會拒絕流量)

#### <a name="denied-web-to-sql-access-through-firewall"></a>(拒絕) Web 透過防火牆對 SQL 進行存取
1. 網際網路使用者從 FrontEnd001.CloudApp.Net (網際網路面向雲端服務) 要求 SQL 資料
2. 因為沒有用於 SQL 的開放端點，此流量不會通過雲端服務到達防火牆
3. 如果基於某些原因而開放端點，Frontend 子網路會開始處理輸入規則：
   1. NSG 規則 1 (DNS) 不適用，移至下一個規則
   2. NSG 規則 2 (RDP) 不適用，移至下一個規則
   3. NSG 規則 3 (網際網路到防火牆) 適用，允許流量，停止處理規則
4. 流量抵達防火牆的內部 IP 位址 (10.0.1.4)
5. 防火牆沒有 SQL 的轉送規則，因此捨棄流量

## <a name="conclusion"></a>結論
這種使用防火牆保護應用程式並隔離後端子網路與輸入流量的方式相當直接簡單。

您可以在[這裡][HOME]找到更多範例和網路安全性界限的概觀。

## <a name="references"></a>參考
### <a name="main-script-and-network-config"></a>主要的指令碼和網路組態
將完整指令碼儲存在 PowerShell 指令碼檔案中。 將網路組態儲存到名為 “NetworkConf2.xml” 的檔案。
視需要修改使用者定義的變數。 執行指令碼，然後依照上面的防火牆規則設定指示進行。

#### <a name="full-script"></a>完整指令碼
根據使用者定義的變數，此指令碼會執行下列動作：

1. 連線到 Azure 訂用帳戶
2. 建立新的儲存體帳戶
3. 依網路組態檔中的定義建立新的 VNet 和兩個子網路
4. 建置 4 個 Windows Server VM
5. 設定 NSG，包括：
   * 建立 NSG
   * 在其中填入規則
   * 將 NSG 繫結至適當的子網路

此 PowerShell 指令碼應該在連線到網際網路的電腦或伺服器上本機執行。

> [!IMPORTANT]
> 此指令碼在執行時，PowerShell 中可能會跳出警告或其他參考訊息。 只有紅色字體的錯誤訊息才需要擔心。
> 
> 

    <# 
     .SYNOPSIS
      Example of DMZ and Network Security Groups in an isolated network (Azure only, no hybrid connections)

     .DESCRIPTION
      This script will build out a sample DMZ setup containing:
       - A default storage account for VM disks
       - Two new cloud services
       - Two Subnets (FrontEnd and BackEnd subnets)
       - A Network Virtual Appliance (NVA), in this case a Barracuda NextGen Firewall
       - One server on the FrontEnd Subnet (plus the NVA on the FrontEnd subnet)
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
       myFirewall - 10.0.1.4
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

    # User Defined Global Variables
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
        $NetworkConfigFile = "C:\Scripts\NetworkConf2.xml"

      # VM Base Disk Image Details
        $SrvImg = Get-AzureVMImage | Where {$_.ImageFamily -match 'Windows Server 2012 R2 Datacenter'} | sort PublishedDate -Descending | Select ImageName -First 1 | ForEach {$_.ImageName}
        $FWImg = Get-AzureVMImage | Where {$_.ImageFamily -match 'Barracuda NextGen Firewall'} | sort PublishedDate -Descending | Select ImageName -First 1 | ForEach {$_.ImageName}

      # NSG Details
        $NSGName = "MyVNetSG"

    # User Defined VM Specific Config
        # Note: To ensure proper NSG Rule creation later in this script:
        #       - The Web Server must be VM 1
        #       - The AppVM1 Server must be VM 2
        #       - The DNS server must be VM 4
        #
        #       Otherwise the NSG rules in the last section of this
        #       script will need to be changed to match the modified
        #       VM array numbers ($i) so the NSG Rule IP addresses
        #       are aligned to the associated VM IP addresses.

        # VM 0 - The Network Virtual Appliance (NVA)
          $VMName += "myFirewall"
          $ServiceName += $FrontEndService
          $VMFamily += "Firewall"
          $img += $FWImg
          $size += "Small"
          $SubnetName += $FESubnet
          $VMIP += "10.0.1.4"

        # VM 1 - The Web Server
          $VMName += "IIS01"
          $ServiceName += $FrontEndService
          $VMFamily += "Windows"
          $img += $SrvImg
          $size += "Standard_D3"
          $SubnetName += $FESubnet
          $VMIP += "10.0.1.5"

        # VM 2 - The First Appliaction Server
          $VMName += "AppVM01"
          $ServiceName += $BackEndService
          $VMFamily += "Windows"
          $img += $SrvImg
          $size += "Standard_D3"
          $SubnetName += $BESubnet
          $VMIP += "10.0.2.5"

        # VM 3 - The Second Appliaction Server
          $VMName += "AppVM02"
          $ServiceName += $BackEndService
          $VMFamily += "Windows"
          $img += $SrvImg
          $size += "Standard_D3"
          $SubnetName += $BESubnet
          $VMIP += "10.0.2.6"

        # VM 4 - The DNS Server
          $VMName += "DNS01"
          $ServiceName += $BackEndService
          $VMFamily += "Windows"
          $img += $SrvImg
          $size += "Standard_D3"
          $SubnetName += $BESubnet
          $VMIP += "10.0.2.4"

    # ----------------------------- #
    # No User Defined Varibles or   #
    # Configuration past this point #
    # ----------------------------- #

      # Get your Azure accounts
        Add-AzureAccount
        Set-AzureSubscription –SubscriptionId $subID -ErrorAction Stop
        Select-AzureSubscription -SubscriptionId $subID -Current -ErrorAction Stop

      # Create Storage Account
        If (Test-AzureName -Storage -Name $StorageAccountName) { 
            Write-Host "Fatal Error: This storage account name is already in use, please pick a diffrent name." -ForegroundColor Red
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
    Else { Write-Host "The network config file was found" -ForegroundColor Green
            If (-Not (Select-String -Pattern $DeploymentLocation -Path $NetworkConfigFile)) {
                Write-Host 'The deployment location was not found in the network config file, please check the network config file to ensure the $DeploymentLocation varible is correct and the netowrk config file matches.' -ForegroundColor Yellow
                $FatalError = $true}
            Else { Write-Host "The deployment location was found in the network config file." -ForegroundColor Green}}

    If ($FatalError) {
        Write-Host "A fatal error has occured, please see the above messages for more information." -ForegroundColor Red
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
            If ($VMFamily[$i] -eq "Firewall") 
                { 
                New-AzureVMConfig -Name $VMName[$i] -ImageName $img[$i] –InstanceSize $size[$i] | `
                    Add-AzureProvisioningConfig -Linux -LinuxUser $LocalAdmin -Password $LocalAdminPwd  | `
                    Set-AzureSubnet  –SubnetNames $SubnetName[$i] | `
                    Set-AzureStaticVNetIP -IPAddress $VMIP[$i] | `
                    New-AzureVM –ServiceName $ServiceName[$i] -VNetName $VNetName -Location $DeploymentLocation
                # Set up all the EndPoints we'll need once we're up and running
                # Note: Web traffic goes through the firewall, so we'll need to set up a HTTP endpoint.
                #       Also, the firewall will be redirecting web traffic to a new IP and Port in a
                #       forwarding rule, so the HTTP endpoint here will have the same public and local
                #       port and the firewall will do the NATing and redirection as declared in the
                #       firewall rule.
                Add-AzureEndpoint -Name "MgmtPort1" -Protocol tcp -PublicPort 801  -LocalPort 801  -VM (Get-AzureVM -ServiceName $ServiceName[$i] -Name $VMName[$i]) | Update-AzureVM
                Add-AzureEndpoint -Name "MgmtPort2" -Protocol tcp -PublicPort 807  -LocalPort 807  -VM (Get-AzureVM -ServiceName $ServiceName[$i] -Name $VMName[$i]) | Update-AzureVM
                Add-AzureEndpoint -Name "HTTP"      -Protocol tcp -PublicPort 80   -LocalPort 80   -VM (Get-AzureVM -ServiceName $ServiceName[$i] -Name $VMName[$i]) | Update-AzureVM
                # Note: A SSH endpoint is automatically created on port 22 when the appliance is created.
                }
            Else
                {
                New-AzureVMConfig -Name $VMName[$i] -ImageName $img[$i] –InstanceSize $size[$i] | `
                    Add-AzureProvisioningConfig -Windows -AdminUsername $LocalAdmin -Password $LocalAdminPwd  | `
                    Set-AzureSubnet  –SubnetNames $SubnetName[$i] | `
                    Set-AzureStaticVNetIP -IPAddress $VMIP[$i] | `
                    Set-AzureVMMicrosoftAntimalwareExtension -AntimalwareConfiguration '{"AntimalwareEnabled" : true}' | `
                    Remove-AzureEndpoint -Name "PowerShell" | `
                    New-AzureVM –ServiceName $ServiceName[$i] -VNetName $VNetName -Location $DeploymentLocation
                    # Note: A Remote Desktop endpoint is automatically created when each VM is created.
                }
            $i++
        }

    # Configure NSG
        Write-Host "Configuring the Network Security Group (NSG)" -ForegroundColor Cyan

      # Build the NSG
        Write-Host "Building the NSG" -ForegroundColor Cyan
        New-AzureNetworkSecurityGroup -Name $NSGName -Location $DeploymentLocation -Label "Security group for $VNetName subnets in $DeploymentLocation"

      # Add NSG Rules
        Write-Host "Writing rules into the NSG" -ForegroundColor Cyan
        Get-AzureNetworkSecurityGroup -Name $NSGName | Set-AzureNetworkSecurityRule -Name "Enable Internal DNS" -Type Inbound -Priority 100 -Action Allow `
            -SourceAddressPrefix VIRTUAL_NETWORK -SourcePortRange '*' `
            -DestinationAddressPrefix $VMIP[4] -DestinationPortRange '53' `
            -Protocol *

        Get-AzureNetworkSecurityGroup -Name $NSGName | Set-AzureNetworkSecurityRule -Name "Enable RDP to $VNetName VNet" -Type Inbound -Priority 110 -Action Allow `
            -SourceAddressPrefix INTERNET -SourcePortRange '*' `
            -DestinationAddressPrefix VIRTUAL_NETWORK -DestinationPortRange '3389' `
            -Protocol *

        Get-AzureNetworkSecurityGroup -Name $NSGName | Set-AzureNetworkSecurityRule -Name "Enable Internet to $($VMName[0])" -Type Inbound -Priority 120 -Action Allow `
            -SourceAddressPrefix Internet -SourcePortRange '*' `
            -DestinationAddressPrefix $VMIP[0] -DestinationPortRange '*' `
            -Protocol *

        Get-AzureNetworkSecurityGroup -Name $NSGName | Set-AzureNetworkSecurityRule -Name "Enable $($VMName[1]) to $($VMName[2])" -Type Inbound -Priority 130 -Action Allow `
            -SourceAddressPrefix $VMIP[1] -SourcePortRange '*' `
            -DestinationAddressPrefix $VMIP[2] -DestinationPortRange '*' `
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
      # Configure Firewall
      # Install Test Web App (Run Post-Build Script on the IIS Server)
      # Install Backend resource (Run Post-Build Script on the AppVM01)
      Write-Host
      Write-Host "Build Complete!" -ForegroundColor Green
      Write-Host
      Write-Host "Optional Post-script Manual Configuration Steps" -ForegroundColor Gray
      Write-Host " - Configure Firewall" -ForegroundColor Gray
      Write-Host " - Install Test Web App (Run Post-Build Script on the IIS Server)" -ForegroundColor Gray
      Write-Host " - Install Backend resource (Run Post-Build Script on the AppVM01)" -ForegroundColor Gray
      Write-Host


#### <a name="network-config-file"></a>網路組態檔
以更新的位置儲存此 xml 檔案，並將此檔案的連結加入到上述指令碼中的 $NetworkConfigFile 變數。

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

#### <a name="sample-application-scripts"></a>範例應用程式指令碼
如果您想要為此範例和其他 DMZ 範例安裝範例應用程式，下列連結中有提供一個：[範例應用程式指令碼][SampleApp]

<!--Image References-->
[1]: ./media/virtual-networks-dmz-nsg-fw-asm/example2design.png "具有 NSG 的輸入 DMZ"
[2]: ./media/virtual-networks-dmz-nsg-fw-asm/dstnaticon.png "目的地 NAT 圖示"
[3]: ./media/virtual-networks-dmz-nsg-fw-asm/firewallrule.png "防火牆規則"
[4]: ./media/virtual-networks-dmz-nsg-fw-asm/firewallruleactivate.png "防火牆規則啟用"

<!--Link References-->
[HOME]: ../best-practices-network-security.md
[SampleApp]: ./virtual-networks-sample-app.md
[Example1]: ./virtual-networks-dmz-nsg-asm.md
