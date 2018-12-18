---
title: 使用 Azure 實驗室服務設定教室實驗室 | Microsoft Docs
description: 在本教學課程中，您會設定在教室中使用的實驗室。
services: devtest-lab, lab-services, virtual-machines
documentationcenter: na
author: spelluru
manager: ''
editor: ''
ms.service: lab-services
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.custom: mvc
ms.date: 10/05/2018
ms.author: spelluru
ms.openlocfilehash: 6696d6e7e53e98dfab2a65c7c66825936020f33c
ms.sourcegitcommit: 67abaa44871ab98770b22b29d899ff2f396bdae3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/08/2018
ms.locfileid: "48856628"
---
# <a name="tutorial-set-up-a-classroom-lab"></a>教學課程：設定教室實驗室 
您將在本教學課程中設定教室實驗室，且其中包含教室中學生所使用的虛擬機器。  

在本教學課程中，您會執行下列動作：

> [!div class="checklist"]
> * 建立教室實驗室
> * 設定教室實驗室
> * 將註冊連結傳送給學生

## <a name="prerequisites"></a>必要條件
若要在實驗室帳戶中設定教室實驗室，您必須是實驗室帳戶中 [實驗室建立者] 角色的成員。 您用來建立實驗室帳戶的帳戶會自動新增至此角色。 實驗室擁有者可以使用下列文章中的步驟，將其他使用者新增至「實驗室建立者」角色：[將使用者新增至實驗室建立者角色](tutorial-setup-lab-account.md#add-a-user-to-the-lab-creator-role)。


## <a name="create-a-classroom-lab"></a>建立教室實驗室

1. 瀏覽至 [Azure 實驗室服務網站](https://labs.azure.com)。 
2. 選取 [登入]，然後輸入您的認證。 Azure 實驗室服務支援組織帳戶和 Microsoft 帳戶。 
3. 在 [新增實驗室] 視窗中，執行下列動作： 
    1. 指定實驗室的**名稱**。 
    2. 指定實驗室中允許的**使用者數目**上限。 
    6. 選取 [ **儲存**]。

        ![建立教室實驗室](../media/tutorial-setup-classroom-lab/new-lab-window.png)
4. 在 [選取虛擬機器規格] 頁面上，執行下列步驟：
    1. 選取在實驗室中建立的虛擬機器 (VM) [大小]。 
    2. 選取您要在其中建立 VM 的 [區域]。 
    3. 選取要用來在實驗室中建立 VM 的 [VM 映像]。 
    4. 選取 [下一步] 。

        ![指定 VM 規格](../media/tutorial-setup-classroom-lab/select-vm-specifications.png)    
5. 在 [設定認證] 頁面上，指定實驗室中所有 VM 的預設認證。 
    1. 指定實驗室中所有 VM 的 [使用者名稱]。
    2. 指定使用者的 [密碼]。 

        > [!IMPORTANT]
        > 請記下使用者名稱和密碼。 它們將不會再次顯示。
    3. 選取 [建立] 。 

        ![設定認證](../media/tutorial-setup-classroom-lab/set-credentials.png)
6. 在 [設定範本] 頁面上，您會看到實驗室建立程序的狀態。 在實驗室中建立範本最多需要 20 分鐘。 

    ![設定範本](../media/tutorial-setup-classroom-lab/configure-template.png)
7. 範本的設定完成後，您會看到下列頁面： 

    ![在範本頁面完成後加以設定](../media/tutorial-setup-classroom-lab/configure-template-after-complete.png)
8. 以下是本教學課程中的選擇性步驟： 
    1. 選取 [啟動] 以啟動範本 VM。
    2. 選取 [連線] 以連線至範本 VM。 
    3. 在您的範本 VM 上安裝並設定軟體。 
    4. **停止** VM。  
    5. 輸入範本的 [描述]

        ![接著在 [設定範本] 頁面中](../media/tutorial-setup-classroom-lab/configure-template-next.png)
9. 選取 [範本] 頁面上的 [下一步]。 
10. 在 [發佈範本] 頁面上，執行下列動作。 
    1. 若要立即發佈範本，請選取 [我了解我無法在發佈之後修改範本。*此程序只能執行一次，最多可能需要一個小時]* 核取方塊，然後選取 [發佈]。  

        > [!WARNING]
        > 在您發佈時，即無法取消發佈。 
    2. 若要稍後再發佈，請選取 [儲存以便稍後使用]。 您可以在精靈完成之後發佈範本 VM。 如需如何在精靈完成之後設定和發佈的詳細資訊，請參閱[如何管理教室實驗室](how-to-manage-classroom-labs.md)一文中的[發佈範本](how-to-manage-classroom-labs.md#publish-the-template)小節。

        ![發佈範本](../media/tutorial-setup-classroom-lab/publish-template.png)
11. 您會看到範本的**發佈進度**。 此程序最多可能需要一小時。 

    ![發佈範本 - 進度](../media/tutorial-setup-classroom-lab/publish-template-progress.png)
12. 範本成功發佈後，您會看到下列頁面。 選取 [完成] 。

    ![發佈範本 - 成功](../media/tutorial-setup-classroom-lab/publish-success.png)
1. 您會看到實驗室的**儀表板**。 
    
    ![教室實驗室儀表板](../media/tutorial-setup-classroom-lab/classroom-lab-home-page.png)
4. 切換到 [虛擬機器] 頁面，確認您看到虛擬機器處於 [未指派] 狀態。 這些虛擬機器尚未指派給任何學生。 它們應處於 [已停止] 狀態。 您可以在此頁面上啟動學生 VM、連線到 VM、停止 VM，以及刪除 VM。 您可以在此頁面啟動 VM，或是讓學生啟動 VM。 

    ![已停止狀態的虛擬機器](../media/tutorial-setup-classroom-lab/virtual-machines-stopped.png)

## <a name="send-registration-link-to-students"></a>將註冊連結傳送給學生

1. 選取左側功能表上 [儀表板]，以切換至 [儀表板] 檢視。 
2. 選取 [使用者註冊] 磚。

    ![學生註冊連結](../media/tutorial-setup-classroom-lab/dashboard-user-registration-link.png)
1. 在 [使用者註冊] 對話方塊中，選取 [複製] 按鈕。 連結會複製到剪貼簿中。 將連結貼到電子郵件編輯器中，然後將電子郵件傳送給學生。 

    ![學生註冊連結](../media/tutorial-setup-classroom-lab/registration-link.png)
2. 在 [使用者註冊] 對話方塊上，選取 [關閉]。 
3. 將註冊連結分享給學生，讓他們能夠註冊課程。 


## <a name="next-steps"></a>後續步驟
在本教學課程中，您已建立教室實驗室並已設定實驗室。 若要了解學生可以如何使用註冊連結，來存取實驗室中的 VM，請前往下一個教學課程：

> [!div class="nextstepaction"]
> [連線到教室實驗室中的 VM](tutorial-connect-virtual-machine-classroom-lab.md)

