---
title: 使用 VPN 將 Azure Stack 連線至 Azure
description: 如何使用 VPN 將 Azure Stack 中的虛擬網路連線至 Azure 中的虛擬網路。
services: azure-stack
documentationcenter: ''
author: sethmanheim
manager: femila
editor: ''
ms.assetid: ''
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: get-started-article
ms.date: 09/12/2018
ms.author: sethm
ms.reviewer: scottnap
ms.openlocfilehash: dcbe222d8dd3d3c658e5778fdc4bc1cc01b5c12d
ms.sourcegitcommit: 4b1083fa9c78cd03633f11abb7a69fdbc740afd1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/10/2018
ms.locfileid: "49078880"
---
# <a name="connect-azure-stack-to-azure-using-vpn"></a>使用 VPN 將 Azure Stack 連線至 Azure

「適用於：Azure Stack 整合系統」

本文示範如何建立站對站 VPN，以將 Azure Stack 中的虛擬網路連線至 Azure 中的虛擬網路。

## <a name="before-you-begin"></a>開始之前

若要完成連線設定，請務必在開始前備妥下列項目：

* 直接連線至網際網路的 Azure Stack 整合系統 (多節點) 部署。 必須可從公用網際網路直接連線至您的外部公用 IP 位址範圍。
* 有效的 Azure 訂用帳戶。 如果您沒有 Azure 訂用帳戶，您可以建立[免費 Azure 帳戶](https://azure.microsoft.com/free/?b=17.06)。

### <a name="vpn-connection-diagram"></a>VPN 連線圖表

下圖顯示完成時連線設定看起來的樣子：

![站對站 VPN 連線組態](media/azure-stack-connect-vpn/image2.png)

### <a name="network-configuration-example-values"></a>網路組態範例值

網路組態範例值表格顯示本文中使用的範例值。 您可以使用這些值，也可以參考這些值，以深入了解本文中的範例。

**網路組態範例**

|   |Azure Stack|Azure|
|---------|---------|---------|
|虛擬網路名稱     |Azs-VNet|AzureVNet |
|虛擬網路位址空間 |10.1.0.0/16|10.100.0.0/16|
|子網路名稱     |FrontEnd|FrontEnd|
|子網路位址範圍|10.1.0.0/24 |10.100.0.0/24 |
|閘道器子網路     |10.1.1.0/24|10.100.1.0/24|

## <a name="create-the-network-resources-in-azure"></a>在 Azure 中建立網路資源

請先為 Azure 建立網路資源。 下列指示說明如何使用 [Azure 入口網站](http://portal.azure.com/)來建立資源。

### <a name="create-the-virtual-network-and-virtual-machine-vm-subnet"></a>建立虛擬網路和虛擬機器 (VM) 子網路

1. 使用 Azure 帳戶登入 [Azure 入口網站](http://portal.azure.com/)。
2. 在使用者入口網站中，選取 [+ 建立資源]。
3. 移至 [Marketplace]，然後選取 [網路]。
4. 選取 [虛擬網路]。
5. 使用網路設定表中的資訊，以識別 Azure [名稱]、[位址空間]、[子網路名稱] 及 [子網路位址範圍] 的值。
6. 針對 [資源群組]，建立資源群組，或選取 [使用現有的] \(如果您已經有資源群組)。
7. 選取 VNet 的 [位置]。  如果您使用範例值，請選取 [美國東部]，或使用您偏好的另一個位置。
8. 選取 [釘選到儀表板]。
9. 選取 [建立] 。

### <a name="create-the-gateway-subnet"></a>建立閘道子網路

1. 從儀表板開啟您建立的虛擬網路資源 (**AzureVNet**)。
2. 在 [設定] 區段上，選取 [子網路]。
3. 選取 [閘道子網路] 以將閘道子網路新增到虛擬網路。
4. 子網路名稱預設為 **GatewaySubnet**。

   >[!IMPORTANT]
   >閘道子網路很特別，**必須**具有此特定名稱，才能正常運作。

5. 在 [位址範圍] 欄位中，確認位址是 **10.100.1.0/24**。
6. 選取 [確定] 以建立閘道子網路。

### <a name="create-the-virtual-network-gateway"></a>建立虛擬網路閘道

1. 在 Azure 入口網站中，選取 [+ 建立資源]。  
2. 移至 [Marketplace]，然後選取 [網路]。
3. 從網路資源清單中，選取 [虛擬網路閘道]。
4. 在 [名稱] 中輸入 **Azure-GW**。
5. 若要選擇虛擬網路，請選取 [虛擬網路]。 然後從清單中選取 [AzureVnet]。
6. 選取 [公用 IP 位址]。 當 [選擇公用 IP 位址] 區段開啟時，請選取 [新建]。
7. 在 [名稱] 中輸入 **Azure-GW-PiP**，然後選取 [確定]。
8. 針對 [VPN 類型]，預設會選取 [依路由]。 保留 [依路由] VPN 類型。
9. 確認 [訂用帳戶] 和 [位置] 均正確無誤。 您可以將資源釘選到儀表板。 選取 [建立] 。

### <a name="create-the-local-network-gateway-resource"></a>建立區域網路閘道資源

1. 在 Azure 入口網站中，選取 [+ 建立資源]。
2. 移至 [Marketplace]，然後選取 [網路]。
3. 從資源清單中，選取 [區域網路閘道]。
4. 在 [名稱] 中輸入 **Azs-GW**。
5. 在 [IP 位址] 中，輸入稍早列在網路設定表中的 Azure Stack 虛擬網路閘道的公用 IP 位址。
6. 在 Azure Stack 的 [位址空間] 中，輸入 **AzureVNet** 的 **10.1.0.0/24** 和 **10.1.1.0/24** 位址空間。
7. 確認您的 [訂用帳戶]、[資源群組] 和 [位置] 正確無誤，然後選取 [建立]。

## <a name="create-the-connection"></a>建立連線

1. 在使用者入口網站中，選取 [+ 建立資源]。
2. 移至 [Marketplace]，然後選取 [網路]。
3. 從資源清單中，選取 [連線]。
4. 在 [基本] 設定區段的 [連線類型] 中，選擇 [站對站 (IPSec)]。
5. 選取 [訂用帳戶]、[資源群組] 和 [位置]，然後選取 [確定]。
6. 在 [設定] 區段上，選取 [虛擬網路閘道]，然後選取 [Azure-GW]。
7. 選取 [區域網路閘道]，然後選取 [Azs-GW]。
8. 在 [連線名稱] 中，輸入 **Azure-Azs**。
9. 在 [共用金鑰 (PSK)] 中，輸入 **12345**。 選取 [確定] 。

   >[!NOTE]
   >如果您使用不同的共用金鑰值，請記住該值與您在連線另一端建立的共用金鑰值「必須」相符。

10. 檢閱 [摘要] 區段，然後選取 [確定]。

## <a name="create-a-virtual-machine"></a>建立虛擬機器

現在，請在 Azure 中建立虛擬機器，然後放入虛擬網路中的 VM 子網路上。

1. 在 Azure 入口網站中，選取 [+ 建立資源]。
2. 移至 [Marketplace]，然後選取 [計算]。
3. 在虛擬機器映像清單中，選取 [Windows Server 2016 Datacenter Oval] 映像。
4. 在 [基本] 區段的 [名稱] 中，輸入 **AzureVM**。
5. 輸入有效的使用者名稱和密碼。 建立虛擬機器之後，您將使用此帳戶來登入虛擬機器。
6. 提供 [訂用帳戶]、[資源群組] 和 [位置]，然後選取 [確定]。
7. 在 [大小] 區段上，選取此執行個體的虛擬機器大小，然後選取 [選取]。
8. 在 [設定] 區段上，您可以使用預設設定。 在您選取 [確定] 之前，請先確認：

   * 已選取 [AzureVnet] 虛擬網路。
   * 子網路已設定為 **10.100.0.0/24**。

   選取 [確定] 。

9. 檢閱 [摘要] 區段上的設定，然後選取 [確定]。

## <a name="create-the-network-resources-in-azure-stack"></a>在 Azure Stack 中建立網路資源

接下來，請在 Azure Stack 中建立網路資源。

### <a name="sign-in-as-a-user"></a>以使用者身分登入

服務系統管理員可以用使用者身分登入，以測試其使用者可能使用的方案、供應項目及訂用帳戶。 如果您還沒有使用者帳戶，請先[建立使用者帳戶](azure-stack-add-new-user-aad.md)再登入。

### <a name="create-the-virtual-network-and-a-vm-subnet"></a>建立虛擬網路和 VM 子網路

1. 以使用者帳戶來登入使用者入口網站。
2. 在使用者入口網站中，選取 [+ 建立資源]。

    ![建立新的虛擬網路](media/azure-stack-create-vpn-connection-one-node-tp2/image3.png)

3. 移至 [Marketplace]，然後選取 [網路]。
4. 選取 [虛擬網路]。
5. 在 [名稱]、[位址空間]、[子網路名稱] 及 [子網路位址範圍] 中，使用網路設定表中的值。
6. 在 [訂用帳戶] 中，會顯示您先前建立的訂用帳戶。
7. 針對 [資源群組]，您可以建立資源群組，或選取 [使用現有的] \(如果您已經有資源群組)。
8. 驗證預設位置。
9. 選取 [釘選到儀表板]。
10. 選取 [建立] 。

### <a name="create-the-gateway-subnet"></a>建立閘道子網路

1. 在儀表板上，開啟您建立的 Azs-VNet 虛擬網路資源。
2. 在 [設定] 區段上，選取 [子網路]。
3. 若要將閘道子網路新增到虛擬網路，請選取 [閘道子網路]。

    ![新增閘道子網路](media/azure-stack-create-vpn-connection-one-node-tp2/image4.png)

4. 子網路名稱預設為 **GatewaySubnet**。 閘道子網路相當特殊。 為了正常運作，它們必須使用 *GatewaySubnet* 名稱。
5. 在 [位址範圍] 中，確認位址是 **10.1.1.0/24**。
6. 選取 [確定] 以建立閘道子網路。

### <a name="create-the-virtual-network-gateway"></a>建立虛擬網路閘道

1. 在 Azure Stack 入口網站中，選取 [+ 建立資源]。
2. 移至 [Marketplace]，然後選取 [網路]。
3. 從網路資源清單中，選取 [虛擬網路閘道]。
4. 在 [名稱] 中輸入 **Azs-GW**。
5. 選取 [虛擬網路] 項目以選擇虛擬網路。 從清單中選取 [Azs-VNet]。
6. 選取 [公用 IP 位址] 功能表項目。 當 [選擇公用 IP 位址] 區段開啟時，請選取 [新建]。
7. 在 [名稱] 中輸入 **Azs-GW-PiP**，然後選取 [確定]。
8. 針對 [VPN 類型]，預設會選取 [路由型]。 保留 [依路由] VPN 類型。
9. 確認 [訂用帳戶] 和 [位置] 均正確無誤。 您可以將資源釘選到儀表板。 選取 [建立] 。

### <a name="create-the-local-network-gateway"></a>建立區域網路閘道

在 Azure Stack 中，「區域網路閘道」的概念稍微不同於 Azure 部署。

在 Azure 部署中，區域網路閘道代表內部部署 (位於使用者位置) 實體裝置，用於連線至 Azure 中的虛擬網路閘道。 但是在 Azure Stack 中，連線的兩端都是虛擬網路閘道！

更廣泛來說，區域網路閘道資源一律是指連線另一端的遠端閘道。

### <a name="create-the-local-network-gateway-resource"></a>建立區域網路閘道資源

1. 登入 Azure Stack 入口網站。
2. 在使用者入口網站中，選取 [+ 建立資源]。
3. 移至 [Marketplace]，然後選取 [網路]。
4. 從資源清單中，選取 [區域網路閘道]。
5. 在 [名稱] 中輸入 **Azure-GW**。
6. 在 [IP 位址] 中，輸入 Azure **Azure-GW-PiP** 中的虛擬網路閘道公用 IP 位址。 此位址先前已顯示在網路組態表中。
7. 在 [位址空間] 中，輸入 **10.100.0.0/24** 和 **10.100.1.0/24**，代表您建立的 Azure VNET 位址空間。
8. 確認您的 [訂用帳戶]、[資源群組] 和 [位置] 正確無誤，然後選取 [建立]。

### <a name="create-the-connection"></a>建立連線

1. 在使用者入口網站中，選取 [+ 建立資源]。
2. 移至 [Marketplace]，然後選取 [網路]。
3. 從資源清單中，選取 [連線]。
4. 在 [基本] 設定區段的 [連線類型] 中，選取 [站對站 (IPSec)]。
5. 選取 [訂用帳戶]、[資源群組] 和 [位置]，然後選取 [確定]。
6. 在 [設定] 區段上，選取 [虛擬網路閘道]，然後選取 [Azs-GW]。
7. 選取 區域網路閘道，然後選取 Azure-GW。
8. 在 [連線名稱] 中，輸入 **Azs-Azure**。
9. 在 [共用金鑰 (PSK)] 中輸入 **12345**，然後選取 [確定]。
10. 在 [摘要] 區段上，選取 [確定]。

### <a name="create-a-virtual-machine-vm"></a>建立虛擬機器 (VM)

若要檢查 VPN 連線，您需要建立兩個 VM：一個在 Azure 中，一個在 Azure Stack 中。 建立這些 VM 之後，您可以使用它們透過 VPN 通道來傳送和接收資料。

1. 在 Azure 入口網站中，選取 [+ 建立資源]。
2. 移至 [Marketplace]，然後選取 [計算]。
3. 在虛擬機器映像清單中，選取 [Windows Server 2016 Datacenter Oval] 映像。
4. 在 [基本] 區段的 [名稱] 中，輸入 **Azs-VM**。
5. 輸入有效的使用者名稱和密碼。 建立 VM 之後，您將使用此帳戶來登入 VM。
6. 提供 [訂用帳戶]、[資源群組] 和 [位置]，然後選取 [確定]。
7. 在 [大小] 區段，選取此執行個體的虛擬機器大小，然後選取 [選取]。
8. 在 [設定] 區段上，接受預設值。 確定已選取 [Azs-VNet] 虛擬網路。 確認子網路已設定為 **10.1.0.0/24**。 然後選取 [確定]。
9. 在 [摘要] 區段上檢視設定，然後選取 [確定]。

## <a name="test-the-connection"></a>測試連線

建立站對站連線之後，您應驗證是否可以取得雙向流入的資料。 測試連線的最簡單方式就是進行 ping 測試：

* 登入您在 Azure Stack 中建立的虛擬機器，然後在 Azure 中偵測該虛擬機器。
* 登入您在 Azure 中建立的虛擬機器，然後在 Azure Stack 中偵測該虛擬機器。

>[!NOTE]
>為了確保您是透過站對站連線傳送流量，請偵測遠端子網路上之虛擬機器的直接 IP (DIP) 位址，而非 VIP。

### <a name="sign-in-to-the-user-vm-in-azure-stack"></a>登入 Azure Stack 中的使用者 VM

1. 登入 Azure Stack 入口網站。
2. 在左側導覽列，選取 [虛擬機器]。
3. 在 VM 清單中，尋找並選取您先前建立的 **Azs-VM**。
4. 在虛擬機器的區段上，選取 [連線]，然後開啟 Azs-VM.rdp 檔案。

     ![[連線] 按鈕](media/azure-stack-create-vpn-connection-one-node-tp2/image17.png)

5. 使用您建立虛擬網路時所設定的帳戶來登入。
6. 開啟已提高權限的 [Windows PowerShell] 視窗。
7. 輸入 **ipconfig /all**。
8. 在輸出中，尋找「IPv4 位址」，然後儲存該位址以供稍後使用。 這是您將從 Azure 偵測的位址。 在範例環境中，位址是 **10.1.0.4**，但在您的環境中可能會不同。 它應該落在您先前建立的 **10.1.0.0/24** 子網路內。
9. 若要建立允許虛擬機器回應 Ping 的防火牆規則，請執行下列 PowerShell 命令：

   ```powershell
   New-NetFirewallRule `
    –DisplayName “Allow ICMPv4-In” `
    –Protocol ICMPv4
   ```

### <a name="sign-in-to-the-tenant-vm-in-azure"></a>登入 Azure 中的租用戶 VM

1. 登入 Azure 入口網站。
2. 在左側導覽列，選取 [虛擬機器]。
3. 從虛擬機器清單中，尋找並選取您先前建立的 **Azure-VM**。
4. 在虛擬機器的區段上，選取 [連線]。
5. 使用您建立虛擬網路時所設定的帳戶來登入。
6. 開啟已提高權限的 [Windows PowerShell] 視窗。
7. 輸入 **ipconfig /all**。
8. 您應該會看到落在 **10.100.0.0/24** 內的 IPv4 位址。 在範例環境中，該位址是 **10.100.0.4**，但您的位址可能會不同。
9. 若要建立允許虛擬機器回應 Ping 的防火牆規則，請執行下列 PowerShell 命令：

   ```powershell
   New-NetFirewallRule `
    –DisplayName “Allow ICMPv4-In” `
    –Protocol ICMPv4
   ```

10. 從 Azure 中的虛擬機器，透過通道偵測 Azure Stack 中的虛擬機器。 若要這麼做，請偵測您從 Azs-VM 記錄的 DIP。 在範例環境中，這是 **10.1.0.4**，但請務必 Ping 您在實驗室中記下的位址。 您應該會看到如下列螢幕擷取畫面的結果：

    ![成功的 Ping](media/azure-stack-create-vpn-connection-one-node-tp2/image19b.png)

11. 來自遠端虛擬機器的回覆表示測試成功！ 您可以關閉虛擬機器視窗。

您也應該執行更嚴格的資料轉送測試。 例如，雙向複製不同大小的檔案。

### <a name="viewing-data-transfer-statistics-through-the-gateway-connection"></a>檢視透過閘道連線的資料傳輸統計資料

如果想要知道有多少資料通過您的站對站連線，您可以在 [連線] 區段上取得此資訊。 此測試也是確認您剛傳送的 Ping 是否真的通過 VPN 連線的另一種方法。

1. 登入 Azure Stack 中的使用者虛擬機器之後，請使用您的使用者帳戶來登入使用者入口網站。
2. 移至 [所有資源]，然後選取 [Azs-Azure] 連線。 [連線] 隨即顯示。
3. [連線] 區段上會顯示 [資料輸入] 和 [資料輸出] 的統計資料。 在以下的螢幕擷取畫面中，那些大的數字歸因於額外的檔案傳輸。 您應該會在該處看到一些非零的值。

    ![資料輸入和輸出](media/azure-stack-connect-vpn/Connection.png)

## <a name="next-steps"></a>後續步驟

[將應用程式部署到 Azure 和 Azure Stack](azure-stack-solution-pipeline.md)
