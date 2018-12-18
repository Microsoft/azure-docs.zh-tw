---
title: 使用 Azure 和 Azure Stack 建立暫存資料分析解決方案 | Microsoft Docs
description: 了解如何使用 Azure 和 Azure Stack 建立暫存資料分析解決方案。
services: azure-stack
documentationcenter: ''
author: mattbriggs
manager: femila
editor: ''
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 10/02/2018
ms.author: mabrigg
ms.reviewer: Anjay.Ajodha
ms.openlocfilehash: b4b81546a267e6fd082f83db8b23010f0742771f
ms.sourcegitcommit: 1981c65544e642958917a5ffa2b09d6b7345475d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/03/2018
ms.locfileid: "48237892"
---
# <a name="tutorial-create-a-staged-data-analytics-solution-with-azure-and-azure-stack"></a>教學課程：使用 Azure 和 Azure Stack 建立暫存資料分析解決方案 

*適用於：Azure Stack 整合系統和 Azure Stack 開發套件*

了解如何使用內部部署和公用雲端環境來滿足有許多設備的企業需求。 Azure Stack 提供快速、安全且彈性的解決方案來收集、處理、儲存和散發本機及遠端資料，尤其會用在安全性、機密性、公司政策和法規需求在位置和使用者之間有所差異的時候。

在此模式中，您的客戶將在收集點上收集需要分析的資料，以便快速做出決策。 此資料收集通常會在沒有網際網路存取時進行。 當連線建立時，您可能必須執行需要大量資源的資料分析，以獲取額外的深入解析。 公用雲端太慢或無法使用時，您仍可以分析資料。

在本教學課程中建置範例環境，用以：

> [!div class="checklist"]
> - 建立原始資料儲存體 Blob。
> - 建立新的 Azure Stack 函式，以將乾淨資料從 Azure Stack 移至 Azure。
> - 建立由 Blob 儲存體所觸發的函式。
> - 建立包含 Blob 和佇列的 Azure Stack 儲存體帳戶。
> - 建立由佇列觸發的函式。
> - 測試由佇列觸發的函式。

> [!Tip]  
> ![hybrid-pillars.png](./media/azure-stack-solution-cloud-burst/hybrid-pillars.png)  
> Microsoft Azure Stack 是 Azure 的延伸模組。 Azure Stack 可將雲端運算的靈活性和創新能力導入您的內部部署環境中，並啟用獨特的混合式雲端，讓您能夠隨處建置及部署混合式應用程式。  
> 
> [混合式應用程式的設計考量](https://aka.ms/hybrid-cloud-applications-pillars)技術白皮書檢閱了設計、部署和操作混合式應用程式時的軟體品質要素 (放置、延展性、可用性、復原、管理性和安全性)。 這些設計考量有助於您設計出最佳的混合式應用程式，讓生產環境遇到最少的挑戰。

## <a name="prerequisites"></a>必要條件

建置此解決方案需要一些準備項目：

-   已安裝且正常運作的 Azure Stack (詳細資訊在此：[Azure Stack 概觀](https://docs.microsoft.com/azure/azure-stack/user/azure-stack-storage-overview))

-   Azure 訂用帳戶。 ([建立免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F))

-   下載並安裝 [Microsoft Azure 儲存體總管](http://storageexplorer.com/)。

-   您必須提供您自己的資料，讓函式進行處理。 資料必須是產生的，並可供上傳至 Azure Stack 儲存體 Blob 容器。

## <a name="issues-and-considerations"></a>問題和考量

### <a name="scalability-considerations"></a>延展性考量

Azure 函式及儲存體解決方案會進行縮放，以滿足資料量和處理需求。 如需有關 Azure 延展性的資訊和目標，請參閱 [Azure 延展性文件](https://docs.microsoft.com/azure/storage/common/storage-scalability-targets)。

### <a name="availability-considerations"></a>可用性考量

儲存體是此模式的主要可用性考量。 大量資料的處理和散發都需要透過快速連結的連線。

### <a name="manageability-considerations"></a>管理性考量

請考量您的開發工具和原始檔控制可讓您如何管理解決方案。

## <a name="create-the-raw-data-storage-blob"></a>建立原始資料儲存體 Blob

儲存體帳戶和 Blob 容器會保留所有內部部署活動產生的原始資料，這些活動包括機器和員工活動、設備資料、生產計量及其他報告。

1.  登入 [Azure Stack 入口網站](https://portal.local.azurestack.external/)。

2.  在 Azure Stack 入口網站中，展開左側功能表以開啟服務的功能表，然後選擇 [所有服務]。 向下捲動至 [儲存體]，然後選擇 [儲存體帳戶]。 在 [儲存體帳戶] 視窗中，選擇 [新增]。

3.  請使用下列帳戶資訊：

    a.  名稱：**由您決定**

    b.  部署模型：**Resource Manager**

    c.  帳戶種類：**儲存體 (一般用途 V1)**

    d.  位置：**美國西部**

    e.  複寫：**本地備援儲存體 (LRS)**

    f.  效能：**標準**

    g.  需要安全傳輸：**停用**

    h.  訂用帳戶：選擇一個訂用帳戶

    i.  資源群組：指定新的資源群組，或選取現有的資源群組

    j.  設定虛擬網路：**停用**

4.  選取 [建立] 以建立儲存體帳戶。

    ![替代文字](media\azure-stack-solution-staged-data-analytics\image1.png)

5.  建好後，選取儲存體帳戶的名稱。

6.  在 [帳戶] 刀鋒視窗中的 [BLOB 服務] 標題底下，選取 [容器]。

7.  在刀鋒視窗的頂端，選取 [+ 容器]，然後選取**容器**。

    ![替代文字](media\azure-stack-solution-staged-data-analytics\image2.png)

8.  名稱：**由您決定**

9.  公用存取層級：**容器** (容器和 Blob 的匿名讀取權限)

10.  選取 [確定] 。

## <a name="create-an-azure-stack-function"></a>建立 Azure Stack 函式

建立新的 Azure Stack 函式，以將乾淨資料從 Azure Stack 移至 Azure。

### <a name="create-the-azure-stack-function-app"></a>建立 Azure Stack 函式應用程式

1. 登入 [Azure Stack 入口網站](https://portal.local.azurestack.external)。
2. 選取 [所有服務]。
3. 選取 [Web + 行動] 群組中的 [函式應用程式]。

4.  請使用影像下方資料表中指定的設定建立函式應用程式。

    | 設定 | 建議的值 | 說明 |
    | ---- | ---- | ---- |
    | 應用程式名稱 | 全域唯一的名稱 | 用以識別新函式應用程式的名稱。 有效字元是 `a`-`z`、`0``-9` 和 `-`。 |
    | 訂用帳戶 | 您的訂用帳戶 | 將在其下建立這個新函式應用程式的訂用帳戶。 |
    | **資源群組** |  |  |
    | myResourceGroup | 要在其中建立函式應用程式的新資源群組名稱。 |  |
    | 作業系統 | Windows | 無伺服器裝載目前只能在執行於 Windows 上時使用。 |
    | **主控方案** |  |  |
    | 取用方案 | 會定義如何將資源配置給函式應用程式的主控方案。 在預設取用方案中，您的函式會根據需要來動態新增資源。 在此無伺服器裝載中，您只需要針對函式有執行的時間來付費。 |  |
    | 位置 | 最接近您的區域 | 選擇的區域應靠近您或靠近函式將會存取的其他服務。 |
    | **儲存體帳戶** |  |  |
    | \<上方所建立的儲存體帳戶> | 函式應用程式所使用之新儲存體帳戶的名稱。 儲存體帳戶名稱長度必須介於 3 到 24 個字元之間。 名稱僅能使用數字和小寫字母。 您也可以使用現有帳戶。 |  |

    **範例：**

    ![定義新的函式應用程式設定](media\azure-stack-solution-staged-data-analytics\image6.png)

5.  選取 [建立] 以佈建並部署函式應用程式。

6.  選取入口網站右上角的 [通知] 圖示，查看是否有**部署成功**訊息。

    ![定義新的函式應用程式設定](media\azure-stack-solution-staged-data-analytics\image7.png)

7.  選取 [前往資源]，以檢視新的函式應用程式。

![已成功建立函式應用程式。](media\azure-stack-solution-staged-data-analytics\image8.png)

### <a name="add-a-function-to-the-azure-stack-function-app"></a>將函式新增至 Azure Stack 函式應用程式

1.  按一下 [函式]，然後按一下 [+ 新增函式] 按鈕，即可建立新的函式。

    ![替代文字](media\azure-stack-solution-staged-data-analytics\image3.png)

2.  選取 [計時器觸發程序]。

    ![替代文字](media\azure-stack-solution-staged-data-analytics\image4.png)

3.  選取 [C\#] 作為語言，並將函式命名為：`upload-to-azure`  將排程設定為 `0 0 * * * *`，這在 CRON 標記法中是指一小時一次。

    ![替代文字](media\azure-stack-solution-staged-data-analytics\image5.png)

## <a name="create-a-blob-storage-triggered-function"></a>建立由 Blob 儲存體所觸發的函式

1.  展開函式應用程式，然後選取 [函式] 旁的 [+] 按鈕。

2.  在 [搜尋] 欄位中，輸入 `blob`，然後選擇需要的 **Blob 觸發程序**範本語言。

  ![選擇 Blob 儲存體觸發程序範本。](media\azure-stack-solution-staged-data-analytics\image10.png)

3.  使用下列表格中所指定的設定：

    | 設定 | 建議的值 | 說明 |
    | ------- | ------- | ------- |
    | 名稱 | 函式應用程式中的唯一名稱 | 這個由 blob 所觸發之函式的名稱。 |
    | Path | \<上述儲存體位置中的路徑> | 受監視 Blob 儲存體中的位置。 在繫結中，Blob 的檔案名稱會以「名稱」參數的形式來傳遞。 |
    | 儲存體帳戶連線 | 函式應用程式連線 | 您可以使用應用程式函式已在使用的儲存體帳戶連線，或建立新的連線。 |

    **範例：**

    ![建立由 Blob 儲存體所觸發的函式。](media\azure-stack-solution-staged-data-analytics\image11.png)

4.  選取 [建立] 以建立函式。

### <a name="test-the-function"></a>測試函式

1.  在 Azure 入口網站中瀏覽至函式。 展開頁面底部的**記錄**，並確定記錄串流並未暫停。

2.  開啟 [儲存體總管] 並連線至本節開頭所建立的儲存體帳戶。

3.  依序展開儲存體帳戶、**Blob 容器**，和您稍早建立的 Blob。 依序選取 [上傳] 和 [上傳檔案]。

    ![將檔案上傳至 Blob 容器。](media\azure-stack-solution-staged-data-analytics\image12.png)

4.  在 [上傳檔案] 對話方塊中，選取 [檔案] 欄位。 瀏覽至本機電腦上的檔案 (例如影像檔)，加以選取，然後依序選取 [開啟] 和 [上傳]。

5.  返回函式記錄，並確認系統已讀取 Blob。

    **範例：**

    ![檢視記錄中的訊息。](media\azure-stack-solution-staged-data-analytics\image13.png)

## <a name="create-an-azure-stack-storage-account"></a>建立 Azure Stack 儲存體帳戶

建立包含 Blob 和佇列的 Azure Stack 儲存體帳戶。

### <a name="storage-blob--data-archiving"></a>封存儲存體 Blob 資料

此儲存體帳戶會包含兩個容器。 這些容器中的一個是 Blob，用來保存封存資料，另一個是佇列，用於主要辦公室散發的資料指派處理。

使用上述步驟和設定來建立另一個儲存體帳戶和 Blob 容器，以作為封存儲存體。

### <a name="storage-queue--filtered-data-holding"></a>保留儲存體佇列篩選資料

1.  使用上述步驟和設定來存取新的儲存體帳戶。

2.  在 [儲存體帳戶概觀] 區段中，選取 [佇列]。

3.  選取 [+ 佇列] 並填妥 [名稱] 欄位，然後填入新佇列的名稱。

4.  選取 [確認]。

    ![替代文字](media\azure-stack-solution-staged-data-analytics\image14.png)

    ![替代文字](media\azure-stack-solution-staged-data-analytics\image15.png)

## <a name="create-a-queue-triggered-function"></a>建立由佇列觸發的函式

1.  使用上方函式建立區段中的步驟，建立其他使用佇列觸發程序 (不是 Blob 觸發程序) 的 Azure Stack 函式。

2.  使用下列表格中所指定的設定：

    | 設定 | 建議的值 | 說明 |
    | ------- | ------- | ------- |
    | 名稱 | 函式應用程式中的唯一名稱 | 這個由佇列所觸發之函式的名稱。 |
    | Path | \<上述儲存體位置中的路徑> | 受監視儲存體中的位置。 在繫結中，佇列的檔案名稱會以「名稱」參數的形式來傳遞。 |
    | 儲存體帳戶連線 | 函式應用程式連線 | 您可以使用應用程式函式已在使用的儲存體帳戶連線，或建立新的連線。 |

3.  選取 [建立] 以建立函式。

## <a name="test-the-queue-triggered-function"></a>測試由佇列觸發的函式

1.  在 Azure 入口網站中瀏覽至函式。 展開頁面底部的**記錄**，並確定記錄串流並未暫停。

2.  開啟 [儲存體總管] 並連線至本節開頭所建立的儲存體帳戶。

3.  依序展開儲存體帳戶、**Blob 容器**，和您稍早建立的 Blob。 依序選取 [上傳] 和 [上傳檔案]。

    ![將檔案上傳至 Blob 容器。](media\azure-stack-solution-staged-data-analytics\image12.png)

4.  在 [上傳檔案] 對話方塊中，選取 [檔案] 欄位。 瀏覽至本機電腦上的檔案 (例如影像檔)，加以選取，然後依序選取 [開啟] 和 [上傳]。

5.  返回函式記錄，並確認系統已讀取 Blob。

  **範例：**

    ![檢視記錄中的訊息。](media\azure-stack-solution-staged-data-analytics\image13.png)

## <a name="securely-stored-and-accessed-compliant-data"></a>安全地儲存和存取符合規範的資料

全球化企業的資料已透過 Azure 和 Azure Stack 暫存資料分析和自訂端點指示詞安全地儲存、處理、散發和存取。 遠端辦公室的員工和機器活動、設備資料和商務計量會根據公司政策和地區性規定持續進行編譯、儲存、合規性測試及散發。

## <a name="next-steps"></a>後續步驟

- 若要深入了解 Azure 雲端模式，請參閱[雲端設計模式](https://docs.microsoft.com/azure/architecture/patterns)。