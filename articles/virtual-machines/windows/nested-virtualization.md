---
title: 如何在 Azure 虛擬機器中啟用巢狀虛擬化 | Microsoft Docs
description: 如何在 Azure 虛擬機器中啟用巢狀虛擬化
services: virtual-machines-windows
documentationcenter: virtual-machines
author: cynthn
manager: jeconnoc
ms.author: cynthn
ms.date: 10/09/2017
ms.topic: howto
ms.service: virtual-machines-windows
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.openlocfilehash: 0d47b89a468aade8cec51ec04709ed99bfd4684c
ms.sourcegitcommit: 1aedb52f221fb2a6e7ad0b0930b4c74db354a569
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/17/2018
ms.locfileid: "42144020"
---
# <a name="how-to-enable-nested-virtualization-in-an-azure-vm"></a>如何在 Azure VM 中啟用巢狀虛擬化

多個 Azure 虛擬機器系列皆支援巢狀虛擬化。 此功能在如開發、測試、訓練和示範環境的支援案例中，可提供更多的彈性。   

本文會在 Azure VM 上逐步啟用 Hyper-V，並設定該客體虛擬機器的網際網路連線。

## <a name="create-a-nesting-capable-azure-vm"></a>建立可使用巢狀結構的 Azure VM

建立新的 Windows Server 2016 Azure VM。  為了方便參考，所有 v3 虛擬機器都支援巢狀虛擬化。 如需支援巢狀結構的虛擬機器完整清單，請參閱 [Azure 計算單位](acu.md)一文。

請務必選擇大小足以支援客體虛擬機器需求的 VM。 在此範例中，我們使用的是 D3_v3 大小的 Azure VM。 

您可以在[這裡](https://azure.microsoft.com/regions/services/)檢視 Dv3 或 Ev3 系列虛擬機器的區域可用性。

>[!NOTE]
>
>如需有關建立新虛擬機器的詳細指示，請參閱[使用 Azure PowerShell 模組建立和管理 Windows VM](https://docs.microsoft.com/azure/virtual-machines/windows/tutorial-manage-vm)
    
## <a name="connect-to-your-azure-vm"></a>連線至您的 Azure VM

建立與虛擬機器的遠端桌面連線。

1. 按一下虛擬機器屬性上的 [連線] 按鈕。 隨即建立並下載遠端桌面通訊協定檔案 (.rdp 檔案)。

2. 若要連線至您的 VM，請開啟下載的 RDP 檔案。 出現提示時，按一下 [連線]。 在 Mac 上，您需要 RDP 用戶端，例如來自 Mac App Store 的[遠端桌面用戶端](https://itunes.apple.com/us/app/microsoft-remote-desktop/id715768417?mt=12)。

3. 輸入在建立虛擬機器時指定的使用者名稱和密碼，然後按一下 [確定]。

4. 您可能會在登入過程中收到憑證警告。 按一下 [是] 或 [繼續] 以繼續進行連線。

## <a name="enable-the-hyper-v-feature-on-the-azure-vm"></a>在 Azure VM 上啟用 Hyper-V 功能
您可以手動設定這些設定，或者使用我們提供的 PowerShell 指令碼，將設定自動化。

### <a name="option-1-use-a-powershell-script-to-configure-nested-virtualization"></a>選項 1：使用 PowerShell 指令碼設定巢狀虛擬化
要在 Windows Server 2016 主機上啟用巢狀虛擬化的 PowerShell 指令碼可在 [GitHub](https://github.com/charlieding/Virtualization-Documentation/tree/live/hyperv-tools/Nested) \(英文\) 上取得。 此指令碼會檢查必要條件，然後在 Azure VM 上設定巢狀虛擬化。 若要完成設定，必須重新啟動 Azure VM。 此指令碼或許能夠在其他環境下運作，但並不能保證。 查看 Azure 部落格文章中有關在 Azure 上執行之巢狀虛擬化的即時影片示範！ https://aka.ms/AzureNVblog。

### <a name="option-2-configure-nested-virtualization-manually"></a>選項 2：手動設定巢狀虛擬化

1. 在 Azure VM 上，以系統管理員身分開啟 PowerShell。 

2. 啟用 Hyper-V 功能和管理工具。

    ```powershell
    Install-WindowsFeature -Name Hyper-V -IncludeManagementTools -Restart
    ```

    >[!WARNING] 
    >
    >此命令會重新啟動 Azure VM。 您將會在重新啟動的過程中失去您的 RDP 連線。
    
3. 在 Azure VM 重新啟動之後，請使用 RDP 重新連線到您的 VM。

## <a name="set-up-internet-connectivity-for-the-guest-virtual-machine"></a>設定客體虛擬機器的網際網路連線
建立客體虛擬機器的新虛擬網路介面卡，並設定 NAT 閘道，以啟用網際網路連線。

### <a name="create-a-nat-virtual-network-switch"></a>建立 NAT 虛擬網路交換器

1. 在 Azure VM 上，以系統管理員身分開啟 PowerShell。
   
2. 建立內部交換器。

    ```powershell
    New-VMSwitch -Name "InternalNATSwitch" -SwitchType Internal
    ```

3. 檢視交換器的屬性，並記下新介面卡的 ifIndex。

    ```powershell
    Get-NetAdapter
    ```

    ![NetAdapter](./media/virtual-machines-nested-virtualization/get-netadapter.png)

    >[!NOTE] 
    >
    >記下您剛建立之虛擬交換器的 "ifIndex"。
    
4. 建立 NAT 閘道的 IP 位址。
    
若要設定閘道，您會需要一些您網路的相關資訊：    
  * IPAddress：NAT 閘道 IP 會指定 IPv4 或 IPv6 位址，當作虛擬網路子網路的預設閘道位址使用。 一般形式是 a.b.c.1 (例如，"192.168.0.1")。 雖然最後一個位置不一定要是 .1，但通常是如此 (根據首碼長度)。 通常您應該使用 RFC 1918 私人網路位址空間。 
  * PrefixLength：子網路首碼長度會定義本機子網路大小 (子網路遮罩)。 子網路首碼長度將是一個介於 0 到 32 之間的整數值。 0 會對應整個網際網路，32 則只允許一個對應的 IP。 常見的值範圍從 24 到 12，視必須附加至 NAT 的 IP 數量而定。 常見的 PrefixLength 是 24，表示子網路遮罩為 255.255.255.0。
  * InterfaceIndex：**ifIndex** 是上一個步驟中建立之虛擬交換器的介面索引。 

    ```powershell
    New-NetIPAddress -IPAddress 192.168.0.1 -PrefixLength 24 -InterfaceIndex 13
    ```

### <a name="create-the-nat-network"></a>建立 NAT 網路

若要設定閘道，您必須提供網路和 NAT 閘道的相關資訊：
  * 名稱︰這是 NAT 網路的名稱。 
  * InternalIPInterfaceAddressPrefix：NAT 子網路首碼說明上述的 NAT 閘道 IP 首碼以及上述的 NAT 子網路首碼長度。 一般形式將是 a.b.c.0/NAT 子網路首碼長度。 

在 PowerShell 中，建立新的 NAT 網路。
```powershell
New-NetNat -Name "InternalNat" -InternalIPInterfaceAddressPrefix 192.168.0.0/24
```


## <a name="create-the-guest-virtual-machine"></a>建立客體虛擬機器

1. 開啟 Hyper-V 管理員，然後建立新的虛擬機器。 將虛擬機器設定為使用您建立的新內部網路。
    
    ![NetworkConfig](./media/virtual-machines-nested-virtualization/configure-networking.png)
    
2. 在客體虛擬機器上安裝作業系統。
    
    >[!NOTE] 
    >
    >您需要作業系統的安裝媒體，才能在 VM 上安裝。 在此案例中，我們會使用 Windows 10 企業版。

## <a name="assign-an-ip-address-to-the-guest-virtual-machine"></a>將 IP 位址指派給客體虛擬機器

您可以在客體虛擬機器上手動設定靜態 IP 位址，或在 Azure VM 上設定 DHCP 以動態方式指派 IP 位址，藉此將 IP 位址指派給客體虛擬機器。

###  <a name="option-1-configure-dhcp-to-dynamically-assign-an-ip-address-to-the-guest-virtual-machine"></a>選項 1：設定 DHCP，將 IP 位址以動態方式指派給客體虛擬機器
請按照下列步驟，在主機虛擬機器上設定 DHCP，以便進行動態位址指派。

#### <a name="install-dchp-server-on-the-azure-vm"></a>在 Azure VM 上安裝 DHCP 伺服器

1. 開啟 [伺服器管理員]。 在 [儀表板] 上，按一下 [新增角色及功能]。 [新增角色及功能精靈] 隨即出現。
  
2. 在精靈中，按一下 [下一步]，直到到達 [伺服器角色] 頁面為止。
  
3. 按一下以選取 [DHCP 伺服器] 核取方塊、按一下 [新增功能]，然後按一下 [下一步]，直到完成精靈為止。
  
4. 按一下 [Install] 。

#### <a name="configure-a-new-dhcp-scope"></a>設定新的 DHCP 範圍

1. 開啟 DHCP 管理員。
  
2. 在瀏覽窗格中，展開伺服器名稱，以滑鼠右鍵按一下 [IPv4]，然後按一下 [新增範圍]。 當 [新增範圍精靈] 出現時，按 [下一步]。
  
3. 輸入範圍的 [名稱] 和 [說明]，然後按 [下一步]。
  
4. 定義 DHCP 伺服器的 IP 範圍 (例如，192.168.0.100 至 192.168.0.200)。
  
5. 按 [下一步]，直到到達 [預設閘道] 頁面為止。 輸入您稍早建立的 IP 位址 (例如 192.168.0.1) 作為預設閘道。
  
6. 按 [下一步]，直到精靈完成為止，並保留所有預設值，然後按一下 [完成]。
    
### <a name="option-2-manually-set-a-static-ip-address-on-the-guest-virtual-machine"></a>選項 2：在客體虛擬機器上手動設定靜態 IP 位址
如果您未設定 DHCP 以動態方式指派 IP 位址給客體虛擬機器，請按照這些步驟來設定靜態 IP 位址。

1. 在 Azure VM 上，以系統管理員身分開啟 PowerShell。

2. 以滑鼠右鍵按一下客體虛擬機器，然後按一下 [連線]。

3. 登入客體虛擬機器。

4. 在客體虛擬機器上，開啟 [網路和共用中心]。

5. 將網路介面卡的位址設定為您在上一節中建立的 NAT 網路範圍內。

在此範例中，您將使用 192.168.0.0/24 範圍內的位址。

## <a name="test-connectivity-in-guest-virtual-machine"></a>在客體虛擬機器中測試連線

在客體虛擬機器中，開啟瀏覽器，然後瀏覽至網頁。
    ![GuestVM](./media/virtual-machines-nested-virtualization/guest-virtual-machine.png)
