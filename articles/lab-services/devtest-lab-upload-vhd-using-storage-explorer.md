---
title: 使用 Microsoft Azure 儲存體總管將 VHD 檔案上傳到 Azure DevTest Labs | Microsoft Docs
description: 使用 Microsoft Azure 儲存體總管將 VHD 檔案上傳到實驗室的儲存體帳戶
services: devtest-lab,virtual-machines,lab-services
documentationcenter: na
author: spelluru
manager: femila
editor: ''
ms.assetid: ''
ms.service: lab-services
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/17/2018
ms.author: spelluru
ms.openlocfilehash: cafb3ec2da335165a3b66c18d83930008c6d72e5
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33782537"
---
# <a name="upload-vhd-file-to-labs-storage-account-using-microsoft-azure-storage-explorer"></a>使用 Microsoft Azure 儲存體總管將 VHD 檔案上傳到實驗室的儲存體帳戶

[!INCLUDE [devtest-lab-upload-vhd-selector](../../includes/devtest-lab-upload-vhd-selector.md)]

在 Azure DevTest Labs 中，可以使用 VHD 檔案來建立自訂映像，這些映像可用來佈建虛擬機器。 本文說明如何使用 [Microsoft Azure 儲存體總管](../vs-azure-tools-storage-manage-with-storage-explorer.md)將 VHD 檔案上傳到實驗室的儲存體帳戶。 在您上傳 VHD 檔案之後，[後續步驟](#next-steps)一節會列出一些說明如何從所上傳的 VHD 檔案建立自訂映像的文章。 如需有關 Azure 中磁碟和 VHD 的詳細資訊，請參閱[關於虛擬機器的磁碟和 VHD](../virtual-machines/linux/about-disks-and-vhds.md)

## <a name="step-by-step-instructions"></a>逐步指示

下列步驟將逐步引導您使用 [Microsoft Azure 儲存體總管](../vs-azure-tools-storage-manage-with-storage-explorer.md)將 VHD 檔案上傳到 DevTest Labs。

1. [下載並安裝最新版 Microsoft Azure 儲存體總管](http://www.storageexplorer.com)。

1. 使用 Azure 入口網站來取得實驗室的儲存體帳戶名稱：

    1. 登入 [Azure 入口網站](http://go.microsoft.com/fwlink/p/?LinkID=525040)。
    
    1. 選取 [所有服務]，然後從清單中選取 [DevTest Labs]。
    
    1. 從實驗室清單中，選取所需的實驗室。  
    
    1. 在實驗室的刀鋒視窗上，選取 [組態] 。 
    
    1. 在實驗室的 [組態] 刀鋒視窗上，選取 [自訂映像 (VHD)]。
    
    1. 在 [自訂映像] 刀鋒視窗上，選取 [+新增]。 
    
    1. 在 [自訂映像] 刀鋒視窗上，選取 [VHD]。
    
    1. 在 [VHD] 刀鋒視窗上，選取 [使用 PowerShell 上傳 VHD]。
    
        ![使用 PowerShell 上傳 VHD][0]
    
    1. [使用 PowerShell 上傳映像] 刀鋒視窗會顯示一個對 **Add-AzureVhd** Cmdlet 的呼叫。 第一個參數 (*Destination*) 包含實驗室的儲存體帳戶，其格式如下：
    
        https://<STORAGE-ACCOUNT-NAME>.blob.core.windows.net/uploads/... 

    1. 記下儲存體帳戶名稱，因為在稍後的步驟中將會用到。
    
1. 使用儲存體總管連線到 Azure 訂用帳戶。

    > [!TIP] 
    > 
    > 儲存體總管支援數個連線選項。 本節說明如何連線到與您的 Azure 訂用帳戶相關聯的儲存體帳戶。 若要查看儲存體總管所支援的其他連線選項，請參閱[開始使用儲存體總管](../vs-azure-tools-storage-manage-with-storage-explorer.md)一文。
 
    1. 開啟儲存體總管。
    
    1. 在儲存體總管中，選取 [Azure 帳戶設定]。 
    
        ![Azure 帳戶設定][1]
    
    1. 左窗格會顯示您已登入的 Microsoft 帳戶。 若要連接到其他帳戶，請選取 [新增帳戶] ，並依照對話方塊使用與至少一個作用中 Azure 訂用帳戶相關聯的 Microsoft 帳戶進行登入。
    
        ![新增帳戶][2]
    
    1. 成功使用 Microsoft 帳戶登入後，左窗格會填入與該帳戶相關聯的 Azure 訂用帳戶。 選取您想要使用的訂用帳戶，然後選取 [套用]。 (選取 [所有訂用帳戶] 切換方塊，可選取全部或不選取任何列出的 Azure 訂用帳戶。)
    
        ![選取 Azure 訂用帳戶][3]
    
    1. 左窗格會顯示與所選 Azure 訂用帳戶相關聯的儲存體帳戶。
    
        ![已選取的 Azure 訂用帳戶][4]

1. 找出實驗室的儲存體帳戶︰

    1. 在儲存體總管的左窗格中，找出並展開擁有實驗室之 Azure 訂用帳戶的節點。
    
    1. 在訂用帳戶的節點下，展開 [儲存體帳戶]。

    1. 展開實驗室的儲存體帳戶節點，以顯示 [Blob 容器]、[檔案共用]、[佇列] 和 [資料表] 的節點。
    
    1. 展開 [Blob 容器] 節點。
    
    1. 選取上傳 Blob 容器以在右窗格中顯示其內容。
        
        ![上傳目錄][5]

1. 使用儲存體總管上傳 VHD 檔案︰

    1. 在儲存體總管的右窗格中，您應該會在實驗室儲存體帳戶的 [上傳] Blob 容器中看到 Blob 清單。 在 Blob 編輯器工具列中，選取 [上傳] 
        
        ![上傳按鈕][6]
    
    1. 從 [上傳] 下拉式功能表中，選取 [上傳檔案...]。
    
    1. 在 [上傳檔案] 對話方塊中，選取省略符號。
        
        ![選取檔案][8]  

    1. 在 [選取要上傳的檔案] 對話方塊中，瀏覽至所需的 VHD 檔案、加以選取，然後選取 [開啟]。
    
    1. 在回到 [上傳檔案] 對話方塊時，將 [Blob 類型] 變更為 [分頁 Blob]。
    
    1. 選取 [上傳] 。

        ![選取檔案][9]  
    
    1. 儲存體總管的 [活動記錄] 窗格會顯示下載狀態 (以及用以取消上傳的連結)。 視 VHD 檔案大小及您的連線速度而定，上傳 VHD 檔案的程序可能會相當長。 

        ![上傳檔案狀態][10]  

## <a name="next-steps"></a>後續步驟

- [使用 Azure 入口網站在 Azure DevTest Labs 中從 VHD 檔案建立自訂映像](devtest-lab-create-template.md)
- [使用 PowerShell 在 Azure DevTest Labs 中從 VHD 檔案建立自訂映像](devtest-lab-create-custom-image-from-vhd-using-powershell.md)

[0]: ./media/devtest-lab-upload-vhd-using-storage-explorer/upload-image-using-psh.png
[1]: ./media/devtest-lab-upload-vhd-using-storage-explorer/settings-icon.png
[2]: ./media/devtest-lab-upload-vhd-using-storage-explorer/add-account-link.png
[3]: ./media/devtest-lab-upload-vhd-using-storage-explorer/subscriptions-list.png
[4]: ./media/devtest-lab-upload-vhd-using-storage-explorer/storage-accounts-list.png
[5]: ./media/devtest-lab-upload-vhd-using-storage-explorer/upload-dir.png
[6]: ./media/devtest-lab-upload-vhd-using-storage-explorer/upload-button.png
[7]: ./media/devtest-lab-upload-vhd-using-storage-explorer/upload-files.png
[8]: ./media/devtest-lab-upload-vhd-using-storage-explorer/select-file.png
[9]: ./media/devtest-lab-upload-vhd-using-storage-explorer/upload-file.png
[10]: ./media/devtest-lab-upload-vhd-using-storage-explorer/upload-status.png
