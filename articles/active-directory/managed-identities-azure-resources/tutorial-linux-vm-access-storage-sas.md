---
title: 使用 Linux 系統指派的受控識別，以利用 SAS 認證來存取 Azure 儲存體
description: 本教學課程說明如何使用 Linux VM 系統指派的受控識別，以利用 SAS 認證 (而非儲存體帳戶來存取金鑰) 來存取 Azure 儲存體。
services: active-directory
documentationcenter: ''
author: daveba
manager: mtillman
editor: daveba
ms.service: active-directory
ms.component: msi
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 11/20/2017
ms.author: daveba
ms.openlocfilehash: 0840f1abb05e97ca8f55cb0003e4f3cd2383e8c7
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47222518"
---
# <a name="tutorial-use-a-linux-vm-system-assigned-identity-to-access-azure-storage-via-a-sas-credential"></a>教學課程：使用 Linux VM 系統指派的受控識別，以透過 SAS 認證存取 Azure 儲存體

[!INCLUDE [preview-notice](../../../includes/active-directory-msi-preview-notice.md)]

本教學課程說明如何將系統指派的受控識別用於 Linux 虛擬機器 (VM)，以取得儲存體共用存取簽章 (SAS) 認證。 具體而言，即[服務 SAS 認證](/azure/storage/common/storage-dotnet-shared-access-signature-part-1?toc=%2fazure%2fstorage%2fblobs%2ftoc.json#types-of-shared-access-signatures)。 

服務 SAS 提供可針對有限的時間和特定的服務，授與儲存體帳戶中物件的有限存取權 (在此情況下，即 Blob 服務)，而不需要公開帳戶存取金鑰。 在執行儲存體作業時 (例如使用儲存體 SDK)，您可以如往常般使用 SAS 認證。 此教學課程中，我們會示範使用 Azure 儲存體 CLI 上傳和下載 blob。 您將了解如何：


> [!div class="checklist"]
> * 建立儲存體帳戶
> * 在儲存體帳戶中建立 Blob 容器
> * 在資源管理員中將 VM 存取權限授與儲存體帳戶 SAS 
> * 使用 VM 身分識別取得存取權杖，並將其用於從資源管理員取出 SAS 

## <a name="prerequisites"></a>必要條件

[!INCLUDE [msi-qs-configure-prereqs](../../../includes/active-directory-msi-qs-configure-prereqs.md)]

[!INCLUDE [msi-tut-prereqs](../../../includes/active-directory-msi-tut-prereqs.md)]

- [登入 Azure 入口網站](https://portal.azure.com)

- [建立 Linux 虛擬機器](/azure/virtual-machines/linux/quick-create-portal)

- [在虛擬機器上啟用系統指派的受控識別](/azure/active-directory/managed-service-identity/qs-configure-portal-windows-vm#enable-system-assigned-identity-on-an-existing-vm)

## <a name="create-a-storage-account"></a>建立儲存體帳戶 

如果您還沒有帳戶，您現在將建立一個儲存體帳戶。  您也可以略過此步驟，並將存取現有儲存體帳戶金鑰的權利，授予 VM 系統指派的受控識別。 

1. 按一下 Azure 入口網站左上角的 [+/建立新服務] 按鈕。
2. 按一下 [儲存體]，然後按一下 [儲存體帳戶]，就會顯示新的 [建立儲存體帳戶] 面板。
3. 輸入儲存體帳戶的 [名稱]，您稍後將會使用它。  
4. [部署模型] 和 [帳戶類型] 應該分別設定為「資源管理員」和「一般用途」。 
5. 確定 [訂用帳戶] 和 [資源群組] 符合您在上一個步驟中建立 VM 時指定的值。
6. 按一下頁面底部的 [新增] 。

    ![建立新的儲存體帳戶](./media/msi-tutorial-linux-vm-access-storage/msi-storage-create.png)

## <a name="create-a-blob-container-in-the-storage-account"></a>在儲存體帳戶中建立 Blob 容器

稍後我們將上傳和下載檔案到新的儲存體帳戶。 由於檔案需要 Blob 儲存體，我們需要建立 Blob 容器，用來儲存檔案。

1. 巡覽回到您新建立的儲存體帳戶。
2. 按一下左側面板 [Blob 服務] 下的 [容器] 連結。
3. 按一下頁面上方的 [+ 容器]，[新的容器] 面板隨即會滑出。
4. 指定容器的名稱，選取存取層級，然後按一下 [確定]。 稍後在教學課程中將會用到您指定的名稱。 

    ![建立儲存體容器](./media/msi-tutorial-linux-vm-access-storage/create-blob-container.png)

## <a name="grant-your-vms-system-assigned-managed-identity-access-to-use-a-storage-sas"></a>將存取儲存體帳戶 SAS 的權利，授予 VM 系統指派的受控識別 

Azure 儲存體原生並不支援 Azure AD 驗證。  不過，您可以使用 VM 系統指派的受控識別，從 Resource Manager 中擷取儲存體 SAS，然後使用該 SAS 來存取儲存體。  在此步驟中，您會將存取儲存體帳戶 SAS 的權利，授予 VM 系統指派的受控識別。   

1. 瀏覽回到您新建立的儲存體帳戶。   
2. 按一下左側面板中的 [存取控制 (IAM)] 連結。  
3. 按一下頁面頂端的 [+ 新增] 以新增 VM 的新角色指派
4. 在頁面右側中，將 [角色] 設定為 [儲存體帳戶參與者]。 
5. 在下一個下拉式清單中，將 [存取權指派對象為] 設定為資源 [虛擬機器]。  
6. 接下來，請確保 [訂用帳戶] 下拉式清單中已列出適當的訂用帳戶，然後將 [資源群組] 設定為 [所有資源群組]。  
7. 最後，在 [選取] 的下拉式清單中，選擇您的 Linux 虛擬機器，然後按一下 [儲存]。  

    ![替代映像文字](./media/msi-tutorial-linux-vm-access-storage/msi-storage-role-sas.png)

## <a name="get-an-access-token-using-the-vms-identity-and-use-it-to-call-azure-resource-manager"></a>使用 VM 的身分識別取得存取權杖，並使用它來呼叫 Azure Resource Manager

其餘課程要從稍早建立的 VM 繼續進行。

若要完成這些步驟，您需要 SSH 用戶端。 如果您使用 Windows，您可以在[適用於 Linux 的 Windows 子系統](https://msdn.microsoft.com/commandline/wsl/install_guide)中使用 SSH 用戶端。 如果您需要設定 SSH 用戶端金鑰的協助，請參閱[如何在 Azure 上搭配 Windows 使用 SSH 金鑰](../../virtual-machines/linux/ssh-from-windows.md)，或[如何在 Azure 中建立和使用 Linux VM 的 SSH 公開和私密金鑰組](../../virtual-machines/linux/mac-create-ssh-keys.md)。

1. 在 Azure 入口網站中，瀏覽至 [虛擬機器]，移至您的 Linux 虛擬機器，然後在 [概觀] 頁面中，按一下頂端的 [連線]。 複製字串以連線到您的 VM。 
2. 使用 SSH 用戶端連線到 VM。  
3. 接下來，系統會提示您輸入建立 Linux VM 時新增的 [密碼]。 您應該可以順利登入。  
4. 使用 CURL 取得 Azure Resource Manager 的存取權杖。  

    存取權杖的 CURL 要求和回應如下：
    
    ```bash
    curl 'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https%3A%2F%2Fmanagement.azure.com%2F' -H Metadata:true    
    ```
    
    > [!NOTE]
    > 在前面的要求中，"resource" 參數的值必須完全符合 Azure AD 的預期。 當使用 Azure Resource Manager 資源 ID 時，必須在 URI 中包含結尾的斜線。
    > 在下列回應中，為求簡單明瞭，已縮短 access_token 元素。
    
    ```bash
    {"access_token":"eyJ0eXAiOiJ...",
    "refresh_token":"",
    "expires_in":"3599",
    "expires_on":"1504130527",
    "not_before":"1504126627",
    "resource":"https://management.azure.com",
    "token_type":"Bearer"} 
     ```

## <a name="get-a-sas-credential-from-azure-resource-manager-to-make-storage-calls"></a>從 Azure Resource Manager 取得 SAS 認證以進行儲存體呼叫

現在請使用上一節中擷取的存取權杖，使用 CURL 對 Resource Manager 進行呼叫，以建立儲存體 SAS 認證。 一旦有了 SAS 認證，我們便可呼叫儲存體進行上傳/下載作業。

針對此要求，我們將使用下列 HTTP 要求參數來建立 SAS 認證：

```JSON
{
    "canonicalizedResource":"/blob/<STORAGE ACCOUNT NAME>/<CONTAINER NAME>",
    "signedResource":"c",              // The kind of resource accessible with the SAS, in this case a container (c).
    "signedPermission":"rcw",          // Permissions for this SAS, in this case (r)ead, (c)reate, and (w)rite.  Order is important.
    "signedProtocol":"https",          // Require the SAS be used on https protocol.
    "signedExpiry":"<EXPIRATION TIME>" // UTC expiration time for SAS in ISO 8601 format, for example 2017-09-22T00:06:00Z.
}
```

要求 SAS 認證的 POST 主體中包含了這些參數。 如需有關建立 SAS 認證參數的詳細資訊，請參閱[清單服務 SAS REST 參考](/rest/api/storagerp/storageaccounts/listservicesas)。

使用下列 CURL 要求取得 SAS 認證。 別忘了以您自己的值取代 `<SUBSCRIPTION ID>`、`<RESOURCE GROUP>`、`<STORAGE ACCOUNT NAME>`、`<CONTAINER NAME>`、`<EXPIRATION TIME>` 參數的值。 將 `<ACCESS TOKEN>` 值取代為您先前擷取的存取權杖。

```bash 
curl https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP>/providers/Microsoft.Storage/storageAccounts/<STORAGE ACCOUNT NAME>/listServiceSas/?api-version=2017-06-01 -X POST -d "{\"canonicalizedResource\":\"/blob/<STORAGE ACCOUNT NAME>/<CONTAINER NAME>\",\"signedResource\":\"c\",\"signedPermission\":\"rcw\",\"signedProtocol\":\"https\",\"signedExpiry\":\"<EXPIRATION TIME>\"}" -H "Authorization: Bearer <ACCESS TOKEN>"
```

> [!NOTE]
> 前述 URL 的文字區分大小寫，因此請確定您使用的資源群組大小寫有正確相應。 此外，務必知道這是 POST 要求而不是 GET 要求。

CURL 回應會傳回 SAS 認證：  

```bash 
{"serviceSasToken":"sv=2015-04-05&sr=c&spr=https&st=2017-09-22T00%3A10%3A00Z&se=2017-09-22T02%3A00%3A00Z&sp=rcw&sig=QcVwljccgWcNMbe9roAJbD8J5oEkYoq%2F0cUPlgriBn0%3D"} 
```

建立範例 Blob 檔案以上傳至您的 Blob 儲存體容器。 在 Linux VM 上，您可以使用下列命令來執行此操作。 

```bash
echo "This is a test file." > test.txt
```

然後使用 SAS 認證透過 `az storage` CLI 命令進行驗證，並將檔案上傳至 blob 容器。 此步驟中，您必須在 VM 上[安裝最新的 Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli) (如果您還沒有這麼做)。

```azurecli-interactive
 az storage blob upload --container-name 
                        --file 
                        --name
                        --account-name 
                        --sas-token
```

回應： 

```JSON
Finished[#############################################################]  100.0000%
{
  "etag": "\"0x8D4F9929765C139\"",
  "lastModified": "2017-09-21T03:58:56+00:00"
}
```

此外，您可以使用 Azure CLI 下載檔案，並透過 SAS 認證進行驗證。 

要求： 

```azurecli-interactive
az storage blob download --container-name
                         --file 
                         --name 
                         --account-name
                         --sas-token
```

回應： 

```JSON
{
  "content": null,
  "metadata": {},
  "name": "testblob",
  "properties": {
    "appendBlobCommittedBlockCount": null,
    "blobType": "BlockBlob",
    "contentLength": 16,
    "contentRange": "bytes 0-15/16",
    "contentSettings": {
      "cacheControl": null,
      "contentDisposition": null,
      "contentEncoding": null,
      "contentLanguage": null,
      "contentMd5": "Aryr///Rb+D8JQ8IytleDA==",
      "contentType": "text/plain"
    },
    "copy": {
      "completionTime": null,
      "id": null,
      "progress": null,
      "source": null,
      "status": null,
      "statusDescription": null
    },
    "etag": "\"0x8D4F9929765C139\"",
    "lastModified": "2017-09-21T03:58:56+00:00",
    "lease": {
      "duration": null,
      "state": "available",
      "status": "unlocked"
    },
    "pageBlobSequenceNumber": null,
    "serverEncrypted": false
  },
  "snapshot": null
}
```

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何使用 Linux VM 系統指派的受控識別，以利用 SAS 認證來存取 Azure 儲存體。  若要深入了解 Azure 儲存體 SAS，請參閱：

> [!div class="nextstepaction"]
>[使用共用存取簽章 (SAS)](/azure/storage/common/storage-dotnet-shared-access-signature-part-1)
