---
title: 包含檔案
description: 包含檔案
services: functions
author: ggailey777
ms.service: functions
ms.topic: include
ms.date: 09/21/2018
ms.author: glenga
ms.custom: include file
ms.openlocfilehash: f1b53c53b1e5fb089eb9b8a9b816b11a1eea126d
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47044504"
---
當您在本機開發函式時，您可以從終端機或命令提示字元中使用 Azure Functions Core Tools，來安裝需要的擴充。

在更新 function.json 檔案以包含函式所需的所有繫結後，請在專案資料夾中執行下列命令。

```bash
func extensions install
```

此命令會讀取 function.json 檔案，查看您需要哪些套件，加以安裝，然後重建擴充專案。 它會在目前版本中新增任何新繫結，但不會更新現有的繫結。 在安裝新的繫結時，使用 `--force` 選項將現有繫結更新為最新版本。

如果您想要安裝特定版本的套件，或是要在編輯 function.json 檔案之前安裝套件，請使用 `func extensions install` 命令搭配套件的名稱，如下列範例所示：

```bash
func extensions install --package Microsoft.Azure.WebJobs.ServiceBus --version <target_version>
```

以特定版本的套件取代 `<target_version>`，例如 `3.0.0-beta5`。 有效的版本會列在 [NuGet.org](https://nuget.org) 的個別套件頁面上。
