---
title: 將虛擬機器提供給您的 Azure Stack 使用者| Microsoft Docs
description: 了解如何將虛擬機器提供給 Azure Stack
services: azure-stack
documentationcenter: ''
author: jeffgilb
manager: femila
editor: ''
ms.assetid: ''
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 09/11/2018
ms.author: jeffgilb
ms.reviewer: ''
ms.custom: mvc
ms.openlocfilehash: d106d9f79498678f08142f952e09c5125c6e5d6c
ms.sourcegitcommit: c29d7ef9065f960c3079660b139dd6a8348576ce
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/12/2018
ms.locfileid: "44721512"
---
# <a name="tutorial-make-virtual-machines-available-to-your-azure-stack-users"></a>教學課程：將虛擬機器提供給 Azure Stack 使用者

身為 Azure Stack 雲端系統管理員，您可以建立供應項目，以供您的使用者 (有時稱為租用戶) 訂閱。 藉由訂閱供應項目，使用者可以取用供應項目所提供的 Azure Stack 服務。

此教學課程會說明如何為虛擬機器建立供應項目，然後以使用者身分登入來測試供應項目。

您將了解：

> [!div class="checklist"]
> * 建立供應項目
> * 新增映像
> * 測試供應項目

在 Azure Stack 中，能透過訂用帳戶、供應項目與方案，為使用者提供服務。 使用者可以訂閱多個供應項目。 供應項目可以有一或多個方案，而方案則可以有一或多個服務。

![訂用帳戶、供應項目與方案](media/azure-stack-key-features/image4.png)

若要深入了解，請參閱 [Azure Stack 中的主要功能與概念](azure-stack-key-features.md)。

## <a name="create-an-offer"></a>建立供應項目

供應項目是一組提供者供使用者購買或訂閱的一或多個方案。 供應項目的建立程序有數個步驟。 首先，系統會提示您建立供應項目，然後建立方案，最後再建立配額。

1. 以雲端系統管理員身分[登入](azure-stack-connect-azure-stack.md)入口網站，然後選取 [+ 建立資源] > [供應項目 + 方案] > [供應項目]。

   ![新增供應項目](media/azure-stack-tutorial-tenant-vm/image01.png)

1. 在 [新增供應項目] 中，輸入 [顯示名稱] 與 [資源名稱]，然後選取新的或現有的 [資源群組]。 [顯示名稱] 是供應項目的易記名稱。 只有雲端操作員可以看到資源名稱，系統管理員會使用此名稱，將供應項目當作 Azure Resource Manager 資源來使用。

   ![顯示名稱](media/azure-stack-tutorial-tenant-vm/image02.png)

1. 選取 [基本方案]，然後在 [方案] 區段中，選取 [新增] 以將新方案新增到供應項目中。

   ![新增方案](media/azure-stack-tutorial-tenant-vm/image03.png)

1. 在 [新增方案] 區段中，填寫 [顯示名稱] 與 [資源名稱]。 [顯示名稱] 是使用者所見的方案易記名稱。 只有雲端操作員可以看到資源名稱，雲端操作員會使用此名稱，將方案當作 Azure Resource Manager 資源來使用。

   ![方案顯示名稱](media/azure-stack-tutorial-tenant-vm/image04.png)

1. 選取 [服務]。 從 [服務] 清單中，選取 [Microsoft.Compute]、[Microsoft.Network] 及 [Microsoft.Storage]。 選擇 [選取] 將這些服務新增至方案。

   ![方案服務](media/azure-stack-tutorial-tenant-vm/image05.png)

1. 選取 [配額]，然後選取要建立配額的第一個服務。 如果是 IaaS 配額，請使用下列範例作為指南，來設定計算、網路和儲存體服務的配額。

   - 首先，為計算服務建立配額。 在命名空間清單中，選取 [Microsoft.Compute]，然後選取 [建立新的配額]。

     ![建立新的配額](media/azure-stack-tutorial-tenant-vm/image06.png)

   - 在 [建立配額] 中，輸入配額的名稱。 您可以針對所要建立的配額，變更或接受所顯示的配額值。 在此範例中，我們會接受預設設定，然後選取 [確定]。

     ![配額名稱](media/azure-stack-tutorial-tenant-vm/image07.png)

   - 在命名空間清單中選取 [Microsoft.Compute]，然後選取您所建立的配額。 這會讓配額連結至計算服務。

     ![選取配額](media/azure-stack-tutorial-tenant-vm/image08.png)

      針對網路和儲存體服務重複這些步驟。 完成時，請選取 [配額] 中的 [確定] 來儲存所有配額。

1. 在 [新增方案] 中，選取 [確定]。

1. 在 [方案] 底下選取新方案，然後 [選取]。

1. 在 [新增供應項目] 中，選取 [建立]。 您會在供應項目建立完成後看到通知。

1. 在儀表板功能表上，選取 [供應項目]，然後選取您所建立的供應項目。

1. 選取 [變更狀態]，然後選擇 [公開]。

    ![公開映像](media/azure-stack-tutorial-tenant-vm/image09.png)

## <a name="add-an-image"></a>新增映像

佈建虛擬機器之前，您必須新增一張影像到 Azure Stack 市集。 您可以從 Azure Marketplace 新增您選擇的影像，包括 Linux 影像。

若您是在已連結情節中作業，且您已向 Azure 註冊 Azure Stack 執行個體，您可以使用[從 Azure 下載 項目到 Azure Stack](azure-stack-download-azure-marketplace-item.md) 主題中所述的步驟，從 Azure Marketplace 下載 Windows Server 2016 VM 映像。

如需有關將不同項目新增到 Marketplace 的資訊，請參閱 [Azure Stack Marketplace](azure-stack-marketplace.md)。

## <a name="test-the-offer"></a>測試供應項目

既然您已經建立供應項目，您可以進行測試。 請以使用者身分登入、訂閱該供應項目，然後新增虛擬機器。

1. **訂閱供應項目**

   a. 以使用者帳戶登入使用者入口網站，然後選取 [取得訂用帳戶] 圖格。
   - 就整合系統而言，URL 會因操作員的區域和外部網域名稱而有所不同，且格式會是 https://portal.&lt;region&gt;.&lt;FQDN&gt;。
   - 如果您使用「Azure Stack 開發套件」，則入口網站位址為 https://portal.local.azurestack.external。

   ![取得訂用帳戶](media/azure-stack-tutorial-tenant-vm/image10.png)

   b. 在 [取得訂用帳戶] 中，於 [顯示名稱] 欄位中輸入訂用帳戶的名稱。 選取 [供應項目]，然後在 [選擇供應項目] 清單中選擇其中一個供應項目。 選取 [建立] 。

   ![建立供應項目](media/azure-stack-tutorial-tenant-vm/image11.png)

   c. 若要檢視訂用帳戶，請選取 [所有服務]，然後在 [一般] 類別底下選取 [訂用帳戶]。 選取新的訂用帳戶，以查看訂用帳戶所含括的服務。

   >[!NOTE]
   >在訂閱供應項目後，請重新整理入口網站以查看新訂用帳戶所含括的服務。

1. **佈建虛擬機器**

   您可以從使用者入口網站中，使用新的訂用帳戶佈建虛擬機器。

   a. 以使用者帳戶登入使用者入口網站。
      - 就整合系統而言，URL 會因操作員的區域和外部網域名稱而有所不同，且格式會是 https://portal.&lt;region&gt;.&lt;FQDN&gt;。
   - 如果您使用「Azure Stack 開發套件」，則入口網站位址為 https://portal.local.azurestack.external。

   b.  在儀表板上，選取 [+ 建立資源] > [計算] > [Windows Server 2016 Datacenter 評估版]，然後選取 [建立]。

   c. 在 [基本資料] 中提供下列資訊：
      - 輸入 [名稱]
      - 輸入 [使用者名稱]
      - 輸入 [密碼]
      - 選擇 [定價層] 
      - 建立 [資源群組] (或選取現有資源群組)。 
      - 選取 [確定] 以儲存此資訊。

   d. 在 [選擇大小] 中，選取 [A1 標準]，然後按一下 [選取]。  

   e. 在 [設定] 中，選取 [虛擬網路]。

   f. 在 [選擇虛擬網路] 中，選取 [新建]。

   g. 在 [建立虛擬網路] 中，接受所有預設值，然後選取 [確定]。

   h. 在 [設定] 中選取 [確定]，以儲存網路組態。

      i. 在 [摘要] 中，選取 [確定] 以建立虛擬機器。  

   j. 若要查看新的虛擬機器，請選取 [所有資源]。 搜尋虛擬機器，然後從搜尋結果中選取其名稱。

   
## <a name="next-steps"></a>後續步驟

在此教學課程中，您已了解如何：

> [!div class="checklist"]
> * 建立供應項目
> * 新增映像
> * 測試供應項目

請前進到下一個教學課程，以了解如何：
> [!div class="nextstepaction"]
> [將 SQL 資料庫提供給您的 Azure Stack 使用者](azure-stack-tutorial-sql-server.md)