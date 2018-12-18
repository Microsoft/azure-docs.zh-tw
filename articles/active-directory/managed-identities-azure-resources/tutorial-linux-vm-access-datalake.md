---
title: 使用 Linux VM 系統指派的受控識別來存取 Azure Data Lake Store
description: 本教學課程說明如何使用 Linux VM 系統指派的受控識別，來存取 Azure Data Lake Store。
services: active-directory
documentationcenter: ''
author: daveba
manager: mtillman
editor: ''
ms.service: active-directory
ms.component: msi
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 11/20/2017
ms.author: daveba
ms.openlocfilehash: 2a3a46b5f2adf052f04530a8d587a577b14b709b
ms.sourcegitcommit: cc4fdd6f0f12b44c244abc7f6bc4b181a2d05302
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47106086"
---
# <a name="tutorial-use-a-linux-vm-system-assigned-managed-identity-to-access-azure-data-lake-store"></a>教學課程：使用 Linux VM 系統指派的受控識別來存取 Azure Data Lake Store

[!INCLUDE [preview-notice](../../../includes/active-directory-msi-preview-notice.md)]

本教學課程說明如何將系統指派的受控識別用於 Linux 虛擬機器 (VM)，以存取 Cosmos DB。 您會了解如何： 

在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 將您的 VM 存取權授與 Azure Data Lake Store。
> * 使用 Linux VM 系統指派的受控識別來取得存取權杖，以存取 Azure Data Lake Store。

## <a name="prerequisites"></a>必要條件

[!INCLUDE [msi-qs-configure-prereqs](../../../includes/active-directory-msi-qs-configure-prereqs.md)]

[!INCLUDE [msi-tut-prereqs](../../../includes/active-directory-msi-tut-prereqs.md)]

- [登入 Azure 入口網站](https://portal.azure.com)

- [建立 Linux 虛擬機器](/azure/virtual-machines/linux/quick-create-portal)

- [在虛擬機器上啟用系統指派的身分識別](/azure/active-directory/managed-service-identity/qs-configure-portal-windows-vm#enable-system-assigned-identity-on-an-existing-vm)

## <a name="grant-your-vm-access-to-azure-data-lake-store"></a>將您的 VM 存取權授與 Azure Data Lake Store

現在您可以將您的 VM 存取權授與 Azure Data Lake Store 中的檔案和資料夾。 在這個步驟，您可以使用現有的 Data Lake Store 執行個體或建立新的執行個體。 若要使用 Azure 入口網站建立 Data Lake Store 執行個體，請遵循 [Azure Data Lake Store 快速入門](https://docs.microsoft.com/azure/data-lake-store/data-lake-store-get-started-portal)。 在 [Azure Data Lake Store 文件](https://docs.microsoft.com/azure/data-lake-store/data-lake-store-overview)中，也有使用 Azure CLI 和 Azure PowerShell 的快速入門做法。

在 Data Lake Store 中建立新資料夾，並將在該資料夾中讀取、寫入和執行檔案的權限，授予 VM 系統指派的受控識別：

1. 在 Azure 入口網站中，選取左側窗格的 [Data Lake Store]。
2. 選取您需要使用的 Data Lake Store 執行個體。
3. 選取命令列上的 [資料總管]。
4. 隨即選取 Data Lake Store 執行個體的根資料夾。 在命令列上選取 [存取]。
5. 選取 [新增] 。  在 [選取] 方塊中，輸入 VM 的名稱，例如 **DevTestVM**。 從搜尋結果中選取您的 VM，然後按一下 [選取]。
6. 按一下 [選取權限]。  選取 [讀取] 和 [執行]，加入到 [此資料夾]，然後新增為 [僅存取權限]。 選取 [確定]。  權限應該已成功加入。
7. 關閉 [存取] 窗格。
8. 針對此教學課程，建立一個新資料夾。 選取命令列中的 [新資料夾]，為新資料夾命名，例如 **TestFolder**。  選取 [確定]。
9. 選取您所建立的資料夾，然後選取命令列中的 [存取]。
10. 如同步驟 5，選取 [新增]。 在 [選取] 方塊中，輸入 VM 的名稱。 從搜尋結果中選取您的 VM，然後按一下 [選取]。
11. 如同步驟 6，選取 [選取權限]。 選取 [讀取]、[寫入] 和 [執行]，新增到 [此資料夾]，並新增為 [存取權限項目及預設權限項目]。 選取 [確定]。  權限應該已成功加入。

現在，適用於 Azure 資源的受控識別，可以對您所建立資料夾中的檔案執行所有作業。 如需管理 Data Lake Store 存取權的詳細資訊，請參閱 [Data Lake Store 中的存取控制](https://docs.microsoft.com/azure/data-lake-store/data-lake-store-access-control)一文。

## <a name="get-an-access-token-and-call-the-data-lake-store-file-system"></a>取得存取權杖，並呼叫 Data Lake Store 檔案系統

Azure Data Lake Store 原生支援 Azure AD 驗證，因此可直接接受透過適用於 Azure 資源的受控識別所取得的存取權杖。 為了向 Data Lake Store 檔案系統進行驗證，您要將 Azure AD 所簽發的存取權杖傳送至 Data Lake Store 檔案系統端點。 存取權杖位於授權標頭，格式為 "Bearer \<ACCESS_TOKEN_VALUE\>"。  若要深入了解 Data Lake Store 的 Azure AD 驗證支援，請參閱[使用 Azure Active Directory 向 Data Lake Store 進行驗證](https://docs.microsoft.com/azure/data-lake-store/data-lakes-store-authentication-using-azure-active-directory)。

在本教學課程中，您使用 cURL 向 Data Lake Store 檔案系統驗證 REST API，以提出 REST 要求。

> [!NOTE]
> 適用於 Data Lake Store 檔案系統的用戶端 SDK，尚不支援適用於 Azure 資源的受控識別。

若要完成這些步驟，您需要 SSH 用戶端。 如果您使用 Windows，您可以在[適用於 Linux 的 Windows 子系統](https://msdn.microsoft.com/commandline/wsl/about)中使用 SSH 用戶端。 如果您需要設定 SSH 用戶端金鑰的協助，請參閱[如何在 Azure 上搭配 Windows 使用 SSH 金鑰](../../virtual-machines/linux/ssh-from-windows.md)，或[如何在 Azure 中建立和使用 Linux VM 的 SSH 公開和私密金鑰組](../../virtual-machines/linux/mac-create-ssh-keys.md)。

1. 在入口網站中，瀏覽至您的 Linux VM。 在 [概觀] 中，選取 [連線]。  
2. 使用您所選擇的 SSH 用戶端來連線到 VM。 
3. 在終端機視窗中，使用 CURL 向 Azure 資源端點的本機受控識別提出要求，以取得 Data Lake Store 的存取權杖。 Data Lake Store 的資源識別碼是 "https://datalake.azure.net/"。  請務必包含資源識別項中的結尾斜線。
    
   ```bash
   curl 'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https%3A%2F%2Fdatalake.azure.net%2F' -H Metadata:true   
   ```
    
   成功的回應會傳回您向 Data Lake Store 驗證時所使用的存取權杖：

   ```bash
   {"access_token":"eyJ0eXAiOiJ...",
    "refresh_token":"",
    "expires_in":"3599",
    "expires_on":"1508119757",
    "not_before":"1508115857",
    "resource":"https://datalake.azure.net/",
    "token_type":"Bearer"}
   ```

4. 使用 cURL 對您 Data Lake Store 檔案系統的 REST 端點提出要求，以列出根資料夾中的資料夾。 這是檢查所有項目皆正確設定的簡單方式。 複製上一個步驟中的存取權杖值。 授權標頭中的字串 "Bearer" 必須是大寫 "B"。 您可以在 Azure 入口網站 [Data Lake Store] 窗格的 [概觀] 區段中，找到您 Data Lake Store 執行個體的名稱。

   ```bash
   curl https://<YOUR_ADLS_NAME>.azuredatalakestore.net/webhdfs/v1/?op=LISTSTATUS -H "Authorization: Bearer <ACCESS_TOKEN>"
   ```
    
   成功的回應看起來會像這樣：

   ```bash
   {"FileStatuses":{"FileStatus":[{"length":0,"pathSuffix":"TestFolder","type":"DIRECTORY","blockSize":0,"accessTime":1507934941392,"modificationTime":1508105430590,"replication":0,"permission":"770","owner":"bd0e76d8-ad45-4fe1-8941-04a7bf27f071","group":"bd0e76d8-ad45-4fe1-8941-04a7bf27f071"}]}}
   ```

5. 現在您可以嘗試將檔案上傳至您的 Data Lake Store 執行個體。 首先，建立要上傳的檔案。

   ```bash
   echo "Test file." > Test1.txt
   ```

6. 使用 cURL 對您的 Data Lake Store 檔案系統 REST 端點提出要求，以將檔案上傳到您先前建立的資料夾。 上傳需要重新導向，cURL 會自動遵循重新導向。 

   ```bash
   curl -i -X PUT -L -T Test1.txt -H "Authorization: Bearer <ACCESS_TOKEN>" 'https://<YOUR_ADLS_NAME>.azuredatalakestore.net/webhdfs/v1/<FOLDER_NAME>/Test1.txt?op=CREATE' 
   ```

    成功的回應看起來會像這樣：

   ```bash
   HTTP/1.1 100 Continue
   HTTP/1.1 307 Temporary Redirect
   Cache-Control: no-cache, no-cache, no-store, max-age=0
   Pragma: no-cache
   Expires: -1
   Location: https://mytestadls.azuredatalakestore.net/webhdfs/v1/TestFolder/Test1.txt?op=CREATE&write=true
   x-ms-request-id: 756f6b24-0cca-47ef-aa12-52c3b45b954c
   ContentLength: 0
   x-ms-webhdfs-version: 17.04.22.00
   Status: 0x0
   X-Content-Type-Options: nosniff
   Strict-Transport-Security: max-age=15724800; includeSubDomains
   Date: Sun, 15 Oct 2017 22:10:30 GMT
   Content-Length: 0
       
   HTTP/1.1 100 Continue
       
   HTTP/1.1 201 Created
   Cache-Control: no-cache, no-cache, no-store, max-age=0
   Pragma: no-cache
   Expires: -1
   Location: https://mytestadls.azuredatalakestore.net/webhdfs/v1/TestFolder/Test1.txt?op=CREATE&write=true
   x-ms-request-id: af5baa07-3c79-43af-a01a-71d63d53e6c4
   ContentLength: 0
   x-ms-webhdfs-version: 17.04.22.00
   Status: 0x0
   X-Content-Type-Options: nosniff
   Strict-Transport-Security: max-age=15724800; includeSubDomains
   Date: Sun, 15 Oct 2017 22:10:30 GMT
   Content-Length: 0
   ```

您可以使用 Data Lake Store 檔案系統的其他 API，來附加至檔案及下載檔案等。

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何使用 Linux VM 系統指派的受控識別，來存取 Azure Data Lake Store。 若要深入了解 Azure Data Lake Store，請參閱：

> [!div class="nextstepaction"]
>[Azure Data Lake Store](/azure/data-lake-store/data-lake-store-overview)
