---
title: 診斷網路之間的通訊問題 - 教學課程 - Azure 入口網站 | Microsoft Docs
description: 了解如何使用網路監看員的 VPN 診斷功能，針對透過 Azure 虛擬網路閘道連線到內部部署或其他虛擬網路的 Azure 虛擬網路，診斷其間的通訊問題。
services: network-watcher
documentationcenter: na
author: jimdial
manager: jeconnoc
editor: ''
Customer intent: I need to determine why resources in a virtual network can't communicate with resources in a different network.
ms.service: network-watcher
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 04/27/2018
ms.author: jdial
ms.custom: mvc
ms.openlocfilehash: d89c5a3f2545edd7c02b67fa9d2e2b78937a9791
ms.sourcegitcommit: ca05dd10784c0651da12c4d58fb9ad40fdcd9b10
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/03/2018
ms.locfileid: "32779519"
---
# <a name="tutorial-diagnose-a-communication-problem-between-networks-using-the-azure-portal"></a>教學課程：使用 Azure 入口網站診斷網路之間的通訊問題

虛擬網路閘道會將 Azure 虛擬網路連線到內部部署或其他虛擬網路。 在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 使用網路監看員的 VPN 診斷功能，診斷虛擬網路閘道的相關問題
> * 診斷閘道連線的相關問題
> * 解決閘道的相關問題

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

## <a name="prerequisites"></a>先決條件

若要使用 VPN 診斷，您必須具有現有且正在執行中的 VPN 閘道。 如果您沒有現有 VPN 閘到可以診斷，可以使用 [PowerShell 指令碼](../vpn-gateway/scripts/vpn-gateway-sample-site-to-site-powershell.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json)部署一個閘道。 您可以從下列位置執行 PowerShell 指令碼：
    - **本機 PowerShell 安裝**：指令碼需要 AzureRM PowerShell 模組 5.7.0 版或更新版本。 執行 `Get-Module -ListAvailable AzureRM` 來了解安裝的版本。 如果您需要升級，請參閱[安裝 Azure PowerShell](/powershell/azure/install-azurerm-ps)。 如果您在本機執行 PowerShell，則也需要執行 `Login-AzureRmAccount` 以建立與 Azure 的連線。
    - **Azure Cloud Shell**：[Azure Cloud Shell](https://shell.azure.com/powershell) 已安裝並設定最新版本的 PowerShell，會將您登入到 Azure。

指令碼建立 VPN 閘道大約需要一小時。 剩餘步驟會假設您要診斷由此指令碼部署的閘道。 如果您改為診斷自己的現有閘道，則結果將會不同。

## <a name="sign-in-to-azure"></a>登入 Azure

登入 [Azure 入口網站](https://portal.azure.com)。

## <a name="enable-network-watcher"></a>啟用網路監看員

如果您已經在美國東部區域中啟用網路監看員，請跳至[診斷閘道](#diagnose-a-gateway)。

1. 在入口網站中，選取 [所有服務]。 在 [篩選] 方塊中，輸入*網路監看員*。 當結果中出現**網路監看員**時，請加以選取。
2. 選取 [區域]、展開它，然後選取 [美國東部] 右邊的 [...]，如下圖所示：

    ![啟用網路監看員](./media/diagnose-communication-problem-between-networks/enable-network-watcher.png)

3. 選取 [啟用網路監看員]。

## <a name="diagnose-a-gateway"></a>診斷閘道

1. 在入口網站的左側，選取 [所有服務]。
2. 開始在 [篩選] 方塊中輸入「網路監看員」。 當搜尋結果中出現**網路監看員**時，請加以選取。
3. 在 [網路診斷工具] 底下，選取 [VPN 診斷]。
4. 選取 [儲存體帳戶]，然後選取您想要將診斷資訊寫入至其中的儲存體帳戶。
5. 從 [儲存體帳戶] 清單中，選取您想要使用的儲存體帳戶。 如果您現在沒有儲存體帳戶，請選取 [+ 儲存體帳戶]，輸入或選取必要資訊，然後選取 [建立] 以建立一個儲存體帳戶。 如果您使用[必要條件](#prerequisites)中的指令碼建立 VPN 閘道，您可以在與閘道相同的資源群組 TestRG1 中建立儲存體帳戶。
6. 從 [容器] 清單中，選取您想要使用的容器，然後按一下 [選取]。 如果您沒有任何容器，選取 [+ 容器]，輸入容器的名稱，然後選取 [確定]。
7. 選取閘道，然後選取 [開始進行疑難排解]。 系統會針對名為 **Vnet1GW** 的閘道執行測試，如下圖所示：

    ![VPN 診斷](./media/diagnose-communication-problem-between-networks/vpn-diagnostics.png)

8. 執行測試時，[疑難排解狀態] 資料行會顯示 [執行中]，在上圖中這個資料行是顯示 [未啟動]。 測試執行的時間可能需要幾分鐘。
9. 檢視已完成測試的狀態。 下圖顯示已完成診斷測試的狀態結果：

    ![狀態](./media/diagnose-communication-problem-between-networks/status.png)

    您可以看到 [疑難排解狀態] 是 [狀況不良]，並且會在 [狀態] 索引標籤上看到問題的 [摘要] 和 [詳細資料]。
10. 當您選取 [動作] 索引標籤時，VPN 診斷會提供其他資訊。 在範例中，VPN 診斷會讓您知道應該要檢查每個連線的健康情況，如下圖所示：

  ![動作](./media/diagnose-communication-problem-between-networks/action.png)

## <a name="diagnose-a-gateway-connection"></a>診斷閘道連線

閘道會透過閘道連線與其他網路連線。 閘道和閘道連線都必須狀況良好，才能讓虛擬網路與連線網路之間的通訊成功。

1. 再次完成[診斷閘道](#diagnose-a-gateway)的步驟 7，但是這次選取連線。 在下列範例中，會測試名為 **VNet1toSite1** 的連線：

    ![連線](./media/diagnose-communication-problem-between-networks/connection.png)

    測試會執行數分鐘。
2. 連線測試完成之後，您會收到類似下圖中 [狀態] 和 [動作] 索引標籤上所顯示的結果：

    ![連線狀態](./media/diagnose-communication-problem-between-networks/connection-status.png)

    ![連線動作](./media/diagnose-communication-problem-between-networks/connection-action.png)

    VPN 診斷會在 [狀態] 索引標籤上通知您哪裡發生錯誤，並且在 [動作] 索引標籤上提供您可能造成問題的數個建議項目。

    如果您測試的閘道是由[必要條件](#prerequisites)中的[指令碼](../vpn-gateway/scripts/vpn-gateway-sample-site-to-site-powershell.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json)所部署，則 [狀態] 索引標籤上的問題，以及 [動作] 索引標籤上的前兩個項目，就是確切的問題所在。 指令碼會為內部部署 VPN 閘道裝置設定預留位置 IP 位址 23.99.221.164。

    若要解決問題，您必須確定您的內部部署 VPN 閘道已[正確設定](../vpn-gateway/vpn-gateway-about-vpn-devices.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json)，並且針對本機網路閘道將指令碼設定的 IP 位址，變更為內部部署 VPN 閘道的實際公用位址。

## <a name="clean-up-resources"></a>清除資源

如果您使用[必要條件](#prerequisites)中的指令碼建立 VPN 閘道，只是為了完成本教學課程，而且不再需要它，那麼請刪除資源群組及其中包含的所有資源：

1. 在入口網站頂端的 [搜尋] 方塊中，輸入 TestRG1。 當您在搜尋結果中看到 **TestRG1** 時，請加以選取。
2. 選取 [刪除資源群組]。
3. 針對 [輸入資源群組名稱：] 輸入 TestRG1，然後選取 [刪除]。

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何診斷與虛擬網路閘道相關的問題。 您可以記錄往返虛擬機器的網路通訊，以便檢視記錄中是否有異常。 若要深入了解，請前進到下一個教學課程。

> [!div class="nextstepaction"]
> [記錄往返 VM 的網路流量](network-watcher-nsg-flow-logging-portal.md)