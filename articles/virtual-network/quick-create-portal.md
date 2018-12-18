---
title: 建立虛擬網路 -快速入門 - Azure 入口網站 | Microsoft Docs
description: 在此快速入門中，您將了解如何使用 Azure 入口網站建立虛擬網路。 虛擬網路可讓 Azure 資源 (例如虛擬機器) 彼此及與網際網路進行私密通訊。
services: virtual-network
documentationcenter: virtual-network
author: jimdial
manager: jeconnoc
editor: ''
tags: azure-resource-manager
Customer intent: I want to create a virtual network so that virtual machines can communicate with privately with each other and with the internet.
ms.assetid: ''
ms.service: virtual-network
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: virtual-network
ms.workload: infrastructure
ms.date: 03/09/2018
ms.author: jdial
ms.custom: mvc
ms.openlocfilehash: 7107dc72686004141d8bea0083089cba065a9f4c
ms.sourcegitcommit: 6fcd9e220b9cd4cb2d4365de0299bf48fbb18c17
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/05/2018
---
# <a name="quickstart-create-a-virtual-network-using-the-azure-portal"></a>快速入門：使用 Azure 入口網站建立虛擬網路

虛擬網路可讓 Azure 資源 (例如虛擬機器 (VM)) 彼此及與網際網路進行私密通訊。 在此快速入門中，您將了解如何建立虛擬網路。 建立虛擬網路之後，您需將兩個 VM 部署至虛擬網路中。 接著，您需從網際網路連線至其中一個 VM，然後在兩個 VM 之間進行私密通訊。

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

## <a name="log-in-to-azure"></a>登入 Azure 

在 https://portal.azure.com 上登入 Azure 入口網站。

## <a name="create-a-virtual-network"></a>建立虛擬網路

1. 選取 Azure 入口網站左上角的 [+ 建立資源]。
2. 選取 [網絡]，然後選取 [虛擬網路]。
3. 輸入或選取下列資訊、接受其餘設定的預設值，然後選取 [建立]：

    |設定|值|
    |---|---|
    |Name|myVirtualNetwork|
    |訂用帳戶| 選取您的訂用帳戶。|
    |資源群組| 選取 [新建]，然後輸入 *myResourceGroup*。|
    |位置| 選取 [美國東部]。|

    ![輸入虛擬網路的相關基本資訊](./media/quick-create-portal/create-virtual-network.png)

## <a name="create-virtual-machines"></a>建立虛擬機器

在虛擬網路內建立兩個 VM：

### <a name="create-the-first-vm"></a>建立第一個 VM

1. 選取 Azure 入口網站左上角的 [+ 建立資源]。
2. 選取 [計算]，然後選取 [Windows Server 2016 Datacenter]。
3. 輸入或選取下列資訊、接受其餘設定的預設值，然後選取 [確定]：

    |設定|值|
    |---|---|
    |Name|myVm1|
    |使用者名稱| 輸入您選擇的使用者名稱。|
    |密碼| 輸入您選擇的密碼。 密碼長度至少必須有 12 個字元，而且符合[定義的複雜度需求](../virtual-machines/windows/faq.md?toc=%2fazure%2fvirtual-network%2ftoc.json#what-are-the-password-requirements-when-creating-a-vm)。|
    |訂用帳戶| 選取您的訂用帳戶。|
    |資源群組| 選取 [使用現有項目]，然後選取 [myResourceGroup]。|
    |位置| 選取 [美國東部]|

    ![虛擬機器基本資訊](./media/quick-create-portal/virtual-machine-basics.png)

4. 選取 VM 的大小，然後選取 [選取]。
5. 在 [設定] 底下，接受所有預設值，然後選取 [確定]。

    ![虛擬機器設定](./media/quick-create-portal/virtual-machine-settings.png)

6. 在 [摘要] 的 [建立] 底下，選取 [建立] 來開始部署 VM。 部署 VM 需要幾分鐘的時間。 

### <a name="create-the-second-vm"></a>建立第二個 VM

再次完成步驟 1-6，但在步驟 3 中，將 VM 命名為 *myVm2*。

## <a name="connect-to-a-vm-from-the-internet"></a>從網際網路連線至 VM

1. 建立 *myVm1* 之後，與其連線。 在 Azure 入口網站頂端，輸入 *myVm1*。 當 **myVm1** 出現在搜尋結果中時，選取它。 選取 [連線]  按鈕。

    ![連接到虛擬機器](./media/quick-create-portal/connect-to-virtual-machine.png)

2. 在選取 [連線] 按鈕之後，將會建立一個「遠端桌面通訊協定」(.rdp) 檔案並下載至您的電腦。  
3. 開啟所下載的 rdp 檔案。 如果出現提示，請選取 [連接]。 輸入您在建立 VM 時指定的使用者名稱和密碼。 您可能需要選取 [其他選擇]，然後選取 [使用不同的帳戶]，以指定您在建立 VM 時輸入的認證。 
4. 選取 [確定] 。
5. 您可能會在登入過程中收到憑證警告。 如果您收到警告，請選取 [是] 或 [繼續] 以繼續進行連線。

## <a name="communicate-between-vms"></a>虛擬機器之間的通訊

1. 從 PowerShell，輸入 `ping myvm2`。 Ping 會失敗，因為 Ping 使用網際網路控制訊息通訊協定 (ICMP)，而 Windows 防火牆依預設並不允許 ICMP 通過。
2. 若要允許 *myVm2* 在稍後的步驟中對 *myVm1* 執行 Ping，請從 PowerShell 輸入下列命令，這會允許連入的 ICMP 通過 Windows 防火牆：

    ```powershell
    New-NetFirewallRule –DisplayName “Allow ICMPv4-In” –Protocol ICMPv4
    ```

3. 關閉對 *myVm1* 的遠端桌面連線。 

4. 再次完成[從網際網路連線至 VM](#connect-to-a-vm-from-the-internet) 中的步驟，但連線至 *myVm2*。 從命令提示字元，輸入 `ping myvm1`。

    您會從 *myVm1* 收到回覆，因為您在先前的步驟中允許 ICMP 通過 *myVm1* VM 上的 Windows 防火牆。

5. 關閉對 *myVm2* 的遠端桌面連線。

## <a name="clean-up-resources"></a>清除資源

當不再需要資源群組時，請將資源群組及其包含的所有資源刪除：

1. 在入口網站頂端的 [搜尋] 方塊中，輸入 myResourceGroup。 當您在搜尋結果中看到 myResourceGroup 時，請加以選取。
2. 選取 [刪除資源群組]。
3. 針對 [輸入資源群組名稱:] 輸入 myResourceGroup，然後選取 [刪除]。

## <a name="next-steps"></a>後續步驟

在此快速入門中，您建立了一個預設的虛擬網路和兩個 VM。 您從網際網路連線至其中一個 VM，然後在該 VM 與另一個 VM 進行私密通訊。 若要深入了解虛擬網路設定，請參閱[管理虛擬網路](manage-virtual-network.md)。

Azure 預設會允許虛擬機器之間進行不受限制的私密通訊，但只允許從網際網路到 Windows VM 的輸入遠端桌面連線。 若要了解如何允許或限制進出 VM 的不同網路通訊類型，請移至[篩選網路流量](tutorial-filter-network-traffic.md)教學課程。