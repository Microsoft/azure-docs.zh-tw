---
title: Azure Stack 驗證即服務中的工作流程常見參數 | Microsoft Docs
description: Azure Stack 驗證即服務的工作流程常見參數
services: azure-stack
documentationcenter: ''
author: mattbriggs
manager: femila
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/24/2018
ms.author: mabrigg
ms.reviewer: johnhas
ms.openlocfilehash: c50e4b5c9eb81c9386e2cb0db96a88de70dcb9e9
ms.sourcegitcommit: 2d961702f23e63ee63eddf52086e0c8573aec8dd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/07/2018
ms.locfileid: "44157798"
---
# <a name="workflow-common-parameters-for-azure-stack-validation-as-a-service"></a>Azure Stack 驗證即服務的工作流程常見參數

[!INCLUDE [Azure_Stack_Partner](./includes/azure-stack-partner-appliesto.md)]

常見的參數包括值，例如驗證即服務 (VaaS) 中所有測試所需的環境變數和使用者認證。 您會在工作流程層級定義這些值。 您需在建立或修改工作流程時，儲存這些值。 在排程時間，工作流程會為測試載入這些值。 

## <a name="environment-parameters"></a>環境參數

環境參數會描述測試中的 Azure Stack 環境。 您必須藉由產生及上傳戳記組態檔來提供這些值 `&lt;link&gt;. [How to get the stamp info link].`

| 參數名稱 | 必要 | 類型 | 說明 |
|----------------------------------|----------|------|---------------------------------------------------------------------------------------------------------------------------------|
| Azure Stack 組建 | 必要 |  | Azure Stack 部署的組建編號 (例如 1.0.170330.9) |
| OEM 版本 | 是 |  | 在 Azure Stack 部署期間所使用 OEM 套件的版本號碼。 |
| OEM 簽章 | 是 |  | 在 Azure Stack 部署期間所使用 OEM 套件的簽章。 |
| AAD 租用戶識別碼 | 必要 |  | 在 Azure Stack 部署期間所指定的 Azure Active Directory 租用戶 GUID。|
| 區域 | 必要 |  | Azure Stack 部署區域。 |
| 租用戶 Resource Manager 端點 | 必要 |  | 租用戶 Azure Resource Manager 作業的端點 (例如 https://management.<ExternalFqdn>) |
| 管理員 Resource Manager 端點 | 是 |  | 租用戶 Azure Resource Manager 作業的端點 (例如 https://adminmanagement.<ExternalFqdn>) |
| 外部 FQDN | 是 |  | 用來作為端點尾碼的外部完整網域名稱 (FQDN)。 (例如 local.azurestack.external 或 redmond.contoso.com)。 |
| 節點數目 | 是 |  | 部署上的節點數目。 |

## <a name="test-parameters"></a>測試參數

常見的測試參數包括不能儲存在組態檔中而必須手動提供的機密資訊。

| 參數名稱 | 必要 | 類型 | 說明 |
|--------------------------------|------------------------------------------------------------------------------|------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 租用戶使用者名稱 | 必要 |  | 已經佈建或必須由「AAD 目錄」中的「服務管理員」佈建的「Azure Active Directory 租用戶管理員」。 如需有關佈建租用戶帳戶的詳細資料，請參閱[開始使用 Azure AD](https://docs.microsoft.com/azure/active-directory/get-started-azure-ad) \(英文\)。 測試會使用此值來執行租用戶層級的作業，例如部署範本來佈建資源 (VM、儲存體帳戶等) 及執行工作負載。 測試會使用此值來執行租用戶層級的作業，例如部署範本來佈建資源 (VM、儲存體帳戶等) 及執行工作負載。 |
| 租用戶密碼 | 必要 |  | 租用戶使用者的密碼。 |
| 服務管理員使用者名稱 | 必要：解決方案驗證、套件驗證<br>非必要：測試進行 |  | 在 Azure Stack 部署期間所指定「AAD 目錄租用戶」的「Azure Active Directory 管理員」。 |
| 服務管理員密碼 | 必要：解決方案驗證、套件驗證<br>非必要：測試進行 |  | 服務管理員使用者的密碼。 |
| 雲端管理員使用者名稱 | 必要 |  | Azure Stack 網域管理員帳戶 (例如 contoso\cloudadmin)。 請在組態檔中搜尋 User Role="CloudAdmin"，然後選取組態檔之 UserName 標記中的值。 |
| 雲端管理員密碼 | 必要 |  | 雲端管理員使用者的密碼。 |
| 診斷連接字串 | 必要 |  | 「Azure 儲存體帳戶」的 SAS URI，在測試執行期間將會把診斷記錄複製到該處。 如需有關產生 SAS URI 的指示，請參閱[設定 Blob 儲存體帳戶](azure-stack-vaas-set-up-account.md)。 |


## <a name="next-steps"></a>後續步驟

- 深入了解 [Azure Stack 驗證即服務](https://docs.microsoft.com/azure/azure-stack/partner)。
