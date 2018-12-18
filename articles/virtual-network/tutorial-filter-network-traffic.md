---
title: 篩選網路流量 - 教學課程 - Azure 入口網站 | Microsoft Docs
description: 在本教學課程中，您將了解如何使用 Azure 入口網站，透過網路安全性群組篩選傳至子網路的網路流量。
services: virtual-network
documentationcenter: virtual-network
author: jimdial
manager: jeconnoc
editor: ''
tags: azure-resource-manager
Customer intent: I want to filter network traffic to virtual machines that perform similar functions, such as web servers.
ms.assetid: ''
ms.service: virtual-network
ms.devlang: ''
ms.topic: tutorial
ms.tgt_pltfrm: virtual-network
ms.workload: infrastructure
ms.date: 06/20/2018
ms.author: jdial
ms.custom: ''
ms.openlocfilehash: a731c1e0617fe0ccf9d571dd2b7d0c2ad107bc9e
ms.sourcegitcommit: d551ddf8d6c0fd3a884c9852bc4443c1a1485899
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/07/2018
ms.locfileid: "37901393"
---
# <a name="tutorial-filter-network-traffic-with-a-network-security-group-using-the-azure-portal"></a>教學課程：使用 Azure 入口網站透過網路安全性群組篩選網路流量

您可以透過網路安全性群組篩選輸入虛擬網路子網路和從中輸出的網路流量。 網路安全性群組包含可依 IP 位址、連接埠和通訊協定篩選網路流量的安全性規則。 安全性規則會套用至子網路中部署的資源。 在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 建立網路安全性群組和安全性規則
> * 建立虛擬網路，並將網路安全性群組與子網路產生關聯
> * 將虛擬機器 (VM) 部署至子網路中
> * 測試流量篩選

您可以依偏好使用 [Azure CLI](tutorial-filter-network-traffic-cli.md) 或 [PowerShell](tutorial-filter-network-traffic-powershell.md) 完成本教學課程。

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

## <a name="log-in-to-azure"></a>登入 Azure

在 https://portal.azure.com 上登入 Azure 入口網站。

## <a name="create-a-virtual-network"></a>建立虛擬網路

1. 選取 Azure 入口網站左上角的 [+ 建立資源]。
2. 選取 [網絡]，然後選取 [虛擬網路]。
3. 輸入或選取下列資訊、接受其餘設定的預設值，然後選取 [建立]：

    | 設定                 | 值                                              |
    | ---                     | ---                                                |
    | Name                    | myVirtualNetwork                                   |
    | 位址空間           | 10.0.0.0/16                                        |
    | 訂用帳戶            | 選取您的訂用帳戶。                          |
    | 資源群組          | 選取 [新建]，然後輸入 *myResourceGroup*。 |
    | 位置                | 選取 [美國東部]。                                |
    | 子網路 - 名稱            | mySubnet                                           |
    | 子網路 - 位址範圍  | 10.0.0.0/24                                        |

## <a name="create-application-security-groups"></a>建立應用程式安全性群組

應用程式安全性群組可讓您將具有類似功能 (例如 web 伺服器) 的伺服器群組在一起。

1. 選取 Azure 入口網站左上角的 [+ 建立資源]。
2. 在 [搜尋 Marketplace] 方塊中，輸入「應用程式安全性群組」。 當搜尋結果中出現 [應用程式安全性群組] 時，請加以選取，在 [所有項目] 下再次選取 [應用程式安全性群組]，然後選取 [建立]。
3. 輸入或選取下列資訊，然後選取 [建立]︰

    | 設定        | 值                                                         |
    | ---            | ---                                                           |
    | Name           | myAsgWebServers                                               |
    | 訂用帳戶   | 選取您的訂用帳戶。                                     |
    | 資源群組 | 選取 [使用現有的]，然後選取 [myResourceGroup]。 |
    | 位置       | 美國東部                                                       |

4. 再次完成步驟 3，並指定下列值：

    | 設定        | 值                                                         |
    | ---            | ---                                                           |
    | Name           | myAsgMgmtServers                                              |
    | 訂用帳戶   | 選取您的訂用帳戶。                                     |
    | 資源群組 | 選取 [使用現有的]，然後選取 [myResourceGroup]。 |
    | 位置       | 美國東部                                                       |

## <a name="create-a-network-security-group"></a>建立網路安全性群組

1. 選取 Azure 入口網站左上角的 [+ 建立資源]。
2. 選擇 [網路]，然後選取 [網路安全性群組]。
3. 輸入或選取下列資訊，然後選取 [建立]︰

    |設定|值|
    |---|---|
    |Name|myNsg|
    |訂用帳戶| 選取您的訂用帳戶。|
    |資源群組 | 選取 [使用現有的]，然後選取 [myResourceGroup]。|
    |位置|美國東部|

## <a name="associate-network-security-group-to-subnet"></a>將網路安全性群組關聯至子網路

1. 在入口網站頂端的 [搜尋資源、服務和文件] 方塊中，開始鍵入 myNsg。 當搜尋結果中出現 **myNsg** 時，請加以選取。
2. 在 [設定] 底下選取 [子網路]，然後選取 [+ 關聯]，如下圖所示：

    ![將 NSG 關聯至子網路](./media/tutorial-filter-network-traffic/associate-nsg-subnet.png)

3. 在 [建立子網路關聯] 底下，選取 [虛擬網路]，然後選取 [myVirtualNetwork]。 選取 [子網路]、選取 [mySubnet]，然後選取 [確定]。

## <a name="create-security-rules"></a>建立安全性規則

1. 在 [設定] 下，選取 [輸入安全性規則]，然後選取 [+ 新增]，如下圖所示：

    ![新增 [輸入安全性規則]](./media/tutorial-filter-network-traffic/add-inbound-rule.png)

2. 建立安全性規則，允許連接埠 80 和 443 連至 **myAsgWebServers** 應用程式安全性群組。 在 [新增輸入安全性規則] 下，輸入或選取下列值、接受其餘預設值，然後選取 [新增]：

    | 設定                 | 值                                                                                                           |
    | ---------               | ---------                                                                                                       |
    | 目的地             | 選取 [應用程式安全性群組]，然後針對 [應用程式安全性群組] 選取 [myAsgWebServers]。  |
    | 目的地連接埠範圍 | 輸入 80,443                                                                                                    |
    | 通訊協定                | 選取 TCP                                                                                                      |
    | Name                    | Allow-Web-All                                                                                                   |

3. 使用下列值再次完成步驟 2：

    | 設定                 | 值                                                                                                           |
    | ---------               | ---------                                                                                                       |
    | 目的地             | 選取 [應用程式安全性群組]，然後針對 [應用程式安全性群組] 選取 [myAsgMgmtServers]。 |
    | 目的地連接埠範圍 | 輸入 3389                                                                                                      |
    | 通訊協定                | 選取 TCP                                                                                                      |
    | 優先順序                | 輸入 110                                                                                                       |
    | Name                    | Allow-RDP-All                                                                                                   |

    在本教學課程中，RDP (連接埠 3389) 會公開至 VM 的網際網路，且這個 VM 會指派給 myAsgMgmtServers 應用程式安全性群組。 在生產環境中則不應將連接埠 3389 公開至網際網路，而是建議您使用 VPN 或私人網路連線連接到您想要管理的 Azure 資源。

一旦您完成步驟 1-3 後，請檢閱您所建立的規則。 您的清單看起來應如下圖中所示的清單︰

![安全性規則](./media/tutorial-filter-network-traffic/security-rules.png)

## <a name="create-virtual-machines"></a>建立虛擬機器

在虛擬網路內建立兩個 VM。

### <a name="create-the-first-vm"></a>建立第一個 VM

1. 選取 Azure 入口網站左上角的 [+ 建立資源]。
2. 選取 [計算]，然後選取 [Windows Server 2016 Datacenter]。
3. 輸入或選取下列資訊、接受其餘設定的預設值，然後選取 [確定]：

    |設定|值|
    |---|---|
    |Name|myVmWeb|
    |使用者名稱| 輸入您選擇的使用者名稱。|
    |密碼| 輸入您選擇的密碼。 密碼長度至少必須有 12 個字元，而且符合[定義的複雜度需求](../virtual-machines/windows/faq.md?toc=%2fazure%2fvirtual-network%2ftoc.json#what-are-the-password-requirements-when-creating-a-vm)。|
    |訂用帳戶| 選取您的訂用帳戶。|
    |資源群組| 選取 [使用現有項目]，然後選取 [myResourceGroup]。|
    |位置| 選取 [美國東部]|

4. 選取 VM 的大小，然後選取 [選取]。
5. 在 [設定] 下，選取下列值、接受其餘預設值，然後選取 [確定]：

    |設定|值|
    |---|---|
    |虛擬網路 |選取 **myVirtualNetwork**|
    |網路安全性群組 | 選取 [進階]。|
    |網路安全性群組 (防火牆)| 選取 [(新建) myVmWeb-nsg]，然後在 [選擇網路安全性群組] 下，選取 [無]。 |

6. 在 [摘要] 的 [建立] 底下，選取 [建立] 來開始部署 VM。

### <a name="create-the-second-vm"></a>建立第二個 VM

再次完成步驟 1-6，但在步驟 3 中，將 VM 命名為 myVmMgmt。 部署 VM 需要幾分鐘的時間。 請勿繼續下一個步驟，直到部署 VM 為止。

## <a name="associate-network-interfaces-to-an-asg"></a>將網路介面關聯至 ASG

當入口網站建立 VM 時，會建立每個 VM 的網路介面，並將網路介面連結至 VM。 請將每個 VM 的網路介面，新增至您先前建立的其中一個應用程式安全性群組：

1. 在入口網站頂端的「搜尋資源、服務和文件」方塊中，開始輸入 myVmWeb。 當搜尋結果中出現 **myVmWeb** VM 時，請加以選取。
2. 在 [設定] 底下，選取 [網路]。  選取 [設定應用程式安全性群組]，針對 [應用程式安全性群組] 選取 myAsgWebServers，然後選取 [儲存]，如下圖所示：

    ![關聯至 ASG](./media/tutorial-filter-network-traffic/associate-to-asg.png)

3. 再次完成步驟 1 和 2、搜尋 **myVmMgmt** VM，然後選取 **myAsgMgmtServers** ASG。

## <a name="test-traffic-filters"></a>測試流量篩選

1. 連線至 myVmMgmt VM。 在入口網站頂端的搜尋方塊中，輸入 myVmMgmt。 當 **myVmMgmt** 出現在搜尋結果中時，請加以選取。 選取 [連線]  按鈕。
2. 選取 [下載 RDP 檔案]。
3. 開啟所下載的 RDP 檔案，然後選取 [連線]。 輸入您在建立 VM 時指定的使用者名稱和密碼。 您可能需要選取 [其他選擇]，然後選取 [使用不同的帳戶]，以指定您在建立 VM 時輸入的認證。
4. 選取 [確定] 。
5. 您可能會在登入過程中收到憑證警告。 如果您收到警告，請選取 [是] 或 [繼續] 以繼續進行連線。

    由於允許連接埠 3389 從網際網路將流量輸入連結至 myVmMgmt VM 的網路介面所在的 myAsgMgmtServers 應用程式安全性群組，因此連線會成功。

6. 在 PowerShell 工作階段中輸入下列命令，從 myVmMgmt VM 連線到 myVmWeb VM：

    ``` 
    mstsc /v:myVmWeb
    ```

    您可以從 myVmMgmt VM 連線至 myVmWeb VM，因為相同虛擬網路中的 VM 依預設可以透過任何連接埠彼此通訊。 不過，您無法建立從網際網路到 myVmWeb VM 的遠端桌面連線，因為 myAsgWebServers 的安全性規則不允許從網際網路的連接埠 3389 輸入，且依預設會拒絕從網際網路傳至所有資源的輸入流量。

7. 若要在 myVmWeb VM 上安裝 Microsoft IIS，請從 myVmWeb VM 的 PowerShell 工作階段輸入下列命令：

    ```powershell
    Install-WindowsFeature -name Web-Server -IncludeManagementTools
    ```

8. 完成 IIS 安裝之後，請將 myVmWeb VM 中斷連線，以留在 myVmMgmt VM 遠端桌面連線中。
9. 從 myVmMgmt VM 中斷連線。
10. 在 Azure 入口網站頂端的「搜尋資源、服務和文件」方塊中，開始從您的電腦輸入 myVmWeb。 當 **myVmWeb** 出現在搜尋結果中時，選取它。 請記下您 VM 的**公用 IP 位址**。 下圖中所示的位址是 137.135.84.74，但您的地址並不同：

    ![公用 IP 位址](./media/tutorial-filter-network-traffic/public-ip-address.png)
  
11. 若要確認您可以從網際網路存取 myVmWeb Web 伺服器，請在電腦上開啟網際網路瀏覽器，並瀏覽至 `http://<public-ip-address-from-previous-step>`。 由於允許連接埠 80 從網際網路將流量輸入連結至 myVmWeb VM 的網路介面所在的 myAsgWebServers 應用程式安全性群組，因此您會看到 IIS 歡迎畫面。

## <a name="clean-up-resources"></a>清除資源

當不再需要資源群組時，請將資源群組及其包含的所有資源刪除：

1. 在入口網站頂端的 [搜尋] 方塊中，輸入 myResourceGroup。 當您在搜尋結果中看到 myResourceGroup 時，請加以選取。
2. 選取 [刪除資源群組]。
3. 針對 [輸入資源群組名稱:] 輸入 myResourceGroup，然後選取 [刪除]。

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已建立網路安全性群組，並將其與虛擬網路子網路產生關聯。 若要深入了解網路安全性群組，請參閱[網路安全性群組概觀](security-overview.md)和[管理網路安全性群組](manage-network-security-group.md)。

Azure 依預設會路由傳送子網路之間的流量。 您可以改採其他方式，例如，透過作為防火牆的 VM 路由傳送子網路之間的流量。 若想了解如何建立路由表，請移至下一個教學課程。

> [!div class="nextstepaction"]
> [建立路由表](./tutorial-create-route-table-portal.md)