---
title: 在網站中使用 ReportViewer | Microsoft Docs
description: 本主題說明如何使用 Visual Studio ReportViewer 控制項 (可顯示儲存在 Microsoft Azure 虛擬機器上的報告)，建置 Microsoft Azure 網站。
services: virtual-machines-windows
documentationcenter: na
author: markingmyname
manager: erikre
editor: monicar
tags: azure-service-management
ms.assetid: 78b76318-d9bf-48ef-9d9e-d1b7d8cf3042
ms.service: virtual-machines-sql
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 01/11/2017
ms.author: maghan
ms.openlocfilehash: af8a4a9c25005925bed3ddb78ced618e669f7f09
ms.sourcegitcommit: 9cdd83256b82e664bd36991d78f87ea1e56827cd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/16/2018
ms.locfileid: "31424613"
---
# <a name="use-reportviewer-in-a-web-site-hosted-in-azure"></a>在裝載於 Azure 上的網站中使用 ReportViewer
> [!IMPORTANT] 
> Azure 建立和處理資源的部署模型有二種： [Resource Manager 和傳統](../../../azure-resource-manager/resource-manager-deployment-model.md)。 本文涵蓋之內容包括使用傳統部署模型。 Microsoft 建議讓大部分的新部署使用 Resource Manager 模式。

您可以使用 Visual Studio ReportViewer 控制項 (可顯示儲存在 Microsoft Azure 虛擬機器上的報告)，建置 Microsoft Azure 網站。 ReportViewer 控制項位於您使用 ASP.NET Web 應用程式範本建置的 Web 應用程式中。

> [!IMPORTANT]
> ASP.NET MVC Web 應用程式範本不支援 ReportViewer 控制項。

若要將 ReportViewer 併入 Microsoft Azure 網站中，您需要完成下列工作。

* **新增** 組件至部署套件
* **設定** 驗證和授權
* **發佈** 至 Azure

## <a name="prerequisites"></a>先決條件
檢閱 [Azure 虛擬機器中的 SQL Server Business Intelligence](../classic/ps-sql-bi.md)中的＜一般建議和最佳作法＞一節。

> [!NOTE]
> ReportViewer 控制項隨附於 Visual Studio (Standard Edition 以上版本) 中。 如果您使用的是 Web Developer Express 版，您必須安裝 [Microsoft Report Viewer 2012 Runtime](https://www.microsoft.com/download/details.aspx?id=35747) ，才能使用 ReportViewer 執行階段功能。
> 
> Microsoft Azure 不支援在本機處理模式中設定的 ReportViewer。

## <a name="adding-assemblies-to-the-deployment-package"></a>將組件新增至部署套件
在內部部署裝載 ASP.NET 應用程式後，通常會在 Visual Studio 安裝期間，於 IIS 伺服器的全域組件快取 (GAC) 中直接安裝 ReportViewer 組件，並可直接透過應用程式存取。 不過，若您是在雲端中裝載 ASP.NET 應用程式，Microsoft Azure 不允許任何項目安裝至 GAC，因此您必須確定 ReportViewer 組件位於本機上，可供應用程式使用。 作法：將組件的參考加入您的專案中，並設定為可從本機複製。

在遠端處理模式中，ReportViewer 控制項會使用下列組件：

* **Microsoft.ReportViewer.WebForms.dll**：包含您要在頁面中使用 ReportViewer 所需的 ReportViewer 程式碼。 將 ReportViewer 控制項拖曳至專案的 ASP.NET 頁面中後，此組件的參考便會加入專案之中。
* **Microsoft.ReportViewer.Common.dll**：包含 ReportViewer 控制項在執行階段使用的類別。 此組件不會自動加入至您的專案。

### <a name="to-add-a-reference-to-microsoftreportviewercommon"></a>加入 Microsoft.ReportViewer.Common 的參考
* 在專案的 [參考] 節點上按一下滑鼠右鍵，選取 [加入參考]，接著在[.NET] 索引標籤中選取該組件，然後按一下 [確定]。

### <a name="to-make-the-assemblies-locally-accessible-by-your-aspnet-application"></a>若要讓組件可由 ASP.NET 應用程式於本機上存取
1. 在 [參考]  資料夾中，按一下 Microsoft.ReportViewer.Common 組件，使其屬性顯示在 [屬性] 窗格中。
2. 在 [屬性] 窗格中，將 [複製到本機]  設為 [True]。
3. 對 Microsoft.ReportViewer.WebForms 重複步驟 1 和 2。

### <a name="to-get-reportviewer-language-pack"></a>取得 ReportViewer 語言套件
1. 安裝從 [Microsoft 下載中心](http://go.microsoft.com/fwlink/?LinkId=317386)下載的適當 Microsoft Report Viewer 2012 Runtime 可轉散發套件。
2. 從下拉式清單中選取語言，然後頁面會重新導向至對應的下載中心頁面。
3. 按 [下載]  即可開始下載 ReportViewerLP.exe。
4. 下載 ReportViewerLP.exe 之後，按一下 [執行] 以立即安裝，或按一下 [儲存]，將其儲存至電腦中。 如果您按一下 [儲存] ，請記住儲存檔案的目的地資料夾名稱。
5. 尋找您儲存檔案的目的地資料夾。 在 ReportViewerLP.exe 上按一下滑鼠右鍵，按一下 [以系統管理員身分執行]，然後按一下 [是]。
6. 執行 ReportViewerLP.exe 之後，您會看到 c:\windows\assembly 中有資源檔案 **Microsoft.ReportViewer.Webforms.Resources** 和 **Microsoft.ReportViewer.Common.Resources**。

### <a name="to-configure-for-localized-reportviewer-control"></a>設定當地語系化的 ReportViewer 控制項
1. 依照上述的指示，下載並安裝 Microsoft Report Viewer 2012 Runtime 可轉散發套件。
2. 在專案中建立 <language> 資料夾，並複製該資料夾中的相關聯資源組件檔案。 需複製的資源組件檔案為：**Microsoft.ReportViewer.Webforms.Resources.dll** 和 **Microsoft.ReportViewer.Common.Resources.dll**。選取資源組件檔案，然後在 [屬性] 窗格中，將 [複製到輸出目錄] 設為 [永遠複製]。
3. 設定 Web 專案的文化特性和 UI 文化特性。 如需關於設定 ASP.NET 網頁的文化特性和 UI 文化特性的詳細資訊，請參閱 [作法：設定 ASP.NET Web 網頁全球化的文化特性和 UI 文化特性](http://go.microsoft.com/fwlink/?LinkId=237461)。

## <a name="configuring-authentication-and-authorization"></a>設定驗證和授權
ReportViewer 必須使用正確的認證對報表伺服器進行驗證，而且認證必須由報表伺服器授權，才能存取您需要的報表。 如需驗證的資訊，請檢閱白皮書《 [Reporting Services 報告檢視器控制項和 Microsoft Azure 虛擬機器型報表伺服器](https://msdn.microsoft.com/library/azure/dn753698.aspx)》。

## <a name="publish-the-aspnet-web-application-to-azure"></a>將 ASP.NET Web 應用程式發佈至 Azure
如需有關將 ASP.NET Web 應用程式發佈至 Azure 的指示，請參閱[做法：從 Visual Studio 將 Web 應用程式移轉並發佈至 Azure](../../../vs-azure-tools-migrate-publish-web-app-to-cloud-service.md) 和[開始使用 Web Apps 和 ASP.NET](../../../app-service/app-service-web-get-started-dotnet.md)。

> [!IMPORTANT]
> 如果新增 Azure 部署專案，或新增 Azure 雲端服務專案命令沒有顯示在 [方案總管] 的捷徑功能表中，您可能需要將專案的目標 Framework 變更為 .NET Framework 4。
> 
> 兩個命令提供的功能基本上相同。 視您安裝的 Microsoft Azure SDK 版本而定，其中一個命令會顯示在捷徑功能表中。
> 
> 

## <a name="resources"></a>資源
[Microsoft 報告](http://go.microsoft.com/fwlink/?LinkId=205399)

[Azure 虛擬機器中的 SQL Server Business Intelligence](../classic/ps-sql-bi.md)

[使用 PowerShell 建立具有原生模式報表伺服器的 Azure VM](../classic/ps-sql-report.md)

