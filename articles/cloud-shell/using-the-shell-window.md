---
title: 使用 Azure Cloud Shell 視窗 | Microsoft Docs
description: 如何使用 Azure Cloud Shell 視窗的概觀。
services: azure
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
ms.date: 01/30/2018
ms.author: juluk
ms.openlocfilehash: 43da2bf5b66ff7db03a6fb5c2e1ceaebe322bcbb
ms.sourcegitcommit: 9d317dabf4a5cca13308c50a10349af0e72e1b7e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 02/01/2018
ms.locfileid: "28919989"
---
# <a name="using-the-azure-cloud-shell-window"></a>使用 Azure Cloud Shell 視窗

本文件說明如何使用 Cloud Shell 視窗。

## <a name="swap-between-bash-and-powershell-environments"></a>在 Bash 與 PowerShell 環境之間交換
![](media/using-the-shell-window/env-selector.png)

使用 Cloud Shell 工具列中的環境選取器，在 Bash 與 PowerShell 環境之間交換。

## <a name="restart-cloud-shell"></a>重新啟動 Cloud Shell
![](media/using-the-shell-window/restart.png)
> [!WARNING]
> 重新啟動 Cloud Shell 將會重設電腦狀態，而所有未由 Azure 檔案共用保存的檔案都將遺失。

* 按一下 Cloud Shell 工具列中的重新啟動圖示，即可重設電腦狀態。

## <a name="minimize--maximize-cloud-shell-window"></a>將 Cloud Shell 視窗最大化和最小化
![](media/using-the-shell-window/minmax.png)
* 按一下視窗右上方的 [最小化] 圖示，即可將視窗隱藏。 再按一下 [Cloud Shell] 圖示，即可取消隱藏。
* 按一下 [最大化] 圖示，即可將視窗設成最大高度。 若要將視窗還原到先前的大小，請按一下 [還原]。

## <a name="concurrent-sessions"></a>並行工作階段
Cloud Shell 可透過允許每個工作階段以個別 Bash 處理序的形式存在，來允許跨瀏覽器索引標籤進行多個並行工作階段。
如果要結束工作階段，請務必從每個工作階段視窗結束，因為每個處理序雖然是在相同的電腦上執行，但卻是獨立執行的。

## <a name="copy-and-paste"></a>複製和貼上
[!INCLUDE [copy-paste](../../includes/cloud-shell-copy-paste.md)]

## <a name="resize-cloud-shell-window"></a>調整 Cloud Shell 視窗大小
* 按一下工具列上邊緣並向上或向下拖曳，即可調整 Cloud Shell 視窗大小。

## <a name="scrolling-text-display"></a>捲動文字顯示
* 使用您的滑鼠或觸控板來捲動終端機文字。

## <a name="changing-the-text-size"></a>變更文字大小
![](media/using-the-shell-window/text-size.png)
* 按一下視窗左上方的設定圖示，並將滑鼠游標移至 [文字大小] 選項上方，然後選取想要的文字大小。 您的選擇將會跨工作階段保存。

## <a name="exit-command"></a>結束命令
執行 `exit` 可終止作用中的工作階段。 此行為預設會在 20 分鐘後發生，不須進行互動。

## <a name="next-steps"></a>後續步驟

[Cloud Shell 中 Bash 的快速入門](quickstart.md)
[Cloud Shell 中 PowerShell 的快速入門](quickstart-powershell.md)