---
title: 命名 Azure Data Factory 實體的規則 |Microsoft Docs
description: 描述 Data Factory 實體的命名規則。
services: data-factory
documentationcenter: ''
author: sharonlo101
manager: craigg
ms.assetid: bc5e801d-0b3b-48ec-9501-bb4146ea17f1
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 01/16/2018
ms.author: shlo
ms.openlocfilehash: cfab1a82c7da0ad596c9989e5a9f3ed800c58e4a
ms.sourcegitcommit: e3d5de6d784eb6a8268bd6d51f10b265e0619e47
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/01/2018
ms.locfileid: "39389305"
---
# <a name="azure-data-factory---naming-rules"></a>Azure Data Factory - 命名規則
下表提供 Data Factory 購件命名規則。

| Name | 名稱唯一性 | 驗證檢查 |
|:--- |:--- |:--- |
| Data Factory |在 Microsoft Azure 中是唯一的。 名稱不區分大小寫，亦即 `MyDF` 和 `mydf` 會指相同的 Data Factory。 |<ul><li>每個 Data Factory 只會繫結至一個 Azure 訂用帳戶。</li><li>物件名稱必須以字母或數字開頭，並且只能包含字母、數字和虛線 (-) 字元。</li><li>每個虛線 (-) 字元的前面和後面必須緊接字母或數字。 容器名稱中不允許使用連續虛線。</li><li>名稱長度可以介於 3-63 個字元。</li></ul> |
| 連結服務/資料集/管線 |在 Data Factory 中是唯一的。 名稱不區分大小寫。 |<ul><li>物件名稱開頭必須為字母、數字或底線 (_)。</li><li>不允許使用下列字元：“.”、“+”、“?”、“/”、“<”、”>”、”*”、”%”、”&”、”:”、”\\”</li><li>只有已連結服務和資料集的名稱中不允許使用連字號 ("-")。</li></ul>  |
| 資源群組 |在 Microsoft Azure 中是唯一的。 名稱不區分大小寫。 | 如需詳細資訊，請參閱 [Azure 命名規則和限制](https://docs.microsoft.com/azure/architecture/best-practices/naming-conventions#naming-rules-and-restrictions)。 |

## <a name="next-steps"></a>後續步驟
遵循[快速入門：建立資料處理站](quickstart-create-data-factory-powershell.md)一文中的逐步指示，以了解如何建立資料處理站。 
