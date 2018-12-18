---
title: 包含檔案
description: 包含檔案
services: functions
author: ggailey777
manager: jeconnoc
ms.service: functions
ms.topic: include
ms.date: 08/12/2018
ms.author: glenga
ms.custom: include file
ms.openlocfilehash: 3cbe634d862682a5f6b06c2cfc77a4d3b03954f9
ms.sourcegitcommit: 58c5cd866ade5aac4354ea1fe8705cee2b50ba9f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/24/2018
ms.locfileid: "42809657"
---
特定函式應用程式中所有函式的程式碼，都位於包含主機設定檔和一或多個子資料夾的根資料夾 (`wwwroot`) 中。 每個子資料夾都包含個別函式的程式碼，如以下範例所示：

```
wwwroot
 | - host.json
 | - mynodefunction
 | | - function.json
 | | - index.js
 | | - node_modules
 | | | - ... packages ...
 | | - package.json
 | - mycsharpfunction
 | | - function.json
 | | - run.csx
 | - bin
 | | - mycompiledcsharp.dll
```

host.json 檔案包含一些執行階段特定的組態，並位在函數應用程式的根資料夾中。 如需可用設定的相關資訊，請參閱 [host.json 參考](../articles/azure-functions/functions-host-json.md)。

每個函數都會有一個資料夾，其中包含一或多個程式碼檔案、function.json 組態及其他相依性。 針對 C# 類別庫專案，已編譯的類別庫 (.dll) 檔案會部署到 `bin` 子資料夾。

