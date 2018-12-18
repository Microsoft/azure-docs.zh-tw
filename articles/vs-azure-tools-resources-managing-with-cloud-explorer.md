---
title: 使用雲端總管管理 Azure 資源 | Microsoft Docs
description: 了解如何在 Visual Studio 內使用 [雲端總管] 瀏覽和管理 Azure 資源。
services: visual-studio-online
author: ghogen
manager: douge
assetId: 6347dc53-f497-49d5-b29b-e8b9f0e939d7
ms.prod: visual-studio-dev15
ms.technology: vs-azure
ms.custom: vs-azure
ms.workload: azure-vs
ms.topic: conceptual
ms.date: 03/25/2017
ms.author: ghogen
ms.openlocfilehash: 596db56a71d4cc73517b9a9cbfe9b97e536aa83a
ms.sourcegitcommit: 5a9be113868c29ec9e81fd3549c54a71db3cec31
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/11/2018
ms.locfileid: "44378841"
---
# <a name="manage-the-resources-associated-with-your-azure-accounts-in-visual-studio-cloud-explorer"></a>在 Visual Studio Cloud Explorer 中管理與 Azure 帳戶關聯的資源
Cloud Explorer 可讓您從 Visual Studio 內檢視您的 Azure 資源和資源群組、檢查其屬性，以及執行重要的開發人員診斷動作。 

與 [Azure 入口網站](http://go.microsoft.com/fwlink/p/?LinkID=525040)相同，Cloud Explorer 也是建立在 Azure Resource Manager 堆疊的基礎上。 因此，Cloud Explorer 了解 Azure 資源群組之類的資源，以及邏輯應用程式和 API 應用程式之類的 Azure 服務，並且支援[角色型存取控制](role-based-access-control/role-assignments-portal.md) (RBAC)。 

## <a name="prerequisites"></a>必要條件
- 已選取「Azure 工作負載」的 [Visual Studio 2017](https://www.visualstudio.com/downloads/)，或是具備 [Microsoft Azure SDK for .NET 2.9](https://www.microsoft.com/en-us/download/details.aspx?id=51657) 的舊版 Visual Studio。
- Microsoft Azure 帳戶 - 如果您沒有帳戶，您可以[申請免費試用](http://go.microsoft.com/fwlink/?LinkId=623901)，或是[啟用您的 Visual Studio 訂閱者權益](http://go.microsoft.com/fwlink/?LinkId=623901)。

> [!NOTE]
> 若要檢視 Cloud Explorer，請在功能表列上，選取 [檢視] > [Cloud Explorer]。   
> 
> 

## <a name="add-an-azure-account-to-cloud-explorer"></a>將 Azure 帳戶新增到 Cloud Explorer
若要檢視與 Azure 帳戶關聯的資源，您必須先將帳戶新增到 Cloud Explorer。 

1. 在 [Cloud Explorer] 中，選取 [Azure 帳戶設定]。

    ![Cloud Explorer 的 [Azure 帳戶設定] 圖示](media/vs-azure-tools-resources-managing-with-cloud-explorer/azure-account-settings.png)

1. 選取 [管理帳戶]。 

    ![Cloud Explorer 的 [新增帳戶] 連結](media/vs-azure-tools-resources-managing-with-cloud-explorer/manage-accounts-link.png)

1. 登入您想要瀏覽資源的 Azure 帳戶。 

1. 登入 Azure 帳戶之後，系統就會顯示與該帳戶關聯的訂用帳戶。 選取您想要瀏覽之帳戶訂用帳戶的核取方塊，然後選取 [套用]。 
 
    ![Cloud Explorer：選取要顯示的 Azure 帳戶](media/vs-azure-tools-resources-managing-with-cloud-explorer/select-subscriptions.png)

1. 選取您想要瀏覽資源的訂用帳戶之後，Cloud Explorer 中就會顯示這些訂用帳戶和資源。

    ![Cloud Explorer 的 Azure 帳戶資源清單](media/vs-azure-tools-resources-managing-with-cloud-explorer/resources-listed.png)

## <a name="remove-an-azure-account-from-cloud-explorer"></a>從 Cloud Explorer 中移除 Azure 帳戶 

1. 在 [Cloud Explorer] 中，選取 [帳戶管理]。

    ![Cloud Explorer 的 [Azure 帳戶設定] 圖示](media/vs-azure-tools-resources-managing-with-cloud-explorer/azure-account-settings.png)

1. 在您要移除的帳戶旁邊，選取 [管理帳戶]。

    ![Cloud Explorer 的 [Azure 帳戶設定] 圖示](media/vs-azure-tools-resources-managing-with-cloud-explorer/remove-account.png)

1. 選擇 [移除] 以移除帳戶。

    ![Cloud Explorer 管理帳戶對話方塊](media/vs-azure-tools-resources-managing-with-cloud-explorer/accountmanage.PNG)

## <a name="view-resource-types-or-resource-groups"></a>檢視資源類型或資源群組
若要檢視您的 Azure 資源，您可以選擇 [資源類型] 或 [資源群組] 檢視。

1. 在 [Cloud Explorer] 中，選取資源檢視下拉式清單。

    ![可供選取所需資源檢視的 Cloud Explorer 下拉式清單](media/vs-azure-tools-resources-managing-with-cloud-explorer/resources-view-dropdown.png)

1. 從操作功能表中，選取所需的檢視： 

    - [資源類型] 檢視 - 在 [Azure 入口網站](http://go.microsoft.com/fwlink/p/?LinkID=525040)上使用的一般檢視，此檢視會依資源的類型 (例如 Web 應用程式、儲存體帳戶及虛擬機器) 分類來顯示 Azure 資源。 
    - [資源群組] 檢視 - 將 Azure 資源依關聯的 Azure 資源群組進行分類。 資源群組是通常由特定應用程式使用的 Azure 資源組合。 若要深入了解 Azure 資源群組，請參閱 [Azure Resource Manager 概觀](./azure-resource-manager/resource-group-overview.md)。

    下圖顯示這兩種資源檢視的比較：

    ![Cloud Explorer 資源檢視比較](media/vs-azure-tools-resources-managing-with-cloud-explorer/resource-views-comparison.png)

## <a name="view-and-navigate-resources-in-cloud-explorer"></a>在 Cloud Explorer 中檢視和瀏覽資源
若要在 Cloud Explorer 中瀏覽至 Azure 資源並檢視其資訊，請展開項目的類型或關聯的資源群組，然後選取資源。 當您選取資源時，Cloud Explorer 底部的 [動作] 和 [屬性] 這兩個索引標籤中會顯示資訊。 

- [動作] 索引標籤 - 列出您可以在 Cloud Explorer 中針對所選資源採取的動作。 您也可以在資源上按一下滑鼠右鍵來檢視其操作功能表，以檢視這些選項。

- [屬性]  索引標籤 - 顯示資源的屬性，例如其類型、地區設定及關聯的資源群組。

下圖顯示您在 App Service 的各個索引標籤上所看到畫面的範例比較：

![](./media/vs-azure-tools-resources-managing-with-cloud-explorer/actions-and-properties.png)

每個資源都有 [在入口網站中開啟] 這個動作。 當您選擇此動作時，[雲端總管] 會在 [Azure 入口網站](http://go.microsoft.com/fwlink/p/?LinkID=525040)中顯示選取的資源。 [在入口網站中開啟] 功能可方便您瀏覽至位於深層巢狀結構中的資源。

根據 Azure 資源而定，也可能出現其他動作和屬性值。 例如，除了 [在入口網站中開啟]，Web 應用程式和邏輯應用程式也有 [在瀏覽器中開啟] 和 [附加偵錯工具] 動作。 當您選擇儲存體帳戶 blob、佇列或資料表時，將會出現開啟編輯器的動作。 Azure 應用程式具有 **URL** 和 **狀態**屬性，而儲存體資源具有索引鍵和連接字串屬性。

## <a name="find-resources-in-cloud-explorer"></a>在 Cloud Explorer 中尋找資源
若要在 Azure 帳戶訂用帳戶中尋找具有特定名稱的資源，請在 Cloud Explorer 的 [搜尋] 方塊中輸入名稱。

![在 [雲端總管] 中尋找資源](./media/vs-azure-tools-resources-managing-with-cloud-explorer/search-for-resources.png)

當您在 [搜尋] 方塊中輸入字元時，只有符合這些字元的資源才會出現在資源樹狀目錄中。
