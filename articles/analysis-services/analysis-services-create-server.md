---
title: 快速入門 - 使用 Azure 入口網站建立 Analysis Services 伺服器 | Microsoft Docs
description: 瞭解如何在 Azure 中建立 Analysis Services 伺服器執行個體。
author: minewiskan
manager: kfile
ms.service: azure-analysis-services
ms.topic: quickstart
ms.date: 07/03/2018
ms.author: owend
ms.reviewer: minewiskan
ms.openlocfilehash: 92209d82a6db18b7f1e1a5adbb52c1899bf4593f
ms.sourcegitcommit: a1e1b5c15cfd7a38192d63ab8ee3c2c55a42f59c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/10/2018
ms.locfileid: "37950694"
---
# <a name="quickstart-create-a-server---portal"></a>快速入門：建立伺服器 - 入口網站

本快速入門說明如何使用入口網站在 Azure 訂用帳戶中建立 Analysis Services 伺服器資源。

## <a name="prerequisites"></a>先決條件 

* **Azure 訂用帳戶**︰瀏覽 [Azure 免費試用](https://azure.microsoft.com/offers/ms-azr-0044p/)建立帳戶。
* **Azure Active Directory**：您的訂用帳戶必須與 Azure Active Directory 租用戶相關聯。 而且，您必須使用該 Azure Active Directory 中的帳戶來登入 Azure。 若要深入了解，請參閱[驗證和使用者權限](analysis-services-manage-users.md)。

## <a name="log-in-to-the-azure-portal"></a>登入 Azure 入口網站 

[登入入口網站](https://portal.azure.com)


## <a name="create-a-server"></a>建立伺服器

1. 按一下 [+ 建立資源] > [資料 + 分析] > [Analysis Services]。

    ![入口網站](./media/analysis-services-create-server/aas-create-server-portal.png)

2. 在 [Analysis Services] 中，填寫必要的欄位，然後按 [建立]。
   
   * **伺服器名稱**︰輸入用來參考伺服器的唯一名稱。
   * **訂用帳戶**：選取要與此伺服器相關聯的訂用帳戶。
   * **資源群組**：建立新的資源群組，或選取您現有的資源群組。 資源群組是為了協助您管理 Azure 資源集合而設計。 若要深入了解，請參閱[資源群組](../azure-resource-manager/resource-group-overview.md)。
   * **位置**︰此 Azure 資料中心位置裝載著伺服器。 請選擇最靠近最大使用者群體的位置。
   * **定價層**：選取定價層。 如果您在進行測試，而且想要安裝範例模型資料庫，請選取免費的 **D1** 層。 若要深入了解，請參閱 [Azure Analysis Services 定價](https://azure.microsoft.com/pricing/details/analysis-services/)。 
    * **系統管理員**：根據預設，這會是您用來登入的帳戶。 您可以從 Azure Active Directory 選擇不同的帳戶。
    * **備份儲存體設定**：選擇性。 如果您已經有[儲存體帳戶](../storage/common/storage-introduction.md)，您可以將它指定為模型資料庫備份的預設值。 您也可以稍後指定[備份和還原](analysis-services-backup.md)設定。
    * **儲存體金鑰到期**：選擇性。 指定儲存體金鑰到期期間。

建立伺服器通常不到一分鐘即可完成。 如果您選取 [新增到入口網站]，請瀏覽至您的入口網站來查看新的伺服器。 或者，導覽至 [所有服務] > [Analysis Services]，以查看您的伺服器是否就緒。

## <a name="clean-up-resources"></a>清除資源

若不再需要，請刪除您的伺服器。 在伺服器的 [概觀] 中，按一下 [刪除]。 

 ![清除](./media/analysis-services-create-server/aas-create-server-cleanup.png)


## <a name="next-steps"></a>後續步驟
在本快速入門中，您已了解如何在 Azure 訂用帳戶中建立伺服器。 既然您有伺服器，您可設定 (選擇性) 伺服器防火牆來協助保護其安全。 您也可以直接從入口網站，將基本範例資料模型新增到您的伺服器。 範例模型有助於您了解如何設定模型資料庫角色以及測試用戶端連線。 若要深入了解，請繼續進行新增範例模型的教學課程。

> [!div class="nextstepaction"]
> [快速入門：設定伺服器防火牆 - 入口網站](analysis-services-qs-firewall.md)   
> [!div class="nextstepaction"]
> [教學課程：將範例模型新增至您的伺服器](analysis-services-create-sample-model.md)
