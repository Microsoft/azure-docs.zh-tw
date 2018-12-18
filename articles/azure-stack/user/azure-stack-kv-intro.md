---
title: Azure Stack Key Vault 簡介 | Microsoft Docs
description: 了解 Azure Stack Key Vault 如何管理金鑰和祕密
services: azure-stack
documentationcenter: ''
author: sethmanheim
manager: femila
editor: ''
ms.assetid: 70f1684a-3fbb-4cd1-bf29-9f9882e98fe9
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: get-started-article
ms.date: 08/15/2018
ms.author: sethm
ms.openlocfilehash: a6b4e8c3543d4681c92fbbde30eec0a543fcb0fd
ms.sourcegitcommit: d2f2356d8fe7845860b6cf6b6545f2a5036a3dd6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/16/2018
ms.locfileid: "43050410"
---
# <a name="introduction-to-key-vault-in-azure-stack"></a>Azure Stack 中的 Key Vault 簡介

## <a name="prerequisites"></a>必要條件 

* 您必須訂閱包含 Azure Key Vault 服務的供應項目。  
* [PowerShell 設定為搭配 Azure Stack 使用](azure-stack-powershell-configure-user.md)。
 
## <a name="key-vault-basics"></a>Key Vault 的基本概念
Azure Stack 中的 Key Vault 可協助保護雲端應用程式和服務所使用的密碼編譯金鑰和祕密。 藉由使用 Key Vault，您可以加密金鑰和秘密，例如：
   * 驗證金鑰 
   * 儲存體帳戶金鑰
   * 資料加密金鑰
   * .pfx 檔案
   * 密碼

金鑰保存庫簡化了金鑰管理程序，並可讓您控管存取和加密資料的金鑰。 開發人員可以在幾分鐘內建立開發和測試的金鑰，然後順利地將他們移轉至生產金鑰。 安全性系統管理員可以視需要授與 (和撤銷) 存取金鑰的權限。

只要擁有 Azure Stack 訂用帳戶，任何人都可以建立和使用金鑰保存庫。 雖然 Key Vault 對開發人員和安全性系統管理員有助益，但管理組織其他 Azure Stack 服務的操作員也可以實作並管理 Key Vault。 例如，Azure Stack 操作員可以使用 Azure Stack 訂用帳戶登入、建立組織要用來儲存金鑰的保存庫，然後負責執行以下操作作業：

* 建立或匯入金鑰或秘密。
* 撤銷或刪除金鑰或秘密。
* 授權使用者或應用程式存取金鑰保存庫，以便管理或使用其金鑰和祕密。
* 設定金鑰使用方法 (例如，簽署或加密)。

然後操作員可以提供統一資源識別項 (URI) 給開發人員，從其應用程式呼叫。 操作員也可以提供金鑰使用方法記錄資訊給安全性系統管理員。

開發人員也可透過使用 API 直接管理金鑰。 如需詳細資訊，請參閱《Key Vault 開發人員指南》。

## <a name="scenarios"></a>案例
下列案例說明 Key Vault 如何協助滿足開發人員和安全性系統管理員的需求。

### <a name="developer-for-an-azure-stack-application"></a>Azure Stack 應用程式的開發人員
**問題：** 我想要撰寫使用金鑰進行簽署和加密的 Azure Stack 應用程式。 我希望這些金鑰會與我的應用程式分開，如此一來此解決方案便適用於位於不同地點的應用程式。

**說明**：金鑰會儲存在保存庫中，並且視需要由 URI 叫用。

### <a name="developer-for-software-as-a-service-saas"></a>軟體即服務 (SaaS) 的開發人員
**問題**：對於客戶的金鑰和祕密，我不想承擔任何實際或潛在法律責任。 我希望客戶擁有並自行管理金鑰，所以我可以將全部精力集中在我的專長上，也就是提供核心軟體功能。

**說明**：客戶可以將他們自己的金鑰匯入 Azure Stack，然後加以管理。 

### <a name="chief-security-officer-cso"></a>資訊安全長 (CSO)
**問題**：我想要確定我的組織掌控著金鑰生命週期，並可監視金鑰使用狀況。

**說明**：Key Vault 的設計，便是讓 Microsoft 無法看見或擷取您的金鑰。 當應用程式必須使用客戶的金鑰來執行加密編譯密碼作業時，Key Vault 會代表應用程式來使用金鑰。 應用程式不會看到客戶的金鑰。 即使我們使用多個 Azure Stack 服務和資源，您可以從 Azure Stack 中的單一位置管理金鑰。 不論您在 Azure Stack 中有幾個保存庫、保存庫支援哪些區域，以及哪些應用程式使用這些保存庫，保存庫都可以提供單一介面。

## <a name="next-steps"></a>後續步驟

* [使用入口網站管理 Azure Stack 中的 Key Vault](azure-stack-kv-manage-portal.md)  
* [使用 PowerShell 管理 Azure Stack 中的 Key Vault](azure-stack-kv-manage-powershell.md)

