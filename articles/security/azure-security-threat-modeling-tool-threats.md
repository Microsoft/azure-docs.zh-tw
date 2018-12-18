---
title: 威脅 - Microsoft 威脅模型化工具 - Azure | Microsoft Docs
description: Microsoft 威脅模型化工具的威脅類別分頁，包含大多數所產生之暴露威脅的類別。
services: security
documentationcenter: na
author: RodSan
manager: RodSan
editor: RodSan
ms.assetid: na
ms.service: security
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/17/2017
ms.author: rodsan
ms.openlocfilehash: 704f9995828866d4d2e4969e3aa922ed1e23c4ea
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "23043303"
---
# <a name="microsoft-threat-modeling-tool-threats"></a>Microsoft 威脅模型化工具威脅

威脅模型化工具是 Microsoft 安全性開發生命週期 (SDL) 的核心元素。 它可讓軟體架構設計人員及早識別和降低潛在安全性問題的風險，以在問題相對簡單且符合成本效益時加以解決。 因此，可大幅降低總開發成本。 此外，我們在設計此工具時已考慮到非安全性專家的問題，因此可讓所有開發人員更加方便地建立威脅模型，為他們提供清楚說明如何建立和分析威脅模型的指導方針。

> 請造訪**[威脅模型化工具](./azure-security-threat-modeling-tool.md)** 立即開始使用！

威脅模型化工具可協助您回答某些問題，例如下列問題︰

* 攻擊者如何變更驗證資料？
* 如果攻擊者可以讀取使用者設定檔資料，會有什麼影響？
* 如果對於使用者設定檔資料庫的存取遭拒會發生什麼事？

## <a name="stride-model"></a>STRIDE 模型

為了協助您更加清楚地闡述上面指出的這些問題，Microsoft 使用了 STRIDE 模型，它會將不同類型的威脅分類，並簡化整體安全性對話。

| 類別 | 說明 |
| -------- | ----------- |
| **詐騙** | 涉及非法存取然後使用其他使用者的驗證資訊，例如使用者名稱和密碼 |
| **竄改** | 涉及惡意修改資料。 範例包括未經授權變更持續性資料 (例如保存在資料庫中的資料)，以及修改在兩部電腦之間透過開放式網路 (例如網際網路) 流動的資料 |
| **否認性** | 與拒絕執行動作但沒有其他任何一方有辦法另外證明的使用者有關，比方說，使用者在無法追蹤禁止作業的系統中執行非法作業。 不可否認性是指系統對抗否認性威脅的能力。 例如，購買商品的使用者可能必須在收到商品時簽名。 然後，廠商可以使用簽名收據做為使用者已收到包裹的證據 |
| **資訊洩漏** | 涉及將資訊暴露給不應該具有其存取權的個人，例如，使用者可以讀取它們未被授權存取的檔案，或入侵者能夠讀取在兩部電腦之間傳輸的資料 |
| **阻斷服務** | 阻斷服務 (DoS) 攻擊可阻斷對有效使用者提供的服務，例如，藉由讓網頁伺服器暫時無法存取或無法使用。 您必須防止特定類型的 DoS 威脅，以便提升系統的可用性和可靠性 |
| **權限提高** | 未授權的使用者取得授權的存取權，因此有足夠存取權危害或摧毀整個系統。 提高權限威脅包含下列情況：攻擊者已有效地滲透所有系統防禦，並成為受信任系統本身的一部分，這確實是危險的情況 |

## <a name="next-steps"></a>後續步驟

繼續**[威脅模型化工具風險降低](./azure-security-threat-modeling-tool-mitigations.md)** 以了解您可以使用 Azure 緩和這些威脅的不同方式。