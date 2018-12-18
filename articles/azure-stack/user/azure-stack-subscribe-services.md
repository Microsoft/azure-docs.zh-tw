---
title: 在本教學課程中，您會了解如何訂閱 Azure Stack 供應項目 | Microsoft Docs
description: 本教學課程會示範如何建立新的 Azure Stack 服務訂用帳戶，並藉由建立測試虛擬機器來測試供應項目。
services: azure-stack
documentationcenter: ''
author: sethmanheim
manager: femila
editor: ''
ms.assetid: ''
ms.service: azure-stack
ms.workload: na
pms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.custom: mvc
ms.date: 09/05/2018
ms.author: sethm
ms.reviewer: ''
ms.openlocfilehash: aff20cba3bc07924f669420a8c367613b41111a1
ms.sourcegitcommit: 4b1083fa9c78cd03633f11abb7a69fdbc740afd1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/10/2018
ms.locfileid: "49076095"
---
# <a name="tutorial-create-and-test-a-subscription"></a>教學課程：建立和測試訂用帳戶
本教學課程說明如何建立含有供應項目的訂用帳戶，然後進行測試。 為了進行測試，您會以雲端管理員的身分登入 Azure Stack 使用者入口網站，訂閱該供應項目，然後建立虛擬機器。

> [!TIP]
> 若要獲得更進階的評估體驗，您可以[為特定使用者建立訂用帳戶](https://docs.microsoft.com/azure/azure-stack/azure-stack-subscribe-plan-provision-vm#create-a-subscription-as-a-cloud-operator)，然後在使用者入口網站中以使用者的身分登入。 

本教學課程說明如何訂閱 Azure Stack 供應項目。

您將了解：

> [!div class="checklist"]
> * 訂閱供應項目 
> * 測試供應項目

## <a name="subscribe-to-an-offer"></a>訂閱供應項目
若要以使用者身分訂閱供應項目，您需要登入 Azure Stack 使用者入口網站，以探索 Azure Stack 操作員已提供的服務。

1. 登入使用者入口網站，並按一下 [取得訂用帳戶]。

   ![取得訂用帳戶](media/azure-stack-subscribe-services/get-subscription.png)

2. 在 [顯示名稱]  欄位中，輸入您的訂閱名稱。 接著，按一下 [供應項目] 以選取 [選擇供應項目] 區段中的其中一個可用供應項目，然後按一下 [建立]。

   ![建立供應項目](media/azure-stack-subscribe-services/create-subscription.png)

   > [!TIP]
   > 您必須立即重新整理使用者入口網站才可開始使用您的訂用帳戶。

3. 若要檢視您所建立的訂用帳戶，請按一下 [所有服務]。  接著，選取 [一般] 類別之下的 [訂用帳戶]，然後選取您的新訂用帳戶。 在訂閱供應項目之後，請重新整理入口網站以查看新訂用帳戶中是否已納入新服務。 在此範例中，我們已新增**虛擬機器**。

   ![檢視訂用帳戶](media/azure-stack-subscribe-services/view-subscription.png)


## <a name="test-the-offer"></a>測試供應項目
在登入使用者入口網站時，您可以藉由使用新的訂用帳戶功能佈建虛擬機器，來測試供應項目。 

> [!NOTE]
> 要進行這項測試，必須已將 Windows Server 2016 資料中心 VM 新增至 Azure Stack 市集。 

1. 登入使用者入口網站。

2. 在使用者入口網站中，按一下 [虛擬機器] > [新增] > [Windows Server 2016 Datacenter]，然後按一下 [建立]。

3. 在 [基本概念] 區段中，輸入 [名稱]、[使用者名稱] 和 [密碼]，選擇 [訂用帳戶]，建立 [資源群組] (或選取現有的)，然後按一下 [確定]。

4. 在 [選擇大小] 區段中，按一下 [A1 標準]，再按一下 [選取]。  

5. 在 [設定] 刀鋒視窗中，接受預設值並按一下 [確定]。

6. 在 [摘要] 區段中，按一下 [確定] 以建立虛擬機器。  

7. 若要查看您的新虛擬機器，請按一下 [虛擬機器]，然後搜尋新的虛擬機器並按一下其名稱。

    ![所有資源](media/azure-stack-subscribe-services/view-vm.png)

> [!NOTE]
> 虛擬機器部署需要數分鐘的時間才能完成。


## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解：

> [!div class="checklist"]
> * 訂閱供應項目 
> * 測試供應項目


> [!div class="nextstepaction"]
> [從社群範本建立 VM](azure-stack-create-vm-template.md)