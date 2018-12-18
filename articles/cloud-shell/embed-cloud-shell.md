---
title: 內嵌 Azure Cloud Shell | Microsoft Docs
description: 了解如何內嵌 Azure Cloud Shell。
services: cloud-shell
documentationcenter: ''
author: jluk
manager: timlt
tags: azure-resource-manager
ms.assetid: ''
ms.service: azure
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: na
ms.topic: article
ms.date: 12/11/2017
ms.author: juluk
ms.openlocfilehash: 3ceddb94336fc2703e6f916f05ab1ec3676cb50d
ms.sourcegitcommit: f1c1789f2f2502d683afaf5a2f46cc548c0dea50
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/18/2018
ms.locfileid: "27864879"
---
# <a name="embed-azure-cloud-shell"></a>內嵌 Azure Cloud Shell

內嵌 Cloud Shell 可讓開發人員和內容撰寫人員直接從專用 URL ([shell.azure.com](https://shell.azure.com)) 開啟 Cloud Shell。這可立即為使用者提供 Cloud Shell 驗證的完整功能、工具處理和最新的 Azure CLI/Azure PowerShell 工具。

標準大小的按鈕

[![](https://shell.azure.com/images/launchcloudshell.png "啟動 Azure Cloud Shell")](https://shell.azure.com)

大型按鈕

[![](https://shell.azure.com/images/launchcloudshell@2x.png "啟動 Azure Cloud Shell")](https://shell.azure.com)

## <a name="how-to"></a>作法

複製下列內容，將 Cloud Shell 的啟動按鈕整合到 Markdown 檔案中：

```markdown
[![Launch Cloud Shell](https://shell.azure.com/images/launchcloudshell.png "Launch Cloud Shell")](https://shell.azure.com)
```

要內嵌快顯 Cloud Shell 的 HTML 如下：
```html
<a style="cursor:pointer" onclick='javascript:window.open("https://shell.azure.com", "_blank", "toolbar=no,scrollbars=yes,resizable=yes,menubar=no,location=no,status=no")'><image src="https://shell.azure.com/images/launchcloudshell.png" /></a>
```

## <a name="customize-experience"></a>自訂體驗

擴大 URL 以設定特定的 Shell 體驗。
|體驗   |URL   |
|---|---|
|最近使用的 Shell   |shell.azure.com           |
|Bash                       |shell.azure.com/bash       |
|PowerShell                 |shell.azure.com/powershell |

## <a name="next-steps"></a>後續步驟
[Cloud Shell 中 Bash 的快速入門](quickstart.md)<br>
[Cloud Shell 中 PowerShell 的快速入門](quickstart-powershell.md)
