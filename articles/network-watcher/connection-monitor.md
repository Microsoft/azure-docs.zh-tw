---
title: 監視網路通訊 - 教學課程 - Azure 入口網站 | Microsoft Docs
description: 了解如何使用 Azure 網路監看員的連線監視功能，監視兩部虛擬機器之間的網路通訊。
services: network-watcher
documentationcenter: na
author: jimdial
manager: jeconnoc
editor: ''
tags: azure-resource-manager
Customer intent: I need to monitor communication between a VM and another VM. If the communication fails, I need to know why, so that I can resolve the problem.
ms.service: network-watcher
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 04/27/2018
ms.author: jdial
ms.custom: mvc
ms.openlocfilehash: 9b13b8ae0b64dc84e476f5fc5da59ea30702fd8d
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/01/2018
ms.locfileid: "34639022"
---
# <a name="tutorial-monitor-network-communication-between-two-virtual-machines-using-the-azure-portal"></a>教學課程：使用 Azure 入口網站監視兩部虛擬機器之間的網路通訊

虛擬機器 (VM) 與另一個端點 (例如另一部虛擬機器) 之間的成功通訊，對於您的組織很重要。 有時候，系統引入的組態變更會中斷通訊。 在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 建立兩部虛擬機器
> * 使用網路監看員的連線監視功能，監視虛擬機器之間的通訊
> * 診斷兩部虛擬機器之間的通訊問題，並且了解可以如何解決

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

## <a name="sign-in-to-azure"></a>登入 Azure

登入 [Azure 入口網站](https://portal.azure.com)。

## <a name="create-vms"></a>建立 VM

建立兩部虛擬機器。

### <a name="create-the-first-vm"></a>建立第一個 VM

1. 選取 Azure 入口網站左上角的 [+ 建立資源]。
2. 選取 [計算]，然後選取作業系統。 本教學課程使用 **Windows Server 2016 Datacenter**。
3. 輸入或選取下列資訊、接受其餘設定的預設值，然後選取 [確定]：

    |設定|值|
    |---|---|
    |Name|myVm1|
    |使用者名稱| 輸入您選擇的使用者名稱。|
    |密碼| 輸入您選擇的密碼。 密碼長度至少必須有 12 個字元，而且符合[定義的複雜度需求](../virtual-machines/windows/faq.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json#what-are-the-password-requirements-when-creating-a-vm)。|
    |訂用帳戶| 選取您的訂用帳戶。|
    |資源群組| 選取 [新建]，然後輸入 **myResourceGroup**。|
    |位置| 選取 [美國東部]|

4. 選取 VM 的大小，然後選取 [選取]。
5. 在 [設定] 底下，選取 [延伸模組]。 選取 [新增擴充功能]，然後選取 [適用於 Windows 的網路監看員代理程式]，如下圖所示：

    ![網路監看員代理程式擴充功能](./media/connection-monitor/nw-agent-extension.png)

6. 在 [適用於 Windows 的網路監看員代理程式] 底下，選取 [建立]，在 [安裝擴充功能] 底下，選取 [確定]，然後在 [擴充功能] 底下，選取 [確定]。
7. 接受其餘 [設定] 的預設值，然後選取 [確定]。
8. 在 [摘要] 的 [建立] 底下，選取 [建立] 來開始部署 VM。

### <a name="create-the-second-vm"></a>建立第二個 VM

再次完成[建立第一部虛擬機器](#create-the-first-vm)中的步驟，但是使用以下變更：

|步驟|設定|值|
|---|---|---|
| 1 | 選取 [Ubuntu Server 17.10 VM] |                                                                         |
| 3 | Name                              | myVm2                                                                   |
| 3 | 驗證類型               | 貼上您的 SSH 公開金鑰，或選取 [密碼]，然後輸入密碼。 |
| 3 | 資源群組                    | 選取 [使用現有項目]，然後選取 [myResourceGroup]。                 |
| 6 | 擴充功能                        | **適用於 Linux 的網路代理程式**                                             |

部署 VM 需要幾分鐘的時間。 等候虛擬機器完成部署，再繼續進行其餘步驟。

## <a name="create-a-connection-monitor"></a>建立連線監視

建立連線監視，以監視透過 TCP 通訊埠 22 從 myVm1 到 myVm2 的通訊。

1. 在入口網站的左側，選取 [所有服務]。
2. 開始在 [篩選] 方塊中輸入「網路監看員」。 當搜尋結果中出現**網路監看員**時，請加以選取。
3. 在 [監視] 下，選取 [連線監視器]。
4. 選取 [+ 新增] 。
5. 輸入或選取您要監視之連線的資訊，然後選取 [新增]。 在下圖所顯示的範例中，受監視的是透過連接埠 22 從 myVm1 虛擬機器到 myVm2 虛擬機器的連線：

    | 設定                  | 值               |
    | ---------                | ---------           |
    | Name                     | myVm1-myVm2(22)     |
    | 來源                   |                     |
    | 虛擬機器          | myVm1               |
    | 目的地              |                     |
    | 選取虛擬機器 |                     |
    | 虛擬機器          | myVm2               |
    | Port                     | 22                  |

    ![新增連線監視器](./media/connection-monitor/add-connection-monitor.png)

## <a name="view-a-connection-monitor"></a>檢視連線監視

1. 完成[建立連線監視器](#create-a-connection-monitor)中的步驟 1-3，以檢視連線監視。 您會看到現有連線監視的清單，如下圖所示：

    ![連線監視](./media/connection-monitor/connection-monitors.png)

2. 選取名稱為 **myVm1-myVm2(22)** 的監視，如上圖所示，以查看監視的詳細資料，如下圖所示：

    ![監視詳細資料](./media/connection-monitor/vm-monitor.png)

    請注意下列資訊︰

    | Item                     | 值                      | 詳細資料                                                     |
    | ---------                | ---------                  |--------                                                     |
    | 狀態                   | 可連接                  | 讓您知道端點是否可連接。|
    | 平均 來回行程          | 讓您知道連線的來回時間，以毫秒為單位。 連線監視會每隔 60 秒探查連線，所以您可以監視一段時間的延遲。                                         |
    | Hops                     | 連線監視可讓您知道兩個端點之間的躍點。 在此範例中，連線是在相同虛擬網路中的兩部虛擬機器之間進行，因此只有一個躍點通往 10.0.0.5 IP 位址。 如果任何現有的系統或自訂路由會透過 VPN 閘道或網路虛擬設備等，路由傳送虛擬機器之間的流量，則會列出其他躍點。                                                                                                                         |
    | 狀態                   | 每個端點的綠色核取記號可讓您知道每個端點狀況良好。    ||

## <a name="view-a-problem"></a>檢視問題

根據預設，Azure 會允許相同虛擬網路中的虛擬機器透過所有連接埠通訊。 經過一段時間，您或您的組織中的其他人，可能會覆寫 Azure 的預設規則，不小心造成通訊失敗。 完成下列步驟以建立通訊問題，然後再次檢視連線監視：

1. 在入口網站頂端的搜尋方塊中，輸入 myResourceGroup。 當 **myResourceGroup** 資源群組出現在搜尋結果中時，選取它。
2. 選取 **myVm2-nsg** 網路安全性群組。
3. 選取 [輸入安全性規則]，然後選取 [新增]，如下圖所示：

    ![輸入安全性規則](./media/connection-monitor/inbound-security-rules.png)

4. 允許虛擬網路中所有虛擬機器之間通訊的預設規則，是名為 **AllowVnetInBound** 的規則。 建立比 **AllowVnetInBound** 規則優先順序還要高 (數字較小) 的規則，讓其拒絕透過連接埠 22 的輸入通訊。 選取或輸入下列資訊、接受其餘預設值，然後選取 [新增]：

    | 設定                 | 值          |
    | ---                     | ---            |
    | 目的地連接埠範圍 | 22             |
    | 動作                  | 拒絕           |
    | 優先順序                | 100            |
    | Name                    | DenySshInbound |

5. 因為連線監視會每隔 60 秒進行探查，所以請稍候幾分鐘，然後在入口網站左側，依序選取 [網路監看員]、[連線監視]，然後再次選取 [myVm1-myVm2(22)] 監視。 現在結果會不同，如下圖所示：

    ![監視詳細資料錯誤](./media/connection-monitor/vm-monitor-fault.png)

    您可以在 **myvm2529** 網路介面的狀態資料行中看到紅色驚嘆號圖示。

6. 若要深入了解狀態為何會變更，請在上圖中選取 10.0.0.5。 連線監視會通知您通訊失敗的原因是：流量遭到封鎖，原因是下列網路安全性群組規則：UserRule_DenySshInbound。

    如果您不知道誰實作了您在步驟 4 中建立的安全性規則，您可以從連線監視知道是該規則造成通訊問題。 然後，您可以變更、覆寫或移除規則，以還原虛擬機器之間的通訊。

## <a name="clean-up-resources"></a>清除資源

當不再需要資源群組時，請將資源群組及其包含的所有資源刪除：

1. 在入口網站頂端的 [搜尋] 方塊中，輸入 myResourceGroup。 當您在搜尋結果中看到 myResourceGroup 時，請加以選取。
2. 選取 [刪除資源群組]。
3. 針對 [輸入資源群組名稱:] 輸入 myResourceGroup，然後選取 [刪除]。

## <a name="next-steps"></a>後續步驟

在本教學課程中，您將會了解如何監視兩部虛擬機器之間的連線。 您已了解是網路安全性群組規則阻止與虛擬機器的通訊。 若要深入了解連線監視傳回的所有不同回應，請參閱[回應類型](network-watcher-connectivity-overview.md#response)。 您也可以監視虛擬機器、完整網域名稱、統一資源識別項或 IP 位址之間的連線。

有時候，您可能會發現虛擬網路中的資源無法透過 Azure 虛擬網路閘道連線，來與其他網路中的資源通訊。 前進到下一個教學課程，以了解如何診斷與虛擬網路閘道相關的問題。

> [!div class="nextstepaction"]
> [診斷網路間的通訊問題](diagnose-communication-problem-between-networks.md)
