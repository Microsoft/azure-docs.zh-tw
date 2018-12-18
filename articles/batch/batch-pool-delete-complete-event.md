---
title: Azure Batch 集區刪除完成事件 | Microsoft Docs
description: Batch 集區刪除完成事件的參考。
services: batch
author: dlepow
manager: jeconnoc
ms.assetid: ''
ms.service: batch
ms.devlang: multiple
ms.topic: article
ms.tgt_pltfrm: ''
ms.workload: big-compute
ms.date: 04/20/2017
ms.author: danlep
ms.openlocfilehash: bfcbcf40efc64ab1c79ee1a86e02502c68ad6d47
ms.sourcegitcommit: 20d103fb8658b29b48115782fe01f76239b240aa
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/03/2018
ms.locfileid: "30310309"
---
# <a name="pool-delete-complete-event"></a>集區刪除完成事件

 集區刪除作業完成時，就會發出此事件。

 下列事件顯示集區刪除完成事件內文。

```
{
    "id": "myPool1",
    "startTime": "2016-09-09T22:13:48.579Z",
    "endTime": "2016-09-09T22:14:08.836Z"
}
```

|元素|類型|注意|
|-------------|----------|-----------|
|id|字串|集區識別碼。|
|startTime|Datetime|集區刪除開始時間。|
|EndTime|DateTime|集區刪除完成時間。|

## <a name="remarks"></a>備註
如需集區調整大小作業狀態與錯誤碼的詳細資訊，請參閱[將集區自帳戶中刪除](https://docs.microsoft.com/rest/api/batchservice/delete-a-pool-from-an-account) (英文)