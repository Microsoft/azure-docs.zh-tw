---
title: Azure Key Vault 概觀 | Microsoft Docs
description: Azure Key Vault 是一項雲端服務，可作為安全的祕密存放區。
services: key-vault
author: barclayn
manager: mbaldwin
tags: azure-resource-manager
ms.assetid: 34af20ee-3fa7-4f28-9d98-6168b1759764
ms.service: key-vault
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: overview
ms.custom: mvc
ms.date: 07/17/2018
ms.author: barclayn
ms.openlocfilehash: aae7836448ff27b4c80d7bb53e108034ee52db1c
ms.sourcegitcommit: 5843352f71f756458ba84c31f4b66b6a082e53df
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/01/2018
ms.locfileid: "47586286"
---
# <a name="what-is-azure-key-vault"></a>什麼是 Azure 金鑰保存庫？

Azure Key Vault 可協助您解決下列問題
- **祕密管理** - Azure Key Vault 可用來安全地儲存權杖、密碼、憑證、API 金鑰和其他祕密，並嚴密控制其存取
- **金鑰管理** - Azure Key Vault 也可作為金鑰管理解決方案。 Azure Key Vault 可讓您輕鬆地建立和控制用來加密資料的加密金鑰。 
- **憑證管理** - Azure Key Vault 也是一項服務，可讓您輕鬆地佈建、管理及部署 Azure 和您內部連線的資源所使用的公用和私人安全通訊端層/傳輸層安全性 (SSL/TLS) 憑證。 
- **儲存受到硬體安全性模型支援的祕密** - 祕密和金鑰可受到軟體的保護，或由通過 FIPS 140-2 Level 2 驗證的 HSM 保護

## <a name="why-use-azure-key-vault"></a>為何使用 Azure Key Vault？

### <a name="centralize-application-secrets"></a>集中儲存應用程式祕密

在 Azure Key Vault 中集中儲存應用程式祕密，可讓您控制其散發。 Key Vault 可大幅降低不小心洩露祕密的風險。 使用 Key Vault 時，應用程式開發人員不再需要在其應用程式中儲存安全性資訊。 這樣就不需要讓此資訊成為程式碼的一部分。 例如，應用程式可能需要連線到資料庫。 在 Key Vault 中妥善保存連接字串，而不用在應用程式程式碼中儲存連接字串。

您的應用程式可以使用 URI 安全地存取所需的資訊，而在應用程式的金鑰或祕密儲存在 Azure Key Vault 之後，URI 可讓它們能擷取特定的祕密版本。 發生這種情況，不需要撰寫自訂程式碼來保護任何祕密資訊。

### <a name="securely-store-secrets-and-keys"></a>安全地儲存秘密和金鑰

Azure 使用會業界標準演算法、金鑰長度和硬體安全性模組 (HSM) 來保護秘密和金鑰。 使用的 HSM 經過美國聯邦資訊處理標準 (FIPS) 140-2 Level 2 驗證。

金鑰保存庫的存取權需要先經過適當的驗證和授權，呼叫者 (使用者或應用程式) 才能取得存取權。 驗證會建立呼叫者的身分識別，授權則會判斷呼叫者可以執行的作業。

驗證會透過 Azure Active Directory 進行。 授權可以透過角色型存取控制 (RBAC) 或 Key Vault 存取原則進行。 在處理保存庫的管理時會使用 RBAC，而在嘗試存取保存庫中儲存的資料時會使用金鑰保存庫存取原則。

Azure Key Vault 可能受軟體或硬體 HSM 保護。 在您需要加強保證的情況下，您可以在硬體安全模組 (HSM) 中匯入或產生無需離開 HSM 界限的金鑰。 Microsoft 會使用 Thales 硬體安全性模型。 您可以使用 Thales 工具將金鑰從您的 HSM 移至 Azure Key Vault。

最後，Azure Key Vault 依設計會使 Microsoft 無法看見或擷取您的資料。

### <a name="monitor-access-and-use"></a>監視存取和使用

在您建立一些 Key Vault 之後，您會想要監視存取金鑰和祕密的方式和時機。 您可以啟用 Key Vault 的記錄功能來達成目的。 您可以如下設定 Azure Key Vault：

- 封存至儲存體帳戶。
- 串流至事件中樞。
- 將記錄傳送至 Log Analytics。

您可以控制您的記錄，藉由限制存取權來保護它們，也可以刪除您不再需要的記錄。

### <a name="simplified-administration-of-application-secrets"></a>簡化應用程式祕密的管理

在儲存珍貴資料時，您必須採取幾個步驟。 安全性資訊必須受到保護，它必須遵循生命週期，同時必須保持高可用性。 Azure Key Vault 簡化此作業的方式如下：

- 無須具備硬體安全性模組的內部知識
- 在短時間內相應增加，以符合組織的使用尖峰。
- 將區域內的 Key Vault 內容複寫到次要區域。 這可確保高可用性，並免除管理員採取任何動作來觸發容錯移轉的需求。
- 透過入口網站、Azure CLI 和 PowerShell 提供標準 Azure 系統管理選項。
- 自動對向公開 CA 購買的憑證執行一些作業，例如註冊或續約。

此外，Azure Key Vault 可讓您隔離應用程式祕密。 應用程式只能存取它們有權存取的保存庫，且受限於僅只執行特定作業。 您可以為每個應用程式建立 Azure Key Vault，並將 Key Vault 中儲存的祕密限制於特定應用程式和開發人員小組。

### <a name="integrate-with-other-azure-services"></a>與其他 Azure 服務整合

Key Vault 是 Azure 中的安全存放區，可用來簡化 [Azure 磁碟加密](../security/azure-security-disk-encryption.md)、SQL 伺服器和 Azure SQL Database 中的[一律加密]( https://docs.microsoft.com/sql/relational-databases/security/encryption/always-encrypted-database-engine)功能、[Azure Web 應用程式]( https://docs.microsoft.com/azure/app-service/web-sites-purchase-ssl-web-site)等案例。 Key Vault 本身可與儲存體帳戶、事件中樞和記錄分析整合。

## <a name="next-steps"></a>後續步驟

- [快速入門：使用 CLI 建立 Azure Key Vault](quick-create-cli.md)
- [將 Azure Web 應用程式設定為從 Key Vault 讀取祕密](tutorial-web-application-keyvault.md)
