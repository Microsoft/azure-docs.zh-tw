---
title: 建立和共用 Azure 入口網站儀表板 | Microsoft Docs
description: 本文說明如何在 Azure 入口網站建立和編輯儀表板。
services: azure-portal
documentationcenter: ''
author: sewatson
manager: doubeby
editor: tysonn
ms.assetid: ff422f36-47d2-409b-8a19-02e24b03ffe7
ms.service: azure-portal
ms.devlang: NA
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: na
ms.date: 09/06/2016
ms.author: sewatson
ms.openlocfilehash: 3bdeb3255c8dc97d2867eca0b9cdec8223d31b21
ms.sourcegitcommit: d78bcecd983ca2a7473fff23371c8cfed0d89627
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/14/2018
ms.locfileid: "34165344"
---
# <a name="create-and-share-dashboards-in-the-azure-portal"></a>在 Azure 入口網站中建立和共用儀表板
您可以建立多個儀表板，並與可存取您 Azure 訂用帳戶的其他人共用。  本文說明建立/編輯、發佈和管理儀表板存取權的基本概念。

## <a name="create-a-dashboard"></a>建立儀表板
若要建立儀表板，請選取目前的儀表板名稱旁的 [新增儀表板]  按鈕。  

![建立儀表板](./media/azure-portal-dashboards/new-dashboard.png)

這個動作會建立全新的空白私人儀表板並讓您進入自訂模式，供您為儀表板命名並新增或重新排列圖格。  處於此模式時，可摺疊的圖格庫會接管左側導覽功能表。  圖格庫可讓您以各種方式尋找 Azure 資源的圖格︰您可以依[資源群組](../azure-resource-manager/resource-group-overview.md#resource-groups)、資源類型、[標籤](../azure-resource-manager/resource-group-using-tags.md)進行瀏覽，或依名稱搜尋資源。  

![自訂儀表板](./media/azure-portal-dashboards/customize-dashboard.png)

拖放圖格到儀表板介面上想要的位置來加以新增。

未與特定資源相關聯的圖格有新的類別，其名稱為**一般**。  在此範例中，我們釘選 [Markdown] 圖格。  您可以使用此圖格將自訂內容新增至儀表板。  圖格支援純文字、[Markdown 語法](https://daringfireball.net/projects/markdown/syntax)和一組有限的 HTML   (基於安全考量，您無法執行插入 `<script>` 標籤或使用可能會干擾入口網站的特定 CSS 樣式元素等動作)。 

![新增 Markdown](./media/azure-portal-dashboards/add-markdown.png)

## <a name="edit-a-dashboard"></a>編輯儀表板
建立儀表板之後，您可以從圖格庫或以圖格表示的刀鋒視窗釘選圖格。 釘選資源群組表示方法。 您可以在瀏覽項目時釘選，或從資源群組刀鋒視窗釘選。 這兩種方法都會釘選以圖格表示的資源群組。

![釘選到儀表板](./media/azure-portal-dashboards/pin-to-dashboard.png)

釘選項目之後，它就會出現在儀表板上。

![檢視儀表板](./media/azure-portal-dashboards/view-dashboard.png)

現在我們已將 Markdown 圖格和資源群組釘選到儀表板，接下來，我們可以將圖格調整大小並重新排列為適合的版面配置。

藉由將滑鼠游標停留在 "..." 並加以選取或是對圖格按一下滑鼠右鍵，您就可以看到該圖格的所有關聯式命令。 預設會有兩個項目：

1. **從儀表板取消釘選** – 從儀表板移除圖格
2. **自訂** – 進入自訂模式

![自訂圖格](./media/azure-portal-dashboards/customize-tile.png)

藉由選取自訂，您可以將圖格調整大小並重新排列。 若要為圖格調整大小，請從關聯式功能表選取新的大小，如下圖所示。

![對圖格調整大小](./media/azure-portal-dashboards/resize-tile.png)

或者，如果圖格支援任何大小，您可以將右下角拖曳到所需大小。

![對圖格調整大小](./media/azure-portal-dashboards/resize-corner.png)

將圖格調整大小之後，檢視儀表板。

![檢視圖格](./media/azure-portal-dashboards/view-tile.png)

在儀表板完成自訂後，只需選取 [完成自訂] 來結束自訂模式，或按一下滑鼠右鍵並從關聯式功能表選取 [完成自訂]。

## <a name="publish-a-dashboard-and-manage-access-control"></a>發佈儀表板和管理存取控制
當您建立儀表板時，它預設是私人用途，這表示只有您會看得到。  若要讓其他人也能看見，請使用與其他儀表板命令一同出現的 [共用]  按鈕。

![共用儀表板](./media/azure-portal-dashboards/share-dashboard.png)

系統會要求您選擇可將儀表板發佈至的訂用帳戶和資源群組。 為了將儀表板完美整合到生態系統，我們已將共用儀表板實作為 Azure 資源 (因此，您無法透過輸入電子郵件地址來共用)。  入口網站中大多數圖格所顯示資訊的存取權是由 [Azure 角色型存取控制](../role-based-access-control/role-assignments-portal.md)所控管。 從存取控制觀點來看，共用儀表板與虛擬機器或儲存體帳戶並無不同。  

假設您有 Azure 訂用帳戶，而且已為小組成員指派訂用帳戶的**擁有者**、**參與者**或**讀取者**角色。  身為擁有者或參與者的使用者能夠列出、檢視、建立、修改或刪除該訂用帳戶內的儀表板。  身為讀取者的使用者能夠列出和檢視儀表板，但無法進行修改或刪除。  具有讀取者存取權的使用者能夠對共用的儀表板進行本機編輯，但無法將這些變更發佈回伺服器。  不過，他們可以製作儀表板的私人複本供自己使用。  一如往常，儀表板上的個別圖格會根據其對應的資源，強制執行自己的存取控制規則。  

為了方便起見，入口網站的發佈體驗會引導您朝向將儀表板放在稱為 **儀表板**的資源群組中的模式。  

![發佈儀表板](./media/azure-portal-dashboards/publish-dashboard.png)

您也可以選擇將儀表板發佈到特定資源群組。  該儀表板的存取控制與資源群組的存取控制相符。  可以管理該資源群組中資源的使用者也可以存取儀表板。

![將儀表板發佈至資源群組](./media/azure-portal-dashboards/publish-to-resource-group.png)

儀表板發佈之後，[共用 + 存取控制]  窗格便會重新整理，並顯示所發佈儀表板的相關資訊，包括用以管理儀表板使用者存取權的連結。  此連結會啟動用來管理任何 Azure 資源存取權的標準角色型存取控制刀鋒視窗。  您隨時都可以透過選取 [共用] 來回到此檢視。

![管理存取控制](./media/azure-portal-dashboards/manage-access.png)

## <a name="next-steps"></a>後續步驟
* 若要管理資源，請參閱 [透過入口網站管理 Azure 資源](../azure-resource-manager/resource-group-portal.md)。
* 若要部署資源，請參閱 [使用 Resource Manager 範本與 Azure 入口網站來部署資源](../azure-resource-manager/resource-group-template-deploy-portal.md)。

