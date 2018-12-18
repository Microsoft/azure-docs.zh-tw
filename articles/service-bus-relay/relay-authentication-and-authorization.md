---
title: Azure 轉送驗證和授權 | Microsoft Docs
description: Azure 轉送中的共用存取簽章 (SAS) 驗證概觀
services: service-bus-relay
documentationcenter: na
author: spelluru
manager: timlt
editor: ''
ms.assetid: ''
ms.service: service-bus-relay
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 01/23/2018
ms.author: spelluru
ms.openlocfilehash: fff484f2ee6722083a34383b8960931cb37b8316
ms.sourcegitcommit: cb61439cf0ae2a3f4b07a98da4df258bfb479845
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/05/2018
ms.locfileid: "43700441"
---
# <a name="azure-relay-authentication-and-authorization"></a>Azure 轉送驗證和授權

應用程式可以使用共用存取簽章 (SAS) 驗證向 Azure 轉送進行驗證。 SAS 驗證可讓應用程式使用轉送命名空間中所設定的存取金鑰，向 Azure 轉送服務進行驗證。 您可以接著使用此金鑰來產生共用存取簽章權杖，以便用戶端用來向轉送服務進行驗證。

## <a name="shared-access-signature-authentication"></a>共用存取簽章驗證

[SAS 驗證](../service-bus-messaging/service-bus-sas.md)可讓您授與使用者具有特定權限之 Azure 轉送資源的存取權。 SAS 驗證牽涉到在資源上設定具有相關權限的密碼編譯金鑰。 接著用戶端可以藉由提出 SAS 權杖 (其中包含正在存取的資源 URI 及利用設定金鑰簽署的到期日) 存取該資源。

您可以在轉送命名空間上設定 SAS 的金鑰。 不同於服務匯流排傳訊，[轉送混合式連線](relay-hybrid-connections-protocol.md)支援未經授權或匿名的寄件者。 您可以在建立實體時啟用匿名存取，如下列入口網站中的螢幕擷取畫面所示︰

![][0]

若要使用 SAS，您可以在轉送命名空間上設定包含下列項目的 [SharedAccessAuthorizationRule](/dotnet/api/microsoft.servicebus.messaging.sharedaccessauthorizationrule) 物件：

* *KeyName* 。
* *PrimaryKey* 是用來簽署/驗證 SAS 權杖的密碼編譯金鑰。
* *SecondaryKey* 是用來簽署/驗證 SAS 權杖的密碼編譯金鑰。
* *權限* 表示接聽、傳送或管理授與權限的集合。

在命名空間層級設定的授權規則可以授與命名空間中所有轉送連線的存取權給具備使用對應金鑰簽署之權杖的用戶端。 在轉送命名空間上最多可以設定 12 條這類授權規則。 根據預設，具備所有權限的 [SharedAccessAuthorizationRule](/dotnet/api/microsoft.servicebus.messaging.sharedaccessauthorizationrule) 會在第一次佈建時為每個命名空間設定。

若要存取實體，用戶端需要使用特定 [SharedAccessAuthorizationRule](/dotnet/api/microsoft.servicebus.messaging.sharedaccessauthorizationrule)產生的 SAS 權杖。 SAS 權杖乃是使用資源字串的 HMAC-SHA256 所產生，該字串由宣告存取的資源 URI 以及具備與授權規則相關聯之密碼編譯金鑰的過期所組成。

Azure 轉送的 SAS 驗證支援包含在 Azure .NET SDK 2.0 版或更新版本中。 SAS 包括 [SharedAccessAuthorizationRule](/dotnet/api/microsoft.servicebus.messaging.sharedaccessauthorizationrule)的支援。 所有接受連接字串做為參數的 API 包括 SAS 連接字串的支援。

## <a name="next-steps"></a>後續步驟

- 如需 SAS 的詳細資料，請繼續閱讀[使用共用存取簽章的服務匯流排驗證](../service-bus-messaging/service-bus-sas.md)。
- 如需混合式連線功能的詳細資訊，請參閱 [Azure 轉送混合式連線通訊協定指南](relay-hybrid-connections-protocol.md)。
- 如需服務匯流排傳訊驗證和授權的相關對應資訊，請參閱[服務匯流排驗證和授權](../service-bus-messaging/service-bus-authentication-and-authorization.md)。 

[0]: ./media/relay-authentication-and-authorization/hcanon.png