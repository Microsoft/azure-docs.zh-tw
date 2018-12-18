---
title: 使用 Key Vault 來儲存及存取 Azure Cosmos DB 金鑰 | Microsoft Docs
description: 使用 Azure Key Vault 來儲存及存取 Azure Cosmos DB 連接字串、金鑰和 URI。
services: cosmos-db
author: rafats
manager: kfile
ms.service: cosmos-db
ms.devlang: dotnet
ms.topic: conceptual
ms.date: 08/21/2018
ms.author: rafats
ms.openlocfilehash: b090c1593b49bec4f51fea8d498860e8af8b2f4b
ms.sourcegitcommit: ebd06cee3e78674ba9e6764ddc889fc5948060c4
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/07/2018
ms.locfileid: "44053079"
---
# <a name="use-key-vault-to-store-and-access-azure-cosmos-db-keys"></a>使用 Key Vault 來儲存及存取 Azure Cosmos DB 金鑰

對應用程式使用 Azure Cosmos DB 時，您可以使用應用程式設定檔內的端點 URI 和金鑰，來存取資料庫、集合和文件。  不過，直接將金鑰和 URL 放在應用程式的程式碼中並不安全，原因是所有使用者都能取得純文字格式的這些資料。 您會希望確定 URI 和金鑰都可供取得，但必須透過安全的機制來進行。 對此，Azure Key Vault 可協助您安全地儲存和管理應用程式祕密。

若要在 Key Vault 儲存和讀取 Azure Cosmos DB 存取金鑰，必須執行下列步驟：

* 建立金鑰保存庫  
* 在 Key Vault 新增 Azure Cosmos DB 存取金鑰  
* 建立 Azure Web 應用程式  
* 註冊應用程式，並對其授與 Key Vault 讀取權限  


## <a name="create-a-key-vault"></a>建立金鑰保存庫

1. 登入 [Azure 入口網站](https://portal.azure.com/)。  
2. 選取 [建立資源] > [安全性] > [Key Vault]。  
3. 在 [建立金鑰保存庫] 區段上提供下列資訊：  
   * **名稱：** 提供 Key Vault 的唯一名稱。  
   * **訂用帳戶：** 選擇要使用的訂用帳戶。  
   * 在 [資源群組] 下選擇 [新建]，然後輸入資源群組名稱。  
   * 在 [位置] 下拉式功能表中選擇位置。  
   * 將其他的選項保留預設值。  
4. 提供上述資訊之後，請選取 [建立]。  

## <a name="add-azure-cosmos-db-access-keys-to-the-key-vault"></a>在 Key Vault 新增 Azure Cosmos DB 存取金鑰。
1. 瀏覽至您在上一個步驟中建立的 Key Vault，然後開啟 [祕密] 索引標籤。  
2. 選取 [+產生/匯入]。 

   * 針對 [上傳選項] 選取 [手動]。
   * 為祕密提供 [名稱]
   * 在 [值] 欄位中提供 Cosmos DB 帳戶的連接字串。 然後選取 [建立]。

   ![建立祕密](./media/access-secrets-from-keyvault/create-a-secret.png)

4. 建立好祕密之後，請將其開啟，並複製具有下列格式的祕密識別碼。 您會在下一節用到這個識別碼。 

   `https://<Key_Vault_Name>.vault.azure.net/secrets/<Secret _Name>/<ID>`

## <a name="create-an-azure-web-application"></a>建立 Azure Web 應用程式

1. 建立 Azure Web 應用程式，或是從 [GitHub 存放庫](https://github.com/Azure/azure-cosmosdb-dotnet/tree/master/Demo/keyvaultdemo)下載應用程式。 這是一個簡單的 MVC 應用程式。  

2. 將所下載的應用程式解壓縮，並開啟 **HomeController.cs** 檔案。 更新下面這一行裡面的祕密識別碼：

   `var secret = await keyVaultClient.GetSecretAsync("<Your Key Vault’s secret identifier>")`

3. [儲存] 檔案，然後 [建置] 解決方案。  
4. 接下來，將應用程式部署至 Azure。 以滑鼠右鍵按一下專案，然後選擇 [發佈]。 建立新的 App Service 設定檔 (您可以將應用程式命名為 WebAppKeyVault1)，然後選取 [發佈]。   

5. 應用程式部署完成後。 從 Azure 入口網站瀏覽至所部署的 Web 應用程式，然後開啟此應用程式的 [受控服務識別]。  

   ![受控服務識別](./media/access-secrets-from-keyvault/turn-on-managed-service-identity.png)

如果您立即執行應用程式，則會看到下列錯誤，原因是您還未在 Key Vault 中對此應用程式提供任何權限。

![已部署但沒有存取權的應用程式](./media/access-secrets-from-keyvault/app-deployed-without-access.png)

## <a name="register-the-application--grant-permissions-to-read-the-key-vault"></a>註冊應用程式，並對其授與 Key Vault 讀取權限

在本節中，您會向 Azure Active Directory 註冊應用程式，並對應用程式提供 Key Vault 的讀取權限。 

1. 瀏覽至 Azure 入口網站，然後開啟您在上一節建立的 **Key Vault**。  

2. 開啟 [存取原則]，選取 [+新增] 以尋找您所部署的 Web 應用程式，選取權限，然後選取 [確定]。  

   ![新增存取原則](./media/access-secrets-from-keyvault/add-access-policy.png)

現在，如果您執行應用程式，便可從 Key Vault 讀取祕密。

![已部署且有存取權的應用程式](./media/access-secrets-from-keyvault/app-deployed-with-access.png)
 
同樣地，您可以新增使用者來存取 Key Vault。 您需要選取 [存取原則] 將自己新增至 Key Vault，然後授與要從 Visual studio 執行應用程式所需的所有權限。 此應用程式從桌面執行時，會擷取您的身分識別。

## <a name="next-steps"></a>後續步驟

* 若要設定 Azure Cosmos DB 的防火牆，請參閱[防火牆支援](firewall-support.md)一文。
* 若要設定虛擬網路服務端點，請參閱[使用 VNet 服務端點來保護存取權](vnet-service-endpoint.md)一文。
