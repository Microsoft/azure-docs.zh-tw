---
title: 管理 Azure 實驗室服務中的實驗室帳戶 | Microsoft Docs
description: 了解如何在 Azure 訂用帳戶中建立實驗室帳戶、檢視所有實驗室帳戶，或刪除實驗室帳戶。
services: lab-services
documentationcenter: na
author: spelluru
manager: ''
editor: ''
ms.service: lab-services
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/17/2018
ms.author: spelluru
ms.openlocfilehash: fd43c62f1a291a59d5d373437a49b263d6af4cb3
ms.sourcegitcommit: 30fd606162804fe8ceaccbca057a6d3f8c4dd56d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/30/2018
ms.locfileid: "39345883"
---
# <a name="manage-lab-accounts-in-azure-lab-services"></a>管理 Azure 實驗室服務中的實驗室帳戶 
在 Azure 實驗室服務中，實驗室帳戶是受控實驗室 (例如教室實驗室) 的容器。 系統管理員會使用 Azure 實驗室服務來設定實驗室帳戶，並向可以在帳戶中建立實驗室的實驗室擁有者提供存取權。 本文說明如何建立實驗室帳戶、檢視所有實驗室帳戶，或刪除實驗室帳戶。

## <a name="create-a-lab-account"></a>建立實驗室帳戶
1. 登入 [Azure 入口網站](https://portal.azure.com)。
2. 在左側主功能表中，選取 [建立資源]。
3. 在 Azure Marketplace 中搜尋 [Lab Services] \(實驗室服務\)，然後選取下拉式清單中的 [Lab Services] \(實驗室服務\)。 
4. 在篩選的服務清單中，選取 [Lab Services (Preview)] \(實驗室服務 (預覽)\)。 
5. 在 [建立實驗室帳戶] 視窗中，選取 [建立]。
7. 在 [實驗室帳戶] 視窗中，執行下列動作： 
    1. 針對 [實驗室帳戶名稱] 輸入名稱。 
    2. 選取您要在其中建立實驗室帳戶的 **Azure 訂用帳戶**。
    3. 針對 [資源群組]，選取 [新建]，並為資源群組輸入名稱。
    4. 針對 [位置]，選取您要在其中建立實驗室帳戶的位置/區域。 
    5. 選取 [建立] 。 

        ![[建立實驗室帳戶] 視窗](../media/how-to-manage-lab-accounts/lab-account-settings.png)
5. 如果您沒有看到實驗室帳戶的頁面，請選取 [通知] 按鈕，然後按一下通知中的 [移至資源] 按鈕。 

    ![[建立實驗室帳戶] 視窗](../media/how-to-manage-lab-accounts/notification-go-to-resource.png)    
6. 您會看到下列 [實驗室帳戶] 頁面：

    ![[實驗室帳戶] 頁面](../media/how-to-manage-lab-accounts/lab-account-page.png)

## <a name="add-a-user-to-the-lab-creator-role"></a>將使用者新增至實驗室建立者角色
若要在實驗室帳戶中設定教室實驗室，使用者必須是實驗室帳戶中的 [實驗室建立者] 角色的成員。 您用來建立實驗室帳戶的帳戶會自動新增至此角色。 如果您打算使用相同的使用者帳戶建立教室實驗室，您可以略過此步驟。 若要使用其他使用者帳戶來建立教室實驗室，請執行下列步驟： 

1. 在 [實驗室帳戶] 頁面上，選取 [存取控制] \(IAM\)，然後按一下工具列上的 [+新增]。 

    ![[實驗室帳戶] 頁面](../media/tutorial-setup-lab-account/access-control.png)
2. 在 [新增權限] 頁面上，針對 [角色] 選取 [實驗室建立者]，選取您要新增到實驗室建立者角色的使用者，然後選取 [儲存]。 

    ![將使用者新增至實驗室建立者角色](../media/tutorial-setup-lab-account/add-user-to-lab-creator-role.png)

## <a name="specify-marketplace-images-available-to-lab-owners"></a>指定適用於實驗室擁有者的 Marketplace 映像
身為實驗室帳戶擁有者的您，可以指定實驗室建立者可用來在實驗室帳戶中建立實驗室的 Marketplace 映像。 

1. 選取左側功能表上的 [Marketplace 映像]。 根據預設，您會看到映像 (包括已啟用和停用) 的完整清單。 您可以從頂端的下拉式清單中選取 [僅限已啟用]/[僅限已停用] 選項來篩選清單，而僅檢視已啟用/已停用的映像。 
    
    ![Marketplace 映像頁面](../media/tutorial-setup-lab-account/marketplace-images-page.png)

    在清單中顯示的是滿足下列條件的 Marketplace 映像：
        
    - 建立單一 VM。
    - 使用 Azure Resource Manager 來佈建 VM
    - 不需要購買額外的授權方案
2. 若要**停用**已啟用的 Marketplace 映像，請執行下列其中一個動作： 
    1. 選取最後一個資料行中的 [...] (省略符號)，然後選取 [停用映像]。 

        ![停用一個映像](../media/tutorial-setup-lab-account/disable-one-image.png) 
    2. 在清單中選取映像名稱前面的核取方塊，以選取清單中的一或多個映像，然後選取 [停用選取的映像]。 

        ![停用多個映像](../media/tutorial-setup-lab-account/disable-multiple-images.png) 
1. 同樣地，若要**啟用** Marketplace 映像，請執行下列其中一個動作： 
    1. 選取最後一個資料行中的 [...] (省略符號)，然後選取 [啟用映像]。 
    2. 在清單中選取映像名稱前面的核取方塊，以選取清單中的一或多個映像，然後選取 [啟用選取的映像]。 

## <a name="view-lab-accounts"></a>檢視實驗室帳戶
1. 登入 [Azure 入口網站](https://portal.azure.com)。
2. 選取功能表中的 [所有資源]。 
3. 針對 [類型]，選取 [實驗室服務]。 
    您也可以依訂用帳戶、資源群組、位置和標籤來篩選。 

## <a name="delete-a-lab-account"></a>刪除實驗室帳戶
請遵循上一節以清單顯示實驗室帳戶的指示。 使用下列指示來刪除實驗室帳戶： 

1. 選取您要刪除的**實驗室帳戶**。 
2. 從工作列中選取 [刪除]。 
3. 輸入 **Yes** 進行確認。
4. 選取 [刪除] 。 

## <a name="next-steps"></a>後續步驟
使用 Azure 實驗室服務以設定實驗室的入門：

- [設定教室實驗室](tutorial-setup-classroom-lab.md)
- [設定實驗室](../tutorial-create-custom-lab.md)
