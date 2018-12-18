---
title: 開始使用適用於 Node.js 的 Azure CDN SDK | Microsoft Docs
description: 了解如何撰寫 Node.js 應用程式來管理 Azure CDN。
services: cdn
documentationcenter: nodejs
author: zhangmanling
manager: erikre
editor: ''
ms.assetid: c4bb6a61-de3d-4f0c-9dca-202554c43dfa
ms.service: cdn
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 01/23/2017
ms.author: mazha
ms.openlocfilehash: 46ae8cd9775432d126cbde856c1fb06ea319297e
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38301559"
---
# <a name="get-started-with-azure-cdn-development"></a>開始使用 Azure CDN 開發
> [!div class="op_single_selector"]
> * [Node.js](cdn-app-dev-node.md)
> * [.NET](cdn-app-dev-net.md)
> 
> 

您可以使用 [Azure CDN SDK for Node.js](https://www.npmjs.com/package/azure-arm-cdn) ，自動建立和管理 CDN 設定檔與端點。  本教學課程會逐步建立簡單的 Node.js 主控台應用程式，示範數個可用的作業。  本教學課程的目的不是詳細說明 Azure CDN SDK for Node.js 的所有層面。

若要完成本教學課程，您應該已安裝和設定 [Node.js](http://www.nodejs.org) **4.x.x** 或更新版本。  您可以使用任何想要的文字編輯器，來建立 Node.js 應用程式。  為了撰寫本教學課程，我使用了 [Visual Studio 程式碼](https://code.visualstudio.com)。  

> [!TIP]
> 您可以在 MSDN 上下載 [本教學課程中完成的專案](https://code.msdn.microsoft.com/Azure-CDN-SDK-for-Nodejs-c712bc74) 。
> 
> 

[!INCLUDE [cdn-app-dev-prep](../../includes/cdn-app-dev-prep.md)]

## <a name="create-your-project-and-add-npm-dependencies"></a>建立專案並新增 NPM 相依性
現在，我們已為 CDN 設定檔建立資源群組，並為 Azure AD 應用程式授與權限來管理該群組內的 CDN 設定檔和端點，我們可以開始建立應用程式。

建立資料夾來儲存您的應用程式。  從主控台中使用您目前路徑中的 Node.js 工具，將您目前的位置設定為這個新資料夾，並執行下列命令來初始化您的專案︰

    npm init

然後您會看到一系列用來初始化專案的問題。  本教學課程使用 app.js 做為 *進入點*。  您可以在下列範例中看到我的其他選擇。

![NPM init 輸出](./media/cdn-app-dev-node/cdn-npm-init.png)

我們的專案現在會使用 packages.json  檔案加以初始化。  我們的專案將使用 NPM 封裝內含的一些 Azure 程式庫。  我們將會針對 Node.js (ms-rest-azure) 使用 Azure 用戶端執行階段，針對 Node.js (azure-arm-cd) 使用 Azure CDN 用戶端程式庫。  讓我們將它們新增至專案做為相依性。

    npm install --save ms-rest-azure
    npm install --save azure-arm-cdn

封裝完成安裝後，package.json  檔案看起來應該類似此範例 (版本號碼可能不同)：

``` json
{
  "name": "cdn_node",
  "version": "1.0.0",
  "description": "Azure CDN Node.js tutorial project",
  "main": "app.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "Cam Soper",
  "license": "MIT",
  "dependencies": {
    "azure-arm-cdn": "^0.2.1",
    "ms-rest-azure": "^1.14.4"
  }
}
```

最後，使用文字編輯器來建立空白文字檔，並將它儲存為專案資料夾根目錄中的 app.js 。  我們現在可以開始撰寫程式碼了。

## <a name="requires-constants-authentication-and-structure"></a>必要項目、常數、驗證和結構
在編輯器中開啟 app.js  ，開始撰寫程式的基本結構。

1. 使用下列內容在頂端為我們的 NPM 封裝新增「必要項目」︰
   
    ``` javascript
    var msRestAzure = require('ms-rest-azure');
    var cdnManagementClient = require('azure-arm-cdn');
    ```
2. 我們必須定義一些我們的方法將用到的常數。  新增下列內容。  務必視需要使用您自己的值來取代預留位置，包括 **&lt;角括號&gt;**。
   
    ``` javascript
    //Tenant app constants
    const clientId = "<YOUR CLIENT ID>";
    const clientSecret = "<YOUR CLIENT AUTHENTICATION KEY>"; //Only for service principals
    const tenantId = "<YOUR TENANT ID>";
   
    //Application constants
    const subscriptionId = "<YOUR SUBSCRIPTION ID>";
    const resourceGroupName = "CdnConsoleTutorial";
    const resourceLocation = "<YOUR PREFERRED AZURE LOCATION, SUCH AS Central US>";
    ```
3. 接下來，我們將會具現化 CDN 管理用戶端，並對它提供我們的認證。
   
    ``` javascript
    var credentials = new msRestAzure.ApplicationTokenCredentials(clientId, tenantId, clientSecret);
    var cdnClient = new cdnManagementClient(credentials, subscriptionId);
    ```
   
    如果您使用個別使用者驗證，這兩行看起來會稍有不同。
   
   > [!IMPORTANT]
   > 如果您選擇使用個別使用者驗證，而不是服務主體，只需使用這個程式碼範例。  請謹慎地保護您的個別使用者認證，並且不要告訴他人。
   > 
   > 
   
    ``` javascript
    var credentials = new msRestAzure.UserTokenCredentials(clientId, 
        tenantId, '<username>', '<password>', '<redirect URI>');
    var cdnClient = new cdnManagementClient(credentials, subscriptionId);
    ```
   
    務必使用正確資訊來取代 **&lt;角括號&gt;** 中的項目。  對於 `<redirect URI>`，請使用您在 Azure AD 中註冊應用程式時所輸入的重新導向 URI。
4. 我們的 Node.js 主控台應用程式將會採用一些命令列參數。  讓我們驗證看看是否已至少傳遞一個參數。
   
   ```javascript
   //Collect command-line parameters
   var parms = process.argv.slice(2);
   
   //Do we have parameters?
   if(parms == null || parms.length == 0)
   {
       console.log("Not enough parameters!");
       console.log("Valid commands are list, delete, create, and purge.");
       process.exit(1);
   }
   ```
5. 此動作會將我們帶到程式的主要部分，根據所傳遞的參數而定，這會讓我們轉往其他函式。
   
    ```javascript
    switch(parms[0].toLowerCase())
    {
        case "list":
            cdnList();
            break;
   
        case "create":
            cdnCreate();
            break;
   
        case "delete":
            cdnDelete();
            break;
   
        case "purge":
            cdnPurge();
            break;
   
        default:
            console.log("Valid commands are list, delete, create, and purge.");
            process.exit(1);
    }
    ```
6. 在程式中有幾個地方，我們必須確定已傳遞正確數目的參數，並在參數不正確時顯示一些說明。  讓我們建立函式來執行此工作。
   
   ```javascript
   function requireParms(parmCount) {
       if(parms.length < parmCount) {
           usageHelp(parms[0].toLowerCase());
           process.exit(1);
       }
   }
   
   function usageHelp(cmd) {
       console.log("Usage for " + cmd + ":");
       switch(cmd)
       {
           case "list":
               console.log("list profiles");
               console.log("list endpoints <profile name>");
               break;
   
           case "create":
               console.log("create profile <profile name>");
               console.log("create endpoint <profile name> <endpoint name> <origin hostname>");
               break;
   
           case "delete":
               console.log("delete profile <profile name>");
               console.log("delete endpoint <profile name> <endpoint name>");
               break;
   
           case "purge":
               console.log("purge <profile name> <endpoint name> <path>");
               break;
   
           default:
               console.log("Invalid command.");
       }
   }
   ```
7. 最後，我們將在 CDN 管理用戶端使用的函式為非同步的，因此需要有方法可在函式完成時回呼。  讓我們建立一個可顯示 CDN 管理用戶端輸出 (如果有的話)，並以正常程序結束程式的方法。
   
    ```javascript
    function callback(err, result, request, response) {
        if (err) {
            console.log(err);
            process.exit(1);
        } else {
            console.log((result == null) ? "Done!" : result);
            process.exit(0);
        }
    }
    ```

程式的基本結構已撰寫完成，接下來我們應該建立會根據我們的參數來呼叫的函式。

## <a name="list-cdn-profiles-and-endpoints"></a>列出 CDN 設定檔和端點
讓我們從列出現有設定檔和端點的程式碼來開始。  我的程式碼註解會提供預期的語法，因此我們會知道每個參數的去處。

```javascript
// list profiles
// list endpoints <profile name>
function cdnList(){
    requireParms(2);
    switch(parms[1].toLowerCase())
    {
        case "profiles":
            console.log("Listing profiles...");
            cdnClient.profiles.listByResourceGroup(resourceGroupName, callback);
            break;

        case "endpoints":
            requireParms(3);
            console.log("Listing endpoints...");
            cdnClient.endpoints.listByProfile(parms[2], resourceGroupName, callback);
            break;

        default:
            console.log("Invalid parameter.");
            process.exit(1);
    }
}
```

## <a name="create-cdn-profiles-and-endpoints"></a>建立 CDN 設定檔和端點
接下來，我們將撰寫函式來建立設定檔和端點。

```javascript
function cdnCreate() {
    requireParms(2);
    switch(parms[1].toLowerCase())
    {
        case "profile":
            cdnCreateProfile();
            break;

        case "endpoint":
            cdnCreateEndpoint();
            break;

        default:
            console.log("Invalid parameter.");
            process.exit(1);
    }
}

// create profile <profile name>
function cdnCreateProfile() {
    requireParms(3);
    console.log("Creating profile...");
    var standardCreateParameters = {
        location: resourceLocation,
        sku: {
            name: 'Standard_Verizon'
        }
    };

    cdnClient.profiles.create(parms[2], standardCreateParameters, resourceGroupName, callback);
}

// create endpoint <profile name> <endpoint name> <origin hostname>        
function cdnCreateEndpoint() {
    requireParms(5);
    console.log("Creating endpoint...");
    var endpointProperties = {
        location: resourceLocation,
        origins: [{
            name: parms[4],
            hostName: parms[4]
        }]
    };

    cdnClient.endpoints.create(parms[3], endpointProperties, parms[2], resourceGroupName, callback);
}
```

## <a name="purge-an-endpoint"></a>清除端點
假設端點已建立，我們可能想要在程式中執行的一個常見工作是清除此端點中的內容。

```javascript
// purge <profile name> <endpoint name> <path>
function cdnPurge() {
    requireParms(4);
    console.log("Purging endpoint...");
    var purgeContentPaths = [ parms[3] ];
    cdnClient.endpoints.purgeContent(parms[2], parms[1], resourceGroupName, purgeContentPaths, callback);
}
```

## <a name="delete-cdn-profiles-and-endpoints"></a>刪除 CDN 設定檔和端點
我們將會包含的最後一個函式會刪除端點和設定檔。

```javascript
function cdnDelete() {
    requireParms(2);
    switch(parms[1].toLowerCase())
    {
        // delete profile <profile name>
        case "profile":
            requireParms(3);
            console.log("Deleting profile...");
            cdnClient.profiles.deleteIfExists(parms[2], resourceGroupName, callback);
            break;

        // delete endpoint <profile name> <endpoint name>
        case "endpoint":
            requireParms(4);
            console.log("Deleting endpoint...");
            cdnClient.endpoints.deleteIfExists(parms[3], parms[2], resourceGroupName, callback);
            break;

        default:
            console.log("Invalid parameter.");
            process.exit(1);
    }
}
```

## <a name="running-the-program"></a>執行程式
現在我們可以使用慣用的偵錯工具或是在主控台執行我們的 Node.js 程式。

> [!TIP]
> 如果您要使用 Visual Studio 程式碼做為偵錯工具，您必須設定您的環境以傳入命令列參數。  Visual Studio 程式碼會在 **lanuch.json** 檔案中進行此動作。  尋找名為 **args** 的屬性，並為您的參數新增字串值陣列，使它看起來類似下列內容︰`"args": ["list", "profiles"]`。
> 
> 

讓我們從列出設定檔來著手。

![列出設定檔](./media/cdn-app-dev-node/cdn-list-profiles.png)

我們得到空白陣列。  我們的資源群組中並沒有任何設定檔，因此這很合理。  現在讓我們建立設定檔。

![建立設定檔](./media/cdn-app-dev-node/cdn-create-profile.png)

現在讓我們新增端點。

![建立端點](./media/cdn-app-dev-node/cdn-create-endpoint.png)

最後，讓我們刪除設定檔。

![刪除設定檔](./media/cdn-app-dev-node/cdn-delete-profile.png)

## <a name="next-steps"></a>後續步驟
若要查看此逐步解說中已完成的專案，請 [下載範例](https://code.msdn.microsoft.com/Azure-CDN-SDK-for-Nodejs-c712bc74)。

若要查看 Azure CDN SDK for Node.js 的參考資料，請檢視 [參考資料](http://azure.github.io/azure-sdk-for-node/azure-arm-cdn/latest/)。

若要尋找其他關於 Azure SDK for Node.js 的文件，請檢視 [完整參考資料](http://azure.github.io/azure-sdk-for-node/)。

使用 [PowerShell](cdn-manage-powershell.md)管理 CDN 資源。

