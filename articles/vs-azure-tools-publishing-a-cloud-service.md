---
title: 使用 Azure Tools 發佈雲端服務 | Microsoft Docs
description: 了解如何使用 Visual Studio 發佈 Azure 雲端服務專案。
services: visual-studio-online
author: ghogen
manager: douge
assetId: 1a07b6e4-3678-4cbf-b37e-4520b402a3d9
ms.prod: visual-studio-dev15
ms.technology: vs-azure
ms.workload: azure
ms.topic: conceptual
ms.date: 11/11/2017
ms.author: ghogen
ms.openlocfilehash: bde00dfbf4a7ffde90d1a9a3d57d3a2decf74cad
ms.sourcegitcommit: fa493b66552af11260db48d89e3ddfcdcb5e3152
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/23/2018
ms.locfileid: "31799459"
---
# <a name="publishing-a-cloud-service-using-visual-studio"></a>使用 Visual Studio 發佈雲端服務

Visual Studio 可直接將應用程式發佈至 Azure，並支援雲端服務的暫存和生產環境。 發佈時，您可以選取針對部署套件暫時要使用的部署環境及儲存體帳戶。

在開發和測試 Azure 應用程式時，可以使用 Web Deploy 以累加方式對 Web 角色發佈變更。 在將應用程式發佈至部署環境後，Web Deploy 可讓您直接將變更部署至執行 Web 角色的虛擬機器。 您不必在每次想要更新 Web 角色以測試變更時封裝和發佈整個 Azure 應用程式。 透過這個方法，即可在雲端提供 Web 角色變更來進行測試，而不必等到應用程式發佈至部署環境後才進行。

請使用下列程序來發佈 Azure 應用程式以及利用 Web Deploy 更新 Web 角色：

- 從 Visual Studio 發佈或封裝 Azure 應用程式
- 在開發和測試週期期間更新 Web 角色

## <a name="publish-or-package-an-azure-application-from-visual-studio"></a>從 Visual Studio 發佈或封裝 Azure 應用程式

在發佈 Azure 應用程式時，可以執行下列其中一項工作：

- 建立服務封裝：您可以從 [Azure 入口網站](https://portal.azure.com)，使用此封裝和服務組態檔將應用程式發佈至部署環境。

- 從 Visual Studio 發佈 Azure 專案：若要將應用程式直接發佈至 Azure，您可以使用 [發佈精靈]。 如需相關資訊，請參閱 [發佈 Azure 應用程式精靈](vs-azure-tools-publish-azure-application-wizard.md)。

### <a name="to-create-a-service-package-from-visual-studio"></a>從 Visual Studio 建立服務封裝

1. 準備好發佈應用程式時，請開啟 [方案總管]，開啟包含角色的 Azure 專案的捷徑功能表，然後選擇 [發佈]。

1. 若只要建立服務封裝，請遵循下列步驟：

   a. 開啟 Azure 專案的捷徑功能表，然後選擇 [封裝] 。

   b. 在 [封裝 Azure 應用程式]  對話方塊中選擇要建立封裝的服務組態，然後選擇組建組態。

   c. (選擇性) 若要在發佈雲端服務後，為其開啟「遠端桌面」，請選取 [啟用所有角色的遠端桌面]，然後選取 [設定] 來設定遠端桌面認證。 如需詳細資訊，請參閱[使用 Visual Studio 啟用 Azure 雲端服務中角色的遠端桌面連線](cloud-services/cloud-services-role-enable-remote-desktop-visual-studio.md)。

      如果要在發佈後偵錯雲端服務，請選取 [啟用所有角色的遠端偵錯工具] 來開啟遠端偵錯。

   d. 若要建立封裝，請選擇 [封裝]  連結。

      [檔案總管] 會顯示新建立之封裝的檔案位置。 您可以複製這個位置，以從 Azure 入口網站使用它。

   e. 若要將此封裝發佈至部署環境，您必須在建立雲端服務時使用此位置做為封裝位置，並透過 Azure 入口網站將此封裝部署到環境中。

1. (選用) 若要取消部署程序，請在活動記錄檔細目的捷徑功能表上，選擇 [取消並移除] 。 此命令會停止部署程序，並從 Azure 中刪除部署環境。 若要在部署後移除環境，請使用 Azure 入口網站。

1. (選用) 角色執行個體啟動後，Visual Studio 會自動在 [伺服器總管] 的 [雲端服務] 節點中顯示部署環境。 您可以從這裡檢視個別角色執行個體的狀態。 請參閱[使用雲端總管管理 Azure 資源](vs-azure-tools-resources-managing-with-cloud-explorer.md)。下圖顯示仍處於初始化狀態的角色執行個體：

    ![VST_DeployComputeNode](./media/vs-azure-tools-publishing-a-cloud-service/IC744134.png)

## <a name="update-a-web-role-as-part-of-the-development-and-testing-cycle"></a>在開發和測試週期期間更新 Web 角色

如果您的應用程式有穩定的後端基礎結構，但 Web 角色需要更頻繁的更新，您可以使用 Web Deploy，僅更新專案中的 Web 角色。 當您不想要重建並重新部署後端背景工作角色，或如果您有多個 Web 角色但只想要更新其中一個 Web 角色，Web Deploy 就能派上用場。

### <a name="requirements-for-using-web-deploy"></a>使用 Web Deploy 的需求

- **只能用於開發和測試目的：** 系統會直接對執行 Web 角色的虛擬機器進行變更。 如果此虛擬機器必須回收，您就會遺失變更，因為您發佈的原始封裝會用來重新建立角色的虛擬機器。 重新發佈應用程式以取得 Web 角色的最新變更。

- **只能更新 Web 角色：** 無法更新背景工作角色。 此外，您不能更新 `web role.cs` 中的 `RoleEntryPoint`。

- **只能支援 Web 角色的單一執行個體：** 所有 Web 角色在部署環境中都不能有多個執行個體。 不過，可支援多個各只有一個執行個體的 Web 角色。

- **啟用遠端桌面連線：** 此需求讓 Web Deploy 能夠使用使用者名稱和密碼來連線到虛擬機器，以將變更部署到執行網際網路資訊服務 (IIS) 的伺服器。 此外，您可能需要連線到虛擬機器，以將信任的憑證新增到此虛擬機器上的 IIS。 (此憑證可確保 Web Deploy 所使用的 IIS 的遠端連線安全無虞。)

下列程序假設您是使用 [發佈 Azure 應用程式]  精靈。

### <a name="enable-web-deploy-when-you-publish-your-application"></a>在發佈應用程式時啟用 Web Deploy

1. 若要啟用 [啟用所有 Web 角色的 Web Deploy]  選項，您必須先設定遠端桌面連線。 選取 [啟用所有角色的遠端桌面]，然後在出現的 [遠端桌面組態] 方塊中提供要用來進行遠端連線的認證。 請參閱[使用 Visual Studio 啟用 Azure 雲端服務中角色的遠端桌面連線](cloud-services/cloud-services-role-enable-remote-desktop-visual-studio.md)。

1. 若要啟用應用程式中所有 Web 角色的 Web Deploy，請選取 [啟用所有 Web 角色的 Web Deploy] 。

    此時會出現黃色警告三角形。 Web Deploy 預設會使用不受信任的自我簽署憑證，在上傳機密資料時不建議使用此憑證。 如果您需要確保機密資料在進行此程序時安全無虞，可以新增 SSL 憑證以用於 Web Deploy 連線。 此憑證必須是受信任的憑證。 如需詳細資訊，請參閱[讓 Web Deploy 安全無虞](#make-web-deploy-secure)。

1. 選擇 [下一步] 以顯示 [摘要] 畫面，然後選擇 [發佈] 部署雲端服務。

    雲端服務隨即進行發佈。 所建立的虛擬機器已啟用遠端連線的 IIS 功能，因此可以使用 Web Deploy 來更新 Web 角色，而不必重新發佈。

   > [!NOTE]
   > 如果您對 Web 角色設定了多個執行個體，則會出現警告訊息，指出在為了發佈應用程式所建立的封裝中，每個 Web 角色限制只能有一個執行個體。 選取 [確定]  以繼續操作。 如＜需求＞一節所述，您可以有多個 Web 角色，但每個角色只能有一個執行個體。

### <a name="update-your-web-role-by-using-web-deploy"></a>使用 Web Deploy 更新 Web 角色

1. 若要使用 Web Deploy，請對 Visual Studio 中您想要發佈的任何 Web 角色，變更其專案的程式碼，然後在方案中的這個專案節點上按一下滑鼠右鍵，並指向 [發佈] 。 [發佈 Web]  對話方塊隨即出現。

1. (選用) 如果您已新增受信任的 SSL 憑證以用於 IIS 的遠端連線，您可以清除 [允許未受信任的憑證]  核取方塊。 如需如何新增憑證以安全執行 Web Deploy，請參閱本文稍後的＜讓 Web Deploy 安全無虞＞  一節。

1. 若要使用 Web Deploy，發佈機制需要您在首次發佈套件時，針對遠端桌面連線所設定的使用者名稱和密碼。

   a. 在 [使用者名稱] 中輸入使用者名稱。

   b. 在 [密碼] 中輸入密碼。

   c. (選用) 如果您想要將此密碼儲存在此設定檔中，請選擇 [儲存密碼] 。

1. 若要發佈 Web 角色的變更，請選擇 [發佈] 。

    狀態列會顯示 [發佈已開始] 。 當發佈完成時，則會出現 [發佈成功]  。 現在變更已部署至虛擬機器上的 Web 角色。 您現在可以在 Azure 環境中啟動 Azure 應用程式來測試變更。

### <a name="make-web-deploy-secure"></a>讓 Web Deploy 安全無虞

1. Web Deploy 預設會使用不受信任的自我簽署憑證，在上傳機密資料時不建議使用此憑證。 如果您需要確保機密資料在進行此程序時安全無虞，可以新增 SSL 憑證以用於 Web Deploy 連線。 此憑證必須是從憑證授權單位 (CA) 取得的受信任憑證。

    若要讓每個 Web 角色的每個虛擬機器都能夠安全使用 Web Deploy，您必須將想要用於 Web Deploy 的受信任憑證上傳到 Azure 入口網站。 此憑證可確保在您發佈應用程式時，針對 Web 角色所建立的虛擬機器中會新增此憑證。

1. 若要將受信任的 SSL 憑證新增至 IIS 以用於遠端連線，請遵循下列步驟：

   a. 若要連接至執行 Web 角色的虛擬機器，請在 [雲端總管] 或 [伺服器總管] 中選取 Web 角色的執行個體，然後選擇 [使用遠端桌面連接] 命令。 如需如何連線至虛擬機器的詳細步驟，請參閱[使用 Visual Studio 啟用 Azure 雲端服務中角色的遠端桌面連線](cloud-services/cloud-services-role-enable-remote-desktop-visual-studio.md)。 您的瀏覽器會提示您下載 `.rdp` 檔案。

   b. 若要新增 SSL 憑證，請開啟 IIS 管理員中的管理服務。 在 IIS 管理員中，開啟 [動作] 窗格中的 [繫結] 連結來啟用 SSL。 [新增站台繫結]  對話方塊隨即出現。 選擇 [新增]，然後在 [類型] 下拉式清單中選擇 HTTPS。 在 [SSL 憑證]  清單中，選擇您已透過 CA 簽署並上傳至 Azure 入口網站的 SSL 憑證。 如需詳細資訊，請參閱 [設定管理服務的連線設定](http://go.microsoft.com/fwlink/?LinkId=215824)。

      > [!NOTE]
      > 如果您新增受信任的 SSL 憑證，[發佈精靈] 中就不會再出現黃色警告三角形。

## <a name="include-files-in-the-service-package"></a>將檔案納入服務封裝

您可能需要在服務封裝中納入特定檔案，以在為角色建立的虛擬機器上提供使用。 例如，您可以在服務封裝中加入啟動指令碼所使用的 `.exe` 或 `.msi` 檔案。 或者您可能需要加入 Web 角色或背景工作角色專案所需的組件。 若要納入檔案，必須將檔案加入 Azure 應用程式的方案中。

1. 若要將組件加入服務封裝中，請使用下列步驟：

   a. 在 [方案總管] 中開啟遺漏所參考組件之專案的專案節點。
   b. 若要將組件加入至專案，請開啟 [參考] 資料夾的捷徑功能表，然後選擇 [加入參考]。 [加入參考] 對話方塊隨即出現。
   c. 選擇您想要加入的參考，然後選擇 [確定]。 參考便會加入 [參考]  資料夾底下的清單。
   d. 開啟您所加入之組件的捷徑功能表，然後選擇 [屬性] 。 [屬性]  視窗隨即出現。

      若要將此組件納入服務封裝，請在 [複製本機清單] 中選擇 [True]。
1. 在 [方案總管]  中開啟遺漏所參考組件之專案的專案節點。

1. 若要將組件加入至專案，請開啟 [參考] 資料夾的捷徑功能表，然後選擇 [加入參考]。 [加入參考]  對話方塊隨即出現。

1. 選擇您想要加入的參考，然後選擇 [確定]  按鈕。

    參考便會加入 [參考]  資料夾底下的清單。

1. 開啟您所加入之組件的捷徑功能表，然後選擇 [屬性] 。 [屬性] 視窗隨即出現。

1. 若要將此組件納入服務封裝，請在 [複製本機清單] 中選擇 [True]。

1. 若要在已新增至 Web 角色專案的服務封裝中納入檔案，請開啟該檔案的捷徑功能表，然後選擇 [屬性] 。 在 [屬性] 視窗中，選擇 [建置動作] 清單方塊中的 [內容]。

1. 若要在已新增至背景工作角色專案的服務封裝中納入檔案，請開啟該檔案的捷徑功能表，然後選擇 [屬性] 。 在 [屬性] 視窗中，選擇 [複製到輸出目錄] 清單方塊中的 [有更新時才複製]。

## <a name="next-steps"></a>後續步驟

若要深入了解如何從 Visual Studio 發佈至 Azure，請參閱 [發佈 Azure 應用程式精靈](vs-azure-tools-publish-azure-application-wizard.md)。
