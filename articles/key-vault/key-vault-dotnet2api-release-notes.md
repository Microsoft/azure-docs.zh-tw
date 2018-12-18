---
title: 金鑰保存庫 .NET 2.x API 版本資訊 | Microsoft Docs
description: .NET 開發人員會使用這個 API 來編寫 Azure 金鑰保存庫的程式碼
services: key-vault
author: bryanla
manager: mbaldwin
editor: bryanla
ms.assetid: 1cccf21b-5be9-4a49-8145-483b695124ba
ms.service: key-vault
ms.devlang: CSharp
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 05/02/2017
ms.author: bryanla
ms.openlocfilehash: f2bcace1ba328aff20971b46880f317295f3a406
ms.sourcegitcommit: 0fcd6e1d03e1df505cf6cb9e6069dc674e1de0be
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/14/2018
ms.locfileid: "42144793"
---
# <a name="azure-key-vault-net-20---release-notes-and-migration-guide"></a>Azure 金鑰保存庫 .NET 2.0 - 版本資訊和移轉指南
下列資訊可協助您移轉至適用於 C# 和 .NET 的 Azure Key Vault 文件庫 2.0 版。  您必須更新針對舊版所撰寫的應用程式，才能支援最新版本。  必須進行這些變更才能完整支援新功能和改進功能，例如 **Key Vault 憑證**。

## <a name="key-vault-certificates"></a>Key Vault 憑證

Key Vault 憑證可管理 x509 憑證，並支援下列行為：  

* 透過 Key Vault 建立流程來建立憑證，或是匯入現有憑證。 這同時包含自我簽署和憑證授權單位 (CA) 所產生的憑證。
* 安全地儲存和管理 X509 憑證儲存區，而不需要使用私密金鑰資料進行互動。  
* 定義原則來將 Key Vault 導向，以管理憑證生命週期。  
* 提供生命週期事件的連絡資訊，例如到期警告和更新通知。  
* 自動更新所選簽發者 (Key Vault 合作夥伴 X509 憑證提供者和憑證授權單位) 的憑證。* 支援替代 (非合作夥伴) 提供者和憑證授權單位 (不支援自動更新) 所核發的憑證。  

## <a name="net-support"></a>.NET 支援

* Azure Key Vault .NET 程式庫 2.0 版不支援 **.NET 4.0**
* Azure Key Vault .NET 程式庫 2.0 版不支援 **.NET Framework 4.5.2**
* Azure Key Vault .NET 程式庫 2.0 版不支援 **.NET Standard 1.4**

## <a name="namespaces"></a>命名空間

* **模型**的命名空間會從 **Microsoft.Azure.KeyVault** 變更為 **Microsoft.Azure.KeyVault.Models**。
* 卸除 **Microsoft.Azure.KeyVault.Internal** 命名空間。
* 下列 Azure SDK 相依性命名空間具有 

    - **Hyak.Common** 現在為 **Microsoft.Rest**。
    - **Hyak.Common.Internals** 現在為 **Microsoft.Rest.Serialization**。

## <a name="type-changes"></a>類型變更

* *Secret* 已變更為 *SecretBundle*
* *Dictionary* 已變更為 *IDictionary*
* *List<T>, string []* 已變更為 *IList<T>*
* *NextList* 已變更為 *NextPageLink*

## <a name="return-types"></a>傳回類型

* **KeyList** 和 **SecretList** 現在會傳回 IPage<T>，而非 ListKeysResponseMessage
* 產生的 **BackupKeyAsync** 現在會傳回 BackupKeyResult，其包含 Value (備份 Blob)。 此方法在過去會進行包裝並只傳回值。

## <a name="exceptions"></a>例外狀況

* *KeyVaultClientException* 變更為 *KeyVaultErrorException*
* 服務錯誤已從 exception.Error 變更為 exception.Body.Error.Message。
* 已移除 **[JsonExtensionData]** 的錯誤訊息中的其他資訊。

## <a name="constructors"></a>建構函式

* 建構函式只會接受 *HttpClientHandler* 或 *DelegatingHandler[]*，而不是接受 *HttpClient* 做為建構函式引數。

## <a name="downloaded-packages"></a>下載的套件

用戶端在處理 Key Vault 相依性時會下載下列套件：

### <a name="previous-package-list"></a>先前的套件清單

* `package id="Hyak.Common" version="1.0.2" targetFramework="net45"`
* `package id="Microsoft.Azure.Common" version="2.0.4" targetFramework="net45"`
* `package id="Microsoft.Azure.Common.Dependencies" version="1.0.0" targetFramework="net45"`
* `package id="Microsoft.Azure.KeyVault" version="1.0.0" targetFramework="net45"`
* `package id="Microsoft.Bcl" version="1.1.9" targetFramework="net45"`
* `package id="Microsoft.Bcl.Async" version="1.0.168" targetFramework="net45"`
* `package id="Microsoft.Bcl.Build" version="1.0.14" targetFramework="net45"`
* `package id="Microsoft.Net.Http" version="2.2.22" targetFramework="net45"`

### <a name="current-package-list"></a>目前套件清單

* `package id="Microsoft.Azure.KeyVault" version="2.0.0-preview" targetFramework="net45"`
* `package id="Microsoft.Rest.ClientRuntime" version="2.2.0" targetFramework="net45"`
* `package id="Microsoft.Rest.ClientRuntime.Azure" version="3.2.0" targetFramework="net45"`

## <a name="class-changes"></a>類別變更

* 已移除 **UnixEpoch** 類別。
* **Base64UrlConverter** 類別已重新命名為 **Base64UrlJsonConverter**。

## <a name="other-changes"></a>其他變更

* 這個版本的 API 已新增暫時性失敗的 KV 作業重試原則組態支援。

## <a name="microsoftazuremanagementkeyvault-nuget"></a>Microsoft.Azure.Management.KeyVault NuGet

* 針對傳回 vault 的作業，傳回類型是包含 **Vault** 屬性的類別。 傳回類型現在是 *Vault*。
* *PermissionsToKeys* 和 *PermissionsToSecrets* 現在是 *Permissions.Keys* 和 *Permissions.Secrets*
* 某些傳回類型變更也會套用至控制平面。

## <a name="microsoftazurekeyvaultextensions-nuget"></a>Microsoft.Azure.KeyVault.Extensions NuGet

* 套件最多分成 **Microsoft.Azure.KeyVault.Extensions** 和 **Microsoft.Azure.KeyVault.Cryptography** 來進行密碼編譯作業。

