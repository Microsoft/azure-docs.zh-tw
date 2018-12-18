---
title: 設定適用於聲場的 Azure 帳戶 - 認知服務
description: 請依照本指南設定與聲場搭配運作所需的 Azure Batch 和「儲存體」帳戶。
services: cognitive-services
author: ashtat
manager: noelc
ms.service: cognitive-services
ms.component: acoustics
ms.topic: article
ms.date: 08/17/2018
ms.author: kegodin
ms.openlocfilehash: d5e78df2cb17e8275aef3694dda90a705ef4bdaa
ms.sourcegitcommit: 1aedb52f221fb2a6e7ad0b0930b4c74db354a569
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/17/2018
ms.locfileid: "40181908"
---
# <a name="create-an-azure-batch-account"></a>建立 Azure Batch 帳戶
請依照本指南設定與聲場搭配運作所需的 Azure Batch 和「儲存體」帳戶。 如需有關隨 Project Acoustics 一起開發之 Unity 外掛程式的資訊，請參閱[什麼是聲場](what-is-acoustics.md)。 如需有關如何將聲場合併到您 Unity 專案中的資訊，請參閱[使用者入門](getting-started.md)。  

## <a name="get-an-azure-subscription"></a>取得 Azure 訂用帳戶
您必須先有 [Azure 訂用帳戶](https://azure.microsoft.com/free/)，才能設定 Batch 與「儲存體」帳戶。 如果您是第一次註冊，Azure 會提供一些有時間限制的免費資源與 200 美元的信用額度。

## <a name="create-azure-batch-and-storage-accounts"></a>建立 Azure Batch 與儲存體帳戶
接著，請依照[這些指示](https://docs.microsoft.com/azure/batch/batch-account-create-portal)來設定 Azure Batch 和相關的「Azure 儲存體」帳戶。

為 Batch 與「儲存體」帳戶選取預設選項：
  
  ![新增 Batch 帳戶](media/NewBatchAccountCreate.png)

  ![新增儲存體帳戶](media/BatchStorageAccountCreate.png)

Azure 需要幾分鐘的時間來部署帳戶。 在入口網站的右上角即可找到完整的通知。
  
  ![已部署帳戶](media/BatchAccountsDeployNotification.png)

您現在應該可以在儀表板上看到您的帳戶。
  
  ![入口網站儀表板](media/AzurePortalDashboard.png)

## <a name="set-up-acoustics-bake-ui-with-azure-credentials"></a>使用 Azure 認證來設定聲場製作 UI
按一下儀表板上的 Batch 帳戶連結，然後按一下 Batch 帳戶頁面上的 [金鑰] 連結以存取您的認證。
  
  ![Batch 金鑰連結](media/BatchAccessKeys.png)

  ![Batch 帳戶認證](media/BatchKeysInfo.png)

按一下頁面上的 [儲存體帳戶] 連結，以存取您的「Azure 儲存體」帳戶認證。
  
  ![儲存體帳戶認證](media/StorageKeysInfo.png)

在 [Bake] \(製作\) 索引標籤中輸入這些認證，如[製作 UI 逐步解說](bake-ui-walkthrough.md)中所述。

## <a name="node-types-and-region-support"></a>節點類型與區域支援
Project Acoustics 需要 F 系列所 H 系列的計算最佳化 Azure VM 節點，但可能並非所有 Azure 區域都支援這些節點。 請參閱[此表格](https://azure.microsoft.com/global-infrastructure/services)，以確保您為 Batch 帳戶選取的位置正確。 目前支援 H 系列虛擬機器的區域包括美國東部、美國中北部、美國中南部、美國西部、美國西部 2、北歐、西歐與日本西部。

## <a name="upgrading-your-quota"></a>升級您的配額
建立 Azure Batch 帳戶時，為帳戶佈建的計算核心數目限制為 20 個。 您可以將此數目限制提高 (最高可達場景中探查點的數目) 來縮短製作時間，因為您可以跨許多節點平行處理聲場工作負載。 您可以按一下 Azure Batch 入口網站頁面上的 [配額] 連結，然後按一下 [申請加大配額]，來申請加大配額：

![加大 Azure 配額](media/azurequotas.png)

## <a name="next-steps"></a>後續步驟
* 開始[將聲場整合到 Unity 專案中](getting-started.md)
* 探索[範例場景](sample-walkthrough.md)

