---
title: 在 Azure DevTest Labs 的金鑰保存庫中儲存秘密 | Microsoft Docs
description: 了解如何將祕密儲存在 Azure Key Vault 中，並在建立 VM、公式或環境時加以使用。
services: devtest-lab
documentationcenter: na
author: spelluru
manager: ''
editor: ''
ms.assetid: ''
ms.service: devtest-lab
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/11/2018
ms.author: spelluru
ms.openlocfilehash: d87c8a46459a9b4bf80bef895ec97e436d38e699
ms.sourcegitcommit: bf522c6af890984e8b7bd7d633208cb88f62a841
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/20/2018
ms.locfileid: "39186827"
---
# <a name="store-secrets-in-a-key-vault-in-azure-devtest-labs"></a>在 Azure DevTest Labs 的金鑰保存庫中儲存秘密
您在使用 Azure DevTest Labs 時可能需要輸入複雜的祕密：Windows VM 的密碼、Linux VM 的公用 SSH 金鑰，或透過成品複製 Git 存放庫所需的個人存取權杖。 祕密通常很長，且包含隨機字元。 因此，在輸入時可能需要技巧且不便，特別是在您多次使用相同的祕密時。

為了解決此問題，並將您的秘密保存在安全之處，DevTest Labs 支援在 [Azure Key Vault](../key-vault/key-vault-overview.md) 中儲存祕密。 當使用者第一次儲存祕密時，DevTest Labs 服務會在包含實驗室的相同資源群組中自動建立金鑰保存庫，並將秘密儲存在金鑰保存庫中。 DevTest Labs 會為每個使用者建立個別的金鑰保存庫。 

## <a name="save-a-secret-in-azure-key-vault"></a>在 Azure Key Vault 中儲存秘密
若要在 Azure Key Vault 中儲存您的秘密，請執行下列步驟：

1. 選取左側功能表上的 [我的祕密]。
2. 輸入祕密的**名稱**。 在建立 VM、公式或環境時，您會在下拉式清單中看到此名稱。 
3. 輸入祕密作為**值**。

    ![儲存祕密](media/devtest-lab-store-secrets-in-key-vault/store-secret.png)

## <a name="use-a-secret-from-azure-key-vault"></a>使用 Azure Key Vault 中的祕密
當您需要輸入祕密以建立 VM、公式或環境時，您可以手動輸入祕密，或從金鑰保存庫中選取已儲存的祕密。 若要使用儲存在金鑰保存庫中的祕密，請執行下列動作：

1. 選取 [使用已儲存的祕密]。 
2. 從 [選擇祕密] 的下拉式清單中選取您的祕密。 

    ![在 VM 中使用祕密](media/devtest-lab-store-secrets-in-key-vault/secret-store-pick-a-secret.png)

## <a name="use-a-secret-in-an-azure-resource-manager-template"></a>在 Azure Resource Manager 範本中使用祕密
您可以在 Azure Resource Manager 範本中指定用來建立 VM 的秘密名稱，如下列範例所示：

![在公式或環境中使用祕密](media/devtest-lab-store-secrets-in-key-vault/secret-store-arm-template.png)

## <a name="next-steps"></a>後續步驟

- [使用祕密建立 VM](devtest-lab-add-vm.md) 
- [使用祕密建立公式](devtest-lab-manage-formulas.md)
- [使用祕密建立環境](devtest-lab-create-environment-from-arm.md)
