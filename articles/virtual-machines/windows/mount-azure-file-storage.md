---
title: 從 Azure Windows VM 掛接 Azure 檔案儲存體 | Microsoft Docs
description: 使用 Azure 檔案儲存體在雲端中儲存檔案，並從 Azure 虛擬機器 (VM) 掛接雲端檔案共用。
documentationcenter: ''
author: cynthn
manager: jeconnoc
editor: tysonn
ms.assetid: ''
ms.service: virtual-machines-windows
ms.workload: ''
ms.tgt_pltfrm: ''
ms.devlang: ''
ms.topic: article
ms.date: 01/02/2018
ms.author: cynthn
ms.openlocfilehash: 8d537bdc882487784baef9f693e4677c76d3bd8d
ms.sourcegitcommit: 2e540e6acb953b1294d364f70aee73deaf047441
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/03/2018
ms.locfileid: "27577546"
---
# <a name="use-azure-file-shares-with-windows-vms"></a>搭配 Windows VM 使用 Azure 檔案共用 

您可以使用 Azure 檔案共用，用以從 VM 儲存及存取檔案。 例如，您可以儲存您想要所有 VM 共用的指令碼或應用程式組態檔。 本文示範如何建立和掛接 Azure 檔案共用，以及如何上傳和下載檔案。

## <a name="connect-to-a-file-share-from-a-vm"></a>從 VM 連線至檔案共用

本節假設您已有想要連線的檔案共用。 如果需要建立檔案共用，請參閱本文稍後的[建立檔案共用](#create-a-file-share)。

1. 登入 [Azure 入口網站](https://portal.azure.com)。
2. 按一下左側功能表的 [儲存體帳戶]。
3. 選擇儲存體帳戶
4. 在 [概觀] 頁面的 [服務] 底下，選取 [檔案]。
5. 選取檔案共用，或按一下 [+ 檔案共用] 來建立可用的新檔案共用。
6. 按一下 [連線] 開啟網頁，該網頁會顯示從 Windows 或 Linux 掛接檔案共用的命令列語法。
7. 在 [磁碟機代號] 中，選取您想要用來識別磁碟機的代號。
8. 選擇要使用的語法，並選取右邊的 [複製] 按鈕，將它複製到剪貼簿。 將它貼到某個您可以輕鬆存取的位置。 
8. 連線至 VM，並開啟命令提示字元。
9. 貼上已編輯的連線語法，然後點擊 [Enter]。
10. 建立連線之後，會收到訊息**命令已順利完成。**
11. 請檢查連線，輸入磁碟機代號以切換至該磁碟機，然後輸入 **dir** 查看檔案共用的內容。



## <a name="create-a-file-share"></a>建立檔案共用 
1. 登入 [Azure 入口網站](https://portal.azure.com)。
2. 按一下左側功能表的 [儲存體帳戶]。
3. 選擇儲存體帳戶
4. 在 [概觀] 頁面的 [服務] 底下，選取 [檔案]。
5. 在 [檔案服務] 頁面上，按一下 [+ 檔案共用]。
6. 填入檔案共用名稱。 檔案共用名稱可以使用小寫字母、數字和單一連字號。 名稱不能以連字號開頭，且不能連續使用多個連字號。 
7. 填入檔案大小上限，最多 5120 GB。
8. 按一下 [確定] 以建立檔案共用。
   
## <a name="upload-files"></a>上傳檔案
1. 登入 [Azure 入口網站](https://portal.azure.com)。
2. 按一下左側功能表的 [儲存體帳戶]。
3. 選擇儲存體帳戶
4. 在 [概觀] 頁面的 [服務] 底下，選取 [檔案]。
5. 選取檔案共用。
6. 按一下 [上傳] 開啟 [上傳檔案] 頁面。
7. 按一下資料夾圖示，瀏覽本機檔案系統，找出要上傳的檔案。   
8. 按一下 [上傳] 將檔案上傳至檔案共用。

## <a name="download-files"></a>下載檔案
1. 登入 [Azure 入口網站](https://portal.azure.com)。
2. 按一下左側功能表的 [儲存體帳戶]。
3. 選擇儲存體帳戶
4. 在 [概觀] 頁面的 [服務] 底下，選取 [檔案]。
5. 選取檔案共用。
6. 在檔案上按一下滑鼠右鍵，選擇 [下載]，將檔案下載到本機電腦。
   

## <a name="next-steps"></a>後續步驟

您也可以使用 PowerShell 建立和管理檔案共用。 如需詳細資訊，請參閱[開始使用 Windows 上的 Azure 檔案儲存體](../../storage/files/storage-dotnet-how-to-use-files.md)。
