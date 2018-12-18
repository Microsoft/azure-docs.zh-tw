---
title: 使用受控識別來向 Azure Data Lake Storage Gen1 輸出驗證 Azure 串流分析作業 (預覽)
description: ''
services: stream-analytics
author: mamccrea
ms.author: mamccrea
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 09/27/2018
ms.openlocfilehash: 41b3dcc03f7cfbfee11798738a3b2daaf5e96741
ms.sourcegitcommit: 6361a3d20ac1b902d22119b640909c3a002185b3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49365283"
---
# <a name="use-managed-identities-to-authenticate-azure-stream-analytics-jobs-to-azure-data-lake-storage-gen1-output-preview"></a>使用受控識別來向 Azure Data Lake Storage Gen1 輸出驗證 Azure 串流分析作業 (預覽)

Azure 串流分析支援向 Azure Data Lake Storage (ADLS) Gen1 輸出進行受控識別驗證。 身分識別是在 Azure Active Directory 中註冊的受控應用程式，代表了指定的串流分析作業，並可用來向指定的資源驗證。 受控識別消除了以使用者為基礎的驗證方法限制，比如因為密碼改變或使用者權杖每 90 天到期一次而需要重新驗證。 此外，受控識別有助於進行輸出至 Azure Data Lake Storage Gen1 的串流分析作業部署自動化。

請瀏覽 [Eight new features in Azure Stream Analytics](https://azure.microsoft.com/blog/eight-new-features-in-azure-stream-analytics/) (Azure 串流分析的八個新功能) 部落格文章，註冊這項預覽功能，並閱讀更多關於新功能的內容。

本文說明兩種方式，讓您用來為輸出至 Azure Data Lake Storage Gen1 的 Azure 串流分析作業啟用受控識別：透過 Azure 入口網站及透過 Azure Resource Manager 範本部署。

## <a name="enable-managed-identity-with-azure-portal"></a>從 Azure 入口網站啟用受控識別

1. 一開始先建立新的串流分析作業，或在 Azure 入口網站中開啟現有作業。 從畫面左側的功能表列選取位在 [設定] 下的 [受控識別 (預覽)]。

   ![設定串流分析受控識別預覽](./media/stream-analytics-managed-identities-adls/stream-analytics-managed-identity-preview.png)

2. 選取右側出現的視窗 [使用系統指派的受控識別 (預覽)]。 按一下 [儲存] 在 Azure Active Directory 中建立串流分析作業的識別服務主體。 Azure 會負責管理新建立的身分識別生命週期。 當串流分析作業刪除時，Azure 會自動刪除已與其建立關聯的身分識別 (亦即服務主體)。

   當組態儲存時，服務主體的物件識別碼 (OID) 會列為主體識別碼，如下所示：

   ![串流分析主體識別碼](./media/stream-analytics-managed-identities-adls/stream-analytics-principal-id.png)
 
   服務主體與串流分析作業的名稱相同。 例如，如果您作業的名稱是 **MyASAJob**，建立的服務主體名稱也會是 **MyASAJob**。

3. 在 ADLS Gen1 輸出接收的輸出屬性視窗中，按一下 [驗證模式] 下拉式清單，然後選擇 [受控識別 (預覽)]。

4. 填寫其餘屬性。 若要深入了解如何建立 ADLS 輸出，請參閱[使用串流分析建立 Data Lake Store 輸出](../data-lake-store/data-lake-store-stream-analytics.md)。 當您完成後，請按一下 [儲存]。

   ![設定 Azure Data Lake Storage](./media/stream-analytics-managed-identities-adls/stream-analytics-configure-adls.png)
 
5. 瀏覽到 ADLS Gen1 的 [概觀] 頁面，然後按一下 [資料總管]。

   ![設定 Data Lake Storage 概觀](./media/stream-analytics-managed-identities-adls/stream-analytics-adls-overview.png)

6. 在 [資料總管] 窗格中選取 [存取]，然後在 [存取] 窗格中按一下 [新增]。

   ![設定 Data Lake Store 存取](./media/stream-analytics-managed-identities-adls/stream-analytics-adls-access.png)

7. 在 [選取使用者或群組] 窗格的文字方塊中，鍵入服務主體的名稱。 請記住，服務主體的名稱也是對應串流分析作業的名稱。 當您開始鍵入主體名稱時，名稱就會顯示在文字方塊的下方。 選擇您想要的服務主體名稱，然後按一下 [選取]。

   ![選取服務主體名稱](./media/stream-analytics-managed-identities-adls/stream-analytics-service-principal-name.png)
 
8. 在 [權限] 窗格中，選取 [寫入] 和 [執行] 權限，並將其指派至 [此資料夾及所有子系]。 然後按一下 [確定]。

   ![選取權限](./media/stream-analytics-managed-identities-adls/stream-analytics-select-permissions.png)
 
9. 服務主體會列在 [存取] 窗格 [指派的權限] 底下，如下所示。 您現在可以返回並啟動您的串流分析作業。

   ![存取清單](./media/stream-analytics-managed-identities-adls/stream-analytics-access-list.png)

   若要深入了解 Data Lake Storage Gen1 檔案系統權限，請參閱 [Azure Data Lake Storage Gen1 的存取控制](../data-lake-store/data-lake-store-access-control.md)。

## <a name="resource-manager-template-deployment"></a>Resource Manager 範本部署

1. 您現在可以建立具有受控識別的 *Microsoft.StreamAnalytics/streamingjobs* 資源，方法是在 Resource Manager 範本的資源區段中加入下列屬性：

   ```json
   "Identity": {
   "Type": "SystemAssigned",
   },
   ```

   這個屬性會要求 Azure Resource Manager 建立及管理您的 Azure 串流分析作業身分識別。

   **範例作業**

   ```json
   { 
   "Name": "AsaJobWithIdentity", 
   "Type": "Microsoft.StreamAnalytics/streamingjobs", 
   "Location": "West US",
   "Identity": {
     "Type": "SystemAssigned", 
     }, 
   "properties": {
      "sku": {
       "name": "standard"
       },
   "outputs": [
         {
           "name": "string",
           "properties":{
             "datasource": {        
               "type": "Microsoft.DataLake/Accounts",
               "properties": {
                 "accountName": “myDataLakeAccountName",
                 "filePathPrefix": “cluster1/logs/{date}/{time}",
                 "dateFormat": "YYYY/MM/DD",
                 "timeFormat": "HH",
                 "authenticationMode": "Msi"
                 }
                 
   }
   ```
  
   **範例作業回應**

   ```json
   { 
   "Name": "mySAJob", 
   "Type": "Microsoft.StreamAnalytics/streamingjobs", 
   "Location": "West US",
   "Identity": {
   "Type": "SystemAssigned",
    "principalId": "GUID", 
    "tenantId": "GUID", 
   }, 
   "properties": {
           "sku": {
             "name": "standard"
           },
   }
   ```

   請記下作業回應中的主體識別碼，以授予必要 ADLS 資源的存取權。

   **租用戶識別碼**是 Azure Active Directory 租用戶的識別碼，而該租用戶是建立服務主體的位置。 服務主體會建立在訂用帳戶信任的 Azure 租用戶中。

   **類型**表示受控識別的類型，如受控識別類型中所述。 僅支援系統指派的類型。

2. 使用 PowerShell 提供服務主體的存取。 若要經由 PowerShell 提供服務主體的存取權，請執行下列命令：

   ```powershell
   Set-AzureRmDataLakeStoreItemAclEntry -AccountName <accountName> -Path <Path> -AceType User -Id <PrinicpalId> -Permissions <Permissions>
   ```

   **PrincipalId** 是服務主體的物件識別碼，並會在服務主體建立時，列於入口網站畫面上。 如果您使用了 Resource Manager 範本部署來建立作業，物件識別碼會列在作業回應的識別屬性中。

   **範例**

   ```powershell
   PS > Set-AzureRmDataLakeStoreItemAclEntry -AccountName "adlsmsidemo" -Path / -AceType
   User -Id 14c6fd67-d9f5-4680-a394-cd7df1f9bacf -Permissions WriteExecute
   ```

   若要深入了解上述 PowerShell 命令，請參閱 [Set-AzureRmDataLakeStoreItemAclEntry](https://docs.microsoft.com/powershell/module/azurerm.datalakestore/set-azurermdatalakestoreitemaclentry?view=azurermps-6.8.1&viewFallbackFrom=azurermps-4.2.0#optional-parameters) 文件。

## <a name="next-steps"></a>後續步驟

* [使用串流分析建立 Data Lake Store 輸出](../data-lake-store/data-lake-store-stream-analytics.md)
