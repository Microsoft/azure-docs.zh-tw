---
title: 風險降低 - Microsoft 威脅模型化工具 - Azure | Microsoft Docs
description: Microsoft 威脅模型化工具的風險降低頁面，反白顯示大多數所產生之暴露威脅的可能解決方案。
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
ms.openlocfilehash: 07ef1fd3d81d795c9164741d22b5a689f86bd720
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "23043433"
---
# <a name="microsoft-threat-modeling-tool-mitigations"></a>Microsoft 威脅模型化工具風險降低

威脅模型化工具是 Microsoft 安全性開發生命週期 (SDL) 的核心元素。 它可讓軟體架構設計人員及早識別和降低潛在安全性問題的風險，以在問題相對簡單且符合成本效益時加以解決。 因此，可大幅降低總開發成本。 此外，我們在設計此工具時已考慮到非安全性專家的問題，因此可讓所有開發人員更加方便地建立威脅模型，為他們提供清楚說明如何建立和分析威脅模型的指導方針。

請造訪**[威脅模型化工具](./azure-security-threat-modeling-tool.md)** 立即開始使用！

## <a name="mitigation-categories"></a>風險降低類別

威脅模型化工具風險降低措施會根據 Web 應用程式安全性架構來分類，其包含下列類別︰

| 類別 | 說明 |
| -------- | ----------- |
| **[稽核和記錄](./azure-security-threat-modeling-tool-auditing-and-logging.md)** | 誰在何時做了什麼？ 稽核和記錄是指應用程式記錄安全性相關事件的方式 |
| **[驗證](./azure-security-threat-modeling-tool-authentication.md)** | 你是誰？ 驗證是某實體證明另一個實體身分識別的程序 (通常透過使用者名稱和密碼等認證) |
| **[授權](./azure-security-threat-modeling-tool-authorization.md)** | 您該怎麼辦？ 授權是應用程式針對資源和作業提供存取控制的方式 |
| **[通訊安全性](./azure-security-threat-modeling-tool-communication-security.md)** | 您在與誰對話？ 通訊安全性可確保所有通訊是在最安全的情況下進行的 |
| **[設定管理](./azure-security-threat-modeling-tool-configuration-management.md)** | 應用程式的執行身分為何？ 其所連線到的資料庫為何？ 應用程式的管理方式為何？ 如何保護這些設定？ 組態管理指的是應用程式處理這些作業問題的方式 |
| **[密碼編譯](./azure-security-threat-modeling-tool-cryptography.md)** | 如何保護機密資料 (機密性)？ 如何防止資料或程式庫遭到竄改 (完整性)？ 如何為必須是密碼編譯增強式的隨機值提供種子？ 密碼編譯是指應用程式強制執行機密性與完整性的方式 |
| **[例外狀況管理](./azure-security-threat-modeling-tool-exception-management.md)** | 當應用程式中的方法呼叫失敗時，應用程式會如何處理？ 您要顯示多少資料？ 您要對使用者傳回容易理解的錯誤資訊嗎？ 您要將重要例外狀況資訊傳回給呼叫者嗎？ 應用程式會正常地失敗嗎？ |
| **[輸入驗證](./azure-security-threat-modeling-tool-input-validation.md)** | 您如何知道應用程式收到的輸入是有效且安全的？ 輸入驗證指的是應用程式如何先篩選、消除或拒絕輸入再進行其他處理。 請考慮透過進入點限制輸入並透過退出點編碼輸出。 您是否信任來源的資料，例如來自資料庫和檔案共用？ |
| **[敏感資料](./azure-security-threat-modeling-tool-sensitive-data.md)** | 應用程式如何處理敏感性資料？ 敏感性資料指的是應用程式如何處理記憶體中、透過網路或持續性存放區中必須受到保護的任何資料 |
| **[工作階段管理](./azure-security-threat-modeling-tool-session-management.md)** | 應用程式如何處理和保護使用者工作階段？ 工作階段是指使用者與 Web 應用程式之間的一系列相關互動 |

這可協助您識別︰

* 最常犯的錯誤
* 最可行的改進措施

因此，您可以使用這些類別來聚焦在您的安全性工作並設定其優先順序，以便在您知道輸入驗證、身分驗證和授權類別中所發生的最普遍安全性問題時，可從中來開始著手。 如需詳細資訊，請造訪**[這個專利連結](https://www.google.com/patents/US7818788)**

## <a name="next-steps"></a>後續步驟

請造訪**[威脅模型化工具威脅](./azure-security-threat-modeling-tool-threats.md)** 以深入了解工具用來產生可能設計威脅的威脅類別。