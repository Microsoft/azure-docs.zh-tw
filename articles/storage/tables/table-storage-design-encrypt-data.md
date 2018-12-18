---
title: 加密 Azure 儲存體資料表資料 | Microsoft Docs
description: 了解 Azure 儲存體中的資料表資料加密。
services: storage
author: MarkMcGeeAtAquent
ms.service: storage
ms.topic: article
ms.date: 04/11/2018
ms.author: sngun
ms.component: tables
ms.openlocfilehash: 8a7b61705f6efc5a76212fdd1345a022f58e0686
ms.sourcegitcommit: 9819e9782be4a943534829d5b77cf60dea4290a2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39526461"
---
# <a name="encrypt-table-data"></a>加密資料表資料
.NET Azure 儲存體用戶端程式庫支援在插入和取代作業時進行字串實體屬性的加密。 加密的字串儲存在服務上作為二進位屬性，且解密後會轉換回字串。    

針對資料表，除了加密原則之外，使用者必須指定要加密的屬性。 作法是指定 [EncryptProperty] 屬性 (針對衍生自 TableEntity 的 POCO 實體)，或在要求選項中指定加密解析程式。 加密解析程式是委派，接受資料分割索引鍵、資料列索引鍵和屬性名稱，然後傳回布林值，指出是否應該加密該屬性。 在加密期間，用戶端程式庫會使用此資訊，決定將屬性寫到網路時是否應該加密。 委派也提供關於屬性如何加密的可能邏輯。 (例如，如果 X，則加密屬性 A，否則加密屬性 A 和 B。)讀取或查詢實體時不需要提供這項資訊。

## <a name="merge-support"></a>合併支援

目前不支援合併。 因為屬性子集先前可能是使用不同的金鑰加密，直接合併新的屬性並更新中繼資料會導致資料遺失。 合併可能需要額外的服務呼叫來從服務讀取預先存在的實體，或在每個屬性上都使用新的金鑰，兩者都不利用於效能。     

如需加密資料表資料的相關資訊，請參閱 [Microsoft Azure 儲存體的用戶端加密和 Azure 金鑰保存庫](../common/storage-client-side-encryption.md)。  

## <a name="next-steps"></a>後續步驟

- [資料表設計模式](table-storage-design-patterns.md)
- [將關聯性模型化](table-storage-design-modeling.md)
- [將關聯性模型化](table-storage-design-modeling.md)
- [資料修改的設計](table-storage-design-for-modification.md)