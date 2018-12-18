---
title: 使用 Visual Studio 的已連接服務加入行動服務 | Microsoft Docs
description: 使用 Visual Studio 的加入已連接服務對話方塊加入行動服務
services: visual-studio-online
documentationcenter: na
author: mlhoop
manager: douge
editor: ''
ms.assetid: 75c3cb93-88e1-476d-a416-f34caa3608e3
ms.service: visual-studio-online
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: mobile
ms.custom: vs-azure
ms.date: 12/16/2015
ms.author: mlearned
ms.openlocfilehash: 6cceb760ab62ea4f7225c9cc0cee7a6baf8ed8be
ms.sourcegitcommit: fab878ff9aaf4efb3eaff6b7656184b0bafba13b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/22/2018
ms.locfileid: "42442206"
---
# <a name="adding-mobile-services-by-using-visual-studio-connected-services"></a>使用 Visual Studio 已連接服務加入行動服務
透過 Visual Studio 2015，您可以使用 [ **加入已連接服務** ] 對話方塊連接到 Azure 行動服務。 您可以從任何 C# 用戶端應用程式、任何 JavaScript 應用程式或跨平台 Cordova 應用程式連接。 連接後，您可以建立和存取資料、建立自訂 API 和排程的工作，或加入推播通知的支援。  已連接服務作業會加入所有適當的參考和連接程式碼。 您也可以利用包含各種熱門身分識別配置 (例如 Azure AD、Facebook、Twitter 和 Microsoft 帳戶) 的內建驗證支援。

## <a name="supported-project-types"></a>支援的專案類型
> [!NOTE]
> 在 Visual Studio 2015 中，不支援使用 [加入已連接服務] 對話方塊將 Azure 行動服務加入至 Windows 通用 (Windows 10) 專案。 使用 NuGet 封裝管理員為您的專案安裝適當的封裝，即可加入 Azure 行動服務。
> 
> 

您可以使用 [已連接服務] 對話方塊來連接到 Azure Mobile Services 中的下列專案類型。

* .NET Windows 8.1 市集、電話和通用應用程式專案
* JavaScript Windows 8.1 市集、電話和通用應用程式專案
* 使用 Visual Studio Tools for Apache Cordova 建立的專案

## <a name="connect-to-azure-mobile-services-using-the-add-connected-services-dialog"></a>使用 [加入已連接服務] 對話方塊連接到 Azure 行動服務
1. 確定您具有 Azure 帳戶。 如果您沒有 Azure 帳戶，您可以註冊 [免費試用](http://go.microsoft.com/fwlink/?LinkId=518146)。
2. 開啟 [加入已連接服務]  對話方塊。
   
   * 若為 .NET 應用程式，請在 Visual Studio 中開啟您的專案，開啟 [方案總管] 中 [參考] 節點的內容功能表，然後選擇 [加入已連接服務]
     
        ![連線至 Azure 行動服務](./media/vs-azure-tools-connected-services-add-mobile-services/IC797635.png)
   * 若為 Apache Cordova 應用程式專案，請在 Visual Studio 中開啟您的專案，開啟 [方案總管] 中專案節點的內容功能表，然後選擇 [加入已連接服務] 。
3. 在 [加入已連接服務] 對話方塊中，選擇 [Azure 行動服務]，然後選擇 [設定] 按鈕。 如果您尚未登入 Azure，系統可能會提示您這麼做。
   
    ![加入 Azure 行動服務](./media/vs-azure-tools-connected-services-add-mobile-services/IC797636.png)
4. 在 [Azure 行動服務]  對話方塊中，選擇現有的行動服務 (如果有的話)。 如果您需要建立新的 Azure 行動服務，請遵循下面的程序。 否則請跳到下一步。
   
    若要建立新的行動服務帳戶︰
   
   1. 選擇對話方塊底部的 [建立服務]**** 連結。
       ![加入新的已連接行動服務](./media/vs-azure-tools-connected-services-add-mobile-services/IC797637.png)
   2. 在 [建立行動服務] 對話方塊上，您可以選擇 JavaScript 後端行動服務，或選擇 [執行階段] 下拉式清單中的 .NET 後端行動服務。 
      
       ![建立行動服務](./media/vs-azure-tools-connected-services-add-mobile-services/IC797638.png)
      
       JavaScript 後端服務不但簡單而且功能強大。 如果您建立 JavaScript 後端行動服務，伺服器端 JavaScript 程式碼會儲存在雲端中，但您可以使用 [伺服器總管] 或 Azure 管理入口網站來編輯伺服器指令碼。 
      
       .NET 後端行動服務讓您擁有 Web API 和 Entity Framework 的完整功能與彈性。 如果您建立 .NET 後端行動服務，則系統會為您建立專案並加入至您的解決方案。 
   3. 選擇您想要行動服務的 [區域]  ，然後輸入伺服器的使用者名稱和密碼。
   4. 在您輸入所有必要的資訊之後，選擇 [建立]  按鈕以建立行動服務。
   5. 新行動服務應該會顯示在 [Azure 行動服務]  對話方塊的服務清單中。 選擇清單中的新行動服務，然後選擇 [新增]  按鈕將服務加入至您的專案。
5. 檢閱所顯示的 [開始使用] 頁面並了解您的專案修改方式。 每當您加入已連接服務時，您的瀏覽器就會出現 [開始使用] 頁面。 您可以檢閱建議的後續步驟和程式碼範例，或切換到 [發生什麼情形] 頁面以查看哪些參考已加入至您的專案，以及您的程式碼和組態檔的修改方式。
6. 使用程式碼範例做為指南，開始撰寫程式碼以存取您的行動服務！

## <a name="how-your-project-is-modified"></a>您的專案修改方式
Visual Studio 修改您的專案的方式視專案類型而定。 若為 C# 用戶端應用程式，請參閱 [發生什麼情形 – C# 專案](http://go.microsoft.com/fwlink/p/?LinkId=513119)。 若為 JavaScript 用戶端應用程式，請參閱 [發生什麼情形 – JavaScript 專案](http://go.microsoft.com/fwlink/p/?LinkId=513120)。 若為 Cordova 應用程式，請參閱 [發生什麼情形 – Cordova 專案](http://go.microsoft.com/fwlink/p/?LinkId=513116)。

## <a name="next-steps"></a>後續步驟
提出問題並取得協助︰ 

* [MSDN 論壇：Azure 行動服務](https://social.msdn.microsoft.com/forums/azure/home?forum=azuremobile)
* [Microsoft Azure 團隊部落格上的 Azure 行動服務](https://azure.microsoft.com/blog/topics/mobile/)
* [azure.microsoft.com 上的 Azure 行動服務](https://azure.microsoft.com/services/mobile-services/)
* [azure.microsoft.com 上的 Azure 行動服務文件](https://azure.microsoft.com/documentation/services/mobile-services/)

