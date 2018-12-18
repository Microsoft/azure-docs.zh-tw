---
title: Azure Functions 執行階段安裝 | Microsoft Docs
description: 如何安裝 Azure Functions 執行階段預覽 2
services: functions
author: apwestgarth
manager: stefsch
ms.assetid: ''
ms.service: azure-functions
ms.devlang: multiple
ms.topic: conceptual
ms.date: 11/28/2017
ms.author: anwestg
ms.openlocfilehash: 1ad1d2c74be97afcb62f3f8e8161111f4938f645
ms.sourcegitcommit: af60bd400e18fd4cf4965f90094e2411a22e1e77
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/07/2018
ms.locfileid: "44093663"
---
# <a name="install-the-azure-functions-runtime-preview-2"></a>安裝 Azure Functions 執行階段預覽 2

如果您想要安裝 Azure Functions 執行階段預覽 2，請遵循下列步驟：

1. 請確定您的電腦符合最低需求。
1. 下載 [Azure Functions 執行階段預覽安裝程式](https://aka.ms/azafrv2)。
1. 解除安裝 Azure Functions 執行階段預覽 1。
1. 安裝 Azure Functions 執行階段預覽 2。
1. 完成 Azure Functions 執行階段預覽 2 的設定。
1. 在 Azure Functions 執行階段預覽中建立您的第一個函式

## <a name="prerequisites"></a>必要條件

安裝 Azure Functions 執行階段預覽之前，您必須備妥下列資源：

1. 執行 Microsoft Windows Server 2016 或 Microsoft Windows 10 Creators Update (Professional 或 Enterprise Edition) 的電腦。
1. 在您的網路內執行的 SQL Server 執行個體。  最小所需版本是 SQL Server Express。

## <a name="uninstall-previous-version"></a>解除安裝舊版本

如果您先前曾安裝 Azure Functions 執行階段預覽，就必須先解除安裝，然後再安裝最新版本。  在 Windows 的 [新增/移除程式] 中移除 Azure Functions 執行階段預覽，以解除安裝該程式。

## <a name="install-the-azure-functions-runtime-preview"></a>安裝 Azure Functions 執行階段預覽

Azure Functions 執行階段預覽安裝程式會引導您完成安裝 Azure Functions 執行階段預覽版的管理角色和背景工作角色。  管理角色和背景工作角色可以安裝在同一部電腦上。  不過，當您新增更多函數應用程式時，必須在其他電腦上部署更多背景工作角色，才能將您的函式擴展至多個背景工作。

## <a name="install-the-management-and-worker-role-on-the-same-machine"></a>在同一部電腦上安裝管理角色和背景工作角色

1. 執行 Azure Functions 執行階段預覽安裝程式。

    ![Azure Functions 執行階段預覽安裝程式][1]

1. 按 [下一步] 。
1. 閱讀 **EULA** 的條款之後，**勾選方塊接受條款**，然後按 [下一步] 前進。
1. 選取您想要在這部電腦上安裝的角色：**Functions 管理角色**和/或 **Functions 背景工作角色**，然後按一下 [下一步]。

    ![Azure Functions 執行階段預覽安裝程式 - 選取角色][3]

    > [!NOTE]
    > 您可以在許多其他機器上安裝 **Functions 背景工作角色**。 若要這樣做，請遵循這些指示，並在安裝程式中只選取 **Functions 背景工作角色**。

1. 按 [下一步] 讓 **Azure Functions 執行階段安裝精靈**開始在您的機器上進行安裝程序。
1. 完成後，安裝精靈會啟動 **Azure Functions 執行階段**設定工具。

    ![Azure Functions 執行階段預覽安裝程式完成][6]

    > [!NOTE]
    > 如果您要安裝在 **Windows 10**，但先前未啟用 [容器] 功能，**Azure Functions 執行階段安裝程式**會提示您重新開機才能完成安裝。

## <a name="configure-the-azure-functions-runtime"></a>設定 Azure Functions 執行階段

若要完成 Azure Functions 執行階段安裝，您必須完成設定。

1. **Azure Functions 執行階段**設定工具會顯示您的機器上安裝哪些角色。

    ![Azure Functions 執行階段預覽設定工具][7]

1. 按一下 [資料庫] 索引標籤、輸入 SQL Server 執行個體的連線詳細資料 (包括指定[資料庫主要金鑰](https://docs.microsoft.com/sql/relational-databases/security/encryption/sql-server-and-database-encryption-keys-database-engine))，然後按一下 [套用]。  需要與 SQL Server 執行個體連線，Azure Functions 執行階段才能建立資料庫以支援執行階段。

    ![Azure Functions 執行階段預覽資料庫設定][8]

1. 按一下 [認證] 索引標籤。您必須在這裡建立兩個新的認證，以使用檔案共用來裝載您的所有函數應用程式。  在 [檔案共用擁有者] 和 [檔案共用使用者] 中，指定 [使用者名稱] 和 [密碼] 組合，然後按一下 [套用]。

    ![Azure Functions 執行階段預覽認證][9]

1. 按一下 [檔案共用] 索引標籤。您必須在這裡指定檔案共用位置的詳細資料。  您可以建立檔案共用，也可以使用現有的檔案共用，然後按一下 [套用]。  如果您選取新的檔案共用位置，您必須指定供 Azure Functions 執行階段使用的目錄。

    ![Azure Functions 執行階段預覽檔案共用][10]

1. 按一下 [IIS] 索引標籤。此索引標籤會顯示 Azure Functions 執行階段設定工具要在 IIS 中建立之網站的詳細資料。  您可以在這裡指定自訂 DNS 名稱，以供 Azure Functions 執行階段預覽入口網站使用。  按一下 [套用] 完成。

    ![Azure Functions 執行階段預覽 IIS][11]

1. 按一下 [服務] 索引標籤。此索引標籤會顯示 Azure Functions 執行階段設定工具中的服務狀態。  如果 **Azure Functions 主機啟用服務**未在初始設定之後執行，請按一下 [啟動服務]。

    ![Azure Functions 執行階段預覽設定完成][12]

1. 瀏覽至 **Azure Functions 執行階段入口網站**：`https://<machinename>.<domain>/`。

    ![Azure Functions 執行階段預覽入口網站][13]

## <a name="create-your-first-function-in-azure-functions-runtime-preview"></a>在 Azure Functions 執行階段預覽中建立您的第一個函式

在 Azure Functions 執行階段預覽中建立您的第一個函式

1. 瀏覽至 **Azure Functions 執行階段入口網站**： https://<machinename>.<domain> 例如 https://mycomputer.mydomain.com
1. 如果是部署在使用您的網域帳戶使用者名稱與密碼的網域，系統就會提示您**登入**，否則請使用您的本機帳戶使用者名稱與密碼來登入入口網站。

![Azure Functions 執行階段預覽入口網站登入][14]

1. 若要建立函數應用程式，您必須建立訂用帳戶。  在入口網站左上角，按一下訂用帳戶旁的 [+] 選項

![Azure Functions 執行階段預覽入口網站訂用帳戶][15]

1. 選擇 [DefaultPlan]、輸入您的訂用帳戶名稱，然後按一下 [建立]。

![Azure Functions 執行階段預覽入口網站訂用帳戶計畫和名稱][16]

1. 入口網站的左側窗格中會列出您的所有函數應用程式。  若要建立新的函數應用程式，請選取 [函數應用程式] 標題，然後按一下 [+] 選項。

1. 輸入您的函數應用程式名稱、選取正確的訂用帳戶、選擇您想要據以進行程式設計的 Azure Functions 執行階段版本，然後按一下 [建立]

![Azure Functions 執行階段預覽入口網站新的函數應用程式][17]

1. 您的新函數應用程式會列於入口網站的左側窗格中。  選取 [函式]，然後按一下入口網站中央窗格頂端的 [新增函式]。

![Azure Functions 執行階段預覽範本][18]

1. 選取 [計時器觸發程序] 函式、在右側彈出式視窗中為您的函式命名並將 [排程] 變更為 `*/5 * * * * *` (這個 cron 運算式會使得您的計時器函式每隔五秒鐘執行一次)，然後按一下 [建立]

![Azure Functions 執行階段預覽新的計時器函式設定][19]

1. 現在已建立您的函式。  您可以透過展開入口網站底部的 [記錄] 窗格，來檢視函數應用程式的執行記錄。

![Azure Functions 執行階段預覽函式執行][20]

<!--Image references-->
[1]: ./media/functions-runtime-install/AzureFunctionsRuntime_Installer1.png
[2]: ./media/functions-runtime-install/AzureFunctionsRuntime_Installer2-EULA.png
[3]: ./media/functions-runtime-install/AzureFunctionsRuntime_Installer3-ChooseRoles.png
[4]: ./media/functions-runtime-install/AzureFunctionsRuntime_Installer4-Install.png
[5]: ./media/functions-runtime-install/AzureFunctionsRuntime_Installer5-Progress.png
[6]: ./media/functions-runtime-install/AzureFunctionsRuntime_Installer6-InstallComplete.png
[7]: ./media/functions-runtime-install/AzureFunctionsRuntime_Configuration1.png
[8]: ./media/functions-runtime-install/AzureFunctionsRuntime_Configuration2_SQL.png
[9]: ./media/functions-runtime-install/AzureFunctionsRuntime_Configuration3_Credentials.png
[10]: ./media/functions-runtime-install/AzureFunctionsRuntime_Configuration4_Fileshare.png
[11]: ./media/functions-runtime-install/AzureFunctionsRuntime_Configuration5_IIS.png
[12]: ./media/functions-runtime-install/AzureFunctionsRuntime_Configuration6_Services.png
[13]: ./media/functions-runtime-install/AzureFunctionsRuntime_Portal.png
[14]: ./media/functions-runtime-install/AzureFunctionsRuntime_Portal_Login.png
[15]: ./media/functions-runtime-install/AzureFunctionsRuntime_Portal_Subscriptions.png
[16]: ./media/functions-runtime-install/AzureFunctionsRuntime_Portal_Subscriptions1.png
[17]: ./media/functions-runtime-install/AzureFunctionsRuntime_Portal_NewFunctionApp.png
[18]: ./media/functions-runtime-install/AzureFunctionsRuntime_v1FunctionsTemplates.png
[19]: ./media/functions-runtime-install/AzureFunctionsRuntime_Portal_NewTimerFunction.png
[20]: ./media/functions-runtime-install/AzureFunctionsRuntime_Portal_RunningV2Function.png
