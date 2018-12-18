---
title: 使用 Visual Studio 的加入已連接服務加入 Azure 儲存體 | Microsoft Docs
description: 使用 Visual Studio 的加入已連接服務對話方塊加入 Azure 儲存體
services: visual-studio-online
author: ghogen
manager: douge
assetId: 521ec044-ad4b-4828-8864-01decde2e758
ms.prod: visual-studio-dev15
ms.technology: vs-azure
ms.workload: azure
ms.topic: conceptual
ms.date: 03/26/2017
ms.author: ghogen
ms.openlocfilehash: 3c5d3dc1d91a6f8bb1816b2985f86ec5c4a12e63
ms.sourcegitcommit: fa493b66552af11260db48d89e3ddfcdcb5e3152
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/23/2018
---
# <a name="adding-azure-storage-by-using-visual-studio-connected-services"></a>使用 Visual Studio 的已連接服務加入 Azure 儲存體
使用 Visual Studio 時，您可以使用 [加入已連接服務] 對話方塊，將下列任何服務連接到「Azure 儲存體」：

- C# 雲端服務
- .NET 後端行動服務
- ASP.NET 網站或服務
- ASP.NET Core 服務
- Azure WebJob 服務 

已連接服務功能會將所有必要的參考和連接程式碼新增到您的專案中，並適當地修改組態檔。 

完成之後，[加入已連接服務] 對話方塊會自動顯示文件，當中會詳細說明開始使用 Blob 儲存體、佇列及資料表所需的步驟。

## <a name="connect-to-azure-storage-using-the-connected-services-dialog"></a>使用已連接服務對話方塊連接到 Azure 儲存體
1. 在 Visual Studio 中開啟您的專案

1. 在 [方案總管] 中，於 [已連接服務] 節點上按一下滑鼠右鍵，然後從操作功能表中選取 [加入已連接服務]。
   
    ![新增 Azure 已連接服務](./media/vs-azure-tools-connected-services-storage/IC796702.png)

1. 在 [已連接服務] 頁面中，選取 [使用 Azure 儲存體的雲端儲存體]。
   
    ![新增 Azure 儲存體](./media/vs-azure-tools-connected-services-storage/add-azure-storage.png)

1. 在 [Azure 儲存體] 對話方塊中，選取現有的儲存體帳戶，然後選取 [新增]。
   
    如果您需要建立儲存體帳戶，請移至下一個步驟。 否則，跳到步驟 6。
    
    ![將現有的儲存體帳戶新增到專案](./media/vs-azure-tools-connected-services-storage/select-azure-storage-account.png)

1. 建立儲存體帳戶： 
   
   1. 選取對話方塊底部的 [建立新的儲存體帳戶]。

   1. 填寫 [建立儲存體帳戶] 對話方塊，然後選取 [建立]。
      
       ![新的 Azure 儲存體帳戶](./media/vs-azure-tools-connected-services-storage/create-storage-account.png)
      
   1. 顯示 [Azure 儲存體] 對話方塊時，新的儲存體帳戶會出現在清單中。 選取清單中的新儲存體帳戶，然後選取 [新增]。

1. 儲存體已連接服務會出現在您專案的 [服務參考] 節點之下。
   
## <a name="how-your-project-is-modified"></a>您的專案修改方式
當您完成對話方塊時，Visual Studio 會加入參考並修改某些組態檔。 特定變更會視專案類型而定： 

- ASP.NET 專案 - [發生什麼事 - ASP.NET 專案](http://go.microsoft.com/fwlink/p/?LinkId=513126)
- ASP.NET Core 專案 - [發生什麼事 - ASP.NET 5 專案](http://go.microsoft.com/fwlink/p/?LinkId=513124) 
- 雲端服務專案 (Web 角色和背景工作角色) - [發生什麼事 - 雲端服務專案](http://go.microsoft.com/fwlink/p/?LinkId=516965)
- WebJob 專案 - [發生什麼事 - WebJob 專案](visual-studio/vs-storage-webjobs-what-happened.md)

## <a name="next-steps"></a>後續步驟
- [MSDN 論壇︰Azure 儲存體](https://social.msdn.microsoft.com/forums/azure/home?forum=windowsazuredata)
- [Microsoft Azure 儲存體小組部落格 (英文)](http://blogs.msdn.com/b/windowsazurestorage/)
- [Azure 儲存體文件](https://docs.microsoft.com/azure/storage/)
