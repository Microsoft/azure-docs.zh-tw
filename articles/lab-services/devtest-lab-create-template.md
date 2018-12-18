---
title: 從 VHD 檔案建立 Azure DevTest Labs 自訂映像 | Microsoft Docs
description: 了解如何使用 Azure 入口網站在 Azure DevTest Labs 中從 VHD 檔案建立自訂映像
services: devtest-lab,virtual-machines,lab-services
documentationcenter: na
author: spelluru
manager: femila
editor: ''
ms.assetid: b795bc61-7c28-40e6-82fc-96d629ee0568
ms.service: lab-services
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/17/2018
ms.author: spelluru
ms.openlocfilehash: a961565815ca0d89dc98a8d6a3e14b338b649398
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38687803"
---
# <a name="create-a-custom-image-from-a-vhd-file"></a>從 VHD 檔案建立自訂映像

[!INCLUDE [devtest-lab-create-custom-image-from-vhd-selector](../../includes/devtest-lab-create-custom-image-from-vhd-selector.md)]

[!INCLUDE [devtest-lab-custom-image-definition](../../includes/devtest-lab-custom-image-definition.md)]

[!INCLUDE [devtest-lab-upload-vhd-options](../../includes/devtest-lab-upload-vhd-options.md)]

## <a name="step-by-step-instructions"></a>逐步指示

下列步驟將逐步引導您使用 Azure 入口網站從 VHD 檔案建立自訂映像：

1. 登入 [Azure 入口網站](http://go.microsoft.com/fwlink/p/?LinkID=525040)。

1. 選取 [所有服務]，然後從清單中選取 [DevTest Labs]。

1. 從實驗室清單中，選取所需的實驗室。  

1. 在實驗室的主窗格上，選取 [設定與原則]。 

1. 在 [設定與原則] 窗格上，選取 [自訂映像]。

1. 在 [自訂映像] 窗格上，選取 [+新增]。

    ![加入自訂映像](./media/devtest-lab-create-template/add-custom-image.png)

1. 輸入自訂映像的名稱。 這個名稱會在建立 VM 時顯示於基本映像清單中。

1. 輸入自訂映像的描述。 這個描述會在建立 VM 時顯示於基本映像清單中。

1. 針對 [OS 類型]，選取 [Windows] 或 [Linux]。

    - 如果選取 [Windows]，請透過核取方塊來指定 *sysprep* 是否已在電腦上執行。 
    - 如果選取 [Linux]，請透過核取方塊來指定 *deprovision* 是否已在電腦上執行。 

1. 從下拉式功能表中選取 **VHD**。 這是將用來建立新自訂映像的 VHD。 如果需要，請選取 [使用 PowerShell 上傳 VHD]。

1. 如果用來建立自訂映像的映像不是授權的映像 (由 Microsoft 發行)，您也可以輸入計劃名稱、計劃供應項目及計劃發行者。

   - **方案名稱：** 輸入建立此自訂映像的 Marketplace 映像 (SKU) 名稱 
   - 
  **方案供應項目：** 輸入建立此自訂映像的 Marketplace 映像產品 (供應項目) 
   - **方案發行者：** 輸入建立此自訂映像的 Marketplace 映像發行者

   > [!NOTE]
   > 如果您用來建立自訂映像的映像**不**是授權的映像，則這些欄位是空的，而且您可選擇填入。 如果影像**是**授權的映像，則欄位會自動填入方案資訊。 如果您嘗試在此情況下變更這些，則會顯示警告訊息。
   >
   >

1. 選取 [確定]  ，以建立自訂映像。

幾分鐘之後，即會建立自訂映像，並儲存於實驗室的儲存體帳戶內。 當實驗室使用者想要建立新的 VM 時，可在基底映像清單中取得映像。

![基底映像清單中可用的自訂映像](./media/devtest-lab-create-template/custom-image-available-as-base.png)


[!INCLUDE [devtest-lab-try-it-out](../../includes/devtest-lab-try-it-out.md)]

## <a name="related-blog-posts"></a>相關部落格文章

- [自訂映像或公式？](https://blogs.msdn.microsoft.com/devtestlab/2016/04/06/custom-images-or-formulas/)
- [在 Azure DevTest Labs 之間複製自訂映像](http://www.visualstudiogeeks.com/blog/DevOps/How-To-Move-CustomImages-VHD-Between-AzureDevTestLabs#copying-custom-images-between-azure-devtest-labs)

## <a name="next-steps"></a>後續步驟

- [將 VM 新增到實驗室](./devtest-lab-add-vm.md)
