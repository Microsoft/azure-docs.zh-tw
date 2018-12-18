---
title: 使用 Visual Studio 疑難排解 Windows 上的 Docker 用戶端錯誤 | Microsoft Docs
description: 疑難排解使用 Visual Studio 2017 在 Windows 上建立及部署 Web 應用程式到 Docker 時您會遇到的問題。
services: azure-container-service
documentationcenter: na
author: devinb
manager: douge
editor: ''
ms.assetid: 346f70b9-7b52-4688-a8e8-8f53869618d3
ms.service: multiple
ms.devlang: dotnet
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: multiple
ms.date: 10/13/2017
ms.author: devinb
ms.openlocfilehash: 90dd5df4a607568e2f3a60791da2948af7ce4e50
ms.sourcegitcommit: 38c9176c0c967dd641d3a87d1f9ae53636cf8260
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/06/2017
ms.locfileid: "24002925"
---
# <a name="troubleshoot-visual-studio-2017-development-with-docker"></a>疑難排解 Docker 的 Visual Studio 2017 開發

您使用 Visual Studio Tools for Docker 時，您可能會在建置或偵錯您的應用程式時遇到問題。 以下是一些常見的疑難排解步驟。

## <a name="volume-sharing-is-not-enabled-enable-volume-sharing-in-the-docker-ce-for-windows-settings--linux-containers-only"></a>未啟用磁碟區共用。 在 Docker CE for Windows 設定中啟用磁碟區共用 (僅限 Linux 容器)

若要解決此問題︰

1. 以滑鼠右鍵按一下通知區域中的 [適用於 Windows 的 Docker]，然後選取 [設定]。
1. 選取 [共用磁碟機] 並共用系統磁碟機以及專案所在的磁碟機。

> [!NOTE]
> 如果檔案似乎已共用，您仍可能需要按一下對話方塊下的 [重設認證...] 連結，以重新啟用磁碟區共用。

![共用的磁碟機](./media/vs-azure-tools-docker-troubleshooting-docker-errors/shareddrives.png)

## <a name="unable-to-start-debugging"></a>無法開始偵錯

其中一個原因可能與您的使用者設定檔資料夾中有過時的偵錯元件有關。 請執行下列命令移除這些資料夾，以便在下一個偵錯工作階段下載最新的偵錯元件。

- del %userprofile%\vsdbg
- del %userprofile%\onecoremsvsmon

## <a name="errors-specific-to-networking-when-debugging-your-application"></a>偵錯您的應用程式時與網路功能相關的錯誤

請嘗試執行可從[清除容器主機網路](https://github.com/MicrosoftDocs/Virtualization-Documentation/tree/master/windows-server-container-tools/CleanupContainerHostNetworking)下載的指令碼，這會重新整理主機電腦上與網路相關的元件。


## <a name="microsoftdockertools-github-repo"></a>Microsoft/DockerTools GitHub 存放庫

對於您遇到的其他任何問題，請參閱 [Microsoft/DockerTools](https://github.com/microsoft/dockertools/issues) 問題。