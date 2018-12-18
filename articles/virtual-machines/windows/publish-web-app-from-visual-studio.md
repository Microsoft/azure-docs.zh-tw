---
title: 從 Visual Studio 將 Web 應用程式發行到 Azure VM
description: 從 Visual Studio 將 ASP.NET Web 應用程式發行到 Azure 虛擬機器
services: virtual-machines-windows
author: ghogen
manager: douge
tags: azure-service-management
ms.assetid: 70267837-3629-41e0-bb58-2167ac4932b3
ms.prod: visual-studio-dev15
ms.technology: vs-azure
ms.custom: vs-azure
ms.workload: azure-vs
ms.topic: conceptual
ms.date: 11/03/2017
ms.author: ghogen
ms.openlocfilehash: d5fd6041f21700ffddd3b2c4755b7ce01691681e
ms.sourcegitcommit: f3bd5c17a3a189f144008faf1acb9fabc5bc9ab7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/10/2018
ms.locfileid: "44304075"
---
# <a name="publish-an-aspnet-web-app-to-an-azure-vm-from-visual-studio"></a>從 Visual Studio 將 ASP.NET Web 應用程式發行到 Azure VM

本文件說明如何使用 Visual Studio 2017 中的 [Microsoft Azure 虛擬機器] 發佈功能，將 ASP.NET Web 應用程式發行到 Azure 虛擬機器 (VM)。  

## <a name="prerequisites"></a>必要條件
若要使用 Visual Studio 將 ASP.NET 專案發行到 Azure VM，必須正確設定 VM。

- 機器必須設定為執行 ASP.NET web 應用程式，並安裝 WebDeploy。

- VM 必須設定 DNS 名稱。 如需詳細資訊，請參閱[在 Azure 入口網站中為 Windows VM 建立完整網域名稱](portal-create-fqdn.md)。

## <a name="publish-your-aspnet-web-app-to-the-azure-vm-using-visual-studio"></a>使用 Visual Studio 將您的 ASP.NET Web 應用程式發行到 Azure VM
下列章節說明如何將現有的 ASP.NET Web 應用程式發行到 Azure 虛擬機器。

1. 在 Visual Studio 2017 中開啟您的 Web 應用程式。
2. 在方案總管中以滑鼠右鍵按一下專案，並選擇 [發行...]
3. 使用頁面右邊的箭頭捲動發行選項，直到您找到 [Microsoft Azure 虛擬機器] 為止。  

   ![發行頁面 - 發行按鈕]

4. 選取 [Microsoft Azure 虛擬機器] 圖示，然後選擇 [發行]。

   ![發行頁面 - Microsoft Azure 虛擬機器圖示]

5. 選擇適當的帳戶 (具有連線到虛擬機器的 Azure 訂用帳戶)。  
   - 若您已登入 Visual Studio，帳戶清單會填入您所有已驗證的帳戶。  
   - 如果您未登入或未列出您需要的帳戶，請選擇 [新增帳戶...]，並遵循提示來登入。  
   ![Azure 帳戶選取器]  

6. 從現有的虛擬機器清單中選取適當的 VM。

   > [!Note]
   > 填入此清單可能需要一些時間。

   ![Azure VM 選取器]

7. 按一下 [確定] 以開始發行。

8. 系統提示您輸入認證時，請提供已設定發佈權限的目標 VM 上使用者帳戶的使用者名稱密碼 (通常是建立 VM 時使用的管理員使用者名稱和密碼)。  

   ![WebDeploy 登入]

9. 接受安全性憑證。

   ![憑證錯誤]

10. 查看 [輸出] 視窗來檢查發行作業的進度。

    ![輸出視窗]

11. 如果已成功發行，便會啟動瀏覽器以開啟新發行網站的 URL。

**成功！**

您現在已經成功將 Web 應用程式發行到 Azure 虛擬機器。

## <a name="publish-page-options"></a>發行頁面選項

完成發行精靈之後，會在文件中開啟 [發行] 頁面，並預先選取新的發行設定檔。

### <a name="re-publish"></a>重新發行

若要將更新發佈到 Web 應用程式，請選取發行頁面上的 [發行] 按鈕。  
- 若出現提示，請輸入使用者名稱和密碼。  
- 會立即開始發行。

![發行頁面 - 發行按鈕]

### <a name="modify-publish-profile-settings"></a>修改發行設定檔設定

若要檢視和修改發行設定檔設定，請選取 [設定...]。  

![發行頁面 - 設定按鈕]

設定應該會看起來像這樣：  

![發行設定 - 連線頁面]

#### <a name="save-user-name-and-password"></a>儲存使用者名稱和密碼
- 若要避免每次發行都需提供驗證資訊，您可以填入 [使用者名稱] 和 [密碼] 欄位，然後勾選 [儲存密碼] 方塊。
- 使用 [驗證連線] 按鈕可確認您已輸入正確的資訊。

#### <a name="deploy-to-clean-web-server"></a>部署到乾淨的 Web 伺服器

- 如果您想要確保每次上傳後 Web 伺服器都會有乾淨的 Web 應用程式副本 (且不會有先前部署所留下的其他檔案)，可以勾選 [設定] 索引標籤中的 [移除目的地的其他檔案] 核取方塊。

- 警告：以這項設定發行會刪除位於 Web 伺服器 (wwwroot 目錄) 上的所有檔案。 請確定您知道啟用這個選項下，發行前的電腦狀態。 

![發行設定 - 設定頁面]

## <a name="next-steps"></a>後續步驟

### <a name="set-up-cicd-for-automated-deployment-to-azure-vm"></a>設定 CI/CD 以自動化部署到 Azure VM

若要以 Azure Pipelines 設定持續傳遞管線，請參閱[部署到 Windows 虛擬機器](https://docs.microsoft.com/vsts/build-release/apps/cd/deploy-webdeploy-iis-deploygroups)。

[VM Overview - DNS Name]: ../../../includes/media/publish-web-app-from-visual-studio/VMOverviewDNSName.png
[IP Address Config - DNS Name]: ../../../includes/media/publish-web-app-from-visual-studio/IPAddressConfigDNSName.png
[VM Overview - DNS Configured]: ../../../includes/media/publish-web-app-from-visual-studio/VMOverviewDNSConfigured.png
[發行頁面 - 發行按鈕]: ../../../includes/media/publish-web-app-from-visual-studio/PublishPageRightArrow.png
[發行頁面 - Microsoft Azure 虛擬機器圖示]: ../../../includes/media/publish-web-app-from-visual-studio/PublishPageMicrosoftAzureVirtualMachineIcon.png
[Azure 帳戶選取器]: ../../../includes/media/publish-web-app-from-visual-studio/ChooseVM-SelectAccount.png
[Azure VM 選取器]: ../../../includes/media/publish-web-app-from-visual-studio/ChooseVM-SelectVM.png
[WebDeploy 登入]: ../../../includes/media/publish-web-app-from-visual-studio/WebDeployLogin.png
[憑證錯誤]: ../../../includes/media/publish-web-app-from-visual-studio/CertificateError.png
[輸出視窗]: ../../../includes/media/publish-web-app-from-visual-studio/OutputWindow.png
[發行頁面 - 發行按鈕]: ../../../includes/media/publish-web-app-from-visual-studio/PublishPagePublishButton.png
[發行頁面 - 設定按鈕]: ../../../includes/media/publish-web-app-from-visual-studio/PublishPageSettingsButton.png
[發行設定 - 連線頁面]: ../../../includes/media/publish-web-app-from-visual-studio/PublishSettingsConnectionPage.png
[發行設定 - 設定頁面]: ../../../includes/media/publish-web-app-from-visual-studio/PublishSettingsSettingsPage.png
