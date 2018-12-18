---
title: 從 Azure Active Directory 存取控制服務移轉至共用存取簽章授權 | Microsoft Docs
description: 將應用程式從存取控制服務移轉至 SAS
services: service-bus-relay
documentationcenter: ''
author: clemensv
manager: timlt
editor: ''
ms.service: service-bus-relay
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 12/20/2017
ms.author: spelluru
ms.openlocfilehash: 39764ce2dce8e00d1b41179e7285a0cea1c808bf
ms.sourcegitcommit: cb61439cf0ae2a3f4b07a98da4df258bfb479845
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/05/2018
ms.locfileid: "43698939"
---
# <a name="migrate-from-azure-active-directory-access-control-service-to-shared-access-signature-authorization"></a>從 Azure Active Directory 存取控制服務移轉至共用存取簽章授權

Azure 轉送應用程式在過去可選擇使用兩個不同的授權模型：由轉送服務直接提供的[共用存取簽章 (SAS)](../service-bus-messaging/service-bus-sas.md) 權杖模型，以及同盟模型，其中授權規則的管理是由 [Azure Active Directory](/azure/active-directory/) 存取控制服務 (ACS) 內部管理，而從 ACS 取得的權杖會傳遞至轉送，以授權存取所需的功能。

ACS 授權模型長期以來已被 [SAS 授權](../service-bus-messaging/service-bus-authentication-and-authorization.md)取代為首選模型，而且今日所有文件、指南和範例皆僅使用 SAS。 此外，無法再建立與 ACS 配對的新轉送命名空間。

SAS 具備的優點在於它不會立即依賴於另一個服務，而是可以藉由提供 SAS 規則名稱和規則索引鍵的用戶端存取權，從用戶端直接使用，而無需任何媒介。 SAS 也可以與一種方法輕鬆整合，用戶端必須先通過其他服務的授權檢查，然後才會發出權杖。 後者方法類似於 ACS 使用模式，但是可以根據在 ACS 中難以表達的應用程式特定條件發出存取權杖。

對於所有相依於 ACS 的現有應用程式，我們鼓勵客戶盡快移轉應用程式，改為依賴 SAS。

## <a name="migration-scenarios"></a>移轉案例

ACS 和轉送會透過*簽署金鑰*的共用知識整合。 ACS 命名空間使用簽署金鑰以簽署授權權杖，而且 Azure 轉送使用該金鑰驗證權杖是否已由配對的 ACS 命名空間發出。 ACS 命名空間會保留服務識別和授權規則。 授權規則定義外部識別提供者發出哪個服務識別或權杖，以最長前置詞相符的形式，取得對部分轉送命名空間圖表的存取權類型。

例如，ACS 規則可將路徑前置詞 `/` 上的**傳送**宣告授與服務識別，這意味著 ACS 根據該規則發出的權杖會授與用戶端權限，使之得以傳送給命名空間中的所有實體。 如果路徑前置詞是 `/abc`，則身分識別會限制為傳送至名稱為 `abc` 的實體或該前置詞下的組織。 假設本移轉指南的讀者已熟悉這些概念。

移轉情節可分成三大類：

1.  **保持不變的預設值**。 某些客戶會使用 [SharedSecretTokenProvider](/dotnet/api/microsoft.servicebus.sharedsecrettokenprovider) 物件，將自動產生的**擁有者**服務識別及其 ACS 命名空間的秘密金鑰 (與傳送命名空間配對) 傳遞出去，而且不會新增規則。

2.  **簡單規則的自訂服務識別**。 某些客戶為了某個特定實體，會新增服務識別，並授與每個新服務識別**傳送**、**接聽**和**管理**的權限。

3.  **複雜規則的自訂服務識別**。 極少數客戶擁有複雜的規則集，其中外部發出的權杖會對應至轉送上的權限，或者單一服務識別會透過多條規則而指派了數個命名空間路徑的不同權限。

如需移轉複雜規則集的協助，可以連絡 [Azure 支援](https://azure.microsoft.com/support/options/)。 其他兩種情況可以直接移轉。

### <a name="unchanged-defaults"></a>保持不變的預設值

如果您的應用程式未變更 ACS 預設值，您可以使用 [SharedAccessSignatureTokenProvider](/dotnet/api/microsoft.servicebus.sharedaccesssignaturetokenprovider) 物件取代所有 [SharedSecretTokenProvider](/dotnet/api/microsoft.servicebus.sharedsecrettokenprovider) 的使用方式，並使用預先設定的 **RootManageSharedAccessKey** 命名空間，而非 ACS **擁有者**帳戶。 請注意，即使是使用 ACS **擁有者**帳戶，通常不建議使用這項設定，因為此帳戶/規則提供了對命名空間的完整管理權限，包括刪除任何實體的權限。

### <a name="simple-rules"></a>簡單規則

如果應用程式使用規則簡單的自訂服務識別，則當 ACS 建立了服務識別，以在特定的轉送上供應存取權限，則移轉就變特別簡單了。 在 SaaS 樣式的解決方案中，通常會出現種情況，每個轉送會被當作通向租用戶網站或分支辦公室的橋樑，並且為該特定網站建立服務識別。 在此情況下，相應的服務識別可以直接轉送至轉送上的共用存取簽章規則。 服務識別名稱可以成為 SAS 規則名稱，而服務識別金鑰可以成為 SAS 規則索引鍵。 然後將 SAS 規則的權限設定為等同於實體個別適用的 ACS 規則。

您可以在與 ACS 同盟的任何現有命名空間上，就地建立 SAS 的新設定和其他設定，並且接著使用 [SharedAccessSignatureTokenProvider](/dotnet/api/microsoft.servicebus.sharedaccesssignaturetokenprovider) 而非 [SharedSecretTokenProvider](/dotnet/api/microsoft.servicebus.sharedsecrettokenprovider)，移轉出 ACS。 命名空間不需要從 ACS 取消連結。

### <a name="complex-rules"></a>複雜規則

SAS 規則並非要作為帳戶使用，而是與權限相關的具名簽署金鑰。 因此，在應用程式建立許多服務識別，並授與它們對多個實體或整個命名空間之存取權限的情況下，仍然需要權杖發放媒介。 您可以藉由[連絡支援人員](https://azure.microsoft.com/support/options/)取得此類媒介的指引。

## <a name="next-steps"></a>後續步驟

若要深入了解 Azure 轉送驗證，請參閱下列主題：

* [Azure 轉送驗證和授權](relay-authentication-and-authorization.md)
* [使用共用存取簽章的服務匯流排驗證](../service-bus-messaging/service-bus-sas.md)


