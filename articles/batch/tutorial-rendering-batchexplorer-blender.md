---
title: 使用 Azure Batch 和 Batch Explorer 轉譯 Blender 場景
description: 教學課程 - 如何使用 Azure Batch 和 Batch Explorer 用戶端應用程式從 Blender 場景轉譯多個畫面格
services: batch
author: mscurrell
ms.author: markscu
ms.date: 08/02/2018
ms.topic: tutorial
ms.openlocfilehash: 46c65cd7ac5734134fa7c4ad6fd85f39d1188e28
ms.sourcegitcommit: d1aef670b97061507dc1343450211a2042b01641
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47392546"
---
# <a name="tutorial-render-a-blender-scene-using-batch-explorer"></a>教學課程：使用 Batch Explorer 轉譯 Blender 場景

此教學課程說明如何轉譯 Blender 示範場景的多個畫面格。 此教學課程會使用 Blender，因為這在用戶端和轉譯的 VM 兩端都可免費使用，但如果您使用 Maya 或 3ds Max 之類的其他應用程式，作業程序也十分類似。

在此教學課程中，您了解如何：
> [!div class="checklist"]
> * 將 Blend 場景上傳至 Azure 儲存體
> * 建立含有多個節點的 Batch 集區以執行轉譯
> * 轉譯多個畫面格
> * 檢視和下載轉譯的畫面格檔案

## <a name="prerequisites"></a>先決條件

您需要預付型訂用帳戶或其他 Azure 購買選項，以按使用量付費的方式，在 Batch 中使用轉譯應用程式。 如果您使用提供信用額度金額的免費 Azure 方案，則不支援按使用量付費授權。

您需要具有關聯之儲存體帳戶的 Azure Batch 帳戶。  請參閱任何 Batch 快速入門文章以建立 Batch 帳戶，例如 [CLI 文章](https://docs.microsoft.com/azure/batch/quick-create-cli)。

此教學課程中指定的 VM 大小和 VM 數目至少需要 50 個核心的低優先順序核心配額；您可以使用預設配額，但將須使用較小的 VM 大小，而這意味著影像轉譯會較為耗時。 要求增加核心配額的程序詳述於[這篇文章](https://docs.microsoft.com/azure/batch/batch-quota-limit)中。

最後，必須要安裝 [Batch Explorer](https://azure.github.io/BatchExplorer/)；它適用於 Windows、OSX 和 Linux。 它是選擇性的，但若安裝了 [Blender](https://www.blender.org/download/)，則可以檢視範例模型檔案。

## <a name="download-the-demo-scene"></a>下載示範場景

從 [Blender 示範檔案網頁](https://www.blender.org/download/demo-files/)下載 Blender 的「課堂」示範 ZIP 檔案，並將其解壓縮至本機磁碟機。

## <a name="upload-a-scene-to-azure-storage"></a>將場景上傳至 Azure 儲存體

建立示範場景檔案的儲存體帳戶容器：

* 啟動 Batch Explorer
* 從左側的主功能表中選取 [資料] 功能表項目。
* 確定已在下拉式清單中選取 [檔案群組]。
* 選取 [+] 按鈕，然後建立新的「空檔案群組」'blender-classroom'
  * 檔案群組就是一個具有 'fgrp-' 前置詞的 Azure 儲存體 Blob 容器；這是用來篩選掉儲存體帳戶中其他容器的慣例

![場景檔案的檔案群組](./media/tutorial-rendering-batchexplorer-blender/batch_explorer_scene_filegroup.png)

上傳場景檔案：

* 選取新的容器，並將 'classroom' 資料夾的內容拖放到 Batch Explorer 中的容器。

![上傳的場景檔案](./media/tutorial-rendering-batchexplorer-blender/batch_explorer_scene_filegroup_uploaded.png)

## <a name="create-azure-storage-container-for-output-images"></a>建立輸出影像的 Azure 儲存體容器

建立示範場景輸出檔案的儲存體帳戶容器：

* 從左側的主功能表中選取 [資料] 功能表項目。
* 選取 [+] 按鈕，然後建立新的「空檔案群組」'render-output'

![輸出檔案的檔案群組](./media/tutorial-rendering-batchexplorer-blender/batch_explorer_output_filegroup.png)

## <a name="create-a-pool-of-vms-for-rendering"></a>建立要轉譯的 VM 集區

使用呈現包含 Blender 應用程式的轉譯 Azure Marketplace VM 映像建立 Batch 集區：

* 從左側的主功能表中選取 [資源庫] 功能表項目。
* 針對應用程式項目清單，選取 'Blender' 項目。
* 選取要在 Windows Server 上轉譯畫面格的項目
* 您可以選擇性地選取項目右側的連結圖示，以檢視將用來建立集區和作業的範本檔案。

![Blender 資源庫項目](./media/tutorial-rendering-batchexplorer-blender/batch_explorer_gallery_item.png)

* 選取 [建立集區以供後續使用] 按鈕
  * 將集區名稱保留為 “blender-windows”
  * 將 [專用 VM 計數] 設定為 "0"
  * 將 [低優先順序 VM 計數] 設定為 "3"
  * 將 [節點大小] 設定為 “Standard_F16” – 您可以選取其他 VM 大小，但轉譯畫面格的所需時間主要會取決於核心數目。
* 選取綠色按鈕以建立集區
  * 集區會隨即建立，但配置和啟動 VM 需要幾分鐘的時間。

![Blender 的集區範本](./media/tutorial-rendering-batchexplorer-blender/batch_explorer_pool_template.png)

> [!WARNING]
> 請注意，當集區中有 VM 存在時，您的 Azure 訂用帳戶將會產生這些 VM 的費用。 必須刪除集區或刪除 VM，才會停止計費。 在完成此教學課程後請刪除集區，以免繼續產生費用。

您可以在 [集區] 檢視中監視集區和 VM 的狀態；下列範例顯示三個都已配置的 VM，其中兩個已啟動但處於閒置狀態，另一個仍在啟動中：![集區熱度圖](./media/tutorial-rendering-batchexplorer-blender/batch_explorer_pool_heatmap.png)

## <a name="create-a-rendering-job"></a>建立轉譯作業

建立會使用已建立的集區來轉譯部分畫面格的轉譯作業：
* 從左側的主功能表中選取 [資源庫] 功能表項目。
* 針對應用程式項目清單，選取 'Blender' 項目。
* 選取要在 Windows Server 上轉譯畫面格的項目。
* 選取 [使用現有集區執行作業] 按鈕
* 選取 'blender-windows' 集區
* 將 [作業名稱] 設定為 'blender-render-tutorial1'
* 針對 [輸入資料] 選取 'fgrp-blender-classroom'
* 選取 [Blend 檔案] 的檔案圖示，然後選取 'classroom.blend'
* 將 [起始畫面格] 保留為 '1'，並將 [結束畫面格] 設定為 '5'
* 將 [輸出] 設定為 'fgrp-render-output'
* 選取綠色按鈕以建立作業；建立的作業會包含五項工作，每個畫面格使用一個

![Blender 的作業範本](./media/tutorial-rendering-batchexplorer-blender/batch_explorer_job_template.png)

作業和所有工作建立後，作業將會連同作業中的工作一起顯示：![作業工作清單](./media/tutorial-rendering-batchexplorer-blender/batch_explorer_task_list.png)

當工作第一次在集區 VM 上開始執行時，系統會執行 Batch 作業準備工作，將場景檔案從儲存體檔案群組複製到 VM 上，使其可供 Blender 存取。
您可以檢視 Blend 所產生的 stdout.txt 記錄檔，以確認轉譯的狀態。  選取工作時依預設會顯示「工作輸出」，此時您可以選取及檢視 'stdout.txt' 檔案。
![stdout 檔案](./media/tutorial-rendering-batchexplorer-blender/batch_explorer_stdout.png)

如果選取 'blender-windows' 集區，集區 VM 將會呈現為執行中狀態：![具有執行中節點的集區熱度圖](./media/tutorial-rendering-batchexplorer-blender/batch_explorer_pool_heatmap_running.png)

轉譯的影像需要數分鐘才會產生，視選取的 VM 大小而定。  使用先前指定的 F16 VM 時，畫面格轉譯的所需時間約為 16 分鐘。

## <a name="view-the-rendering-output"></a>檢視轉譯輸出

畫面格完成轉譯後，這些工作將顯示為已完成：![工作完成](./media/tutorial-rendering-batchexplorer-blender/batch_explorer_tasks_complete.png)

已轉譯的影像會先寫入至 VM，然後選取 'wd' 資料夾即可加以檢視：![集區節點上的轉譯影像](./media/tutorial-rendering-batchexplorer-blender/batch_explorer_output_image.png)

作業範本也會指定，輸出畫面格和記錄檔應寫回至作業建立時所指定的 Azure 儲存體帳戶檔案群組。  [資料] UI 可用來檢視輸出檔案和記錄，也可用來下載檔案：![儲存檔案群組中的轉譯影像](./media/tutorial-rendering-batchexplorer-blender/batch_explorer_output_image_storage.png)

當所有工作都完成後，作業將會標示為已完成：![作業和所有工作均已完成](./media/tutorial-rendering-batchexplorer-blender/batch_explorer_job_alltasks_complete.png)

## <a name="clean-up-resources"></a>清除資源

> [!WARNING]
> 您必須刪除集區 (或將其大小調整為零個節點)，以避免 Azure 訂用帳戶持續產生 VM 費用。

* 選取 [集區]
* 選取 'blender-windows' 集區
* 按一下滑鼠右鍵再按一下 [刪除]，或選取集區上方的垃圾桶圖示

## <a name="next-steps"></a>後續步驟
* 在 [資源庫] 區段中，透過 Batch Explorer 瀏覽可用的轉譯應用程式。
* 每個應用程式會有數個可用的範本，且數量會隨著時間而增加。  例如，其中會有 Blender 範本，可將單一影像分割為圖格，因而能夠以平行方式轉譯影像的某些部分。
* 如需轉譯功能的完整說明，請參閱[此處](https://docs.microsoft.com/azure/batch/batch-rendering-service)的系列文章。
