---
title: 限制對 PaaS 資源的網路存取 -教學課程 - Azure 入口網站 | Microsoft Docs
description: 在本教學課程中，您將了解如何使用 Azure 入口網站透過虛擬網路服務端點來限制對 Azure 資源 (例如 Azure 儲存體和 Azure SQL Database) 的網路存取。
services: virtual-network
documentationcenter: virtual-network
author: jimdial
manager: jeconnoc
editor: ''
tags: azure-resource-manager
Customer intent: I want only resources in a virtual network subnet to access an Azure PaaS resource, such as an Azure Storage account.
ms.assetid: ''
ms.service: virtual-network
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: virtual-network
ms.workload: infrastructure
ms.date: 08/23/2018
ms.author: jdial
ms.openlocfilehash: b951386fbeca883ae61a7f8040893e55467c8e5d
ms.sourcegitcommit: 58c5cd866ade5aac4354ea1fe8705cee2b50ba9f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/24/2018
ms.locfileid: "42810079"
---
# <a name="tutorial-restrict-network-access-to-paas-resources-with-virtual-network-service-endpoints-using-the-azure-portal"></a>教學課程：使用 Azure 入口網站透過虛擬網路服務端點來限制對 PaaS 資源的網路存取

虛擬網路服務端點可讓您限制某些 Azure 服務資源對虛擬網路子網路的網路存取。 您也可以移除對資源的網際網路存取。 服務端點提供從您虛擬網路到受支援 Azure 服務的直接連線，讓您可以使用虛擬網路的私人位址空間來存取 Azure 服務。 透過服務端點預定流向 Azure 資源的流量，一律會留在 Microsoft Azure 骨幹網路中。 在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 建立具有一個子網路的虛擬網路
> * 新增子網路，並啟用服務端點
> * 建立 Azure 資源，並僅允許從子網路對其進行網路存取
> * 將虛擬機器 (VM) 部署到每個子網路
> * 確認從子網路對資源的存取
> * 確認從子網路和網際網路對資源的存取遭到拒絕

您可以依偏好使用 [Azure CLI](tutorial-restrict-network-access-to-resources-cli.md) 或 [Azure PowerShell](tutorial-restrict-network-access-to-resources-powershell.md) 完成本教學課程。

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

## <a name="log-in-to-azure"></a>登入 Azure

在 https://portal.azure.com 上登入 Azure 入口網站。

## <a name="create-a-virtual-network"></a>建立虛擬網路

1. 選取 Azure 入口網站左上角的 [+ 建立資源]。
2. 選取 [網絡]，然後選取 [虛擬網路]。
3. 輸入或選取下列資訊，然後選取 [建立]︰

   |設定|值|
   |----|----|
   |名稱| myVirtualNetwork |
   |位址空間| 10.0.0.0/16|
   |訂用帳戶| 選取您的訂用帳戶|
   |資源群組 | 選取 [新建]，然後輸入 *myResourceGroup*。|
   |位置| 選取 [美國東部] |
   |子網路名稱| 公開|
   |子網路位址範圍| 10.0.0.0/24|
   |服務端點| 已停用|

   ![輸入虛擬網路的相關基本資訊](./media/tutorial-restrict-network-access-to-resources/create-virtual-network.png)

## <a name="enable-a-service-endpoint"></a>啟用服務端點

每項服務、每個子網路都會啟用服務端點。 建立子網路，並啟用子網路的服務端點。

1. 在入口網站頂端的 [搜尋資源、服務和文件] 方塊中，輸入 myVirtualNetwork。 當搜尋結果中出現 **myVirtualNetwork** 時加以選取。
2. 將子網路新增至虛擬網路。 在 [設定] 底下選取 [子網路]，然後選取 [+ 子網路]，如下圖所示：

    ![新增子網路](./media/tutorial-restrict-network-access-to-resources/add-subnet.png) 

3. 在 [新增子網路] 底下選取或輸入下列資訊，然後選取 [確定]︰

    |設定|值|
    |----|----|
    |名稱| 私人 |
    |位址範圍| 10.0.1.0/24|
    |服務端點| 選取 [服務] 底下的 [Microsoft.Storage]|

> [!CAUTION]
> 針對具有資源的現有子網路啟用服務端點之前，請參閱[變更子網路設定](virtual-network-manage-subnet.md#change-subnet-settings)。

## <a name="restrict-network-access-for-a-subnet"></a>限制子網路的網路存取

根據預設，子網路中的所有 VM 可以與所有資源通訊。 建立網路安全性群組，並將它與子網路產生關聯，即可限制與子網路中所有資源之間的通訊。

1. 選取 Azure 入口網站左上角的 [+ 建立資源]。
2. 選擇 [網路]，然後選取 [網路安全性群組]。
3. 在 [建立網路安全性群組] 底下，輸入或選取下列資訊，然後選取 [建立]︰

    |設定|值|
    |----|----|
    |名稱| myNsgPrivate |
    |訂用帳戶| 選取您的訂用帳戶|
    |資源群組 | 選取 [使用現有項目]，然後選取 [myResourceGroup]。|
    |位置| 選取 [美國東部] |

4. 建立網路安全性群組之後，請在入口網站頂端的 [搜尋資源、服務和文件] 方塊中，輸入 myNsgPrivate。 當 **myNsgPrivate** 出現在搜尋結果中時，請加以選取。
5. 在 [設定] 下，選取 [輸出安全性規則]。
6. 選取 [+ 新增] 。
7. 建立一個規則，允許 Azure 儲存體服務的輸出通訊。 輸入或選取下列資訊，然後選取 [新增]︰

    |設定|值|
    |----|----|
    |來源| 選取 [VirtualNetwork] |
    |來源連接埠範圍| * |
    |目的地 | 選取 [服務標記]|
    |目的地服務標記 | 選取 [儲存體]|
    |目的地連接埠範圍| * |
    |通訊協定|任意|
    |動作|允許|
    |優先順序|100|
    |名稱|Allow-Storage-All|

8. 建立另一個拒絕網際網路通訊的輸出安全性規則。 此規則會覆寫所有網路安全性群組中允許輸出網際網路通訊的預設規則。 使用下列值，再次完成步驟 5-7：

    |設定|值|
    |----|----|
    |來源| 選取 [VirtualNetwork] |
    |來源連接埠範圍| * |
    |目的地 | 選取 [服務標記]|
    |目的地服務標記| 選取 [網際網路]|
    |目的地連接埠範圍| * |
    |通訊協定|任意|
    |動作|拒絕|
    |優先順序|110|
    |名稱|Deny-Internet-All|

9. 在 [設定] 下，選取 [輸入安全性規則]。
10. 選取 [+ 新增] 。
11. 建立，允許從任一處對子網路輸入遠端桌面通訊協定 (RDP) 流量的輸入安全性規則。 此規則會覆寫拒絕來自網際網路之所有輸入流量的預設安全性規則。 允許與子網路的遠端桌面連線，以便在稍後步驟中可以測試連線。 在 [設定] 下選取 [輸入安全性規則]，然後選取 [+新增]、輸入下列值，再選取 [新增]：

    |設定|值|
    |----|----|
    |來源| 任意 |
    |來源連接埠範圍| * |
    |目的地 | 選取 [VirtualNetwork]|
    |目的地連接埠範圍| 3389 |
    |通訊協定|任意|
    |動作|允許|
    |優先順序|120|
    |名稱|Allow-RDP-All|

12. 在 [設定] 底下，選取 [子網路]。
13. 選取 [+ 關聯]
14. 在 [建立子網路關聯] 底下，選取 [虛擬網路]，然後在 [選擇虛擬網路] 底下選取 [myVirtualNetwork]。
15. 在 [選取子網路] 下，選取 [私人]，然後選取 [確定]。

## <a name="restrict-network-access-to-a-resource"></a>限制對資源的網路存取

如果資源是透過已針對服務端點啟用的 Azure 服務建立，則限制其網路存取的必要步驟會因為服務不同而有所差異。 請參閱個別服務的文件，以了解每個服務的特定步驟。 本教學課程的其餘部分包含對 Azure 儲存體帳戶的網路存取進行限制的步驟 (以範例形式說明)。

### <a name="create-a-storage-account"></a>建立儲存體帳戶

1. 選取 Azure 入口網站左上角的 [+ 建立資源]。
2. 選取 [儲存體]，然後選取 [儲存體帳戶 - blob、檔案、資料表、佇列]。
3. 輸入或選取下列資訊、接受其餘預設值，然後選取 [建立]：

    |設定|值|
    |----|----|
    |名稱| 請輸入在所有 Azure 位置間具有唯一性、長度介於 3-24 個字元，且僅使用數字和小寫字母的名稱。|
    |帳戶類型|StorageV2 (一般用途 v2)|
    |位置| 選取 [美國東部] |
    |複寫| 本地備援儲存體 (LRS)|
    |訂用帳戶| 選取您的訂用帳戶|
    |資源群組 | 選取 [使用現有項目]，然後選取 [myResourceGroup]。|

### <a name="create-a-file-share-in-the-storage-account"></a>在儲存體帳戶中建立檔案共用

1. 建立儲存體帳戶之後，請在入口網站頂端的 [搜尋資源、服務和文件] 方塊中，輸入儲存體帳戶的名稱。 當您儲存體帳戶的名稱出現在搜尋結果時，請選取它。
2. 選取 [檔案]，如下圖所示：

   ![儲存體帳戶](./media/tutorial-restrict-network-access-to-resources/storage-account.png) 

3. 選取 [+ 檔案共用]。
4. 在 [名稱] 下，輸入 my-file-share，然後選取 [確定]。
5. 關閉 [檔案服務] 方塊。

### <a name="restrict-network-access-to-a-subnet"></a>限制對子網路的網路存取

根據預設，儲存體帳戶會接受來自任何網路用戶端的網路連線，包括網際網路。 拒絕來自網際網路以及所有虛擬網路中所有其他子網路的網路存取，myVirtualNetwork 虛擬網路中的「私人」子網路除外。

1. 在儲存體帳戶的 [設定] 下，選取 [防火牆和虛擬網路]。
2. 選取 [選取的網路]。
3. 選取 [+新增現有的虛擬網路]。
4. 在 [新增網路] 下，選取下列值，並選取 [新增]：

    |設定|值|
    |----|----|
    |訂用帳戶| 選取您的訂用帳戶。|
    |虛擬網路|在 [虛擬網路] 下，選取 [myVirtualNetwork]|
    |子網路| 在 [子網路] 下，選取 [私人]|

    ![防火牆與虛擬網路](./media/tutorial-restrict-network-access-to-resources/storage-firewalls-and-virtual-networks.png)

5. 選取 [ **儲存**]。
6. 關閉 [防火牆和虛擬網路] 方塊。
7. 在儲存體帳戶的 [設定] 下，選取 [存取金鑰]，如下圖所示：

      ![防火牆與虛擬網路](./media/tutorial-restrict-network-access-to-resources/storage-access-key.png)

8. 將 [金鑰] 值記下，因為當您在稍後步驟中將檔案共用對應至 VM 中的磁碟機代號時，必須以手動方式將它輸入。

## <a name="create-virtual-machines"></a>建立虛擬機器

若要測試對儲存體帳戶的網路存取，請將 VM 部署至每個子網路。

### <a name="create-the-first-virtual-machine"></a>建立第一部虛擬機器

1. 選取 Azure 入口網站左上角的 [+ 建立資源]。
2. 選取 [計算]，然後選取 [Windows Server 2016 Datacenter]。
3. 輸入或選取下列資訊，然後選取 [確定]︰

   |設定|值|
   |----|----|
   |名稱| myVmPublic|
   |使用者名稱|輸入您選擇的使用者名稱。|
   |密碼| 輸入您選擇的密碼。 密碼長度至少必須有 12 個字元，而且符合[定義的複雜度需求](../virtual-machines/windows/faq.md?toc=%2fazure%2fvirtual-network%2ftoc.json#what-are-the-password-requirements-when-creating-a-vm)。|
   |訂用帳戶| 選取您的訂用帳戶。|
   |資源群組| 選取 [使用現有項目]，然後選取 [myResourceGroup]。|
   |位置| 選取 [美國東部]。|

   ![輸入虛擬機器的相關基本資訊](./media/tutorial-restrict-network-access-to-resources/virtual-machine-basics.png)
4. 選取虛擬機器的大小，然後選取 [選取]。
5. 在 [設定] 下，選取 [網路]，然後選取 [myVirtualNetwork]。 然後選取 [子網路]，並選取 [公用]，如下圖所示：

   ![選取虛擬網路](./media/tutorial-restrict-network-access-to-resources/virtual-machine-settings.png)

6. 在 [網路安全性群組] 下方，選取 [進階]。 入口網站會自動為您建立允許連接埠 3389 的網路安全性群組；您必須開啟此連接埠，才能在後續的步驟中連線至虛擬機器。 在 [設定] 頁面上選取 [確定]。
7. 在 [摘要] 頁面上，選取 [建立] 來開始進行虛擬機器部署。 VM 需要幾分鐘的時間進行部署，但您在 VM 建立時可以繼續下一個步驟。

### <a name="create-the-second-virtual-machine"></a>建立第二部虛擬機器

再次完成步驟 1-7，但在步驟 3 中，請將虛擬機器命名為 myVmPrivate，並在步驟 5 中選取 [私人] 子網路。

部署 VM 需要幾分鐘的時間。 請等到 VM 建立完成，並在入口網站開啟設定後，再繼續下一個步驟。

## <a name="confirm-access-to-storage-account"></a>確認對儲存體帳戶的存取

1. 一旦 myVmPrivate VM 完成建立後，Azure 會開啟它的設定。 選取 [連線] 按鈕以連線到 VM，如下圖所示：

   ![連接到虛擬機器](./media/tutorial-restrict-network-access-to-resources/connect-to-virtual-machine.png)

2. 在選取 [連線] 按鈕之後，將會建立一個「遠端桌面通訊協定」(.rdp) 檔案並下載至您的電腦。  
3. 開啟所下載的 rdp 檔案。 如果出現提示，請選取 [連接]。 輸入您在建立 VM 時指定的使用者名稱和密碼。 您可能需要選取 [其他選擇]，然後選取 [使用不同的帳戶]，以指定您在建立 VM 時輸入的認證。 
4. 選取 [確定] 。
5. 您可能會在登入過程中收到憑證警告。 如果您收到警告，請選取 [是] 或 [繼續] 以繼續進行連線。
6. 在 myVmPrivate VM 上，使用 PowerShell 將 Azure 檔案共用對應至磁碟機 Z。 在執行後續命令之前，請將 `<storage-account-key>` 和 `<storage-account-name>` 取代為您在[建立儲存體帳戶](#create-a-storage-account)中提供或取出的值。

   ```powershell
   $acctKey = ConvertTo-SecureString -String "<storage-account-key>" -AsPlainText -Force
   $credential = New-Object System.Management.Automation.PSCredential -ArgumentList "Azure\<storage-account-name>", $acctKey
   New-PSDrive -Name Z -PSProvider FileSystem -Root "\\<storage-account-name>.file.core.windows.net\my-file-share" -Credential $credential
   ```

   PowerShell 會傳回類似以下範例輸出的輸出：

   ```powershell
   Name           Used (GB)     Free (GB) Provider      Root
   ----           ---------     --------- --------      ----
   Z                                      FileSystem    \\vnt.file.core.windows.net\my-f...
   ```

   Azure 檔案共用已成功對應至 Z 磁碟機。

7. 從命令提示字元確認 VM 沒有網際網路的輸出連線：

   ```
   ping bing.com
   ```

   您不會收到回應，因為與「私人」子網路相關聯的網路安全性群組不允許對網際網路進行輸出存取。

8. 關閉 myVmPrivate 虛擬機器的遠端桌面工作階段。

## <a name="confirm-access-is-denied-to-storage-account"></a>確認對儲存體帳戶的存取遭到拒絕

1. 在入口網站頂端的 [搜尋資源、服務和文件] 方塊中，輸入 myVmPublic。
2. 當 **myVmPublic** 出現在搜尋結果中時，請選取它。
3. 在 myVmPublic VM 的 [確認對儲存體帳戶的存取](#confirm-access-to-storage-account) 中，完成步驟 1-6。

   短暫等候之後，您會收到 `New-PSDrive : Access is denied` 錯誤。 存取遭到拒絕，因為 myVmPublic VM 是部署在「公用」子網路中。 「公用」子網路沒有已啟用的服務端點可供 Azure 儲存體使用。 儲存體帳戶只允許「私人」子網路，而不允許「公用」子網路的網路存取。

4. 關閉 myVmPublic VM 的遠端桌面工作階段。

5. 從您的電腦，瀏覽至 Azure [入口網站](https://portal.azure.com)。
6. 在 [搜尋資源、服務和文件] 方塊中，輸入您建立的儲存體帳戶之名稱。 當您儲存體帳戶的名稱出現在搜尋結果時，請選取它。
7. 選取 [檔案]。
8. 您會收到下圖中所示的錯誤：

   ![拒絕存取錯誤](./media/tutorial-restrict-network-access-to-resources/access-denied-error.png)

   存取遭到拒絕，因為您的電腦不在 MyVirtualNetwork 虛擬網路的「私人」子網路中。

## <a name="clean-up-resources"></a>清除資源

當不再需要資源群組時，請將資源群組及其包含的所有資源刪除：

1. 在入口網站頂端的 [搜尋] 方塊中，輸入 myResourceGroup。 當您在搜尋結果中看到 myResourceGroup 時，請加以選取。
2. 選取 [刪除資源群組]。
3. 針對 [輸入資源群組名稱:] 輸入 myResourceGroup，然後選取 [刪除]。

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已啟用虛擬網路子網路的服務端點。 您已了解可以針對從多項 Azure 服務部署的資源啟用服務端點。 您已建立 Azure 儲存體帳戶，並將儲存體帳戶的網路存取限制為只能存取虛擬網路子網路內的資源。 若要深入了解服務端點，請參閱[服務端點概觀](virtual-network-service-endpoints-overview.md)和[管理子網路](virtual-network-manage-subnet.md)。

如果您的帳戶中有多個虛擬網路，您可以同時連線兩個虛擬網路，讓每個虛擬網路內的資源都可互相進行通訊。 若要了解如何連線虛擬網路，請移至下一個教學課程。

> [!div class="nextstepaction"]
> [連線虛擬網路](./tutorial-connect-virtual-networks-portal.md)
