---
title: Azure 快速入門 - 建立 Batch AI 叢集 - 入口網站 | Microsoft Docs
description: 快速入門 - 建立 Batch AI 叢集來訓練機器學習和 AI 模型 - Azure 入口網站
services: batch-ai
documentationcenter: na
author: dlepow
manager: jeconnoc
editor: ''
ms.assetid: ''
ms.custom: ''
ms.service: batch-ai
ms.workload: ''
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: quickstart
ms.date: 08/15/2018
ms.author: danlep
ms.openlocfilehash: 8b9daa0fbbf84e0f602498a0847c9e120f709b17
ms.sourcegitcommit: ebd06cee3e78674ba9e6764ddc889fc5948060c4
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/07/2018
ms.locfileid: "44057479"
---
# <a name="quickstart-create-a-cluster-for-batch-ai-training-jobs-using-the-azure-portal"></a>快速入門：使用 Azure 入口網站建立 Batch AI 訓練作業的叢集

本快速入門說明如何使用 Azure 入口網站來建立 Batch AI 叢集，以供用來訓練 AI 和機器學習模型。 Batch AI 是一項受控服務，可讓資料科學家和 AI 研究人員在 Azure 虛擬機器的叢集上大規模地訓練 AI 和機器學習模型。

此叢集一開始只有一個 GPU 節點和一個連結的檔案伺服器。 完成本快速入門之後，您就會擁有可相應增加並用來訓練深度學習模型的叢集。 請使用 Batch AI、[Azure Machine Learning](../machine-learning/service/overview-what-is-azure-ml.md) 工具或 [Visual Studio Tools for AI](https://github.com/Microsoft/vs-tools-for-ai) 將訓練作業提交至叢集。

[!INCLUDE [quickstarts-free-trial-note.md](../../includes/quickstarts-free-trial-note.md)]

## <a name="create-ssh-key-pair"></a>建立 SSH 金鑰組

您必須要有 SSH 金鑰組，才能完成本快速入門的操作。 如果現有的 SSH 金鑰組，則可略過此步驟。

若要建立 SSH 金鑰組，請從 Bash 殼層執行下列命令，並依照螢幕上的指示操作。 例如，您可以使用 [Azure Cloud Shell](../cloud-shell/overview.md) 或在 Windows 上使用[適用於 Linux 的 Windows 子系統](/windows/wsl/install-win10)。 命令輸出包含公開金鑰檔案的檔案名稱。 將公開金鑰檔案 (`cat ~/.ssh/id_rsa.pub`) 的內容複製到剪貼簿或其他位置，以便在稍後的步驟中存取。

```bash
ssh-keygen -t rsa -b 2048
```

如需如何建立 SSH 金鑰組的詳細資訊，請參閱[在 Azure 中建立和使用 Linux VM 的 SSH 公開和私密金鑰組](../virtual-machines/linux/mac-create-ssh-keys.md)。

## <a name="sign-in-to-azure"></a>登入 Azure

在 https://portal.azure.com 登入 Azure 入口網站。

## <a name="create-a-batch-ai-workspace"></a>建立 Batch AI 工作區

一開始先建立 Batch AI 工作區，以便組織 Batch AI 資源。 工作區可以包含一或多個叢集或其他 Batch AI 資源。

1. 選取 [所有服務] 並篩選 [Batch AI]。

2. 選取 [新增工作區]。

3. 輸入 [工作區名稱] 和 [資源群組] 的值。 如果您想要的話，也可以為工作區選取不同的 [訂用帳戶] 和 [位置] 選項。 選取 [建立工作區]。

  ![建立 Batch AI 工作區](./media/quickstart-create-cluster-portal/create-workspace.png)

當**部署成功**訊息出現時，移至您所建立的資源，然後選取工作區。

## <a name="create-a-file-server"></a>建立檔案伺服器

Batch AI 檔案伺服器是單一節點 NFS，可自動掛接在叢集節點上。 在可為訓練作業的輸入資料和輸出提供儲存體的數種方法之中，這是其中的一種。

1. 在工作區中，選取 [檔案伺服器] > [新增 Batch AI 檔案伺服器]。

2. 輸入 [檔案伺服器名稱] 和 [VM 大小] 的值。 在此快速入門中，建議您為檔案伺服器使用*標準 D1_v2* 的 VM 大小。 您也可以選擇不同的大小，以便儲存更大量的訓練作業輸入或輸出資料。

3. 輸入 [系統管理員使用者名稱]，並將 SSH 公用金鑰檔案的內容複製到 [SSH 金鑰]。 接受其餘值的預設值，然後選取 [建立檔案伺服器]。

  ![建立 Batch AI 檔案伺服器](./media/quickstart-create-cluster-portal/create-file-server.png)

部署檔案伺服器需要幾分鐘的時間。

伺服器建立完成後，按一下 [屬性] 並記下 [掛接設定]。 您可以透過 SSH 連線至伺服器的公用 IP 位址，以便在指定的目錄 (/data) 上傳和下載訓練資料和輸出檔。

![檔案伺服器屬性](./media/quickstart-create-cluster-portal/file-server-properties.png)

## <a name="create-a-cluster"></a>建立叢集

下列步驟會建立具有單一 GPU 節點的叢集。 叢集節點會執行預設 Ubuntu Server 映像，此映像的設計目的是要裝載容器型應用程式，以供用於大部分的訓練工作負載。 叢集節點會將檔案伺服器掛接在其掛接點上。 

1. 在 Batch AI 工作區中，選取 [叢集] > [新增 Batch AI 叢集]。

2. 輸入 [叢集名稱] 和下列設定的值。 建議的 VM 大小具有一個 NVIDIA Tesla K80 GPU。
  
   |設定  |值  |
   |---------|---------|
   |**VM 大小**     |標準 NC6|
   |**目標節點數目**     |1|

3. 輸入 [系統管理員使用者名稱]，並將 SSH 公用金鑰檔案的內容複製到 [SSH 金鑰]。 接受這個頁面上其餘值的預設值，然後選取 [下一步: 節點設定]。

   ![輸入基本的叢集資訊](./media/quickstart-create-cluster-portal/create-cluster.png)

4. 在 [掛接磁碟區] 底下，選取 [檔案伺服器參考] > [新增]。 選取您先前建立的檔案伺服器。 輸入每個叢集節點上用來掛接檔案伺服器的 [相對掛接路徑]。 選取 [儲存並繼續]。

   ![新增檔案伺服器參考](./media/quickstart-create-cluster-portal/file-server-reference.png)

儲存節點設定，並選取 [建立叢集]。

Batch AI 會花幾分鐘的時間來配置節點。 在這段期間，叢集的 [配置狀態] 為 [調整大小]。 幾分鐘之後，叢集的狀態為 [穩定]，而節點會啟動。

![叢集正在啟動](./media/quickstart-create-cluster-portal/cluster-resizing.png)

選取叢集名稱，以檢查節點的狀態。 當節點的狀態為 [閒置] 時，便已準備好執行訓練作業。

### <a name="list-cluster-nodes"></a>列出叢集節點

如果您需要連線到叢集節點 (在此案例為單一節點) 以安裝應用程式或進行維護，請在入口網站中取得連線資訊。 叢集建立好之後，按一下 [節點]，並記下 SSH [連線] 設定 (IP 位址和連接埠號碼)。

![叢集節點](./media/quickstart-create-cluster-portal/cluster-nodes.png)

您可以使用此資訊對節點建立 SSH 連線。 例如，在下列命令中替代成節點的正確 IP 位址和連接埠號碼：

```bash
ssh myusername@137.135.82.15 -p 50000
``` 

### <a name="resize-the-cluster"></a>調整叢集大小

當您使用叢集來訓練模型時，您可能需要更多計算資源。 例如，若要將分散式訓練作業的大小增加為 2 個節點，請選取 [調整] 並將 [節點的目標數目] 設定為 2。 儲存組態。

![調整叢集](./media/quickstart-create-cluster-portal/scale-cluster.png)

叢集會花幾分鐘的時間來調整大小。

## <a name="clean-up-resources"></a>清除資源

如果您想要繼續進行 Batch AI 教學課程和範例，請使用本快速入門所建立的 Batch AI 工作區、檔案伺服器和叢集。

當基礎虛擬機器執行時，即使您沒有排定要進行的作業，仍需支付 Batch AI 叢集和檔案伺服器的費用。 如果您想要在沒有作業需要執行時保留叢集設定，請將叢集大小調整為 0 個節點。 之後，請將其大小調整為 1 個以上的節點，以便執行作業。 

若不再需要，請刪除包含叢集和檔案伺服器的 Batch AI 工作區。 若要這樣做，請選取 Batch AI 工作區，然後選取 [刪除]。

## <a name="next-steps"></a>後續步驟

在本快速入門中，您已了解如何使用 Azure 入口網站建立 Batch AI 叢集和連結的檔案伺服器。 若要了解如何使用 Batch AI 叢集來訓練模型，請繼續進行用來訓練深度學習模型的快速入門。

> [!div class="nextstepaction"]
> [訓練深度學習模型](./quickstart-tensorflow-training-cli.md)
