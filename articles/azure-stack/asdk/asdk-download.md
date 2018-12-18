---
title: 下載 Azure Stack 開發套件 (ASDK) 並解壓縮 | Microsoft Docs
description: 說明如何下載 Azure Stack 開發套件 (ASDK) 並解壓縮。
services: azure-stack
documentationcenter: ''
author: jeffgilb
manager: femila
editor: ''
ms.assetid: ''
ms.service: azure-stack
ms.workload: na
pms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/10/2018
ms.author: jeffgilb
ms.reviewer: misainat
ms.openlocfilehash: a6ccfa439b58d36ee44d5f8441c2058622965653
ms.sourcegitcommit: 387d7edd387a478db181ca639db8a8e43d0d75f7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/10/2018
ms.locfileid: "41946527"
---
# <a name="download-and-extract-the-azure-stack-development-kit-asdk"></a>下載 Azure Stack 開發套件 (ASDK) 並解壓縮
確認您的開發套件主機電腦符合安裝 ASDK 的基本需求後，下一個步驟是下載 ASDK 部署套件並解壓縮以取得 Cloudbuilder.vhdx。

## <a name="download-the-asdk"></a>下載 ASDK
1. 開始下載之前，請確定您的電腦符合下列必要條件：

  - 除了作業系統磁碟以外，電腦在四個個別相同的邏輯硬碟上必須有至少 60 GB 的可用磁碟空間。
  - 必須安裝 [.NET framework 4.6 (或更新版本)](https://aka.ms/r6mkiy)。

2. [移至 [開始使用] 頁面](https://azure.microsoft.com/overview/azure-stack/try/?v=try)，下載 Azure Stack 開發套件，提供您的詳細資料，然後按一下 [提交]。
3. 下載並執行 [Azure Stack 開發套件的部署檢查程式](https://go.microsoft.com/fwlink/?LinkId=828735&clcid=0x409)必要條件檢查程式指令碼。 此獨立指令碼會經歷 Azure Stack 開發套件設定期間所進行的先決條件檢查。 在下載較大型的 Azure Stack 開發套件之前，這可用來確認您符合硬體和軟體需求。
4. 在 [下載軟體] 底下，按一下 [Azure Stack 開發套件]。

  > [!NOTE]
  > ASDK 下載 (AzureStackDevelopmentKit.exe) 約 10 GB。

## <a name="extract-the-asdk"></a>將 ASDK 解壓縮
1. 下載完成之後，按一下 [執行] 以啟動 ASDK 自我解壓縮程式 (AzureStackDevelopmentKit.exe)。
2. 檢閱並接受自我解壓縮程式精靈的 [授權合約] 頁面上顯示的授權合約，然後按 [下一步]。
3. 檢閱自我解壓縮程式精靈的 [重要聲明] 頁面上顯示的隱私權聲明資訊，然後按 [下一步]。
4. 在自我解壓縮程式精靈的 [選取目的地位置] 頁面上，選取 Azure Stack 安裝程式檔案要解壓縮至的位置，然後按 [下一步]。 預設位置是「目前資料夾」\Azure Stack 開發套件。 
5. 檢閱「自我解壓縮程式精靈」之 [準備解壓縮] 頁面上的目的地位置摘要，然後按一下 [解壓縮]以將 CloudBuilder.vhdx (大約 28GB) 和 ThirdPartyLicenses.rtf 檔案解壓縮。 這個程序需要一些時間才能完成。
6. 將 CloudBuilder.vhdx 檔案複製或移動至 ASDK 主機電腦上的 C:\ 磁碟機根目錄 (C:\CloudBuilder.vhdx)。

> [!NOTE]
> 將檔案解壓縮之後，您可以刪除 .EXE 和 .BIN 檔案來恢復硬碟空間。 或者，您也可以備份這些檔案，如此一來，如果您需要重新部署 ASDK，就不需要下載這些檔案。


## <a name="next-steps"></a>後續步驟
[準備 ASDK 主機電腦](asdk-prepare-host.md)