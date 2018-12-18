---
title: DMZ 範例 – 建置 DMZ 以透過防火牆、UDR 和 NSG 保護網路 | Microsoft Docs
description: 建置具有防火牆、使用者定義的路由 (UDR) 和網路安全性群組 (NSG) 的 DMZ
services: virtual-network
documentationcenter: na
author: tracsman
manager: rossort
editor: ''
ms.assetid: dc01ccfb-27b0-4887-8f0b-2792f770ffff
ms.service: virtual-network
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/01/2016
ms.author: jonor;sivae
ms.openlocfilehash: fdb3c5cbd3acee90386352c6f180a71aa81f54fe
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "23127156"
---
# <a name="example-3--build-a-dmz-to-protect-networks-with-a-firewall-udr-and-nsg"></a>範例 3 – 建置 DMZ 以透過防火牆、UDR 和 NSG 保護網路
[返回 [安全性界限最佳作法] 頁面][HOME]

此範例會建立 DMZ，其內含防火牆、四個 Windows 伺服器、使用者定義的路由、IP 轉送和網路安全性群組。 此範例也會逐步解說每個相關命令，以讓您更加深入地了解每個步驟。 另外您還會看到＜流量案例＞一節，本節提供深入的逐步說明，讓您知道流量是如何流經 DMZ 內的各個防禦層。 最後則有＜參考＞一節，本節提供完整的程式碼和指示，以供您建置此環境來測試和試驗各種案例。 

![具有 NVA、NSG 和 UDR 的雙向 DMZ][1]

## <a name="environment-setup"></a>環境設定
此範例中，有一個訂用帳戶包含下列項目：

* 三個雲端服務：“SecSvc001”、“FrontEnd001” 和 “BackEnd001”
* 一個虛擬網路 "CorpNetwork"，包含三個子網路：“SecNet”、“FrontEnd” 和 “BackEnd”
* 一個網路虛擬應用裝置，在此範例中為防火牆，並連接到 SecNet 子網路
* 一個代表應用程式 Web 伺服器的 Windows Server (“IIS01”)
* 兩個代表應用程式後端伺服器的 Windows Server (“AppVM01”、“AppVM02”)
* 一個代表 DNS 伺服器的 Windows Server ("DNS01")

在下面的＜參考＞一節中有提供 PowerShell 指令碼，其可建置上述的大部分環境。 至於 VM 和虛擬網路的建置，雖然也是由此範例指令碼來完成，但本文不會詳加敘述。

若要建置環境：

1. 儲存＜參考＞一節中所包含的網路組態 xml 檔 (更新名稱、位置和 IP 位址以符合給定的案例)
2. 更新指令碼中的使用者變數，以符合要用來執行指令碼的環境 (訂用帳戶、服務名稱等)
3. 在 PowerShell 中執行指令碼

**注意**：PowerShell 指令碼中所指的區域必須符合網路組態 xml 檔中所指的區域。

指令碼順利執行後，即可採取下列指令碼後續步驟：

1. 設定防火牆規則，下面的 [防火牆規則說明] 一節會有這部分的說明。
2. (選擇性)＜參考＞一節中有兩個指令碼，其會設定 Web 伺服器和具有簡單 Web 應用程式的應用程式伺服器，以便能使用此 DMZ 組態進行測試。

一旦指令碼順利執行，就必須完成防火牆規則，[防火牆規則] 一節中會有這部分的說明。

## <a name="user-defined-routing-udr"></a>使用者定義的路由 (UDR)
根據預設，下列系統路由的定義如下：

        Effective routes : 
         Address Prefix    Next hop type    Next hop IP address Status   Source     
         --------------    -------------    ------------------- ------   ------     
         {10.0.0.0/16}     VNETLocal                            Active   Default    
         {0.0.0.0/0}       Internet                             Active   Default    
         {10.0.0.0/8}      Null                                 Active   Default    
         {100.64.0.0/10}   Null                                 Active   Default    
         {172.16.0.0/12}   Null                                 Active   Default    
         {192.168.0.0/16}  Null                                 Active   Default

VNETLocal 一律是該特定網路之 VNet 的定義位址前置詞 (也就是說，根據每個特定 VNet 的定義方式，不同 VNet 會有不同前置詞)。 其餘系統路由則如上所述，屬於靜態路由並採用預設值。

至於這些路由的優先順序，由於路由是透過最長前置詞比對 (LPM) 方法來進行處理，因此路由表中最特定的路由會適用於給定的目的地位址。

因此，用於區域網路 (10.0.0.0/16) 的流量 (例如流往 DNS01 伺服器 10.0.2.4)，會由於 10.0.0.0/16 路由而穿過 VNet 路由傳送到目的地。 換句話說，針對 10.0.2.4，10.0.0.0/16 路由是最特定的路由，即使 10.0.0.0/8 以及 0.0.0.0/0 也可適用，但是因為它們較不特定，所以不會影響此流量。 因此，流往 10.0.2.4 的流量會有本機 VNet 的下個躍點，因而會直接路由傳送至目的地。

舉例來說，如果是用於 10.1.1.1 的流量，10.0.0.0/16 路由就不適用，但 10.0.0.0/8 則會是最特定的，因此會捨棄此流量 (「黑洞化」)，因為下個躍點是 Null。 

如果目的地不適用於任何 Null 前置詞或 VNETLocal 前置詞，則會遵循最不特定路由 0.0.0.0/0 並路由傳送出網際網路做為下個躍點，因此會傳送到 Azure 的網際網路邊緣之外。

如果路由表中有兩個相同的前置詞，則根據路由的 “source” 屬性，其喜好順序如下：

1. "VirtualAppliance" = 手動新增到表格中的「使用者定義路由」
2. “VPNGateway” = 由動態網路通訊協定新增的動態路由 (與混合式網路搭配使用時為 BGP)，這些路由會隨時間改變，因為動態通訊協定會自動反映對等網路中的變更
3. “Default” = 上述路由表所示的系統路由、本機 VNet 和靜態項目。

> [!NOTE]
> 您現在可以將使用者定義路由 (UDR) 與 ExpressRoute 和 VPN 閘道搭配使用，以強制將輸入和輸出的跨單位流量路由傳送至網路虛擬設備 (NVA)。
> 
> 

#### <a name="creating-the-local-routes"></a>建立本機路由
此範例需要兩個路由表，以分別用於 Frontend 和 Backend 子網路。 每個資料表都會載入適用於給定子網路的靜態路由。 為進行此範例，每個資料表內有三個路由：

1. 未定義下一個躍點的本機子網路流量，以允許本機子網路流量略過防火牆
2. 下一個躍點定義為防火牆的虛擬網路流量，這會覆寫允許本機 VNet 流量直接路由的預設規則
3. 下一個躍點定義為防火牆的所有其餘流量 (0/0)

路由表一經建立就會繫結至其子網路。 Frontend 子網路路由表在建立並繫結至子網路後，應該會像下面這樣：

        Effective routes : 
         Address Prefix    Next hop type    Next hop IP address Status   Source     
         --------------    -------------    ------------------- ------   ------     
         {10.0.1.0/24}     VNETLocal                            Active 
         {10.0.0.0/16}     VirtualAppliance 10.0.0.4            Active    
         {0.0.0.0/0}       VirtualAppliance 10.0.0.4            Active


針對此範例，會使用下列命令來建置路由表、新增使用者定義的路由，然後將路由表繫結至子網路 (注意：底下以貨幣符號開頭的項目 (例如 $BESubnet) 皆為來自本文＜參考＞一節之指令碼的使用者定義變數)：

1. 首先必須建立基礎路由表。 此程式碼片段顯示如何建立 Backend 子網路的路由表。 指令碼中也會針對 Frontend 子網路建立對應的路由表。
   
     New-AzureRouteTable -Name $BERouteTableName `
   
         -Location $DeploymentLocation `
         -Label "Route table for $BESubnet subnet"
2. 一旦建立路由表，就可以新增特定的使用者定義路由。 在此程式碼片段中，會透過虛擬應用裝置路由傳送所有流量 (0.0.0.0/0) (指令碼稍早建立虛擬應用裝置時，是使用變數 $VMIP[0] 來傳入指派的 IP 位址)。 指令碼也會在 Frontend 路由表中建立對應的規則。
   
     Get-AzureRouteTable $BERouteTableName | `
   
         Set-AzureRoute -RouteName "All traffic to FW" -AddressPrefix 0.0.0.0/0 `
         -NextHopType VirtualAppliance `
         -NextHopIpAddress $VMIP[0]
3. 上述路由項目會覆寫預設的 "0.0.0.0/0" 路由，但預設的 10.0.0.0/16 規則仍會存在，以允許 VNet 內的流量直接路由傳送到目的地，而非傳送到網路虛擬應用裝置。 若要修正此行為，您必須新增下列規則。
   
        Get-AzureRouteTable $BERouteTableName | `
            Set-AzureRoute -RouteName "Internal traffic to FW" -AddressPrefix $VNetPrefix `
            -NextHopType VirtualAppliance `
            -NextHopIpAddress $VMIP[0]
4. 此時要做一項選擇。 在上述兩個路由中，所有流量都會路由傳送至防火牆進行評估，甚至單一子網路內的流量也是如此。 這可能是您想要的結果，但若要允許子網路內的流量直接在本機路由傳送，而不要防火牆介入，則可以新增第三個特定規則。 此路由會指出，目的地為本機子網路的位址可以直接路由傳送至該處 (NextHopType = VNETLocal)。
   
        Get-AzureRouteTable $BERouteTableName | `
            Set-AzureRoute -RouteName "Allow Intra-Subnet Traffic" -AddressPrefix $BEPrefix `
            -NextHopType VNETLocal
5. 最後，在建立好路由表並填入使用者定義的路由後，路由表必須立即繫結至子網路。 在此指令碼中，Frontend 路由表也會繫結到 Frontend 子網路。 以下是 Backend 子網路的繫結指令碼。
   
     Set-AzureSubnetRouteTable -VirtualNetworkName $VNetName `
   
        -SubnetName $BESubnet `
        -RouteTableName $BERouteTableName

## <a name="ip-forwarding"></a>IP 轉送
UDR 隨附 IP 轉送功能。 這是虛擬應用裝置的一項設定，以允許它接收不是要特別傳送到應用裝置的流量，再將流量轉送到其最終目的地。

舉例來說，如果來自 AppVM01 的流量對 DNS01 伺服器提出要求，UDR 會將此流量路由傳送到防火牆。 在啟用 IP 轉送時，目的地為 DNS01 (10.0.2.4) 的流量會被應用裝置 (10.0.0.4) 所接受，然後轉送到其最終目的地 (10.0.2.4) 。 若防火牆未啟用 IP 轉送，即使路由表將防火牆做為下一個躍點，流量也不會被應用裝置所接受。 

> [!IMPORTANT]
> 請務必記得一同啟用 IP 轉送和使用者定義的路由。
> 
> 

設定 IP 轉送是單一命令，可在建立 VM 時執行。 在此範例的流程中，這個程式碼片段靠近指令碼結尾處，與 UDR 命令放在一起：

1. 呼叫代表您的虛擬應用裝置的 VM 執行個體 (在此案例中是防火牆)，並啟用 IP 轉送 (注意：以貨幣符號開頭的紅色項目 (例如 $VMName[0]) 皆為來自本文＜參考＞一節之指令碼的使用者定義變數。 以方括弧括住的零 ([0]) 代表 VM 陣列中的第一個 VM，為了讓範例指令碼無須修改即可運作，第一個 VM (VM 0) 必須是防火牆)：
   
     Get-AzureVM -Name $VMName[0] -ServiceName $ServiceName[0] | `
   
        Set-AzureIPForwarding -Enable

## <a name="network-security-groups-nsg"></a>網路安全性群組 (NSG)
此範例會建置 NSG 群組，然後在其中載入單一規則。 此群組接著只會繫結到 Frontend 和 Backend 子網路 (不會繫結到 SecNet)。 指令碼會以宣告方式建置下列規則：

1. 拒絕從網際網路到整個 VNet (所有子網路) 的任何流量 (所有連接埠)

雖然此範例使用 NSG，但它的主要目的是做為防止人為設定錯誤的第二道防線。 我們想要封鎖由網際網路流往 Frontend 或 Backend 子網路的所有輸入流量，流量只應流經 SecNet 子網路前往防火牆 (並於合適時流往 Frontend 或 Backend 子網路)。 此外，在具有 UDR 規則時，確實能夠流往 Frontend 或 Backend 子網路的流量皆會被引導出來並流向防火牆 (多虧有了 UDR)。 防火牆會將此流量視為非對稱流量，並且會捨棄輸出流量。 因此，Frontend 和 Backend 子網路共有三層安全防護：1) FrontEnd001 和 BackEnd001 雲端服務上沒有開放的端點，2) NSG 會拒絕來自網際網路的流量，3) 防火牆會捨棄非對稱流量。

此範例的網路安全性群組有一個有趣的地方，那就是它只有一個規則 (如下所示)，此規則是為了要拒絕由網際網路流往整個虛擬網路 (會包含安全性子網路) 的流量。 

    Get-AzureNetworkSecurityGroup -Name $NSGName | `
        Set-AzureNetworkSecurityRule -Name "Isolate the $VNetName VNet `
        from the Internet" `
        -Type Inbound -Priority 100 -Action Deny `
        -SourceAddressPrefix INTERNET -SourcePortRange '*' `
        -DestinationAddressPrefix VIRTUAL_NETWORK `
        -DestinationPortRange '*' `
        -Protocol *

不過，由於 NSG 只會繫結至 Frontend 和 Backend 子網路，因此不會對流往安全性子網路的輸入流量處理此規則。 如此一來，即使 NSG 規則指出因為 NSG 永遠不會繫結到安全性子網路，所以沒有由網際網路流往 VNet 上任何位址的流量，流量還是會流向安全性子網路。

    Set-AzureNetworkSecurityGroupToSubnet -Name $NSGName `
        -SubnetName $FESubnet -VirtualNetworkName $VNetName

    Set-AzureNetworkSecurityGroupToSubnet -Name $NSGName `
        -SubnetName $BESubnet -VirtualNetworkName $VNetName

## <a name="firewall-rules"></a>防火牆規則
防火牆上必須建立轉送規則。 因為防火牆會封鎖或轉送所有輸入、輸出和內部 VNet 流量，所以需要許多防火牆規則。 此外，所有輸入流量都會抵達安全性服務的公用 IP 位址 (在不同連接埠上)，並由防火牆進行處理。 最佳作法是先繪製邏輯流量圖，再設定子網路和防火牆規則，以避免事後還要修改。 下圖是此範例中的防火牆規則的邏輯視圖：

![防火牆規則的邏輯視圖][2]

> [!NOTE]
> 根據所使用的網路虛擬應用裝置，會有不同的管理連接埠。 在此範例中，所參考的 Barracuda NextGen 防火牆使用連接埠 22、801 和 807。 請參閱應用裝置廠商的說明文件來尋找用於管理所使用裝置的確切連接埠。
> 
> 

### <a name="logical-rule-description"></a>邏輯規則說明
安全性子網路上只有防火牆這個資源，所以上述邏輯視圖未顯示該子網路，而且這個視圖會顯示防火牆規則以及這些規則在邏輯上是如何允許或拒絕流量的流動，而不會顯示實際的路由路徑。 此外，針對 RDP 流量選取的外部連接埠皆為較高範圍的連接埠 (8014 – 8026)，且在選取時會稍微配合本機 IP 位址的最後兩個八位元數字，以方便閱讀 (例如，本機伺服器位址 10.0.1.4 會與外部連接埠 8014 相關聯)，不過您可以使用任何更高範圍的連接埠，只要連接埠未衝突即可。

在此範例中，我們需要 7 種規則，以下說明這些規則類型：

* 外部規則 (適用於輸入流量)：
  1. 防火牆管理規則：此應用程式重新導向規則允許流量傳遞至網路虛擬應用裝置的管理連接埠。
  2. RDP 規則 (適用於每個 Windows 伺服器)：這四個規則 (每部伺服器一個) 會允許透過 RDP 管理個別伺服器。 根據所使用的網路虛擬應用裝置功能而定，也可將其結合成一個規則。
  3. 應用程式流量規則：應用程式流量規則有兩個，第一個適用於前端 Web 流量，第二個適用於後端流量 (例如 Web 伺服器流往資料層)。 這些規則的設定將取決於網路架構 (伺服器的放置位置) 和流量流動行為 (流量的流動方向，以及使用的連接埠)。
     * 第一個規則會允許實際的應用程式流量抵達應用程式伺服器。 其他規則所考慮的是安全性、管理等方面的事項，而應用程式規則則是用來允許外部使用者或服務存取應用程式。 就此範例來說，連接埠 80 上有單一 Web 伺服器，因此單一防火牆應用程式規則會將流往外部 IP 的輸入流量，重新導向到 Web 伺服器的內部 IP 位址。 重新導向的流量工作階段會經過 NAT 轉譯到內部伺服器。
     * 第二個應用程式流量規則是後端規則，可允許Web 伺服器透過任何連接埠與 AppVM01 伺服器 (而非 AppVM02) 對談。
* 內部規則 (適用於內部 VNet 流量)
  1. 輸出到網際網路規則：此規則會允許來自任何網路的流量傳遞至選定的網路。 此規則通常是防火牆上既有的預設規則，但會處於停用狀態。 在此範例中，應啟用此規則。
  2. DNS 規則：此規則只會允許 DNS (連接埠 53) 流量傳遞至 DNS 伺服器。 在此環境中，會封鎖大部分由 Frontend 流往 Backend 的流量，而此規則特別允許來自任何本機子網路的 DNS。
  3. 子網路對子網路規則：此規則會允許 Backend 子網路上的任何伺服器連接至 Frontend 子網路上的任何伺服器 (但不允許反向連接)。
* 保險規則 (適用於不符合上述任一規則的流量)：
  1. 拒絕所有流量規則：請一律將此規則做為最終規則 (依據優先順序)，這樣一來，如果流量的流動行為無法符合上述任何規則時，便會由此規則將其捨棄。 這是預設規則且通常會啟動，一般來說並不需要修改。

> [!TIP]
> 為簡化此範例，會允許第二個應用程式流量規則上的任何連接埠，但在真實案例中，則應使用最特定的連接埠和位址範圍，以降低此規則的攻擊面。
> 
> 

<br />

> [!IMPORTANT]
> 建立好上述所有規則後，請務必檢閱每個規則的優先順序，以確保可依需要允許或拒絕流量。 在此範例中，規則是依照優先順序排列。 規則順序若錯誤，會很容易造成防火牆遭到鎖定。 因此，最優先的規則便是至少要確實管理防火牆本身。
> 
> 

### <a name="rule-prerequisites"></a>規則的必要條件
執行防火牆之虛擬機器的必要條件之一是公用端點。 為了讓防火牆能夠處理流量，必須開放適當的公用端點。 此範例有三種類型的流量：1) 管理流量，用以控制防火牆和防火牆規則，2) RDP 流量，用以控制 Windows 伺服器，和 3) 應用程式流量。 這些流量便是上述防火牆規則的邏輯視圖上半部中的三排流量類型。

> [!IMPORTANT]
> 這段內容的重點是要記得 **所有** 流量都會通過防火牆傳來。 遠端桌面到 IIS01 伺服器也是如此，即使是在 FrontEnd 雲端服務和 FrontEnd 子網路上，若要存取此伺服器，我們仍需要透過 RDP 連接到連接埠 8014 上的防火牆，然後允許防火牆在內部將 RDP 要求路由至 IIS01 RDP 連接埠。 由於沒有直接通往 IIS01 的 RDP 路徑 (就入口網站上所見)，Azure 入口網站的 [連線] 按鈕將不會有作用。 這表示所有來自網際網路的連線將會連往安全性服務和連接埠，例如 secscv001.cloudapp.net:xxxx (請參考上面的圖表，以對應外部連接埠和內部 IP 和連接埠)。
> 
> 

在建立 VM 或後續建置時皆可開放端點，其方式在範例指令碼和下面這個程式碼片段中都有說明 (注意：以貨幣符號開頭的項目 (例如 $VMName[$i]) 皆為來自本文＜參考＞一節之指令碼的使用者定義變數。 以方括弧括住的 “$i” ([$i]) 代表 VM 陣列中特定 VM 的陣列號碼)：

    Add-AzureEndpoint -Name "HTTP" -Protocol tcp -PublicPort 80 -LocalPort 80 `
        -VM (Get-AzureVM -ServiceName $ServiceName[$i] -Name $VMName[$i]) | `
        Update-AzureVM

雖然因為使用變數的關係所以此處未清楚顯示，但實際上 **只** 會開啟安全性雲端服務上的端點。 這是為了確保能由防火牆處理 (路由傳送、進行 NAT 處理、捨棄) 所有輸入流量。

電腦上必須安裝管理用戶端，才能管理防火牆和建立所需的組態。 請參閱防火牆 (或其他 NVA) 廠商提供的說明文件，以了解如何管理裝置。 本節剩餘部分和下一節＜建立防火牆規則＞將說明如何透過廠商的管理用戶端 (亦即不是使用 Azure 入口網站或 PowerShell) 設定防火牆本身。

適用於下載用戶端和連線到此範例所用 Barracuda 的指示，可以在這裡找到： [Barracuda NG Admin](https://techlib.barracuda.com/NG61/NGAdmin)

在登入防火牆後、建立防火牆規則前，有兩個必要的物件類別可讓您更輕鬆地建立規則：分別是網路和服務物件。

在此範例中，應定義三個具名的網路物件 (Frontend 子網路和 Backend 子網路各一個，還有一個網路物件則用於 DNS 伺服器的 IP 位址)。 若要建立具名網路：先從 Barracuda NG Admin 用戶端儀表板瀏覽至 [設定] 索引標籤，在 [作業組態] 區段中按一下 [規則集]，接著按一下 [防火牆物件] 功能表底下的 [網路]，然後在 [編輯網路] 功能表中按一下 [新增]。 現在您可以藉由新增名稱和前置詞來建立網路物件：

![建立 FrontEnd 網路物件][3]

這會建立 FrontEnd 子網路的具名網路，您也應該對 BackEnd 子網路建立類似物件。 現在您可以更輕鬆地在防火牆規則中依名稱參考子網路。

針對 DNS 伺服器物件：

![建立 DNS 伺服器物件][4]

本文稍後將會在 DNS 規則中利用這個單一 IP 位址參考。

第二個必要物件是服務物件。 這些物件會代表每部伺服器的 RDP 連線連接埠。 因為既有 RDP 服務物件繫結至預設 RDP 連接埠 3389，您可以建立新服務以允許來自外部連接埠 (8014-8026) 的流量。 新的連接埠也可以新增至既有 RDP 服務，但為了便於示範，可以為每部伺服器各建立一個規則。 若要為伺服器建立新的 RDP 規則：先從 Barracuda NG Admin 用戶端儀表板瀏覽至 [設定] 索引標籤，在 [作業組態] 區段中按一下 [規則集]，接著按一下 [防火牆物件] 功能表底下的 [服務]，往下捲動服務清單，然後選取 [RDP] 服務。 按一下滑鼠右鍵並選取 [複製]，然後按一下滑鼠右鍵並選取 [貼上]。 現在會有可供編輯的 RDP-Copy1 服務物件。 在 RDP-Copy1 上按一下滑鼠右鍵並選取 [編輯]，便會跳出 [編輯服務物件] 視窗，如下所示：

![預設 RDP 規則的複本][5]

接著可以編輯值來代表特定伺服器的 RDP 服務。 針對 AppVM01，應該修改上述的預設 RDP 規則以反映圖 8 的圖表中所用的新服務名稱、說明和外部 RDP 連接埠 (注意：連接埠會從 RDP 預設值 3389 變更為要用於此特定伺服器的外部連接埠，在 AppVM01 這個案例中，外部連接埠是 8025)。修改過後的服務如下所示：

![AppVM01 規則][6]

您必須重複此程序來建立其餘伺服器的 RDP 服務：AppVM02、DNS01 和 IIS01。 建立這些服務可讓下一節的規則建立作業變得更簡單明瞭。

> [!NOTE]
> 防火牆不需要 RDP 服務有兩個理由：1) 首先防火牆 VM 是 Linux 映像，因此會在連接埠 22 上使用 SSH 來管理 VM，而不是使用 RDP，2) 下面所述的第一個管理規則允許連接埠 22 和另外兩個管理連接埠，以允許進行管理連線。
> 
> 

### <a name="firewall-rules-creation"></a>建立防火牆規則
此範例使用三種類型的防火牆規則，它們各有不同的圖示：

應用程式重新導向規則：![應用程式重新導向圖示][7]

目的地 NAT 規則：![目的地 NAT 圖示][8]

傳遞規則：![傳遞圖示][9]

Barracuda 網站可以找到這些規則的詳細資訊。

若要建立下列規則 (或驗證現有的預設規則)，請先從 Barracuda NG Admin 用戶端儀表板瀏覽至 [設定] 索引標籤，在 [作業組態] 區段中按一下 [規則集]。 此時會出現「主要規則」方格，顯示此防火牆現有的作用中規則和已停用規則。 此方格右上角有一個小型的綠色 "+" 按鈕，按一下此按鈕即可建立新規則 (注意：您的防火牆可能會遭「鎖定」不準變更，如果您看到標示為 [鎖定] 的按鈕且無法建立或編輯規則，請按一下此按鈕以「解除鎖定」規則集並允許編輯)。 如果您想要編輯現有規則，請選取該規則，以滑鼠右鍵按一下並選取 [編輯規則]。

您的規則一經建立和 (或) 修改，就必須推送至防火牆並啟用，如果沒有這麼做，規則的變更就不會生效。 詳細的規則說明下面會說明此推送和啟用程序。

完成此範例所需的每個規則的細節說明如下：

* **防火牆管理規則**︰此「應用程式重新導向」規則允許將流量傳遞到網路虛擬設備 (在此範例中為「Barracuda NextGen 防火牆」) 的管理連接埠。 管理連接埠是 801 和 807，以及 22 (選擇性)。 外部和內部連接埠相同 (亦即沒有連接埠轉譯)。 此 SETUP-MGMT-ACCESS 規則 (在 Barracuda NextGen 防火牆 6.1 版中) 是預設規則，且為預設啟用。
  
    ![防火牆管理規則][10]

> [!TIP]
> 此規則中的來源位址空間是 [任何]，如果已知管理 IP 位址範圍，減少此範圍也可減少管理連接埠的攻擊面。
> 
> 

* **RDP 規則**：這些目的地 NAT 規則將允許透過 RDP 管理個別伺服器。
  要建立此規則需要四個重要欄位：
  
  1. 來源 – 允許來自任何地方的 RDP，[來源] 欄位中使用參考值 [任何]。
  2. 服務 – 使用先前建立的適當服務物件，在此案例中為 [AppVM01 RDP]，外部連接埠會重新導向至伺服器本機 IP 位址和連接埠 3386 (預設 RDP 連接埠)。 此特定規則用於 AppVM01 的 RDP 存取。
  3. 目的地 – 應該是防火牆上的本機連接埠 [DCHP 1 本機 IP]，如果使用靜態 IP 則為 [eth0]。 如果您的網路應用裝置有多個本機介面，序號 (eth0、eth1 等等) 可能會不同。 這是防火牆用來往外傳送的連接埠 (可能與接收連接埠相同)，實際的路由目的地在 [目標清單] 欄位中。
  4. 重新導向 – 此區段會告知虛擬應用裝置最終要將此流量重新導向至什麼地方。 最簡單的重新導向是在 [目標清單] 欄位中放置 IP 和連接埠 (選擇性)。 如果沒有使用連接埠，則會使用輸入要求上的目的地連接埠 (即未轉譯)，如果指定了連接埠，連接埠會連同 IP 位址一起進行 NAT 處理。
     
     ![防火牆 RDP 規則][11]
     
     共需要建立四個 RDP 規則： 
     
     | 規則名稱 | 伺服器 | 服務 | 目標清單 |
     | --- | --- | --- | --- |
     | RDP-to-IIS01 |IIS01 |IIS01 RDP |10.0.1.4:3389 |
     | RDP-to-DNS01 |DNS01 |DNS01 RDP |10.0.2.4:3389 |
     | RDP-to-AppVM01 |AppVM01 |AppVM01 RDP |10.0.2.5:3389 |
     | RDP-to-AppVM02 |AppVM02 |AppVm02 RDP |10.0.2.6:3389 |

> [!TIP]
> 縮減 [來源] 和 [服務] 欄位的範圍會減少攻擊面。 請使用可讓功能正常運作的最小範圍。
> 
> 

* **應用程式流量規則**：應用程式流量規則有兩個，第一個適用於前端 Web 流量，第二個適用於後端流量 (例如 Web 伺服器流往資料層)。 這些規則將取決於網路架構 (伺服器的放置位置) 和流量流動行為 (流量的流動方向，以及使用的連接埠)。
  
    首先討論的是 Web 流量的前端規則：
  
    ![防火牆 Web 規則][12]
  
    此目的地 NAT 規則允許實際的應用程式流量抵達應用程式伺服器。 其他規則所考慮的是安全性、管理等方面的事項，而應用程式規則則是用來允許外部使用者或服務存取應用程式。 就此範例來說，連接埠 80 上有單一 Web 伺服器，因此單一防火牆應用程式規則會將流往外部 IP 的輸入流量，重新導向到 Web 伺服器的內部 IP 位址。
  
    **注意**：[目標清單] 欄位中未指定連接埠，因此輸入連接埠 80 (若為所選服務則為 443) 將會用於 Web 伺服器的重新導向。 如果 Web 伺服器正在接聽不同連接埠，例如連接埠 8080，[目標清單] 欄位則可更新為 [10.0.1.4:8080] 以同時允許連接埠重新導向。
  
    下一個應用程式流量規則是後端規則，可允許Web 伺服器透過任何服務與 AppVM01 伺服器 (而非 AppVM02) 對談：
  
    ![防火牆 AppVM01 規則][13]
  
    此傳遞規則允許 Frontend 子網路上的任何 IIS 伺服器在任何連接埠上連線到 AppVM01 (IP 位址 10.0.2.5)，使用任何通訊協定來存取 Web 應用程式所需的資料。
  
    在此螢幕擷取畫面中，[目的地] 欄位使用 "\<explicit-dest\>" 來表示以 10.0.2.5 做為目的地。 這可以是如所示的明確位址或具名網路物件 (如 DNS 伺服器的必要條件中所做的)。 要使用哪一種方法全由防火牆的系統管理員來決定。 若要將 10.0.2.5 新增為明確目的地，請在 \<explicit-dest\> 底下的第一個空白資料列按兩下，並在彈出的視窗中輸入位址。
  
    使用此傳遞規則時不需要 NAT，因為這是內部流量，因此 [連線方法] 可以設為 [不使用 SNAT]。
  
    **注意**：此規則的 [來源] 網路是 FrontEnd 子網路上的任何資源，如果只會有一個或已知特定數目的 Web 伺服器，則可以將網路物件資源建立為更針對這些確切 IP 位址，而不是針對整個 FrontEnd 子網路。

> [!TIP]
> 此規則使用 [任何] 服務讓您更容易設定和使用範例應用程式，這也會在單一規則中允許 ICMPv4 (ping)。 不過，這不是建議的作法。 連接埠和通訊協定 (「服務」) 應該盡可能縮小到允許應用程式作業的程度，以減少跨越此界限的攻擊面。
> 
> 

<br />

> [!TIP]
> 雖然此規則顯示正在使用 explicit-dest 參考，但整個防火牆設定應該使用一致的方法。 建議在整個設定中使用具名網路物件，以方便閱讀和支援。 在此使用 explicit-dest 只是為了顯示替代參考方法，一般並不建議這麼做 (尤其是在複雜設定中)。
> 
> 

* **輸出到網際網路規則**：此傳遞規則會允許來自任何 [來源] 網路的流量傳遞至選定的 [目的地] 網路。 此規則是 Barracuda NextGen 防火牆上通常已經有的預設規則，但會處於停用狀態。 以滑鼠右鍵按一下此規則可以存取 [啟用規則] 命令。 此處顯示的規則已經過修改，新增兩個本文＜必要條件＞一節中所建立做為參考的本機子網路到此規則的 [來源] 屬性。
  
    ![防火牆輸出規則][14]
* **DNS 規則**：此傳遞規則只會允許 DNS (連接埠 53) 流量傳遞至 DNS 伺服器。 在此環境中，會封鎖大部分由 Frontend 流往 Backend 的流量，而此規則特別允許 DNS。
  
    ![防火牆 DNS 規則][15]
  
    **注意**：在此螢幕擷取畫面中，包含了 [連線方法]。 此規則用於內部 IP 到內部 IP的位址流量，因此不需要 NATing，此傳遞規則的 [連線方法] 設定為 [不使用 SNAT]。
* **子網路對子網路規則**：此傳遞規則是經過啟用和修改的預設規則，會允許 Backend 子網路上的任何伺服器連接至 Frontend 子網路上的任何伺服器。 此規則全是內部流量，因此可將 [連線方法] 設為 [不使用 SNAT]。
  
    ![防火牆內部 VNet 規則][16]
  
    **注意**：未核取 [雙向] 核取方塊 (也未簽入大部分規則)，這一點對於此規則很重要，因為它可讓此規則只能「單向」進行，也就是可以由 Backend 子網路連接至 Frontend 網路，但不能反向進行。 如果核取該核取方塊，此規則就會啟用雙向流量，從我們的邏輯圖來看並不需要這麼做。
* **拒絕所有流量規則**：請一律將此規則做為最終規則 (依據優先順序)，這樣一來，如果流量的流動行為無法符合上述任何規則時，便會由此規則將其捨棄。 這是預設規則且通常會啟動，一般來說並不需要修改。 
  
    ![防火牆拒絕規則][17]

> [!IMPORTANT]
> 建立好上述所有規則後，請務必檢閱每個規則的優先順序，以確保可依需要允許或拒絕流量。 在此範例中，規則的排列順序是它們應在 Barracuda 管理用戶端之轉送規則的 [主要方格] 中出現的順序。
> 
> 

## <a name="rule-activation"></a>啟用規則
規則集在修改為邏輯圖的規格後，必須上傳至防火牆並加以啟用。

![防火牆規則啟用][18]

管理用戶端右上角是按鈕叢集。 按一下 [傳送變更] 按鈕將修改過的規則傳送到防火牆，然後按一下 [啟用] 按鈕。

在啟用防火牆規則集後，這個範例環境的建置便已完成。

## <a name="traffic-scenarios"></a>流量案例
> [!IMPORTANT]
> 重點是要記得 **所有** 流量都會通過防火牆傳來。 遠端桌面到 IIS01 伺服器也是如此，即使是在 FrontEnd 雲端服務和 FrontEnd 子網路上，若要存取此伺服器，我們仍需要透過 RDP 連接到連接埠 8014 上的防火牆，然後允許防火牆在內部將 RDP 要求路由至 IIS01 RDP 連接埠。 由於沒有直接通往 IIS01 的 RDP 路徑 (就入口網站上所見)，Azure 入口網站的 [連線] 按鈕將不會有作用。 這表示來自網際網路的所有連線都會流往安全性服務和連接埠，例如 secscv001.cloudapp.net:xxxx。
> 
> 

針對這些案例，應備妥下列防火牆規則：

1. 防火牆管理
2. RDP 到 IIS01
3. RDP 到 DNS01
4. RDP 到 AppVM01
5. RDP 到 AppVM02
6. 到 Web 的應用程式流量
7. 到 AppVM01 的應用程式流量
8. 輸出到網際網路
9. Frontend 到 DNS01
10. 內部子網路流量 (僅限 Backend 到 Frontend)
11. 全部拒絕

實際的防火牆規則集很可能會有上述規則以外的許多其他規則，任何給定防火牆上的規則也會有與此處所列編號不同的優先順序編號。 此清單和關聯的編號只為提供這十一個規則之間的相關性，以及它們彼此之間的相對優先順序。 換句話說，在實際的防火牆上，「RDP 到 IIS01」可能是規則編號 5，但只要它低於「防火牆管理」規則並高於「RDP 到 DNS01」規則，它就符合此清單的目的。 此清單也有助於以下案例變得簡單明瞭，例如「FW 規則 9 (DNS)」。 同樣是為了簡潔起見，當流量案例與 RDP 無關時，四個 RDP 規則將會統稱為「RDP 規則」。

也請記得已有網路安全性群組可用於 Frontend 和 Backend 子網路上的輸入網際網路流量。

#### <a name="allowed-internet-to-web-server"></a>(允許) 網際網路到 Web 伺服器
1. 網際網路使用者從 SecSvc001.CloudApp.Net (網際網路面向雲端服務) 要求 HTTP 頁面
2. 雲端服務透過連接埠 80 上的開放端點將流量傳遞至 10.0.0.4:80 上的防火牆介面
3. 未對安全性子網路指派 NSG，因此系統 NSG 規則允許流量流往防火牆
4. 流量抵達防火牆的內部 IP 位址 (10.0.1.4)
5. 防火牆開始處理規則：
   1. FW 規則 1 (FW Mgmt) 不適用，移至下一個規則
   2. FW 規則 2 - 5 (RDP 規則) 不適用，移至下一個規則
   3. FW 規則 6 (應用程式：Web) 適用，允許流量，防火牆將其進行 NAT 處理至 10.0.1.4 (IIS01)
6. Frontend 子網路開始處理輸入規則：
   1. NSG 規則 1 (封鎖網際網路) 不適用 (此流量由防火牆進行 NAT 處理，因此來源位址現在是位於安全性子網路上的防火牆，被 Frontend 子網路 NSG 認為是「本機」流量，因此加以允許)，移至下一個規則
   2. 預設 NSG 規則允許子網路對子網路的流量，允許流量，停止處理 NSG 規則
7. IIS01 正在接聽 Web 流量，接收此要求並開始處理要求
8. IIS01 嘗試在 Backend 子網路上起始連往 AppVM01 的 FTP 工作階段
9. Frontend 子網路上的 UDR 路由讓防火牆成為下一個躍點
10. Frontend 子網路上沒有輸出規則，允許流量
11. 防火牆開始處理規則：
    1. FW 規則 1 (FW Mgmt) 不適用，移至下一個規則
    2. FW 規則 2 - 5 (RDP 規則) 不適用，移至下一個規則
    3. FW 規則 6 (應用程式：Web) 不適用，移至下一個規則
    4. FW 規則 7 (應用程式：Backend) 適用，允許流量，防火牆轉送流量到 10.0.2.5 (AppVM01)
12. Backend 子網路開始處理輸入規則：
    1. NSG 規則 1 (封鎖網際網路) 不適用，移至下一個規則
    2. 預設 NSG 規則允許子網路對子網路的流量，允許流量，停止處理 NSG 規則
13. AppVM01 接收要求，啟動工作階段並回應
14. Backend 子網路上的 UDR 路由讓防火牆成為下一個躍點
15. Backend 子網路上沒有輸出 NSG 規則，所以允許回應
16. 因為這會傳回已建立的工作階段上的流量，防火牆將回應傳回給 Web 伺服器 (IIS01)
17. Frontend 子網路開始處理輸入規則：
    1. NSG 規則 1 (封鎖網際網路) 不適用，移至下一個規則
    2. 預設 NSG 規則允許子網路對子網路的流量，允許流量，停止處理 NSG 規則
18. IIS 伺服器接收回應，完成 AppVM01 的交易，然後建置好 HTTP 回應，此 HTTP 回應傳送給要求者
19. Frontend 子網路上沒有輸出 NSG 規則，所以允許回應
20. HTTP 回應抵達防火牆，因為這是已建立的 NAT 工作階段的回應，所以被防火牆接受
21. 防火牆接著將回應重新導向回網際網路使用者
22. Frontend 子網路上沒有輸出 NSG 規則或 UDR 躍點，所以允許回應，網際網路使用者會收到要求的網頁。

#### <a name="allowed-internet-rdp-to-backend"></a>(允許) 網際網路 RDP 到 Backend
1. 網際網路上的伺服器管理員要求透過 SecSvc001.CloudApp.Net:8025 啟動 AppVM01 的 RDP 工作階段，其中 8025 是使用者針對「RDP 到 AppVM01」防火牆規則所指派的連接埠號碼
2. 雲端服務透過連接埠 8025 上的開放端點將流量傳遞至 10.0.0.4:8025 上的防火牆介面
3. 未對安全性子網路指派 NSG，因此系統 NSG 規則允許流量流往防火牆
4. 防火牆開始處理規則：
   1. FW 規則 1 (FW Mgmt) 不適用，移至下一個規則
   2. FW 規則 2 (RDP IIS) 不適用，移至下一個規則
   3. FW 規則 3 (RDP DNS01) 不適用，移至下一個規則
   4. FW 規則 4 (RDP AppVM01) 適用，允許流量，防火牆將其進行 NAT 處理至 10.0.2.5:3386 (AppVM01 上的 RDP 連接埠)
5. Backend 子網路開始處理輸入規則：
   1. NSG 規則 1 (封鎖網際網路) 不適用 (此流量由防火牆進行 NAT 處理，因此來源位址現在是位於安全性子網路上的防火牆，被 Backend 子網路 NSG 認為是「本機」流量，因此加以允許)，移至下一個規則
   2. 預設 NSG 規則允許子網路對子網路的流量，允許流量，停止處理 NSG 規則
6. AppVM01 正在接聽 RDP 流量並回應
7. 由於沒有輸出 NSG 規則，會套用預設規則並允許傳回的流量
8. UDR 將輸出流量路由傳送至防火牆做為下一個躍點
9. 因為這會傳回已建立的工作階段上的流量，防火牆將回應傳回給網際網路使用者
10. 已啟用 RDP 工作階段
11. AppVM01 會提示輸入使用者名稱和密碼

#### <a name="allowed-web-server-dns-lookup-on-dns-server"></a>(允許) DNS 伺服器上的 Web 伺服器 DNS 查閱
1. Web 伺服器 IIS01 需要 www.data.gov 的資料摘要，但需要解析位址。
2. VNet 的網路組態將 DNS01 (Backend 子網路上的 10.0.2.4) 列為主要 DNS 伺服器，IIS01 將 DNS 要求傳送至 DNS01
3. UDR 將輸出流量路由傳送至防火牆做為下一個躍點
4. 沒有輸出 NSG 規則繫結至 Frontend 子網路，允許流量
5. 防火牆開始處理規則：
   1. FW 規則 1 (FW Mgmt) 不適用，移至下一個規則
   2. FW 規則 2 - 5 (RDP 規則) 不適用，移至下一個規則
   3. FW 規則 6 & 7 (應用程式規則) 不適用，移至下一個規則
   4. FW 規則 8 (到網際網路) 不適用，移至下一個規則
   5. FW 規則 9 (DNS) 適用，允許流量，防火牆轉送流量到 10.0.2.4 (DNS01)
6. Backend 子網路開始處理輸入規則：
   1. NSG 規則 1 (封鎖網際網路) 不適用，移至下一個規則
   2. 預設 NSG 規則允許子網路對子網路的流量，允許流量，停止處理 NSG 規則
7. DNS 伺服器收到要求
8. DNS 伺服器沒有快取的位址，並要求網際網路上的根 DNS 伺服器
9. UDR 將輸出流量路由傳送至防火牆做為下一個躍點
10. Backend 子網路上沒有輸出 NSG 規則，允許流量
11. 防火牆開始處理規則：
    1. FW 規則 1 (FW Mgmt) 不適用，移至下一個規則
    2. FW 規則 2 - 5 (RDP 規則) 不適用，移至下一個規則
    3. FW 規則 6 & 7 (應用程式規則) 不適用，移至下一個規則
    4. FW 規則 8 (到網際網路) 適用，允許流量，工作階段是 SNAT 往外到網際網路上的根 DNS 伺服器
12. 網際網路 DNS 伺服器回應，因為此工作階段是從防火牆起始，回應會被防火牆所接受
13. 因為這是已建立的工作階段，防火牆會轉送回應到起始的伺服器 DNS01
14. Backend 子網路開始處理輸入規則：
    1. NSG 規則 1 (封鎖網際網路) 不適用，移至下一個規則
    2. 預設 NSG 規則允許子網路對子網路的流量，允許流量，停止處理 NSG 規則
15. DNS 伺服器接收和快取回應，然後將初始要求回應到 IIS01
16. Backend 子網路上的 UDR 路由讓防火牆成為下一個躍點
17. Backend 子網路上沒有輸出 NSG 規則，允許流量
18. 這是防火牆上的已建立工作階段，防火牆將回應轉送回到 IIS 伺服器
19. Frontend 子網路開始處理輸入規則：
    1. Backend 子網路到 Frontend 子網路的輸入流量沒有適用的 NSG 規則，因此不會套用任何 NSG 規則
    2. 允許子網路間流量的預設系統規則會允許此流量，因此允許流量
20. IIS01 從 DNS01 接收回應

#### <a name="allowed-backend-server-to-frontend-server"></a>(允許) Backend 伺服器到 Frontend 伺服器
1. 透過 RDP 登入 AppVM02 的管理員直接透過 Windows 檔案總管從 IIS01 伺服器要求檔案
2. Backend 子網路上的 UDR 路由讓防火牆成為下一個躍點
3. Backend 子網路上沒有輸出 NSG 規則，所以允許回應
4. 防火牆開始處理規則：
   1. FW 規則 1 (FW Mgmt) 不適用，移至下一個規則
   2. FW 規則 2 - 5 (RDP 規則) 不適用，移至下一個規則
   3. FW 規則 6 & 7 (應用程式規則) 不適用，移至下一個規則
   4. FW 規則 8 (到網際網路) 不適用，移至下一個規則
   5. FW 規則 9 (DNS) 不適用，移至下一個規則
   6. FW 規則 10 (內部子網路) 適用，允許流量，防火牆將流量傳遞至 10.0.1.4 (IIS01)
5. Frontend 子網路開始處理輸入規則：
   1. NSG 規則 1 (封鎖網際網路) 不適用，移至下一個規則
   2. 預設 NSG 規則允許子網路對子網路的流量，允許流量，停止處理 NSG 規則
6. 假設有適當的驗證和授權，IIS01 會接受要求和回應
7. Frontend 子網路上的 UDR 路由讓防火牆成為下一個躍點
8. Frontend 子網路上沒有輸出 NSG 規則，所以允許回應
9. 這是防火牆上現有的工作階段，因此允許此回應，防火牆會將回應傳回給 AppVM02
10. Backend 子網路開始處理輸入規則：
    1. NSG 規則 1 (封鎖網際網路) 不適用，移至下一個規則
    2. 預設 NSG 規則允許子網路對子網路的流量，允許流量，停止處理 NSG 規則
11. AppVM02 接收回應

#### <a name="denied-internet-direct-to-web-server"></a>(拒絕) 網際網路直接到 Web 伺服器
1. 網際網路使用者嘗試透過 FrontEnd001.CloudApp.Net 服務存取 Web 伺服器 IIS01
2. 因為沒有用於 HTTP 流量的開放端點，此流量不會穿過雲端服務到達伺服器
3. 如果基於某些原因而開放端點，Frontend 子網路上的 NSG (封鎖網際網路) 會封鎖此流量
4. 最後，Frontend 子網路 UDR 路由會將來自 IIS01 的任何輸出流量傳送到防火牆做為下一個躍點，防火牆會將此流量視為非對稱流量，並捨棄輸出回應。因此，網際網路和 IIS01 之間至少有三個獨立防禦層，可透過其雲端服務防止未經授權/不適當的存取。

#### <a name="denied-internet-to-backend-server"></a>(拒絕) 網際網路到 Backend 伺服器
1. 網際網路使用者嘗試透過 BackEnd001.CloudApp.Net 服務存取 AppVM01 上的檔案
2. 因為沒有用於檔案共用的開放端點，此流量不會通過雲端服務到達伺服器
3. 如果基於某些原因而開放端點，NSG (封鎖網際網路) 會封鎖此流量
4. 最後，UDR 路由會將來自 AppVM01 的任何輸出流量傳送到防火牆做為下一個躍點，防火牆會將此流量視為非對稱流量，並捨棄輸出回應。因此，網際網路和 AppVM01 之間至少有三個獨立防禦層，可透過其雲端服務防止未經授權/不適當的存取。

#### <a name="denied-frontend-server-to-backend-server"></a>(拒絕) Frontend 伺服器到 Backend 伺服器
1. 假設 IIS01 遭到入侵，正在執行惡意程式碼嘗試掃描 Backend 子網路的伺服器。
2. Frontend 子網路 UDR 路由會傳送任何來自 IIS01 的輸出流量到防火牆做為下一個躍點。 遭到入侵的 VM 無法改變這一點。
3. 如果要求是對 AppVM01 提出，或對 DNS 伺服器提出以進行防火牆可能會允許流量的 DNS 查閱 (因為 FW 規則 7 和 9)，防火牆就會處理流量。 所有其他流量則會遭 FW 規則 11 (全部拒絕) 封鎖。
4. 若防火牆上已啟用進階威脅偵測 (本文未涵蓋這方面的內容，請參閱廠商的說明文件，以了解特定網路應用裝置的進階威脅功能)，且流量包含會加上進階威脅規則旗標的已知簽章或模式，即使是本文所討論的基本轉送規則所會允許的流量可能也會遭到阻止。

#### <a name="denied-internet-dns-lookup-on-dns-server"></a>(拒絕) DNS 伺服器上的網際網路 DNS 查閱
1. 網際網路使用者嘗試透過 BackEnd001.CloudApp.Net 服務查閱 DNS01 上的內部 DNS 記錄 
2. 因為沒有用於 DNS 流量的開放端點，此流量不會穿過雲端服務到達伺服器
3. 如果基於某些原因而開放端點，Frontend 子網路上的 NSG 規則 (封鎖網際網路) 會封鎖此流量
4. 最後，Backend 子網路 UDR 路由會將來自 DNS01 的任何輸出流量傳送到防火牆做為下一個躍點，防火牆會將此流量視為非對稱流量，並捨棄輸出回應。因此，網際網路和 DNS01 之間至少有三個獨立防禦層，可透過其雲端服務防止未經授權/不適當的存取。

#### <a name="denied-internet-to-sql-access-through-firewall"></a>(拒絕) 網際網路透過防火牆對 SQL 進行存取
1. 網際網路使用者從 SecSvc001.CloudApp.Net (網際網路面向雲端服務) 要求 SQL 資料
2. 因為沒有用於 SQL 的開放端點，此流量不會通過雲端服務到達防火牆
3. 如果基於某些原因而開放 SQL 端點，防火牆就會開始處理規則：
   1. FW 規則 1 (FW Mgmt) 不適用，移至下一個規則
   2. FW 規則 2 - 5 (RDP 規則) 不適用，移至下一個規則
   3. FW 規則 6 & 7 (應用程式規則) 不適用，移至下一個規則
   4. FW 規則 8 (到網際網路) 不適用，移至下一個規則
   5. FW 規則 9 (DNS) 不適用，移至下一個規則
   6. FW 規則 10 (內部子網路) 不適用，移至下一個規則
   7. FW 規則 11 (全部拒絕) 適用，封鎖流量，停止處理規則

## <a name="references"></a>參考
### <a name="main-script-and-network-config"></a>主要的指令碼和網路組態
將完整指令碼儲存在 PowerShell 指令碼檔案中。 將網路組態儲存到名為 “NetworkConf2.xml” 的檔案。
視需要修改使用者定義的變數。 執行指令碼，然後依照上面的防火牆規則設定指示進行。

#### <a name="full-script"></a>完整指令碼
根據使用者定義的變數，此指令碼會執行下列動作：

1. 連線到 Azure 訂用帳戶
2. 建立新的儲存體帳戶
3. 依網路組態檔中的定義建立新的 VNet 和三個子網路
4. 建置五部虛擬機器：1 個防火牆和 4 個 Windows Server VM
5. 設定 UDR，包括：
   1. 建立兩個新的路由表
   2. 將路由新增至路由表
   3. 將路由表繫結至適當子網路
6. 啟用 NVA 上的 IP 轉送
7. 設定 NSG，包括：
   1. 建立 NSG
   2. 新增規則
   3. 將 NSG 繫結至適當的子網路

此 PowerShell 指令碼應該在連線到網際網路的電腦或伺服器上本機執行。

> [!IMPORTANT]
> 此指令碼在執行時，PowerShell 中可能會跳出警告或其他參考訊息。 只有紅色字體的錯誤訊息才需要擔心。
> 
> 

    <# 
     .SYNOPSIS
      Example of DMZ and User Defined Routing in an isolated network (Azure only, no hybrid connections)

     .DESCRIPTION
      This script will build out a sample DMZ setup containing:
       - A default storage account for VM disks
       - Three new cloud services
       - Three Subnets (SecNet, FrontEnd, and BackEnd subnets)
       - A Network Virtual Appliance (NVA), in this case a Barracuda NextGen Firewall
       - One server on the FrontEnd Subnet
       - Three Servers on the BackEnd Subnet
       - IP Forwading from the FireWall out to the internet
       - User Defined Routing FrontEnd and BackEnd Subnets to the NVA

      Before running script, ensure the network configuration file is created in
      the directory referenced by $NetworkConfigFile variable (or update the
      variable to reflect the path and file name of the config file being used).

     .Notes
      Everyone's security requirements are different and can be addressed in a myriad of ways.
      Please be sure that any sensitive data or applications are behind the appropriate
      layer(s) of protection. This script serves as an example of some of the techniques
      that can be used, but should not be used for all scenarios. You are responsible to
      assess your security needs and the appropriate protections needed, and then effectively
      implement those protections.

      Security Service (SecNet subnet 10.0.0.0/24)
       myFirewall - 10.0.0.4

      FrontEnd Service (FrontEnd subnet 10.0.1.0/24)
       IIS01      - 10.0.1.4

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
        $SecureService = "SecSvc001"
        $FrontEndService = "FrontEnd001"
        $BackEndService = "BackEnd001"

      # Network Details
        $VNetName = "CorpNetwork"
        $VNetPrefix = "10.0.0.0/16"
        $SecNet = "SecNet"
        $FESubnet = "FrontEnd"
        $FEPrefix = "10.0.1.0/24"
        $BESubnet = "BackEnd"
        $BEPrefix = "10.0.2.0/24"
        $NetworkConfigFile = "C:\Scripts\NetworkConf3.xml"

      # VM Base Disk Image Details
        $SrvImg = Get-AzureVMImage | Where {$_.ImageFamily -match 'Windows Server 2012 R2 Datacenter'} | sort PublishedDate -Descending | Select ImageName -First 1 | ForEach {$_.ImageName}
        $FWImg = Get-AzureVMImage | Where {$_.ImageFamily -match 'Barracuda NextGen Firewall'} | sort PublishedDate -Descending | Select ImageName -First 1 | ForEach {$_.ImageName}

      # UDR Details
        $FERouteTableName = "FrontEndSubnetRouteTable"
        $BERouteTableName = "BackEndSubnetRouteTable"

      # NSG Details
        $NSGName = "MyVNetSG"

    # User Defined VM Specific Config
        # Note: To ensure UDR and IP forwarding is setup
        # properly this script requires VM 0 be the NVA.

        # VM 0 - The Network Virtual Appliance (NVA)
          $VMName += "myFirewall"
          $ServiceName += $SecureService
          $VMFamily += "Firewall"
          $img += $FWImg
          $size += "Small"
          $SubnetName += $SecNet
          $VMIP += "10.0.0.4"

        # VM 1 - The Web Server
          $VMName += "IIS01"
          $ServiceName += $FrontEndService
          $VMFamily += "Windows"
          $img += $SrvImg
          $size += "Standard_D3"
          $SubnetName += $FESubnet
          $VMIP += "10.0.1.4"

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

    If (Test-AzureName -Service -Name $SecureService) { 
        Write-Host "The SecureService service name is already in use, please pick a different service name." -ForegroundColor Yellow
        $FatalError = $true}
    Else { Write-Host "The FrontEndService service name is valid for use." -ForegroundColor Green}

    If (Test-AzureName -Service -Name $FrontEndService) { 
        Write-Host "The FrontEndService service name is already in use, please pick a different service name." -ForegroundColor Yellow
        $FatalError = $true}
    Else { Write-Host "The FrontEndService service name is valid for use" -ForegroundColor Green}

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
        New-AzureService -Location $DeploymentLocation -ServiceName $SecureService -ErrorAction Stop
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
                # Note: All traffic goes through the firewall, so we'll need to set up all ports here.
                #       Also, the firewall will be redirecting traffic to a new IP and Port in a forwarding
                #       rule, so all of these endpoint have the same public and local port and the firewall
                #       will do the mapping, NATing, and/or redirection as declared in the firewall rules.
                Add-AzureEndpoint -Name "MgmtPort1" -Protocol tcp -PublicPort 801  -LocalPort 801  -VM (Get-AzureVM -ServiceName $ServiceName[$i] -Name $VMName[$i]) | Update-AzureVM
                Add-AzureEndpoint -Name "MgmtPort2" -Protocol tcp -PublicPort 807  -LocalPort 807  -VM (Get-AzureVM -ServiceName $ServiceName[$i] -Name $VMName[$i]) | Update-AzureVM
                Add-AzureEndpoint -Name "HTTP"      -Protocol tcp -PublicPort 80   -LocalPort 80   -VM (Get-AzureVM -ServiceName $ServiceName[$i] -Name $VMName[$i]) | Update-AzureVM
                Add-AzureEndpoint -Name "RDPWeb"    -Protocol tcp -PublicPort 8014 -LocalPort 8014 -VM (Get-AzureVM -ServiceName $ServiceName[$i] -Name $VMName[$i]) | Update-AzureVM
                Add-AzureEndpoint -Name "RDPApp1"   -Protocol tcp -PublicPort 8025 -LocalPort 8025 -VM (Get-AzureVM -ServiceName $ServiceName[$i] -Name $VMName[$i]) | Update-AzureVM
                Add-AzureEndpoint -Name "RDPApp2"   -Protocol tcp -PublicPort 8026 -LocalPort 8026 -VM (Get-AzureVM -ServiceName $ServiceName[$i] -Name $VMName[$i]) | Update-AzureVM
                Add-AzureEndpoint -Name "RDPDNS01"  -Protocol tcp -PublicPort 8024 -LocalPort 8024 -VM (Get-AzureVM -ServiceName $ServiceName[$i] -Name $VMName[$i]) | Update-AzureVM
                # Note: A SSH endpoint is automatically created on port 22 when the appliance is created.
                }
            Else
                {
                New-AzureVMConfig -Name $VMName[$i] -ImageName $img[$i] –InstanceSize $size[$i] | `
                    Add-AzureProvisioningConfig -Windows -AdminUsername $LocalAdmin -Password $LocalAdminPwd  | `
                    Set-AzureSubnet  –SubnetNames $SubnetName[$i] | `
                    Set-AzureStaticVNetIP -IPAddress $VMIP[$i] | `
                    Set-AzureVMMicrosoftAntimalwareExtension -AntimalwareConfiguration '{"AntimalwareEnabled" : true}' | `
                    Remove-AzureEndpoint -Name "RemoteDesktop" | `
                    Remove-AzureEndpoint -Name "PowerShell" | `
                    New-AzureVM –ServiceName $ServiceName[$i] -VNetName $VNetName -Location $DeploymentLocation
                }
            $i++
        }

    # Configure UDR and IP Forwarding
        Write-Host "Configuring UDR" -ForegroundColor Cyan

      # Create the Route Tables
        Write-Host "Creating the Route Tables" -ForegroundColor Cyan 
        New-AzureRouteTable -Name $BERouteTableName `
            -Location $DeploymentLocation `
            -Label "Route table for $BESubnet subnet"
        New-AzureRouteTable -Name $FERouteTableName `
            -Location $DeploymentLocation `
            -Label "Route table for $FESubnet subnet"

      # Add Routes to Route Tables
        Write-Host "Adding Routes to the Route Tables" -ForegroundColor Cyan 
        Get-AzureRouteTable $BERouteTableName `
            |Set-AzureRoute -RouteName "All traffic to FW" -AddressPrefix 0.0.0.0/0 `
            -NextHopType VirtualAppliance `
            -NextHopIpAddress $VMIP[0]
        Get-AzureRouteTable $BERouteTableName `
            |Set-AzureRoute -RouteName "Internal traffic to FW" -AddressPrefix $VNetPrefix `
            -NextHopType VirtualAppliance `
            -NextHopIpAddress $VMIP[0]
        Get-AzureRouteTable $BERouteTableName `
            |Set-AzureRoute -RouteName "Allow Intra-Subnet Traffic" -AddressPrefix $BEPrefix `
            -NextHopType VNETLocal
        Get-AzureRouteTable $FERouteTableName `
            |Set-AzureRoute -RouteName "All traffic to FW" -AddressPrefix 0.0.0.0/0 `
            -NextHopType VirtualAppliance `
            -NextHopIpAddress $VMIP[0]
        Get-AzureRouteTable $FERouteTableName `
            |Set-AzureRoute -RouteName "Internal traffic to FW" -AddressPrefix $VNetPrefix `
            -NextHopType VirtualAppliance `
            -NextHopIpAddress $VMIP[0]
        Get-AzureRouteTable $FERouteTableName `
            |Set-AzureRoute -RouteName "Allow Intra-Subnet Traffic" -AddressPrefix $FEPrefix `
            -NextHopType VNETLocal

      # Assoicate the Route Tables with the Subnets
        Write-Host "Binding Route Tables to the Subnets" -ForegroundColor Cyan 
        Set-AzureSubnetRouteTable -VirtualNetworkName $VNetName `
            -SubnetName $BESubnet `
            -RouteTableName $BERouteTableName
        Set-AzureSubnetRouteTable -VirtualNetworkName $VNetName `
            -SubnetName $FESubnet `
            -RouteTableName $FERouteTableName

     # Enable IP Forwarding on the Virtual Appliance
        Get-AzureVM -Name $VMName[0] -ServiceName $ServiceName[0] `
            |Set-AzureIPForwarding -Enable

    # Configure NSG
        Write-Host "Configuring the Network Security Group (NSG)" -ForegroundColor Cyan

      # Build the NSG
        Write-Host "Building the NSG" -ForegroundColor Cyan
        New-AzureNetworkSecurityGroup -Name $NSGName -Location $DeploymentLocation -Label "Security group for $VNetName subnets in $DeploymentLocation"

      # Add NSG Rule
        Write-Host "Writing rules into the NSG" -ForegroundColor Cyan
        Get-AzureNetworkSecurityGroup -Name $NSGName | Set-AzureNetworkSecurityRule -Name "Isolate the $VNetName VNet from the Internet" -Type Inbound -Priority 100 -Action Deny `
            -SourceAddressPrefix INTERNET -SourcePortRange '*' `
            -DestinationAddressPrefix VIRTUAL_NETWORK -DestinationPortRange '*' `
            -Protocol *

      # Assign the NSG to two Subnets
        # The NSG is *not* bound to the Security Subnet. The result
        # is that internet traffic flows only to the Security subnet
        # since the NSG bound to the Frontend and Backback subnets
        # will Deny internet traffic to those subnets.
        Write-Host "Binding the NSG to two subnets" -ForegroundColor Cyan
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
              <Subnet name="SecNet">
                <AddressPrefix>10.0.0.0/24</AddressPrefix>
              </Subnet>
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
[1]: ./media/virtual-networks-dmz-nsg-fw-udr-asm/example3design.png "具有 NVA、NSG 和 UDR 的雙向 DMZ"
[2]: ./media/virtual-networks-dmz-nsg-fw-udr-asm/example3firewalllogical.png "防火牆規則的邏輯視圖"
[3]: ./media/virtual-networks-dmz-nsg-fw-udr-asm/createnetworkobjectfrontend.png "建立前端網路物件"
[4]: ./media/virtual-networks-dmz-nsg-fw-udr-asm/createnetworkobjectdns.png "建立 DNS 伺服器物件"
[5]: ./media/virtual-networks-dmz-nsg-fw-udr-asm/createnetworkobjectrdpa.png "預設 RDP 規則的複本"
[6]: ./media/virtual-networks-dmz-nsg-fw-udr-asm/createnetworkobjectrdpb.png "AppVM01 規則"
[7]: ./media/virtual-networks-dmz-nsg-fw-udr-asm/iconapplicationredirect.png "應用程式重新導向圖示"
[8]: ./media/virtual-networks-dmz-nsg-fw-udr-asm/icondestinationnat.png "目的地 NAT 圖示"
[9]: ./media/virtual-networks-dmz-nsg-fw-udr-asm/iconpass.png "傳遞圖示"
[10]: ./media/virtual-networks-dmz-nsg-fw-udr-asm/rulefirewall.png "防火牆管理規則"
[11]: ./media/virtual-networks-dmz-nsg-fw-udr-asm/rulerdp.png "防火牆 RDP 規則"
[12]: ./media/virtual-networks-dmz-nsg-fw-udr-asm/ruleweb.png "防火牆 Web 規則"
[13]: ./media/virtual-networks-dmz-nsg-fw-udr-asm/ruleappvm01.png "防火牆 AppVM01 規則"
[14]: ./media/virtual-networks-dmz-nsg-fw-udr-asm/ruleoutbound.png "防火牆輸出規則"
[15]: ./media/virtual-networks-dmz-nsg-fw-udr-asm/ruledns.png "防火牆 DNS 規則"
[16]: ./media/virtual-networks-dmz-nsg-fw-udr-asm/ruleintravnet.png "防火牆內部 VNet 規則"
[17]: ./media/virtual-networks-dmz-nsg-fw-udr-asm/ruledeny.png "防火牆拒絕規則"
[18]: ./media/virtual-networks-dmz-nsg-fw-udr-asm/firewallruleactivate.png "防火牆規則啟用"

<!--Link References-->
[HOME]: ../best-practices-network-security.md
[SampleApp]: ./virtual-networks-sample-app.md
