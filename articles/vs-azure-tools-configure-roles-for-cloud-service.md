---
title: 使用 Visual Studio 設定 Azure 雲端服務的角色 | Microsoft Docs
description: 了解如何使用 Visual Studio 設定 Azure 雲端服務的角色。
services: visual-studio-online
author: ghogen
manager: douge
assetId: d397ef87-64e5-401a-aad5-7f83f1022e16
ms.prod: visual-studio-dev15
ms.technology: vs-azure
ms.workload: azure
ms.topic: conceptual
ms.date: 03/21/2017
ms.author: ghogen
ms.openlocfilehash: 09e6c3a9c27342ef27d49674d62ccf74d70d2e0f
ms.sourcegitcommit: fa493b66552af11260db48d89e3ddfcdcb5e3152
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/23/2018
ms.locfileid: "31798721"
---
# <a name="configure-azure-cloud-service-roles-with-visual-studio"></a>使用 Visual Studio 設定 Azure 雲端服務角色
Azure 雲端服務可以有一或多個背景工作角色或 web 角色。 您必須為每個角色定義其設定方式，並設定該角色的執行方式。 若要深入了解雲端服務中的角色，請參閱視訊 [Azure 雲端服務簡介](https://channel9.msdn.com/Series/Windows-Azure-Cloud-Services-Tutorials/Introduction-to-Windows-Azure-Cloud-Services)。 

您的雲端服務資訊儲存在下列檔案中：

- **ServiceDefinition.csdef** - 服務定義檔可定義雲端服務的執行階段設定，包括需要哪些角色、端點和虛擬機器大小。 當您的角色正在執行時，不可以變更儲存在 `ServiceDefinition.csdef` 中的任何資料。
- **ServiceConfiguration.cscfg** - 服務組態檔可設定一個角色可以執行的執行個體數目，以及為角色定義的設定值。 當您的角色正在執行時，可以變更儲存在 `ServiceConfiguration.cscfg` 中的資料。

若要為可控制角色執行方式的設定儲存不同的值，您可以定義多個服務組態。 您可以將不同的服務組態用於每個部署環境。 例如，您可以設定儲存體帳戶連接字串，在本機服務組態中使用本機 Azure 儲存體模擬器，並建立另一個服務組態，在雲端中使用 Azure 儲存體。

當您在 Visual Studio 中建立 Azure 雲端服務時，會自動建立兩個服務組態並新增至您的 Azure 專案：

- `ServiceConfiguration.Cloud.cscfg`
- `ServiceConfiguration.Local.cscfg`

## <a name="configure-an-azure-cloud-service"></a>設定 Azure 雲端服務
您可以從 Visual Studio 中的 [方案總管] 設定 Azure 雲端服務，如下列步驟所示：

1. 在 Visual Studio 中建立或開啟 Azure 雲端服務專案。

1. 在 [方案總管] 中，以滑鼠右鍵按一下專案，然後從操作功能表中選取 [屬性]。
   
    ![方案總管專案操作功能表](./media/vs-azure-tools-configure-roles-for-cloud-service/solution-explorer-project-context-menu.png)

1. 在專案的屬性頁面中，選取 [開發] 索引標籤。 

    ![專案屬性頁面- 開發索引標籤](./media/vs-azure-tools-configure-roles-for-cloud-service/project-properties-development-tab.png)

1. 在 [服務組態]  清單中，選取您想要編輯的服務組態名稱。 (如果您想要變更此角色的所有服務組態，請選取 [所有組態]。)
   
    > [!IMPORTANT]
    > 如果您選擇特定服務組態，某些屬性會停用，因為它們只能針對所有組態設定。 若要編輯這些屬性，您必須選取 [所有組態]。
    > 
    > 
   
    ![Azure 雲端服務的服務組態清單](./media/vs-azure-tools-configure-roles-for-cloud-service/cloud-service-service-configuration-property.png)

## <a name="change-the-number-of-role-instances"></a>變更角色執行個體的數目
若要改善雲端服務的效能，您可以根據使用者數目或特定角色的預期負載，變更正在執行之角色的執行個體數目。 雲端服務在 Azure 中執行時，會為角色的每個執行個體建立個別的虛擬機器。 這會影響這個雲端服務的部署計費。 如需計費的詳細資訊，請參閱[了解 Microsoft Azure 帳單](billing/billing-understand-your-bill.md)。

1. 在 Visual Studio 中建立或開啟 Azure 雲端服務專案。

1. 在 [方案總管] 中，展開專案節點。 在 [角色] 節點之下，以滑鼠右鍵按一下您要更新的節點，然後從操作功能表中選取 [屬性]。

    ![方案總管 Azure 角色操作功能表](./media/vs-azure-tools-configure-roles-for-cloud-service/solution-explorer-azure-role-context-menu.png)

1. 選取 [組態] 索引標籤。

    ![組態索引標籤](./media/vs-azure-tools-configure-roles-for-cloud-service/role-configuration-properties-page.png)

1. 在 [服務組態] 清單中，選取您要更新的服務組態。
   
    ![服務組態清單](./media/vs-azure-tools-configure-roles-for-cloud-service/role-configuration-properties-page-select-configuration.png)

1. 在 [執行個體計數]  文字方塊中，輸入您要為此角色啟動的執行個體數目。 當您將雲端服務發佈至 Azure 時，每個執行個體會在個別的虛擬機器上執行。

    ![更新執行個體計數](./media/vs-azure-tools-configure-roles-for-cloud-service/role-configuration-properties-page-instance-count.png)

1. 從 Visual Studio 工具列，選取 [儲存]。

## <a name="manage-connection-strings-for-storage-accounts"></a>管理儲存體帳戶的連接字串
您可以新增、移除或修改服務組態的連接字串。 例如，針對具有 `UseDevelopmentStorage=true`值的本機服務組態，您可能想要本機連接字串。 您也能也會想要設定在 Azure 中使用儲存體帳戶的雲端服務組態。

> [!WARNING]
> 當您輸入儲存體帳戶連接字串的 Azure 儲存體帳戶金鑰資訊時，這項資訊會儲存在本機服務組態檔中。 不過，這項資訊目前不會儲存為加密文字。
> 
> 

針對每個服務組態使用不同的值，您就不必在您的雲端服務中使用不同的連接字串，或在您將雲端服務發佈至 Azure 時修改程式碼。 您可以在程式碼中將相同名稱用於連接字串，根據您在建置雲端服務或發佈該服務時選取的服務組態，此值會有所不同。

1. 在 Visual Studio 中建立或開啟 Azure 雲端服務專案。

1. 在 [方案總管] 中，展開專案節點。 在 [角色] 節點之下，以滑鼠右鍵按一下您要更新的節點，然後從操作功能表中選取 [屬性]。

    ![方案總管 Azure 角色操作功能表](./media/vs-azure-tools-configure-roles-for-cloud-service/solution-explorer-azure-role-context-menu.png)

1. 選取 [設定] 索引標籤。

    ![[設定] 索引標籤](./media/vs-azure-tools-configure-roles-for-cloud-service/project-properties-settings-tab.png)

1. 在 [服務組態] 清單中，選取您要更新的服務組態。

    ![服務組態](./media/vs-azure-tools-configure-roles-for-cloud-service/project-properties-settings-tab-select-configuration.png)

1. 若要新增連接字串，請選取 [新增設定]。

    ![新增連接字串](./media/vs-azure-tools-configure-roles-for-cloud-service/project-properties-settings-tab-add-setting.png)

1. 將新的設定新增至清單後，以必要資訊更新清單中的資料列。

    ![New connection string](./media/vs-azure-tools-configure-roles-for-cloud-service/project-properties-settings-tab-add-setting-new-setting.png)

    - **名稱** - 輸入您要用於連接字串的名稱。
    - **類型** - 從下拉式清單中選取 [連接字串]。
    - **值** - 您可以直接將連接字串輸入到 值 儲存格中，或選取省略符號 (...) 以在 建立儲存體連接字串 對話方塊中作業。  

1. 在 [建立儲存體連接字串] 對話方塊中，針對 [連接方式] 選取一個選項。 針對您選取的選項依照指示操作：

    - **Microsoft Azure 儲存體模擬器** - 如果您選取此選項，對話方塊上的其餘設定會停用，因為它們只會套至 Azure。 選取 [確定] 。
    - **您的訂用帳戶** - 如果您選取此選項，請使用下拉式清單來選取並登入 Microsoft 帳戶，或新增 Microsoft 帳戶。 選取 Azure 訂用帳戶和儲存體帳戶。 選取 [確定] 。
    - **手動輸入的認證** - 輸入儲存體帳戶名稱，以及主要或次要金鑰。 選取 **[連線]** 的 (在大部分情況下建議適用 HTTPS)。選取 [確定] 。

1. 若要刪除連接字串，請選取連接字串，然後選取 [移除設定]。

1. 從 Visual Studio 工具列，選取 [儲存]。

## <a name="programmatically-access-a-connection-string"></a>以程式設計方式存取連接字串

下列步驟顯示如何使用 C# 以程式設計方式存取連接字串。

1. 將下列 using 指示詞新增至您要使用此設定的 C# 檔案︰

    ```csharp
    using Microsoft.WindowsAzure;
    using Microsoft.WindowsAzure.Storage;
    using Microsoft.WindowsAzure.ServiceRuntime;
    ```

1. 下列程式碼說明如何存取連接字串的範例。 使用適當的值來取代 &lt;ConnectionStringName> 預留位置。 

    ```csharp
    // Setup the connection to Azure Storage
    var storageAccount = CloudStorageAccount.Parse(RoleEnvironment.GetConfigurationSettingValue("<ConnectionStringName>"));
    ```

## <a name="add-custom-settings-to-use-in-your-azure-cloud-service"></a>新增要在 Azure 雲端服務中使用的自訂設定
服務組態檔中的自訂設定可讓您為特定服務組態的字串新增名稱和值。 您可以藉由讀取設定的值並使用該值控制程式碼中的邏輯，即可選擇使用此設定在您的雲端服務中設定功能。 您可以變更這些服務組態值，而不需要重建服務封裝或在雲端服務執行時。 您的程式碼可以檢查設定變更時的通知。 請參閱 [RoleEnvironment.Changing 事件](https://msdn.microsoft.com/library/azure/microsoft.windowsazure.serviceruntime.roleenvironment.changing.aspx)。

您可以新增、移除或修改服務組態的自訂設定。 針對不同的服務組態，您可能會想要這些連接字串的不同值。

針對每個服務組態使用不同的值，您就不必在您的雲端服務中使用不同的字串，或在您將雲端服務發佈至 Azure 時修改程式碼。 您可以在程式碼中將相同名稱用於字串，根據您在建置雲端服務或發佈該服務時選取的服務組態，此值會有所不同。

1. 在 Visual Studio 中建立或開啟 Azure 雲端服務專案。

1. 在 [方案總管] 中，展開專案節點。 在 [角色] 節點之下，以滑鼠右鍵按一下您要更新的節點，然後從操作功能表中選取 [屬性]。

    ![方案總管 Azure 角色操作功能表](./media/vs-azure-tools-configure-roles-for-cloud-service/solution-explorer-azure-role-context-menu.png)

1. 選取 [設定] 索引標籤。

    ![[設定] 索引標籤](./media/vs-azure-tools-configure-roles-for-cloud-service/project-properties-settings-tab.png)

1. 在 [服務組態] 清單中，選取您要更新的服務組態。

    ![服務組態清單](./media/vs-azure-tools-configure-roles-for-cloud-service/project-properties-settings-tab-select-configuration.png)

1. 若要新增自訂設定，請選取 [新增設定]。

    ![新增自訂設定](./media/vs-azure-tools-configure-roles-for-cloud-service/project-properties-settings-tab-add-setting.png)

1. 將新的設定新增至清單後，以必要資訊更新清單中的資料列。

    ![新增自訂設定](./media/vs-azure-tools-configure-roles-for-cloud-service/project-properties-settings-tab-add-setting-new-setting.png)

    - **名稱** - 輸入設定的名稱。
    - **類型** - 從下拉式清單中選取 [字串]。
    - **值** - 輸入設定的值。 您可以直接將值輸入到 [值] 儲存格中，或選取省略符號 (...) 以在 [編輯字串] 對話方塊中輸入值。  

1. 若要刪除自訂設定，請選取設定，然後選取 [移除設定]。

1. 從 Visual Studio 工具列，選取 [儲存]。

## <a name="programmatically-access-a-custom-settings-value"></a>以程式設計方式存取自訂設定的值
 
下列步驟顯示如何使用 C# 以程式設計方式存取自訂設定。

1. 將下列 using 指示詞新增至您要使用此設定的 C# 檔案︰

    ```csharp
    using Microsoft.WindowsAzure;
    using Microsoft.WindowsAzure.Storage;
    using Microsoft.WindowsAzure.ServiceRuntime;
    ```

1. 下列程式碼說明如何存取自訂設定的範例。 使用適當的值來取代 &lt;SettingName> 預留位置。 
    
    ```csharp
    var settingValue = RoleEnvironment.GetConfigurationSettingValue("<SettingName>");
    ```

## <a name="manage-local-storage-for-each-role-instance"></a>管理每個角色執行個體的本機儲存體
您可以為角色的每個執行個體新增本機檔案系統儲存體。 為其儲存資料之角色的其他執行個體，或其他角色無法存取儲存在該儲存體中的資料。  

1. 在 Visual Studio 中建立或開啟 Azure 雲端服務專案。

1. 在 [方案總管] 中，展開專案節點。 在 [角色] 節點之下，以滑鼠右鍵按一下您要更新的節點，然後從操作功能表中選取 [屬性]。

    ![方案總管 Azure 角色操作功能表](./media/vs-azure-tools-configure-roles-for-cloud-service/solution-explorer-azure-role-context-menu.png)

1. 選取 [本機儲存體] 索引標籤。

    ![本機儲存體索引標籤](./media/vs-azure-tools-configure-roles-for-cloud-service/role-local-storage-tab.png)

1. 在 [服務組態] 清單中，確保已選取 [所有組態]，因為本機儲存體設定會套用至所有服務組態。 任何其他值都會導致頁面上的所有輸入欄位遭到停用。 

    ![服務組態清單](./media/vs-azure-tools-configure-roles-for-cloud-service/role-local-storage-tab-service-configuration.png)

1. 若要新增本機儲存體項目，請選取 [新增本機儲存體]。

    ![新增本機儲存體](./media/vs-azure-tools-configure-roles-for-cloud-service/role-local-storage-tab-add-local-storage.png)

1. 將新的本機儲存體項目新增至清單後，以必要資訊更新清單中的資料列。

    ![新增本機儲存體項目](./media/vs-azure-tools-configure-roles-for-cloud-service/role-local-storage-tab-new-local-storage.png)

    - **名稱** - 輸入您要用於新本機儲存體的名稱。
    - **大小 (MB)** - 輸入您要用於新本機儲存體的大小 (MB)。
    - **於角色回收時清除** - 選取此選項，以在回收角色的虛擬機器時，移除新本機儲存體中的資料。

1. 若要刪除本機儲存體項目，請選取此項目，然後選取 [移除本機儲存體]。

1. 從 Visual Studio 工具列，選取 [儲存]。

## <a name="programmatically-accessing-local-storage"></a>以程式設計方式存取本機儲存體

本節說明如何藉由撰寫測試文字檔案 `MyLocalStorageTest.txt`，使用 C# 以程式設計方式存取本機儲存體。  

### <a name="write-a-text-file-to-local-storage"></a>將文字檔案寫入本機儲存體

下列程式碼顯示如何將文字檔案寫入本機儲存體的範例。 使用適當的值來取代 &lt;LocalStorageName> 預留位置。 

    ```csharp
    // Retrieve an object that points to the local storage resource
    LocalResource localResource = RoleEnvironment.GetLocalResource("<LocalStorageName>");
    
    //Define the file name and path
    string[] paths = { localResource.RootPath, "MyLocalStorageTest.txt" };
    String filePath = Path.Combine(paths);
    
    using (FileStream writeStream = File.Create(filePath))
    {
        Byte[] textToWrite = new UTF8Encoding(true).GetBytes("Testing Web role storage");
        writeStream.Write(textToWrite, 0, textToWrite.Length);
    }

    ```

### <a name="find-a-file-written-to-local-storage"></a>尋找已寫入本機儲存體的檔案

若要檢視上一節中的程式碼所建立的檔案，請遵循下列步驟︰
    
1.  在 Windows 通知區域中，以滑鼠右鍵按一下 Azure 圖示，然後從操作功能表中選取 [顯示計算模擬器 UI]。 

    ![顯示 Azure 計算模擬器](./media/vs-azure-tools-configure-roles-for-cloud-service/show-compute-emulator.png)

1. 選取 Web 角色。

    ![Azure 計算模擬器](./media/vs-azure-tools-configure-roles-for-cloud-service/compute-emulator.png)

1. 在 [Microsoft Azure 計算模擬器] 功能表上，選取 [工具] > [開啟本機存放區]。

    ![開啟本機存放區功能表項目](./media/vs-azure-tools-configure-roles-for-cloud-service/compute-emulator-open-local-store-menu.png)

1. 當 [Windows 檔案總管] 視窗開啟時，請在 [搜尋] 文字方塊中輸入 `MyLocalStorageTest.txt`，然後選取 **Enter** 開始搜尋。 

## <a name="next-steps"></a>後續步驟
藉由參閱 [設定 Azure 專案](vs-azure-tools-configuring-an-azure-project.md)深入了解 Visual Studio 中的 Azure 專案。 藉由參閱 [結構描述參考](https://msdn.microsoft.com/library/azure/dd179398)深入了解雲端服務結構描述。

