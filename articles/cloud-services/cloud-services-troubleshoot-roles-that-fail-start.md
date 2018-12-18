---
title: 針對無法啟動的角色進行疑難排解 | Microsoft Docs
description: 以下是雲端服務角色無法啟動的一些常見原因。 此外也提供這些問題的解決方案。
services: cloud-services
documentationcenter: ''
author: simonxjx
manager: felixwu
editor: ''
tags: top-support-issue
ms.assetid: 674b2faf-26d7-4f54-99ea-a9e02ef0eb2f
ms.service: cloud-services
ms.devlang: na
ms.topic: troubleshooting
ms.tgt_pltfrm: na
ms.workload: tbd
ms.date: 06/15/2018
ms.author: v-six
ms.openlocfilehash: 4b4669ecdae474c8926a346ed02f40913cf67265
ms.sourcegitcommit: e8f443ac09eaa6ef1d56a60cd6ac7d351d9271b9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/12/2018
ms.locfileid: "35766926"
---
# <a name="troubleshoot-cloud-service-roles-that-fail-to-start"></a>對無法啟動的雲端服務角色進行疑難排解
以下是與無法啟動的 Azure 雲端服務角色相關的一些常見問題和解決方案。

[!INCLUDE [support-disclaimer](../../includes/support-disclaimer.md)]

## <a name="missing-dlls-or-dependencies"></a>遺失 Dll 或相依性
角色沒有回應，以及角色在 [正在初始化]、[忙碌] 和 [正在停止] 狀態之間循環，有可能是因為遺失 DLL 或組件所致。

遺失 DLL 或組件的徵狀可能是：

* 您的角色執行個體在 [正在初始化]、[忙碌] 及 [正在停止] 狀態之間循環。
* 您的角色執行個體已進入 [就緒]  狀態，但當您瀏覽至 Web 應用程式時，發現頁面並未顯示。

有數個建議的方法可調查這些問題。

## <a name="diagnose-missing-dll-issues-in-a-web-role"></a>診斷 Web 角色中遺失 DLL 的問題
當您瀏覽至在 Web 角色中部署的網站時，瀏覽器顯示如下的伺服器錯誤，表示可能遺失 DLL。

!['/' 應用程式中有伺服器錯誤。](./media/cloud-services-troubleshoot-roles-that-fail-start/ic503388.png)

## <a name="diagnose-issues-by-turning-off-custom-errors"></a>關閉自訂錯誤以診斷問題
設定 Web 角色的 web.config 以將自訂錯誤模式設為關閉，並重新部署服務，可以檢視更完整的錯誤資訊。

若要檢視更完整的錯誤而不使用遠端桌面：

1. 在 Microsoft Visual Studio 中開啟解決方案。
2. 在 [方案總管] 中找出 web.config 檔案，並加以開啟。
3. 在 web.config 檔案中找出 system.web 區段，並加入下面這一行：

    ```xml
    <customErrors mode="Off" />
    ```
4. 儲存檔案。
5. 重新封裝並重新部署服務。

重新部署服務之後，您會看到下列錯誤訊息，以及遺失組件或 DLL 的名稱。

## <a name="diagnose-issues-by-viewing-the-error-remotely"></a>從遠端檢視錯誤以診斷問題
您可以使用遠端桌面存取角色，並從遠端檢視更完整的錯誤資訊。 使用下列步驟，可使用遠端桌面檢視錯誤：

1. 確定已安裝 Azure SDK 1.3 或更新版本。
2. 在使用 Visual Studio 部署解決方案期間，啟用「遠端桌面」。 如需詳細資訊，請參閱[使用 Visual Studio 啟用 Azure 雲端服務中角色的遠端桌面連線](cloud-services-role-enable-remote-desktop-visual-studio.md)。
3. 在 Microsoft Azure 入口網站中，一旦執行個體的狀態顯示為 [就緒]，即可遠端存取執行個體。 如需透過雲端服務使用遠端桌面的詳細資訊，請參閱[角色執行個體的遠端存取](cloud-services-role-enable-remote-desktop-new-portal.md#remote-into-role-instances)。
5. 使用在遠端桌面設定期間指定的認證登入虛擬機器。
6. 開啟命令視窗。
7. 輸入 `IPconfig`。
8. 記下 IPV4 位址值。
9. 開啟 Internet Explorer。
10. 輸入 Web 應用程式的位址和名稱。 例如， `http://<IPV4 Address>/default.aspx`。

瀏覽至網站現在會傳回更明確的錯誤訊息：

* '/' 應用程式中有伺服器錯誤。
* 說明：執行目前的 Web 要求期間發生未處理的例外狀況。 請檢閱堆疊追蹤，以進一步了解錯誤以及它產生於程式碼中的何處。
* 例外狀況詳細資料：System.IO.FIleNotFoundException：無法載入檔案或組件 ‘Microsoft.WindowsAzure.StorageClient，Version=1.1.0.0，Culture=neutral，PublicKeyToken=31bf856ad364e35’ 或其相依性之一。 系統找不到指定的檔案。

例如︰

!['/' 應用程式中有明確的伺服器錯誤。](./media/cloud-services-troubleshoot-roles-that-fail-start/ic503389.png)

## <a name="diagnose-issues-by-using-the-compute-emulator"></a>使用計算模擬器診斷問題
您可以使用 Microsoft Azure 計算模擬器來診斷和排解遺失相依性與 web.config 錯誤的問題。

若要讓這種診斷方法獲得最佳結果，您應該使用具有 Windows 全新安裝的電腦或虛擬機器。 若要達到模擬 Azure 環境的最佳效果，請使用 Windows Server 2008 R2 x64。

1. 安裝獨立版的 [Azure SDK](https://azure.microsoft.com/downloads/)。
2. 在開發電腦上建置雲端服務專案。
3. 在 Windows 檔案總管中，瀏覽至雲端服務專案的 bin\debug 資料夾。
4. 將 .csx 資料夾及 .cscfg 檔案複製到您用來偵錯問題的電腦。
5. 在初始狀態的電腦上開啟 Azure SDK 命令提示字元視窗，並輸入 `csrun.exe /devstore:start`。
6. 在命令提示字元中，輸入 `run csrun <path to .csx folder> <path to .cscfg file> /launchBrowser`。
7. 在角色啟動時，您會在 Internet Explorer 中看到詳細的錯誤資訊。 您也可以使用標準 Windows 疑難排解工具進一步診斷問題。

## <a name="diagnose-issues-by-using-intellitrace"></a>使用 IntelliTrace 診斷問題
對於使用 .NET Framework 4 的背景工作角色和 Web 角色，您可以使用 Microsoft Visual Studio Enterprise 所提供的 [IntelliTrace](https://msdn.microsoft.com/library/dd264915.aspx)。

請遵循下列步驟，部署已啟用 IntelliTrace 的服務：

1. 確定已安裝 Azure SDK 1.3 或更新版本。
2. 使用 Visual Studio 部署解決方案。 在部署期間，勾選 [為 .NET 4 角色啟用 IntelliTrace]  核取方塊。
3. 執行個體啟動後，請開啟 [伺服器總管] 。
4. 展開 [Azure]\\[雲端服務] 節點，並找出部署。
5. 展開部署，直到您看見角色執行個體。 以滑鼠右鍵按一下其中一個執行個體。
6. 選擇 [檢視 IntelliTrace 記錄檔] 。 [IntelliTrace 摘要]  隨即開啟。
7. 找出摘要的例外狀況區段。 如果有例外狀況，區段會標示 [例外狀況資料] 。
8. 展開 [例外狀況資料]，並尋找類似如下的 **System.IO.FileNotFoundException** 錯誤：

![例外狀況資料、遺失檔案或組件](./media/cloud-services-troubleshoot-roles-that-fail-start/ic503390.png)

## <a name="address-missing-dlls-and-assemblies"></a>解決遺失 Dll 和組件的問題
若要解決遺失 DLL 和組件錯誤，請遵循下列步驟：

1. 在 Visual Studio 中開啟解決方案。
2. 在 [方案總管] 中，開啟 [參考] 資料夾。
3. 按一下錯誤中識別的組件。
4. 在 [屬性] 窗格中找出 [複製本機] 屬性，並將值設為 **True**。
5. 重新部署雲端服務。

在確認所有錯誤皆已修正後，即可在未勾選 [為 .NET 4 角色啟用 IntelliTrace]  核取方塊的情況下部署服務。

## <a name="next-steps"></a>後續步驟
檢視更多雲端服務的 [疑難排解文章](https://azure.microsoft.com/documentation/articles/?tag=top-support-issue&product=cloud-services) 。

若要了解如何利用 Azure PaaS 電腦診斷資料對雲端服務角色問題進行疑難排解，請參閱 [Kevin Williamson 的部落格系列](http://blogs.msdn.com/b/kwill/archive/2013/08/09/windows-azure-paas-compute-diagnostics-data.aspx)。
