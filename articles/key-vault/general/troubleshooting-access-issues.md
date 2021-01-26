---
title: 針對 Azure Key Vault 存取原則問題進行疑難排解
description: 針對 Azure Key Vault 存取原則問題進行疑難排解
author: sebansal
ms.author: sebansal
ms.date: 08/10/2020
ms.service: key-vault
ms.subservice: general
ms.topic: how-to
ms.openlocfilehash: c5fab8b856ff9c82a0de887dc9c322dbf541348b
ms.sourcegitcommit: a055089dd6195fde2555b27a84ae052b668a18c7
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/26/2021
ms.locfileid: "98791402"
---
# <a name="troubleshooting-azure-key-vault-access-policy-issues"></a>針對 Azure Key Vault 存取原則問題進行疑難排解

## <a name="frequently-asked-questions"></a>常見問題集

### <a name="i-am-not-able-to-list-or-get-secretskeyscertificate-i-am-seeing-something-went-wrong-error"></a>我無法列出或取得祕密/金鑰/憑證。 我看到「發生錯誤...」錯誤。
如果您在列出/取得/建立或存取秘密時遇到問題，請確定您已定義存取原則以執行該作業：[金鑰保存庫存取原則](./assign-access-policy-cli.md)

### <a name="how-can-i-identify-how-and-when-key-vaults-are-accessed"></a>如何識別存取金鑰保存庫的方式和時間？

在建立一或多個金鑰保存庫之後，您可能會想要監視金鑰保存庫的存取方式、時間和存取者。 若要監視，您可以啟用 Azure Key Vault 的記錄功能，請[在此深入了解](./logging.md)啟用記錄功能的逐步指南。

### <a name="how-can-i-monitor-vault-availability-service-latency-periods-or-other-performance-metrics-for-key-vault"></a>如何監視金鑰保存庫的保存庫可用性、服務延遲期間或其他效能計量？

開始調整服務時，傳送至金鑰保存庫的要求數將會增加。 這項要求會增加要求的延遲，而且在極端情況下，會導致您的要求受到節流處理，進而影響服務效能。 您可以監視金鑰保存庫效能計量，並取得特定閾值的警示，請[在此深入了解](./alert.md)設定監視功能的逐步指南。

### <a name="i-am-not-able-to-modify-access-policy-how-can-it-be-enabled"></a>我無法修改存取原則，請問如何啟用？
使用者必須有足夠的 AAD 權限才能修改存取原則。 在此情況下，使用者必須擁有較高的參與者角色。

### <a name="i-am-seeing-unknown-policy-error-what-does-that-mean"></a>我看到「未知的原則」錯誤。 這代表什麼？
您有兩種不同的可能性會看到未知的存取原則：
* 可能是先前有存取權的使用者，現在因為某些原因而不存在了。
* 是否透過 powershell 新增存取原則，而且是為了應用程式 objectid (而不是服務主體) 新增存取原則

### <a name="how-can-i-assign-access-control-per-key-vault-object"></a>如何為每個金鑰保存庫物件指派存取控制？ 

您可以在這裡收到每個秘密/金鑰/憑證存取控制功能的可用性通知，[深入了解](https://feedback.azure.com/forums/906355-azure-key-vault/suggestions/32213176-per-secret-key-certificate-access-control)

### <a name="how-can-i-provide-key-vault-authenticate-using-access-control-policy"></a>如何使用存取控制原則來提供金鑰保存庫驗證？

若要驗證 Key Vault 的雲端式應用程式，最簡單的方法是使用受控識別；如需詳細資訊，請參閱[向 Azure Key Vault 進行驗證](authentication.md)。
如果您要建立內部部署應用程式、進行本機開發，或無法使用受控識別，您可以改為以手動方式註冊服務主體，並使用存取控制原則提供金鑰保存庫的存取權。 請參閱[指派存取控制原則](assign-access-policy-portal.md)。

### <a name="how-can-i-give-the-ad-group-access-to-the-key-vault"></a>如何為 AD 群組授與金鑰保存庫的存取權？

使用 Azure CLI 的 `az keyvault set-policy` 命令或 Azure PowerShell 的 Set-AzKeyVaultAccessPolicy Cmdlet，為 AD 群組授與金鑰保存庫的權限。 請參閱[指派存取原則 - CLI](assign-access-policy-cli.md) 和[指派存取原則 - PowerShell](assign-access-policy-powershell.md)。

應用程式也需要至少一個指派給金鑰保存庫的身分識別與存取管理 (IAM) 角色。 否則，應用程式將無法登入，且將會因為沒有足夠的權限可存取訂用帳戶而失敗。 具有受控識別的 Azure AD 群組可能需要多達八小時才能重新整理權杖並生效。

### <a name="how-can-i-redeploy-key-vault-with-arm-template-without-deleting-existing-access-policies"></a>如何使用 ARM 範本重新部署 Key Vault，而不需要刪除現有的存取原則？

目前只要重新部署 Key Vault，就會刪除 Key Vault 中的任何存取原則，並將其取代為 ARM 範本中的存取原則。 Key Vault 存取原則沒有累加選項。 若要在 Key Vault 中保留存取原則，您必須在 Key Vault 中讀取現有的存取原則，並以這些原則填入 ARM 範本，以避免任何存取中斷。

另一個有助於此案例的選項是使用 Azure RBAC 和角色做為存取原則的替代方案。 使用 Azure RBAC，您可以重新部署金鑰保存庫，而不需要再次指定原則。 您可以在[這裡](./rbac-guide.md)深入閱讀此解決方案。

### <a name="recommended-troubleshooting-steps-for-following-error-types"></a>下列錯誤類型的建議疑難排解步驟

* HTTP 401：未經驗證的要求 - [疑難排解步驟](rest-error-codes.md#http-401-unauthenticated-request)
* HTTP 403：權限不足 - [疑難排解步驟](rest-error-codes.md#http-403-insufficient-permissions)
* HTTP 429：太多要求 - [疑難排解步驟](rest-error-codes.md#http-429-too-many-requests)
* 檢查您是否有金鑰保存庫的刪除存取權限：請參閱[指派存取原則 - CLI](assign-access-policy-cli.md) 和[指派存取原則 - PowerShell](assign-access-policy-powershell.md) 或[指派存取原則 - 入口網站](assign-access-policy-portal.md)。
* 如果您在透過程式碼向金鑰保存庫進行驗證時發生問題，請使用[驗證 SDK](https://azure.github.io/azure-sdk/posts/2020-02-25/defaultazurecredentials.html)

### <a name="what-are-the-best-practices-i-should-implement-when-key-vault-is-getting-throttled"></a>當金鑰保存庫受到節流處理時，我應該執行哪些最佳做法？
請遵循[此處](overview-throttling.md#how-to-throttle-your-app-in-response-to-service-limits)所記載的最佳做法

## <a name="next-steps"></a>後續步驟

了解如何對金鑰保存庫驗證錯誤進行疑難排解：[Key Vault 疑難排解指南](rest-error-codes.md)。