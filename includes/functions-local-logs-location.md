---
title: 包含檔案
description: 包含檔案
services: functions
author: ggailey777
manager: cfowler
ms.service: functions
ms.topic: include
ms.date: 05/01/2018
ms.author: glenga
ms.custom: include file
ms.openlocfilehash: 88c01e8e57d4a92478b8b1ca0689ff0f8e499b39
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38739371"
---
Functions 主機在本機執行時，會將記錄寫入至下列路徑：

```
<DefaultTempDirectory>\LogFiles\Application\Functions
```

在 Windows 上，第一個找到的 TMP、TEMP、USERPROFILE 環境變數值會是 `<DefaultTempDirectory>`，或是 Windows 目錄。
在 MacOS 或 Linux 上，`<DefaultTempDirectory>` 則是 TMPDIR 環境變數。

> [!NOTE]
> Functions 主機啟動時，會覆寫該目錄中現有的檔案結構。