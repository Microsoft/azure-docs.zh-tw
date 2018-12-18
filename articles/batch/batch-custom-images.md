---
title: 從自訂映像佈建 Azure Batch 集區 | Microsoft Docs
description: 您可以從自訂映像建立 Batch 集區，以佈建含有您應用程式所需軟體與資料的計算節點。 自訂映像是設定計算節點以執行 Batch 工作負載的有效方式。
services: batch
author: dlepow
manager: jeconnoc
ms.service: batch
ms.topic: article
ms.date: 04/23/2018
ms.author: danlep
ms.openlocfilehash: 78bc50a1189d8f42281f81643a5e907d94480082
ms.sourcegitcommit: e2adef58c03b0a780173df2d988907b5cb809c82
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/28/2018
ms.locfileid: "32158607"
---
# <a name="use-a-managed-custom-image-to-create-a-pool-of-virtual-machines"></a>使用受控自訂映像來建立虛擬機器的集區 

當您使用虛擬機器設定建立 Azure Batch 集區時，需指定 VM 映像，以提供集區中每個計算節點的作業系統。 您可以使用 Azure Marketplace 映像或自訂映像 (即您自行建立並設定的 VM 映像) 來建立虛擬機器的集區。 自訂映像必須是*受控映像*資源，且位於和 Batch 帳戶相同的 Azure 訂用帳戶和區域中。

## <a name="why-use-a-custom-image"></a>為何要使用自訂映像？
當您提供自訂映像，您可以控制作業系統設定以及要使用的作業系統和資料磁碟類型。 自訂映像可以包含應用程式和參考資料，它們在所有 Batch 集區節點上一經佈建便可使用。

使用自訂映像可以節省讓集區的計算節點可以執行 Batch 工作負載的準備時間。 雖然您可以使用 Azure Marketplace 映像，並且在佈建後在每個節點上安裝軟體，但使用自訂映像可能比較有效率。

使用專為您的案例設定的自訂映像有幾個優點：

- **設定作業系統 (OS)**。 您可以在自訂映像的作業系統磁碟上執行特別的作業系統設定。 
- **預先安裝應用程式。** 您可以建立在作業系統磁碟上預先安裝應用程式的自訂映像，這樣更有效率，而且相較於使用 StartTask 佈建計算節點後安裝應用程式，更不容易發生錯誤。
- **節省 VM 的重新開機時間。** 應用程式安裝通常需要重新啟動 VM，這相當耗時。 預先安裝應用程式可以節省重新開機時間。 
- **一次複製大量資料。** 您可以將靜態資料複製到受控映像的資料磁碟，藉此將靜態資料加入受控自訂映像。 這只需要執行一次，就能將資料提供給集區的每個節點。
- **選擇磁碟類型。** 您可以從 VHD、從 Azure VM 的受控磁碟、從這些磁碟的快照集、或從您自己的已設定 Linux 或 Windows 安裝建立受控自訂映像。 您可以選擇在 OS 磁碟和資料磁碟使用進階儲存體。
- **集區成長為任何大小。** 當您使用受控自訂映像建立集區時，集區可以成長到您要求的任何大小。 您不需要建立映像 blob VHD 的複本來容納這個數目的 VM。 


## <a name="prerequisites"></a>先決條件

- **受控映像資源**。 若要使用自訂映像建立虛擬機器的集區，需要在和 Batch 帳戶相同的 Azure 訂用帳戶和區域中建立受控映像資源。 如需準備受控映像的選項，請參閱下一節。
- **Azure Active Directory (AAD) 驗證**。 Batch 用戶端 API 必須使用 AAD 驗證。 Azure Batch 對於 AAD 的支援記載於[使用 Active Directory 驗證 Batch 服務解決方案](batch-aad-auth.md)中。

    
## <a name="prepare-a-custom-image"></a>準備自訂映像
您可以從 VHD、從包含受控磁碟的 Azure VM、或從 VM 快照集，準備受控映像。 針對批次，建議您從具有受控磁碟的 VM 或 VM 快照來建立受控映像。 受控映像及基礎資源應存在以供集區進行相應增加，並可以在刪除集區之後加以移除。 

準備映像時，請記住下列事項：

* 請確認您用來佈建 Batch 集區的基本 OS 映像沒有任何預先安裝的 Azure 擴充，例如自訂指令碼擴充。 如果映像包含預先安裝的擴充，Azure 在部署 VM 時可能會遇到問題。
* 確定您提供的基本 OS 映像使用預設的暫存磁碟機。 Batch 節點代理程式目前需要有預設的暫存磁碟機。
* Batch 集區所參考的受控映像資源，在集區的存留期內無法刪除。 如果受控映像資源遭到刪除，集區便無法再成長。 

### <a name="to-create-a-managed-image"></a>建立受控映像
您可以使用任何現有已備妥的 Windows 或 Linux 作業系統磁碟來建立受控映像。 例如，如果您需要使用本機映像，請使用 AzCopy 或另一個上傳工具，將本機磁碟上傳到和 Batch 帳戶相同訂用帳戶和區域中的 Azure 儲存體帳戶。 如需上傳 VHD 和建立受控映像的詳細步驟，請參閱 [Windows](../virtual-machines/windows/upload-generalized-managed.md) 或 [Linux](../virtual-machines/linux/upload-vhd.md) VM 的指導方針。

您也可以從新的或現有的 Azure VM 或 VM 快照集準備受控映像。 

* 如果您要建立新的 VM，可以使用 Azure Marketplace 映像作為受控映像的基礎映像，然後加以自訂。 

* 如果您打算使用入口網站擷取映像，請確定 VM 建立時包含受控磁碟。 當您建立 VM 時，這是預設的儲存體設定。

* 一旦 VM 開始執行之後，請透過 RDP (適用於 Windows) 或 SSH (適用於 Linux) 向它連線。 安裝任何必要的軟體或複製所需的資料，然後將 VM 一般化。  

如需 Azure VM 一般化以及建立受控映像的步驟，請參閱 [Windows](../virtual-machines/windows/capture-image-resource.md) 或 [Linux](../virtual-machines/linux/capture-image.md) VM 的指導方針。

依據您計劃如何以映像建立 Batch 集區，您需要下列映像識別碼：

* 如果您打算使用 Batch API 以映像建立集區，則需要映像的**資源識別碼**，其形式為 `/subscriptions/xxxx-xxxxxx-xxxxx-xxxxxx/resourceGroups/myResourceGroup/providers/Microsoft.Compute/images/myImage`。 
* 如果您打算使用入口網站中，則需要映像的**名稱**。 





## <a name="create-a-pool-from-a-custom-image-in-the-portal"></a>在 Azure 入口網站中從自訂映像建立集區

一旦您儲存自訂映像，且知道它的資源識別碼或名稱之後，就可以從該映像建立 Batch 集區。 下列步驟顯示如何從 Azure 入口網站建立集區。

> [!NOTE]
> 如果您要使用其中一個 Batch API 建立集區，請確定您用於 AAD 驗證的身分識別具備映像資源的權限。 請參閱[使用 Active Directory 驗證 Batch 服務解決方案](batch-aad-auth.md)。
>

1. 在 Azure 入口網站中瀏覽至您的 Batch 帳戶。 此帳戶必須與包含自訂映像的資源群組位於相同的訂用帳戶和區域中。 
2. 在左側的 [設定] 視窗中，選取 [集區] 功能表項目。
3. 在 [集區] 視窗中，選取 [新增] 命令。
4. 在 [新增集區] 視窗上，從 [映像類型] 下拉式清單選取 [自訂映像 (Linux/Windows)]。 從 [自訂 VM 映像] 下拉式清單中，選取映像名稱 (資源識別碼的簡短形式)。
5. 依據您的自訂映像選取正確的**發行者/提供項目/SKU**。
6. 指定其餘的必要設定，包括**節點大小**、**目標專用的節點**和**低優先權的節點**，以及所需的任何選擇性設定。

    例如，針對 Microsoft Windows Server Datacenter 2016 自訂映像，[新增集區] 視窗隨即顯示，如下所示：

    ![從自訂 Windows 映像新增集區](media/batch-custom-images/add-pool-custom-image.png)
  
若要檢查現有的集區是否以自訂映像作為基礎，請參閱 [集區] 視窗的資源摘要區段中的 **Operating System** 屬性。 如果已從自訂映像建立集區，它就會設定為**自訂 VM 映像**。

與集區相關聯的所有自訂映像都會顯示在集區的 [屬性] 視窗中。
 
## <a name="next-steps"></a>後續步驟

- 如需 Batch 的深入概觀，請參閱[使用 Batch 開發大規模的平行計算解決方案](batch-api-basics.md)。
