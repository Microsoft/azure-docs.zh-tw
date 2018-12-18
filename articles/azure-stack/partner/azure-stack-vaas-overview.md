---
title: Azure Stack 的驗證即服務概觀 | Microsoft Docs
description: Azure Stack 驗證即服務的已知問題概觀。
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
ms.openlocfilehash: c96e7385356354d398108ad69492603d38667e46
ms.sourcegitcommit: 3f8f973f095f6f878aa3e2383db0d296365a4b18
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/20/2018
ms.locfileid: "40234505"
---
# <a name="what-is-validation-as-a-service-for-azure-stack"></a>什麼是 Azure Stack 的驗證即服務？

[!INCLUDE[Azure_Stack_Partner](./includes/azure-stack-partner-appliesto.md)]

驗證即服務 (VaaS) 是原生的 Azure 服務，專供與 Microsoft 共同設計製造 Azure Stack 供應項目的解決方案合作夥伴使用。 解決方案合作夥伴可以使用此服務，檢查其解決方案是否符合 Microsoft 的需求並可搭配 Azure Stack 正常運作。

VaaS 的主要用途為：

- 驗證新的 Azure Stack 解決方案
- 驗證對 Azure Stack 軟體進行的變更
- 取得部署期間所使用以數位方式簽署的解決方案合作夥伴套件
- 預覽 Azure Stack 驗證附隨品

## <a name="validate-new-azure-stack-solution"></a>驗證新的 Azure Stack 解決方案

合作夥伴可使用解決方案驗證工作流程來檢查新的 Azure Stack 解決方案。 解決方案必須通過必要的 Hardware Lab Kit (HKL) Azure Stack 測試元件測試。 您只需為每個新的解決方案執行此工作流程兩次：最小值和最大值設定各一次。

如需詳細資訊，請參閱[驗證新的 Azure Stack 解決方案](azure-stack-vaas-validate-solution-new.md)。

## <a name="validate-changes-to-the-azure-stack-software"></a>驗證對 Azure Stack 軟體進行的變更

合作夥伴可使用套件驗證工作流程，來檢查其解決方案是否可與最近的 Azure Stack 軟體更新搭配運作。 套件驗證工作流程必須至少在 Microsoft 所建議的硬體環境上執行，以及在已使用修補程式和更新 (P&U) 來套用更新的解決方案上執行。 您應該在基準組建上執行套件驗證。

如需詳細資訊，請參閱[驗證來自 Microsoft 的軟體更新](azure-stack-vaas-validate-microsoft-updates.md)。

## <a name="get-digitally-signed-solution-partner-packages"></a>取得以數位方式簽署的解決方案合作夥伴套件

除了驗證 Azure Stack 更新之外，您還可以使用套件驗證工作流程來檢查對 OEM 自訂套件進行的更新，這包括 Azure Stack 合作夥伴特定的驅動程式、韌體及其他在 Azure Stack 軟體部署期間所使用的軟體。 請至少使用將可支援的最小大小解決方案，將您要檢查的套件部署在目前版本的 Azure Stack 軟體上。 進行測試之前，必須先將已更新的套件上傳至服務。 如果測試成功，請通知 vaashelp@microsoft.com。 告訴 Azure Stack 合作夥伴該套件已完成測試，應該使用 Azure Stack 數位簽章以數位方式簽署該套件。 Microsoft 會簽署該套件，並通知 Azure Stack 合作夥伴該套件已在入口網站中可供下載。

如需詳細資訊，請參閱[驗證 OEM 套件](azure-stack-vaas-validate-oem-package.md)。

## <a name="preview-azure-stack-validation-collateral"></a>預覽 Azure Stack 驗證附隨品

Microsoft 會定期在 Azure Stack 中提供新的功能。 在為市場提供這些功能的開發過程中，會在測試進行工作流程中提供新的測試附隨品。 測試進行工作流程包含來自其他工作流程的測試附隨品，用來允許執行非官方測試。 請勿使用測試進行工作流程來提交結果以供核准。 請使用解決方案驗證和套件驗證工作流程，來為您的解決方案取得官方核准。

## <a name="next-steps"></a>後續步驟

- 開始著手進行，並[設定您的驗證即服務帳戶](azure-stack-vaas-validate-solution-new.md)
