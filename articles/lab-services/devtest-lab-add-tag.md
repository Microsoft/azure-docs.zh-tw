---
title: 在 Azure DevTest Labs 中將標記新增到實驗室 | Microsoft Docs
description: 了解如何在 Azure DevTest Labs 中將標記新增到實驗室
services: devtest-lab,virtual-machines
documentationcenter: na
author: spelluru
manager: ''
editor: ''
ms.assetid: dc5b327a-62e4-41bc-80ef-deb3c23d51b2
ms.service: devtest-lab
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/17/2018
ms.author: spelluru
ms.openlocfilehash: 3d9a5b3c0ae0b6058d3e8ccf8cdb340bd1200edc
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38299106"
---
# <a name="add-tags-to-a-lab-in-azure-devtest-labs"></a>在 Azure DevTest Labs 中將標記新增到實驗室

您可以建立自訂標記並套用到 DevTest Labs 資源，以利用邏輯方式分類資源。 之後，您可以快速而輕鬆地查看訂用帳戶中所有加上標記的資源。 當您需要整理資源以進行計費或管理時，標記就能發揮功用。

支援標記的資源包括

* 計算 VM
* NIC
* IP 位址
* 負載平衡器
* 儲存體帳戶
* 受控磁碟

您可以在[建立實驗室](devtest-lab-create-lab.md)時套用標記，稍後再透過 [組態和設定] 下方的 [標記] 刀鋒視窗進行管理。

每個標記都包含一對**名稱**/**值**。 例如，您可以建立名稱為 *costcenter*，值為 *34543* 的標記。 這類標記能協助您在日後辨識實驗室資源，以便向組織的此特定領域收費。 您可以選擇方便讓自己整理訂用帳戶的名稱和值。

## <a name="steps-to-manage-tags-in-an-existing-lab"></a>在現有實驗室中管理標記的步驟

1. 登入 [Azure 入口網站](http://go.microsoft.com/fwlink/p/?LinkID=525040)。
1. 視需要選取 [所有服務]，然後從清單中選取 [DevTest Labs]。 您的實驗室可能已顯示在 [所有資源] 下方的儀表板上。
1. 從實驗室清單中，選取您想要新增或管理標記的實驗室。  
1. 在實驗室的 [概觀] 區域中選取 [組態和原則]。  

    ![組態和原則按鈕](./media/devtest-lab-add-tag/devtestlab-config-and-policies.png)

1. 在左側的 [管理] 下方選取 [標記]。
1. 若要為這個實驗室建立新標記，請輸入**名稱**/**值**組，然後選取 [儲存]。 您也可以從清單選取現有標記，以便檢視或管理與該標記相關的資源。

    ![管理標記](./media/devtest-lab-add-tag/devtestlab-manage-tags.png)

## <a name="understanding-limitations-to-tags"></a>了解標記的限制

標籤具有下列限制：

* 每個資源或資源群組最多都可以有 15 個標記名稱/值組。 此限制只適用於直接套用至資源群組或資源的標記。 資源群組可以包含許多資源，其各自有 15 個標記名稱/值組。 
* 標記名稱上限為 512 個字元，且標記值上限為 256 字元。 儲存體帳戶的標記名稱上限為 128 個字元，且標記值上限為 256 個字元。
* 資源群組中的資源不會繼承套用至該資源群組的標籤。

[使用標記來組織 Azure 資源](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags)提供在 Azure 中使用標記的更多詳細資料，包括如何使用 PowerShell 或 Azure CLI 管理標記。

[!INCLUDE [devtest-lab-try-it-out](../../includes/devtest-lab-try-it-out.md)]

## <a name="next-steps"></a>後續步驟
* 您可以使用自訂原則，在訂用帳戶內套用限制和慣例。 您所定義的原則可能需要所有資源都具有特定標籤的值。 如需詳細資訊，請參閱[設定原則和排程](devtest-lab-set-lab-policy.md)。
* 瀏覽 [DevTest Labs Azure Resource Manager 快速入門範本資源庫 (英文)](https://github.com/Azure/azure-devtestlab/tree/master/Samples)。
