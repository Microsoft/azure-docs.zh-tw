---
title: 使用 Azure Cloud Shell 編輯器 | Microsoft Docs
description: Azure Cloud Shell 編輯器使用方式概觀。
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
ms.date: 07/24/2018
ms.author: juluk
ms.openlocfilehash: caf6e18a9a30654710f5445ed6ab957a5253d62e
ms.sourcegitcommit: c2c64fc9c24a1f7bd7c6c91be4ba9d64b1543231
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/26/2018
ms.locfileid: "39259485"
---
# <a name="using-the-azure-cloud-shell-editor"></a>使用 Azure Cloud Shell 編輯器

Azure Cloud Shell 包含從開放原始碼 [Monaco Editor](https://github.com/Microsoft/monaco-editor) \(英文\) 建置的整合式檔案編輯器。 Cloud Shell 編輯器支援各種功能，例如語言反白顯示、命令選擇區和檔案總管。

![Cloud Shell 編輯器](media/using-cloud-shell-editor/open-editor.png)

## <a name="opening-the-editor"></a>開啟編輯器

若要快速建立和編輯檔案，請在 Cloud Shell 終端機中執行 `code .` 以啟動編輯器。 這個動作會開啟編輯器並進入終端機中設定的作用中工作目錄。

若要直接開啟檔案以快速編輯，請執行 `code <filename>` 來開啟編輯器，但不開啟檔案總管。

若要透過 UI 按鈕來開啟編輯器，從工具列按一下 `{}` 編輯器圖示。 這樣會開啟編輯器而且檔案總管預設會進入 `/home/<user>` 目錄。

## <a name="closing-the-editor"></a>關閉編輯器

若要關閉編輯器，請開啟編輯器右上角的`...` 動作面板窗格，然後選取 `Close editor`。

![關閉編輯器](media/using-cloud-shell-editor/close-editor.png)

## <a name="command-palette"></a>命令選擇區

若要啟動命令選擇區，當焦點是設定在編輯器上時，使用 `F1` 鍵。 透過動作面板也可以開啟命令選擇區。

![命令選擇區](media/using-cloud-shell-editor/cmd-palette.png)

## <a name="contributing-to-the-monaco-editor"></a>對 Monaco Editor 做出貢獻

Cloud Shell 編輯器中的語言反白顯示支援，是透過 [Monaco Editor](https://github.com/Microsoft/monaco-editor) 使用之 Monarch 語法所定義的上游功能實現的。 若要了解如何做出貢獻，請閱讀 [Monaco 參與者指南](https://github.com/Microsoft/monaco-editor/blob/master/CONTRIBUTING.md) \(英文\)。

## <a name="next-steps"></a>後續步驟
[嘗試 Cloud Shell 中的 Bash 快速入門](quickstart.md)
[檢視整合式的 Cloud Shell 工具的完整清單](features.md)