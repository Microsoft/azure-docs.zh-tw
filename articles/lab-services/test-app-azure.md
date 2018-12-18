---
title: 如何在 Azure 中測試應用程式 | Microsoft Docs
description: 了解如何在實驗室中建立檔案共用，並且在本機電腦和實驗室中的虛擬機器上掛接，然後將傳統型/Web 應用程式部署到檔案共用並進行測試。
services: devtest-lab,virtual-machines,lab-services
documentationcenter: na
author: spelluru
manager: femila
ms.service: lab-services
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 09/04/2018
ms.author: spelluru
ms.openlocfilehash: 099bdc25c27e264c3c7732243068307856840409
ms.sourcegitcommit: d211f1d24c669b459a3910761b5cacb4b4f46ac9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/06/2018
ms.locfileid: "44030465"
---
# <a name="test-your-app-in-azure"></a>在 Azure 測試您的應用程式 
本文提供在 Azure 中使用 DevTest Labs 測試應用程式的步驟。 首先，您會在實驗室內設定檔案共用，並將它掛接為本機開發電腦和實驗室內 VM 上的磁碟機。 然後，您可使用 Visual Studio 2017 將應用程式部署到檔案共用，以便在實驗室中的 VM 上執行應用程式。  

## <a name="prerequisites"></a>必要條件 
1. [建立 Azure 訂用帳戶](https://azure.microsoft.com/free/) (如果您還沒有的話)，然後登入 [Azure 入口網站](https://portal.azure.com)。
2. 遵循[這篇文章](devtest-lab-create-lab.md)中的指示，使用 Azure DevTest Labs 建立實驗室。 將實驗室釘選至儀表板，以便下次登入時可以輕鬆找到它。 Azure DevTest Labs 可讓您藉由減少浪費並控制成本，在 Azure 內快速建立資源。 若要深入了解 DevTest Labs，請參閱[概觀](devtest-lab-overview.md)。 
3. 遵循[建立儲存體帳戶](../storage/common/storage-create-storage-account.md)一文中的指示，在實驗室的資源群組中建立 Azure 儲存體帳戶。 在 [建立儲存體帳戶] 頁面上，針對 [資源群組] 選取 [使用現有的]，然後選取 [實驗室的資源群組]。 
4. 遵循[在 Azure 檔案服務中建立檔案共用](../storage/files/storage-how-to-create-file-share.md)一文中的指示，在 Azure 儲存體中建立檔案共用。 

## <a name="mount-the-file-share-on-your-local-machine"></a>在本機電腦上掛接檔案共用
1. 在本機電腦上，使用[搭配 Windows 使用 Azure 檔案共用](../storage/files/storage-how-to-use-files-windows.md)一文的[在 Windows 中保存 Azure 檔案共用認證](../storage/files/storage-how-to-use-files-windows.md#persisting-azure-file-share-credentials-in-windows)一節中的指令碼。 
2. 然後，使用 `net use` 命令在本機電腦上掛接檔案共用。 以下是範例命令：請先指定您的 Azure 儲存體名稱和檔案共用名稱，再執行命令。 

    `net use Z: \\<YOUR AZURE STORAGE NAME>.file.core.windows.net\<YOUR FILE SHARE NAME> /persistent:yes`

## <a name="create-a-vm-in-the-lab"></a>在實驗室中建立虛擬機器
1. 在 [檔案共用] 頁面上，選取頂端階層連結功能表中的**資源群組**。 您會看見 [資源群組] 頁面。 
    
    ![從階層連結功能表中選取資源群組](media/test-app-in-azure/select-resource-group-bread-crump.png)
2. 在 [資源群組] 頁面上，選取您在 DevTest Labs 中建立的**實驗室**。

    ![選取實驗室](media/test-app-in-azure/select-devtest-lab-in-resource-group.png)
3. 在實驗室的 [DevTest Lab] 頁面上，選取工具列上的 [+ 新增]。 

    ![實驗室的 [新增] 按鈕](media/test-app-in-azure/add-button-in-lab.png)
4. 在 [選擇基底] 頁面上，搜尋 **smalldisk**，然後選取 [[smalldisk] Windows Server 2016 資料中心]。 

    ![選擇小型磁碟 Windows Server](media/test-app-in-azure/choose-small-disk-windows-server.png)
5. 在 [虛擬機器] 頁面上，指定 [虛擬機器名稱]、[使用者名稱]、[密碼]，然後選取 [建立]。    
    
    ![建立虛擬機器角色](media/test-app-in-azure/create-virtual-machine-page.png)    

## <a name="mount-the-file-share-on-your-vm"></a>在 VM 上掛接檔案共用
1. 成功建立虛擬機器之後，請從清單中選取**虛擬機器**。    

    ![選取實驗室 VM](media/test-app-in-azure/select-lab-vm.png)
2. 選取工具列上的 [連線]，以連線到 VM。 
3. 利用 [命令列工具] 區段中的 [Windows 安裝] 連結，[安裝 Azure PowerShell](https://azure.microsoft.com/downloads/)。 如需安裝 Azure PowerShell 的其他方式，請參閱[這篇文章](/powershell/azure/install-azurerm-ps?view=azurermps-6.8.1)。
4. 遵循[掛接檔案共用](#mount-the-file-share)一節中的指示。 

## <a name="publish-your-app-from-visual-studio"></a>從 Visual Studio 發佈應用程式
在這一節中，您會將應用程式從 Visual Studio 發佈至雲端的測試 VM。

1. 使用 Visual Studio 2017 建立傳統型/Web 應用程式。
2. 建置您的應用程式。
3. 若要發佈應用程式，請在 [方案總管] 中以滑鼠右鍵按一下您的專案，然後選取 [發佈]。 
4. 在 [發佈精靈] 中，輸入您的檔案共用所對應的**磁碟機**。

    **傳統型應用程式：**

    ![傳統型應用程式](media/test-app-in-azure/desktop-app.png)

    **Web 應用程式：**

    ![Web 應用程式](media/test-app-in-azure/web-app.png)

1. 選取 [下一步] 來完成發佈工作流程，然後選取 [完成]。 當您完成精靈步驟時，Visual Studio 會建置您的應用程式並將其發佈到您的檔案共用。 


## <a name="test-the-app-on-your-test-vm-in-the-lab"></a>在實驗室中的測試 VM 上測試應用程式

1. 瀏覽至實驗室中您的 VM 的虛擬機器頁面。 
2. 選取工具列上的 [開始] 來啟動 VM (如果它處於停止狀態的話)。 您可以為 VM 設定自動啟動和自動關機原則，避免每次啟動和停止。 
3. 選取 [ **連接**]。

    ![虛擬機器頁面](media/test-app-in-azure/virtual-machine-page.png)
4. 在虛擬機器中，啟動 [檔案總管]，然後選取 [這部電腦] 來尋找您的檔案共用。

    ![在 VM 上尋找共用](media/test-app-in-azure/find-share-on-vm.png)

    > [!NOTE]
    > 基於任何原因，如果您在虛擬機器或本機電腦上找不到檔案共用，您可以執行 `net use` 命令予以重新掛接。 您可以在 Azure 入口網站中您的 [檔案共用] 的 [連線] 精靈上尋找 `net use` 命令。
1. 開啟檔案共用，並確認您看到從 Visual Studio 部署的應用程式。 

    ![在 VM 上開啟共用](media/test-app-in-azure/open-file-share.png)

    您現在可以在 Azure 中所建立的測試 VM 內存取及測試應用程式。

## <a name="next-steps"></a>後續步驟
請參閱下列文章，了解如何使用實驗室中的 VM。 

- [將 VM 新增至實驗室](devtest-lab-add-vm.md)
- [重新啟動實驗室 VM](devtest-lab-restart-vm.md)
- [調整實驗室 VM 的大小](devtest-lab-resize-vm.md)
