---
title: 使用 Windows VM 系統指派的受控識別來存取 Azure 儲存體
description: 本教學課程會逐步引導您使用 Windows VM 系統指派的受控識別，以存取 Azure 儲存體。
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
ms.date: 04/12/2018
ms.author: daveba
ms.openlocfilehash: 46be9469e67a4f456be100823d475b8720262b1b
ms.sourcegitcommit: 2d961702f23e63ee63eddf52086e0c8573aec8dd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/07/2018
ms.locfileid: "44163170"
---
# <a name="tutorial-use-a-windows-vm-system-assigned-managed-identity-to-access-azure-storage"></a>教學課程：使用 Windows VM 系統指派的受控識別來存取 Azure 儲存體

[!INCLUDE [preview-notice](../../../includes/active-directory-msi-preview-notice.md)]

本教學課程說明如何將系統指派的受控識別用於 Windows 虛擬機器 (VM)，以存取 Azure 儲存體。 您會了解如何：

> [!div class="checklist"]
> * 在儲存體帳戶中建立 Blob 容器
> * 將 Windows VM 系統指派的受控識別存取權授與儲存體帳戶 
> * 取得存取權，並用來呼叫 Azure 儲存體 

> [!NOTE]
> Azure 儲存體的 Azure Active Directory 驗證處於公開預覽狀態。

## <a name="prerequisites"></a>必要條件

[!INCLUDE [msi-qs-configure-prereqs](../../../includes/active-directory-msi-qs-configure-prereqs.md)]

[!INCLUDE [msi-tut-prereqs](../../../includes/active-directory-msi-tut-prereqs.md)]

- [登入 Azure 入口網站](https://portal.azure.com)

- [建立 Windows 虛擬機器](/azure/virtual-machines/windows/quick-create-portal)

- [在虛擬機器上啟用系統指派的受控識別](/azure/active-directory/managed-service-identity/qs-configure-portal-windows-vm#enable-system-assigned-identity-on-an-existing-vm)

## <a name="create-a-storage-account"></a>建立儲存體帳戶 

在本節中，您會建立儲存體帳戶。 

1. 按一下 Azure 入口網站左上角的 [+ 建立資源] 按鈕。
2. 按一下 [儲存體]，然後按一下 [儲存體帳戶 - Blob、檔案、資料表、佇列]。
3. 在 [名稱] 下，輸入儲存體帳戶的名稱。  
4. [部署模型] 和 [帳戶類型] 應該設定為**資源管理員**和儲存體 (一般用途 v1)。 
5. 確定 [訂用帳戶] 和 [資源群組] 符合您在上一個步驟中建立 VM 時指定的值。
6. 按一下頁面底部的 [新增] 。

    ![建立新的儲存體帳戶](./media/msi-tutorial-linux-vm-access-storage/msi-storage-create.png)

## <a name="create-a-blob-container-and-upload-a-file-to-the-storage-account"></a>建立 Blob 容器，並將檔案上傳至儲存體帳戶

檔案需要 Blob 儲存體，因此您必須建立 Blob 容器，用來儲存檔案。 然後，您可以在新的儲存體帳戶中，將檔案上傳到 Blob 容器。

1. 巡覽回到您新建立的儲存體帳戶。
2. 在 [Blob 服務] 下，按一下 [容器]。
3. 按一下頁面頂端的 [+ 容器]。
4. 在 [新增容器] 下，輸入容器的名稱，然後在 [公用存取層級] 下保留預設值。

    ![建立儲存體容器](./media/msi-tutorial-linux-vm-access-storage/create-blob-container.png)

5. 使用您選擇的編輯器，在本機電腦上建立標題為 hello world.txt 的檔案。  開啟檔案並新增以下文字 (不含引號)："Hello world! :)" 然後加以儲存。 
6. 按一下容器名稱，然後按一下 [上傳]，即可將檔案上傳至新建立的容器
7. 在 [上傳 blob] 窗格的 [檔案] 下，按一下資料夾圖示，然後在本機電腦瀏覽至檔案 **hello_world.txt**、選取檔案，然後按一下 [上傳]。
    ![上傳文字檔](./media/msi-tutorial-linux-vm-access-storage/upload-text-file.png)

## <a name="grant-your-vm-access-to-an-azure-storage-container"></a>將您的虛擬機器存取權授與 Azure 儲存體容器 

您可以使用 VM 系統指派的受控識別，來擷取 Azure 儲存體 Blob 中的資料。   

1. 巡覽回到您新建立的儲存體帳戶。  
2. 按一下左側面板中的 [存取控制 (IAM)] 連結。  
3. 按一下頁面頂端的 [+ 新增] 以新增虛擬機器的新角色指派。
4. 在 [角色] 下，從下拉式清單中，選取 [儲存體 Blob 資料讀取器 (預覽)]。 
5. 在下一個下拉式清單的 [將存取權指派給] 下，選取 [虛擬機器]。  
6. 接下來，請確保 [訂用帳戶] 下拉式清單中已列出適當的訂用帳戶，然後將 [資源群組] 設定為 [所有資源群組]。  
7. 在 [選取] 下，選擇您的虛擬機器，然後按一下 [儲存]。 

    ![指派權限](./media/tutorial-linux-vm-access-storage/access-storage-perms.png)

## <a name="get-an-access-token-and-use-it-to-call-azure-storage"></a>取得存取權杖，並用來呼叫 Azure 儲存體 

Azure 儲存體原生支援 Azure AD 驗證，因此可直接接受使用受控識別取得的存取權杖。 這是 Azure AD 與 Azure 儲存體整合的一部分，與在連接字串上提供認證不同。

以下 .Net 程式碼範例，是使用存取權杖開啟與 Azure 儲存體的連線，然後再讀取您稍早所建立檔案的內容。 此程式碼必須在 VM 上執行，才能夠存取 VM 的受控識別端點。 需使用 .Net Framework 4.6 或更高版本，才可使用存取權杖方法。 據以取代 `<URI to blob file>` 的值。 您可以瀏覽至您建立並上傳至 Blob 儲存體的檔案，並複製 [概觀] 頁面 [屬性] 下的 [URL]，即可取得此值。

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.IO;
using System.Net;
using System.Web.Script.Serialization; 
using Microsoft.WindowsAzure.Storage.Auth;
using Microsoft.WindowsAzure.Storage.Blob;

namespace StorageOAuthToken
{
    class Program
    {
        static void Main(string[] args)
        {
            //get token
            string accessToken = GetMSIToken("https://storage.azure.com/");
           
            //create token credential
            TokenCredential tokenCredential = new TokenCredential(accessToken);

            //create storage credentials
            StorageCredentials storageCredentials = new StorageCredentials(tokenCredential);

            Uri blobAddress = new Uri("<URI to blob file>");

            //create block blob using storage credentials
            CloudBlockBlob blob = new CloudBlockBlob(blobAddress, storageCredentials);
        
            //retrieve blob contents
            Console.WriteLine(blob.DownloadText());
            Console.ReadLine();
        }

        static string GetMSIToken(string resourceID)
        {
            string accessToken = string.Empty;
            // Build request to acquire MSI token
            HttpWebRequest request = (HttpWebRequest)WebRequest.Create("http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=" + resourceID);
            request.Headers["Metadata"] = "true";
            request.Method = "GET";

            try
            {
                // Call /token endpoint
                HttpWebResponse response = (HttpWebResponse)request.GetResponse();

                // Pipe response Stream to a StreamReader, and extract access token
                StreamReader streamResponse = new StreamReader(response.GetResponseStream());
                string stringResponse = streamResponse.ReadToEnd();
                JavaScriptSerializer j = new JavaScriptSerializer();
                Dictionary<string, string> list = (Dictionary<string, string>)j.Deserialize(stringResponse, typeof(Dictionary<string, string>));
                accessToken = list["access_token"];
                return accessToken;
            }
            catch (Exception e)
            {
                string errorText = String.Format("{0} \n\n{1}", e.Message, e.InnerException != null ? e.InnerException.Message : "Acquire token failed");
                return accessToken;
            }
        }            
    }
}
```

回應會包含檔案的內容：

`Hello world! :)`

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何啟用 Windows VM 系統指派的身分識別來存取 Azure 儲存體。  若要深入了解 Azure 儲存體，請參閱：

> [!div class="nextstepaction"]
> [Azure 儲存體](/azure/storage/common/storage-introduction)



