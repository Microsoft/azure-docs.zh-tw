---
title: 在 Azure 入口網站中建立 Batch 帳戶 | Microsoft Docs
description: 了解如何在 Azure 入口網站中建立 Azure Batch 帳戶，以在雲端中執行大規模的平行工作負載
services: batch
documentationcenter: ''
author: dlepow
manager: jeconnoc
editor: ''
ms.assetid: 3fbae545-245f-4c66-aee2-e25d7d5d36db
ms.service: batch
ms.workload: big-compute
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: get-started-article
ms.date: 06/18/2018
ms.author: danlep
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: dfaee72be883ee8902fe4550890d757f114ff932
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39426100"
---
# <a name="create-a-batch-account-with-the-azure-portal"></a>使用 Azure 入口網站建立 Batch 帳戶

了解如何在 [Azure 入口網站][azure_portal]中建立 Azure Batch 帳戶，並選擇符合您計算案例的帳戶屬性。 了解如何尋找存取金鑰和帳戶 URL 等重要帳戶屬性。

如需 Batch 帳戶和案例的相關背景，請參閱[功能概觀](batch-api-basics.md)。

## <a name="create-a-batch-account"></a>建立批次帳戶：

[!INCLUDE [batch-account-mode-include](../../includes/batch-account-mode-include.md)]

1. 登入 [Azure 入口網站][azure_portal]。

1. 選取 [建立資源] > [計算] > [Batch 服務]。

    ![Marketplace 中的批次][marketplace_portal]

1. 輸入 [新增 Batch 帳戶] 設定。 請參閱下列詳細資料。

    ![建立批次帳戶：][account_portal]

    a. **帳戶名稱**：您選擇的名稱必須是將建立帳戶的 Azure 區域內的唯一名稱 (請參閱下面的「位置」)。 帳戶名稱只能包含小寫字元或數字，且長度必須為 3-24 個字元。

    b. **訂用帳戶**：要在其中建立 Batch 帳戶的訂用帳戶。 如果您只有一個訂用帳戶，則預設會選取此項目。

    c. **資源群組**：為新的 Batch 帳戶選取現有的資源群組，或選擇性地建立一個新的資源群組。

    d. **位置**：要在其中建立 Batch 帳戶的 Azure 區域。 只有您的訂用帳戶和資源群組所支援的區域會顯示為選項。

    e. **儲存體帳戶** (選用)：與 Batch 帳戶相關聯的 Azure 儲存體帳戶。 這是大部分 Batch 帳戶的建議作法。 如需 Batch 中的儲存體帳戶選項，請參閱 [Batch 功能概觀](batch-api-basics.md#azure-storage-account)。 在入口網站中選取現有的儲存體帳戶，或選擇性地建立新的帳戶。

      ![建立儲存體帳戶][storage_account]

    f. **集區配置模式**：在多數情況下，請接受預設的 [Batch 服務]。

1. 選取 [建立] 以建立帳戶。



## <a name="view-batch-account-properties"></a>檢視 Batch 帳戶屬性
一旦建立帳戶後，選取帳戶以存取其設定和屬性。 您可以使用左側功能表來存取所有的帳戶設定和屬性。

![Azure 入口網站中的 Batch 帳戶頁面][account_blade]

* **Batch 帳戶名稱、URL 和金鑰**︰當您使用 [Batch API](batch-apis-tools.md#azure-accounts-for-batch-development) 開發應用程式時，必須要有帳戶 URL 和金鑰才能存取 Batch 資源。 (Batch 也支援 Azure Active Directory 驗證。)

    若要檢視 Batch 帳戶存取資訊，請選取 [金鑰]。

    ![Azure 入口網站中的 Batch 帳戶金鑰][account_keys]

* 若要檢視與 Batch 帳戶相關聯的儲存體帳戶名稱和金鑰，請選取 [儲存體帳戶]。

* 若要檢視套用至 Batch 帳戶的資源配額，請選取 [配額]。 如需詳細資訊，請參閱 [Batch 服務配額和限制](batch-quota-limit.md)。


## <a name="additional-configuration-for-user-subscription-mode"></a>使用者訂用帳戶模式的其他組態

如果您選擇在使用者訂用帳戶模式中建立 Batch 帳戶，請在建立帳戶之前執行下列其他的步驟。

### <a name="allow-azure-batch-to-access-the-subscription-one-time-operation"></a>允許 Azure Batch 存取訂用帳戶 (一次性作業)
當您在使用者訂用帳戶模式中建立第一個 Batch 帳戶時，您必須向 Batch 註冊訂用帳戶。 (如果您先前已做過，請跳至下一節。)

1. 登入 [Azure 入口網站][azure_portal]。

1. 選取 [更多服務] > [訂用帳戶]，然後選取您想要用於 Batch 帳戶的訂用帳戶。

1. 在 [訂用帳戶] 頁面中，選取 [資源提供者]，並搜尋 **Microsoft.Batch**。 請檢查 **Microsoft.Batch** 資源提供者是否已在訂用帳戶中註冊。 如果未註冊，請選取**註冊**連結。

    ![註冊 Microsoft.Batch 提供者][register_provider]

1. 在 [訂用帳戶] 頁面中，選取 [存取控制 (IAM)] > [新增]。

    ![訂用帳戶存取控制][subscription_access]

1. 在 [新增權限] 頁面中，選取 [參與者] 角色，並搜尋 Batch API。 搜尋這些字串，直到您找到 API 為止：
    1. **MicrosoftAzureBatch**。
    1. **Microsoft Azure Batch**。 較新的 Azure AD 租用戶可以使用這個名稱。
    1. **ddbf3205-c6bd-46ae-8127-60eb93363864** 是 Batch API 的識別碼。 

1. 一旦您找到 Batch API 之後，請選取它，然後選取 [儲存]。

    ![新增 Batch 權限][add_permission]

### <a name="create-a-key-vault"></a>建立金鑰保存庫
在使用者訂用帳戶模式中，需要與要建立之 Batch 帳戶屬於相同資源群組的 Azure Key Vault。 請確定資源群組位於 Batch 為[可用](https://azure.microsoft.com/regions/services/)且您訂用帳戶支援的區域中。

1. 在 [Azure 入口網站][azure_portal]中，選取 [新增] > [安全性] > [Key Vault]。

1. 在 [建立 Key Vault] 頁面中，輸入 Key Vault 的名稱，並在您需要的 Batch 帳戶區域中建立資源群組。 將其餘設定保留為預設值，然後選取 [建立]。

在使用者訂用帳戶模式中建立 Batch 帳戶時，請使用金鑰保存庫的資源群組、指定 [使用者訂用帳戶] 作為集區配置模式，並選取金鑰保存庫。

## <a name="other-batch-account-management-options"></a>其他 Batch 帳戶管理選項
除了使用 Azure 入口網站以外，您也可以使用下列工具來建立及管理 Batch 帳戶︰

* [Batch PowerShell Cmdlet](batch-powershell-cmdlets-get-started.md)
* [Azure CLI](batch-cli-get-started.md)
* [Batch Management .NET](batch-management-dotnet.md)

## <a name="next-steps"></a>後續步驟
* 若要深入了解 Batch 服務概念和功能，請參閱 [Batch 功能概觀](batch-api-basics.md) 。 本文討論主要 Batch 資源 (例如集區、計算節點、作業和工作)，並提供能夠進行大規模計算工作負載的服務功能概觀。
* 了解使用 [Batch .NET 用戶端程式庫](quick-run-dotnet.md)或 [Python](quick-run-python.md) 開發啟用 Batch 之應用程式的基本概念。 這些快速入門會介紹使用 Batch 服務在多個計算節點上執行工作負載的範例應用程式，並說明如何使用 Azure 儲存體進行工作負載檔案的預備和擷取。

[azure_portal]: https://portal.azure.com
[batch_pricing]: https://azure.microsoft.com/pricing/details/batch/

[marketplace_portal]: ./media/batch-account-create-portal/marketplace-batch.png
[account_blade]: ./media/batch-account-create-portal/batch_blade.png
[account_portal]: ./media/batch-account-create-portal/batch-account-portal.png
[account_keys]: ./media/batch-account-create-portal/batch-account-keys.png
[account_url]: ./media/batch-account-create-portal/account_url.png
[storage_account]: ./media/batch-account-create-portal/storage_account.png
[subscription_access]: ./media/batch-account-create-portal/subscription_iam.png
[add_permission]: ./media/batch-account-create-portal/add_permission.png
[register_provider]: ./media/batch-account-create-portal/register_provider.png

