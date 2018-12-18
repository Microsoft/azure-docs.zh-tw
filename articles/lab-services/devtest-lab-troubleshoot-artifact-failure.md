---
title: 針對 Azure DevTest Labs 虛擬機器中的構件失敗進行診斷 | Microsoft Docs
description: 了解如何針對 Azure DevTest Labs 中的構件失敗進行疑難排解。
services: devtest-lab,virtual-machines,lab-services
documentationcenter: na
author: spelluru
manager: femila
editor: ''
ms.assetid: 115e0086-3293-4adf-8738-9f639f31f918
ms.service: lab-services
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/17/2018
ms.author: spelluru
ms.openlocfilehash: ebc64215683989ce07f4dd88dc352ecaefe184cd
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38697267"
---
# <a name="diagnose-artifact-failures-in-the-lab"></a>針對實驗室中的構件失敗進行診斷 
建立構件之後，您可以檢查以確認其成功或失敗。 Azure DevTest Labs 中的構件記錄會提供資訊供您用來診斷構件失敗。 有幾個選項可供您檢視 Windows VM 的構件記錄資訊：

* 在 Azure 入口網站
* 在 VM 中

> [!NOTE]
> 為了確保正確識別並說明失敗，請務必讓構件具有正確結構。 如需如何正確建構構件的資訊，請參閱[建立自訂構件](devtest-lab-artifact-author.md)。 若要查看正確結構化的構件範例，請參閱[測試參數類型](https://github.com/Azure/azure-devtestlab/tree/master/Artifacts/windows-test-paramtypes)構件。

## <a name="troubleshoot-artifact-failures-by-using-the-azure-portal"></a>使用 Azure 入口網站為構件失敗進行疑難排解

1. 在 Azure 入口網站的資源清單中選取您的實驗室。
2. 選擇您要調查之構件所在的 Windows VM。
3. 在左面板的 [一般] 下，選取 [構件]。 與該 VM 相關聯的構件清單隨即出現。 清單中會指出構件的名稱及構件狀態。

   ![構件狀態](./media/devtest-lab-troubleshoot-artifact-failure/devtest-lab-artifacts-failure.png)

4. 選取顯示 [失敗] 狀態的構件。 構件會隨即開啟。 系統會顯示延伸訊息，其中包含有關構件失敗的詳細資料。

   ![構件錯誤訊息](./media/devtest-lab-troubleshoot-artifact-failure/devtest-lab-artifact-error.png)


## <a name="troubleshoot-artifact-failures-from-within-the-virtual-machine"></a>針對虛擬機器中的構件失敗進行疑難排解

1. 登入您要診斷之構件所在的 VM。
2. 移至 C:\Packages\Plugins\Microsoft.Compute.CustomScriptExtension\\1.9\Status，其中 1.9 是 Azure 自訂指令碼延伸模組的版本號碼。

   ![狀態檔案](./media/devtest-lab-troubleshoot-artifact-failure/devtest-lab-artifact-error-vm-status.png)

3. 開啟**狀態**檔案。

[!INCLUDE [devtest-lab-try-it-out](../../includes/devtest-lab-try-it-out.md)]

## <a name="related-blog-posts"></a>相關部落格文章
* [Join a VM to an existing Active Directory domain by using a Resource Manager template in DevTest Labs](http://www.visualstudiogeeks.com/blog/DevOps/Join-a-VM-to-existing-AD-domain-using-ARM-template-AzureDevTestLabs) (在 DevTest Labs 中使用 Resource Manager 範本將 VM 加入至現有 Active Directory 網域)

## <a name="next-steps"></a>後續步驟
* 了解如何[將 Git 存放庫新增至實驗室](devtest-lab-add-artifact-repo.md)。

