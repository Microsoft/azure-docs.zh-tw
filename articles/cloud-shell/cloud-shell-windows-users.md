---
title: 適用於 Windows 使用者的 Azure Cloud Shell | Microsoft Docs
description: 引導不熟悉 Linux 系統之使用者的指南
services: azure
documentationcenter: ''
author: maertendMSFT
manager: hemantm
tags: azure-resource-manager
ms.assetid: ''
ms.service: azure
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: na
ms.topic: article
ms.date: 08/03/2018
ms.author: damaerte
ms.openlocfilehash: aad474195060c01a3f9d85e6f9037b568b0c16ad
ms.sourcegitcommit: 4de6a8671c445fae31f760385710f17d504228f8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/08/2018
ms.locfileid: "39630381"
---
# <a name="powershell-in-azure-cloud-shell-for-windows-users"></a>Azure Cloud Shell 中的 PowerShell (適用於 Windows 使用者)

在 2018 年 5 月，Microsoft [宣佈](https://azure.microsoft.com/blog/pscloudshellrefresh/)變更 Azure Cloud Shell 中的 PowerShell。
Azure Cloud Shell 中的 PowerShell 體驗現在會在 Linux 環境中執行 [PowerShell Core 6](https://github.com/powershell/powershell)。
變更後，相較於 Windows PowerShell 體驗中的預期內容，Cloud Shell 中的 PowerShell 體驗可能會有某些差異。

## <a name="file-system-case-sensitivity"></a>檔案系統區分大小寫

在 Windows 中，檔案系統不區分大小寫，而在 Linux 上，檔案系統會區分大小寫。
先前系統會將 `file.txt` 與 `FILE.txt` 視為同一個檔案，不過現在則會視為不同的檔案。
在檔案系統中進行 `tab-completing` 時，必須使用正確的大小寫。
PowerShell 特有體驗 (如 `tab-completing` Cmdlet 名稱、參數和值) 則不區分大小寫。

## <a name="windows-powershell-aliases-vs-linux-utilities"></a>Windows PowerShell 別名與 Linux 公用程式

某些現有 PowerShell 別名具有與內建 Linux 命令 (例如 `cat`、`ls`、`sort`、`sleep` 等等) 相同的名稱。PowerShell Core 6 已移除與內建 Linux 命令衝突的別名。
以下是已移除的常見別名以及對等的命令：  

|已移除的別名   |對等命令   |
|---|---|
|`cat`    | `Get-Content` |
|`curl`   | `Invoke-WebRequest` |
|`diff`   | `Compare-Object` |
|`ls`     | `dir` <br> `Get-ChildItem` |
|`mv`     | `Move-Item`   |
|`rm`     | `Remove-Item` |
|`sleep`  | `Start-Sleep` |
|`sort`   | `Sort-Object` |
|`wget`   | `Invoke-WebRequest` |

## <a name="persisting-home"></a>保留 $HOME

以前，使用者只能在其雲端磁碟機中保留指令碼和其他檔案。
現在，使用者的 $HOME 目錄也能跨工作階段保留下來。

## <a name="powershell-profile"></a>PowerShell 設定檔

根據預設，系統不會建立使用者的 PowerShell 設定檔。
若要建立設定檔，請在 `$HOME/.config` 之下建立 `PowerShell` 目錄。

```azurepowershell-interactive
mkdir (Split-Path $profile.CurrentUserAllHosts)
```

在 `$HOME/.config/PowerShell` 底下，您可以建立設定檔 - `profile.ps1` 和/或 `Microsoft.PowerShell_profile.ps1`。

## <a name="whats-new-in-powershell-core-6"></a>PowerShell Core 6 的新功能

如需 PowerShell Core 6 新功能的詳細資訊，請參考 [PowerShell 文件](https://docs.microsoft.com/powershell/scripting/whats-new/what-s-new-in-powershell-core-60?view=powershell-6)和[開始使用 PowerShell Core](https://blogs.msdn.microsoft.com/powershell/2017/06/09/getting-started-with-powershell-core-on-windows-mac-and-linux/) 的部落格文章。
