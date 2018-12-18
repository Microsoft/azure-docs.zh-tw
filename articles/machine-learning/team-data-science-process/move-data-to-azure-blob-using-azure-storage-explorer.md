---
title: 使用 Azure 儲存體總管從 Blob 儲存體移入或移出資料 | Microsoft Docs
description: 使用 Azure 儲存體總管從 Azure Blob 儲存體來回移動資料
services: machine-learning,storage
documentationcenter: ''
author: deguhath
manager: cgronlun
editor: cgronlun
ms.assetid: 10bd283f-0875-4c67-af63-6492270b7656
ms.service: machine-learning
ms.component: team-data-science-process
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 11/04/2017
ms.author: deguhath
ms.openlocfilehash: 295fadd69a86769f74ec45ed3e77e8a5b807e14f
ms.sourcegitcommit: b4a46897fa52b1e04dd31e30677023a29d9ee0d9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49392517"
---
# <a name="move-data-to-and-from-azure-blob-storage-using-azure-storage-explorer"></a>使用 Azure 儲存體總管從 Azure Blob 儲存體來回移動資料
Azure 儲存體總管是 Microsoft 所提供的免費工具，可讓您在 Windows、MacOS 和 Linux 上使用 Azure 儲存體資料。 本主題說明如何使用它來於 Azure Blob 儲存體中上傳及下載資料。 您可以從 [Microsoft Azure 儲存體總管](http://storageexplorer.com/)下載此工具。

[!INCLUDE [blob-storage-tool-selector](../../../includes/machine-learning-blob-storage-tool-selector.md)]

> [!NOTE]
> 如果您是使用以 [Azure 中的資料科學虛擬機器](virtual-machines.md)提供的指令碼所設定的 VM，則 Azure 儲存體總管已經安裝在 VM 上。
> 
> [!NOTE]
> 如需 Azure Blob 儲存體的完整介紹，請參閱 [Azure Blob 基本概念](../../storage/blobs/storage-dotnet-how-to-use-blobs.md)和 [Azure Blob 服務](https://msdn.microsoft.com/library/azure/dd179376.aspx)。   
> 
> 

## <a name="prerequisites"></a>必要條件
本文件假設您擁有 Azure 訂用帳戶、儲存體帳戶和該帳戶的對應儲存體金鑰。 上傳/下載資料之前，您必須知道 Azure 儲存體帳戶名稱和帳戶金鑰。 

* 若要設定 Azure 訂用帳戶，請參閱 [免費試用一個月](https://azure.microsoft.com/pricing/free-trial/)。
* 如需建立儲存體帳戶的指示，以及取得帳戶和金鑰的資訊，請參閱 [關於 Azure 儲存體帳戶](../../storage/common/storage-create-storage-account.md)。 記下儲存體帳戶的存取金鑰，因為您需要此金鑰，才能連接到具有 Azure 儲存體總管工具的帳戶。
* 您可以從 [Microsoft Azure 儲存體總管](http://storageexplorer.com/)下載 Azure 儲存體總管工具。 安裝期間請接受預設值。

<a id="explorer"></a>

## <a name="use-azure-storage-explorer"></a>使用 Azure 儲存體總管
下列步驟說明如何使用 Azure 儲存體總管上傳/下載資料。 

1. 啟動 Microsoft Azure 儲存體總管。
2. 若要啟動 [登入您的帳戶...] 精靈，請選取 [Azure 帳戶設定] 圖示，然後選取 [新增帳戶] 並輸入您的認證。 ![新增 Azure 儲存體帳戶](./media/move-data-to-azure-blob-using-azure-storage-explorer/add-an-azure-store-account.png)
3. 若要啟動 [連接到 Azure 儲存體] 精靈，請選取 [連接到 Azure 儲存體] 圖示。 ![連接到 Azure 儲存體](./media/move-data-to-azure-blob-using-azure-storage-explorer/connect-to-azure-storage-1.png)
4. 在 [連接到 Azure 儲存體] 精靈中輸入 Azure 儲存體帳戶的存取金鑰，然後選取 [下一步]。 ![連接到 Azure 儲存體](./media/move-data-to-azure-blob-using-azure-storage-explorer/connect-to-azure-storage-2.png)
5. 在 [帳戶名稱] 方塊中輸入儲存體帳戶名稱，然後選取 [下一步]。 ![附加外部儲存體](./media/move-data-to-azure-blob-using-azure-storage-explorer/attach-external-storage.png)
6. 現在應該會列出新增的儲存體帳戶。 若要在儲存體帳戶中建立 Blob 容器，請以滑鼠右鍵按一下該帳戶中的 [Blob 容器] 節點、選取 [建立 Blob 容器]，然後輸入名稱。
7. 若要將資料上傳至容器，請選取目標容器，然後按一下 [上傳] 按鈕。![儲存體帳戶](./media/move-data-to-azure-blob-using-azure-storage-explorer/storage-accounts.png)
8. 按一下 [檔案] 方塊右邊的 [...]，從檔案系統中選取一或多個要上傳的檔案，然後按一下 [上傳]，開始上傳檔案。![上傳檔案](./media/move-data-to-azure-blob-using-azure-storage-explorer/upload-files-to-blob.png)
9. 若要下載資料，選取對應容器中要下載的 Blob，然後按一下 [下載] 。 ![下載檔案](./media/move-data-to-azure-blob-using-azure-storage-explorer/download-files-from-blob.png)

