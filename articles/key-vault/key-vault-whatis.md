---
title: 什麼是 Azure 金鑰保存庫？ | Microsoft Docs
description: Azure 金鑰保存庫可協助保護雲端應用程式和服務所使用的密碼編譯金鑰和密碼。 使用 Azure 金鑰保存庫之後，客戶可以加密金鑰和密碼 (例如驗證金鑰、儲存體帳戶金鑰、資料加密金鑰、.PFX 檔案和密碼)，方法是使用受硬體安全模組 (HSM) 保護的金鑰。
services: key-vault
documentationcenter: ''
author: barclayn
manager: mbaldwin
tags: azure-resource-manager
ms.assetid: e759df6f-0638-43b1-98ed-30b3913f9b82
ms.service: key-vault
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: get-started-article
ms.date: 08/02/2018
ms.author: barclayn
ms.openlocfilehash: 26828efedac9953ce1c7375fc62269e93019ce50
ms.sourcegitcommit: f6e2a03076679d53b550a24828141c4fb978dcf9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43094865"
---
# <a name="what-is-azure-key-vault"></a>什麼是 Azure 金鑰保存庫？

Azure Key Vault 可協助您解決下列問題
- Azure Key Vault 可用來安全地儲存權杖、密碼、憑證、API 金鑰和其他祕密，並嚴密控制其存取
- Azure Key Vault 也可作為金鑰管理解決方案。 Azure Key Vault 可讓您輕鬆地建立和控制用來加密資料的加密金鑰。 
- Azure Key Vault 也是一項服務，可讓您輕鬆地佈建、管理及部署 Azure 和您內部連線的資源所使用的公用和私人安全通訊端層/傳輸層安全性 (SSL/TLS) 憑證。 
- 祕密和金鑰可受到軟體的保護，或由通過 FIPS 140-2 Level 2 驗證的 HSM 保護

## <a name="basic-concepts"></a>基本概念

Azure Key Vault 是可安全儲存及存取祕密的工具。 祕密是指任何需受到嚴密存取控制的項目，例如 API 金鑰、密碼或憑證。
以下提供部分主要詞彙：
- **租用戶**：租用戶是擁有並且可管理特定 Microsoft 雲端服務執行個體的組織。 租用戶最常以明確的方式用來參照組織的 Azure 和 Office 365 服務集合。
- **保存庫擁有者**：保存庫擁有者可建立金鑰保存庫，並取得其完整存取權和控制權。 保存庫擁有者也可以設定稽核，以記錄存取祕密和金鑰的人員。 系統管理員可控制金鑰的生命週期。 他們可將金鑰變換為新版本、備份金鑰，以及執行相關工作。
- **資源**：資源是可透過 Azure 提供的可管理項目。 部分常見資源有虛擬機器、儲存體帳戶、Web 應用程式、資料庫和虛擬網路，但這只是其中一小部分。
- **資源群組**：資源群組是可保存 Azure 解決方案相關資源的容器。 資源群組可以包含方案的所有資源，或只包含您要以群組方式管理的資源。 您可決定如何根據對組織最有利的方式，將資源配置到資源群組。
- **保存庫取用者**：保存庫取用者可在保存庫擁有者為取用者授與存取權時，對金鑰保存庫內的資產執行動作。 可用的動作取決於授與的權限。
- **[Azure Active Directory (Azure AD)](../active-directory/active-directory-whatis.md)**：Azure AD 是租用戶的 Active Directory 服務。 每個目錄都有一或多個網域。 一個目錄可以有多個相關聯的訂用帳戶，但只能有一個租用戶。 
- **Azure 租用戶識別碼**：租用戶識別碼是對 Azure 訂用帳戶內的 Azure AD 執行個體進行識別的唯一方法。
- **受控服務識別**：Azure Key Vault 可讓您安全地儲存認證和其他金鑰及秘密，但是您的程式碼必須向 Key Vault 進行驗證，才可擷取這些項目。 受控服務識別可以輕易地解決此問題，因為它可在 Azure AD 中將自動受控識別提供給 Azure 服務。 您可以使用此身分識別向 Key Vault 或任何支援 Azure AD 驗證的服務進行驗證，而無須在程式碼中使用任何認證。 如需詳細資訊，請參閱[適用於 Azure 資源的受控服務識別](../active-directory/managed-service-identity/overview.md)。

    ![受控服務識別運作方式圖表](./media/key-vault-whatis/msi.png)

## <a name="key-vault-roles"></a>Key Vault 角色

使用下表將有助於瞭解金鑰保存庫可以如何協助達到開發人員和安全性系統管理員的需求。

| 角色 | 問題陳述 | Azure 金鑰保存庫已解決問題 |
| --- | --- | --- |
| Azure 應用程式的開發人員 |「我想要撰寫使用金鑰進行簽署和加密的 Azure 應用程式，但我希望這些金鑰會與我的應用程式分開，如此一來此解決方案便適用於位於不同地點的應用程式。 <br/><br/>我希望這些金鑰和密碼會受到保護，但我可以不用自己撰寫程式碼。 我也希望從我的應用程式就可以輕鬆使用這些金鑰和密碼，並得到最佳效能。」 |√ 金鑰會儲存在保存庫中，並且視需要由 URI 叫用。<br/><br/> √ Azure 使用會業界標準演算法、金鑰長度和硬體安全性模組保護金鑰。<br/><br/> √ 金鑰會在與應用程式位於相同 Azure 資料中心的 HSM 中處理。 此方法會比金鑰位於不同位置 (例如內部部署) 提供更高的可靠性並減少延遲。 |
| 軟體即服務 (SaaS) 的開發人員 |「對於客戶之租用戶的金鑰和密碼，我不想承擔任何實際或潛在法律責任。 <br/><br/>我希望客戶擁有並自行管理金鑰，這樣我就可以將全部精力集中在我的專長上，也就是提供核心軟體功能。」 |√ 客戶可以將他們自己的金鑰匯入 Azure 並加以管理。 當 SaaS 應用程式必須使用其客戶金鑰來執行加密編譯密碼作業時，金鑰保存庫會代表應用程式執行這些作業。 應用程式不會看到客戶的金鑰。 |
| 資訊安全長 (CSO) |「我想要知道我們的應用程式是否遵守 FIPS 140-2 Level 2 HSM 的安全金鑰管理規定。 <br/><br/>我想要確定我的組織妥善掌控金鑰生命週期，並可監視金鑰使用情形。 <br/><br/>而且即使我們使用多個 Azure 服務和資源，我仍想要從 Azure 中的單一位置管理金鑰。」 |√ HSM 已通過 FIPS 140-2 Level 2 驗證。<br/><br/>√ 金鑰保存庫經過設計，所以 Microsoft 不會看見或擷取您的金鑰。<br/><br/>√ 金鑰使用情形會近乎即時地進行記錄。<br/><br/>√ 不論您在 Azure 中有幾個保存庫、保存庫支援哪些區域，以及哪些應用程式使用這些保存庫，保存庫均提供單一介面讓您清楚了解。 |

只要擁有 Azure 訂用帳戶，任何人都可以建立和使用金鑰保存庫。 雖然金鑰保存庫有益於開發人員和安全性系統管理員，但管理組織其他 Azure 服務的組織系統管理員也可以實作並管理金鑰保存庫。 例如，此系統管理員可以使用 Azure 訂用帳戶登入、建立組織要用來儲存金鑰的保存庫，然後負責執行營運工作，例如：

* 建立或匯入金鑰或密碼
* 撤銷或刪除金鑰或密碼
* 授權使用者或應用程式存取金鑰保存庫，以便他們管理或使用其金鑰和密碼
* 設定金鑰使用方法 (例如，簽署或加密)
* 監視金鑰使用方法

此系統管理員接著可以為開發人員提供可從其應用程式進行呼叫的 URI，並為其安全性系統管理員提供金鑰使用方法記錄資訊。 

![Azure 金鑰保存庫概觀][1]

開發人員也可透過使用 API 直接管理金鑰。 如需詳細資訊，請參閱 [金鑰保存庫開發人員指南](key-vault-developers-guide.md)。

## <a name="next-steps"></a>後續步驟

如需適用於系統管理員的開始使用教學課程，請參閱[開始使用 Azure Key Vault](key-vault-get-started.md)。

如需 Key Vault 使用記錄的詳細資訊，請參閱 [Azure Key Vault 記錄](key-vault-logging.md)。

如需搭配 Azure Key Vault 使用金鑰和秘密的詳細資訊，請參閱 [關於金鑰、秘密和憑證](https://msdn.microsoft.com/library/azure/dn903623\(v=azure.1\).aspx)。

<!--Image references-->
[1]: ./media/key-vault-whatis/AzureKeyVault_overview.png
大部分地區均提供 Azure 金鑰保存庫。 如需詳細資訊，請參閱 [金鑰保存庫價格頁面](https://azure.microsoft.com/pricing/details/key-vault/)。
