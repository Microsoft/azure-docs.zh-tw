---
title: 使用 Visual Studio 在 ASP.NET 專案中新增 Key Vault 支援 | Microsoft Docs
description: 使用此教學課程，以協助您了解如何在 ASP.NET 或 ASP.NET Core Web 應用程式中新增 Key Vault 支援。
services: key-vault
author: ghogen
manager: douge
ms.prod: visual-studio-dev15
ms.technology: vs-azure
ms.custom: vs-azure
ms.workload: azure-vs
ms.topic: conceptual
ms.date: 04/15/2018
ms.author: ghogen
ms.openlocfilehash: 9cf49ae97da3bf67300bdc222c86bb712aeaed37
ms.sourcegitcommit: 06724c499837ba342c81f4d349ec0ce4f2dfd6d6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46465787"
---
# <a name="add-key-vault-to-your-web-application-by-using-visual-studio-connected-services"></a>使用 Visual Studio 連線服務在 Web 應用程式中新增 Key Vault

在此教學課程中，您將了解如何輕鬆地新增所需的項目，以便開始在 Visual Studio 中使用 Azure Key Vault 來管理 Web 專案的祕密，而不論您使用的是 ASP.NET Core 或任何類型的 ASP.NET 專案。 透過使用 Visual Studio 2017 中的「連線服務」功能，您可以讓 Visual Studio 自動新增所需的所有 NuGet 套件和組態設定，以連線到 Azure 中的 Key Vault。 

針對「連線服務」在您專案中為啟用 Key Vault 所做的變更，如需詳細資料，請參閱 [Key Vault 連線服務 - 我的 ASP.NET 4.7.1 專案發生什麼情形](vs-key-vault-aspnet-what-happened.md)或 [Key Vault 連線服務 - 我的 ASP.NET Core 專案發生什麼情形](vs-key-vault-aspnet-core-what-happened.md)。

## <a name="prerequisites"></a>先決條件

- **Azure 訂用帳戶**。 如果您沒有訂用帳戶，您可以註冊 [免費帳戶](https://azure.microsoft.com/pricing/free-trial/)。
- **Visual Studio 2017 版本 15.7**，並且已安裝**網頁程式開發**工作負載。 [立即下載](https://aka.ms/vsdownload?utm_source=mscom&utm_campaign=msdocs)。
- 針對 ASP.NET (不是 Core)，您需要預設不會安裝的 .NET Framework 4.7.1 開發工具。 若要安裝它們，請啟動 Visual Studio 安裝程式、選擇 [修改]，然後選擇 [個別元件]，接著在右側展開 [ASP.NET 與網頁程式開發]，並選擇 [.NET Framework 4.7.1 開發工具]。
- ASP.NET 4.7.1 或 ASP.NET Core 2.0 Web 專案隨即開啟。

## <a name="add-key-vault-support-to-your-project"></a>在專案中新增 Key Vault 支援

1. 在 [方案總管] 中，選擇 [新增] > [連線服務]。
   [連線服務] 頁面隨即出現，並顯示您可新增至專案的服務。
1. 在可用服務的功能表中，選擇 [運用 Azure Key Vault 保護祕密的安全]。

   ![選擇 [運用 Azure Key Vault 保護祕密的安全]](media/vs-key-vault-add-connected-service/KeyVaultConnectedService1.PNG)

   如果您已登入 Visual Studio，並具有與您帳戶相關聯的 Azure 訂用帳戶，則會出現一個頁面，其中顯示含有您訂用帳戶的下拉式清單。 請確定您已登入 Visual Studio，並且您登入的帳戶與您用於 Azure 訂用帳戶的帳戶相同。

1. 選取您想要使用的訂用帳戶，然後選擇新的或現有的 Key Vault，或選擇 [編輯] 連結來修改自動產生的名稱。

   ![選取您的訂用帳戶](media/vs-key-vault-add-connected-service/KeyVaultConnectedService3.PNG)

1. 輸入您想要針對 Key Vault 使用的名稱。

   ![將 Key Vault 重新命名並選擇資源群組](media/vs-key-vault-add-connected-service/KeyVaultConnectedService-Edit.PNG)

1. 選取現有的資源群組，或選擇要使用自動產生的唯一名稱來建立新的資源群組。  如果想要使用不同的名稱來建立新群組，您可以使用 [Azure 入口網站](https://portal.azure.com)，接著關閉頁面並重新啟動，以重新載入資源群組的清單。
1. 選擇要在其中建立 Key Vault 的區域。 如果您的 Web 應用程式裝載於 Azure 中，請選擇裝載該 Web 應用程式的區域以獲得最佳效能。
1. 選擇定價模型。 如需詳細資訊，請參閱 [Key Vault 價格](https://azure.microsoft.com/pricing/details/key-vault/)。
1. 選擇 [確定] 以接受設定選項。
1. 選擇 [新增] 以建立 Key Vault。 如果您選擇已使用的名稱，則建立程序可能會失敗。  如果發生這種情況，使用 [編輯] 連結來將 Key Vault 重新命名，然後再試一次。

   ![正在將連線服務新增至專案](media/vs-key-vault-add-connected-service/KeyVaultConnectedService4.PNG)

1. 現在，在 Azure 的 Key Vault 中新增祕密。 若要到達入口網站中的正確位置，按一下 [管理儲存在此 Key Vault 中的祕密] 連結。 如果您關閉了頁面或專案，可以在 [Azure 入口網站](https://portal.azure.com)中，選擇 [安全性] 下方的 [所有服務]、選擇 [Key Vault]，然後選擇您剛才建立的 Key Vault，就能瀏覽到該頁面或專案。

   ![瀏覽至入口網站](media/vs-key-vault-add-connected-service/manage-secrets-link.jpg)

1. 在您所建立 Key Vault 的 [Key Vault] 區段中，依序選擇 [祕密] 和 [產生/匯入]。

   ![產生/匯入祕密](media/vs-key-vault-add-connected-service/generate-secrets.jpg)

1. 輸入祕密 (例如 "MySecret")，並為它指定任何字串值作為測試，然後選擇 [建立] 按鈕。

   ![建立祕密](media/vs-key-vault-add-connected-service/create-a-secret.jpg)

1. (選擇性) 輸入其他祕密，但此時請將它命名為 "Secrets--MySecret"，藉以將它放置於分類中。 此語法會指定 "Secrets" 分類，其中包含 "MySecret" 祕密。
 
現在，您可透過程式碼存取祕密。 後續步驟會根據您使用的是 ASP.NET 4.7.1 或 ASP.NET Core 而有所差異。

## <a name="access-your-secrets-in-code-aspnet-core-projects"></a>透過程式碼存取祕密 (ASP.NET Core 專案)

Key Vault 的連線會在啟動時由實作 [Microsoft.AspNetCore.Hosting.IHostingStartup](/dotnet/api/microsoft.aspnetcore.hosting.ihostingstartup?view=aspnetcore-2.1) 的類別以擴充啟動行為的方式設定，其擴充方式的說明請見[透過 IHostingStartup 從 ASP.NET Core 中的外部組件增強應用程式](/aspnet/core/fundamentals/host/platform-specific-configuration)。 啟動類別會使用兩個包含 Key Vault 連線資訊的環境變數：ASPNETCORE_HOSTINGSTARTUP__KEYVAULT__CONFIGURATIONENABLED (設定為 true) 和 ASPNETCORE_HOSTINGSTARTUP__KEYVAULT__CONFIGURATIONVAULT (設定為您的 Key Vault URL)。 當您執行**新增已連線的服務**程序時，這些變數將會新增至 launchsettings.json 檔案。

若要存取您的祕密：

1. 在 Visual Studio 的 ASP.NET Core 專案中，您現在可以透過在程式碼中使用下列運算式來參考這些祕密：
 
   ```csharp
      config["MySecret"] // Access a secret without a section
      config["Secrets:MySecret"] // Access a secret in a section
      config.GetSection("Secrets")["MySecret"] // Get the configuration section and access a secret in it.
   ```

1. 在 .cshtml 頁面 (假設為 About.cshtml) 中，在接近檔案最上方處加入 @inject 指示詞，以設定可用來存取 Key Vault 設定的變數。

   ```cshtml
      @inject Microsoft.Extensions.Configuration.IConfiguration config
   ```

1. 進行測試時，您可以透過在其中一個頁面上顯示祕密的值，來確認其可供使用。 使用 @config 來參考 config 變數。
 
   ```cshtml
      <p> @config["MySecret"] </p>
      <p> @config.GetSection("Secrets")["MySecret"] </p>
      <p> @config["Secrets:MySecret"] </p>
   ```

1. 建置並執行 Web 應用程式、瀏覽至 [關於] 頁面，然後查看「祕密」值。

## <a name="access-your-secrets-in-code-aspnet-471-projects"></a>透過程式碼存取祕密 (ASP.NET 4.7.1 專案)

Key Vault 的連線會由 ConfigurationBuilder 類別使用您在執行**新增已連線的服務**程序時新增至 web.config 檔案中的資訊來設定。

若要存取您的祕密：

1. 修改 web.config，如下所示。 金鑰乃是預留位置，將由 AzureKeyVault ConfigurationBuilder 以 Key Vault 中的祕密值來取代。

   ```xml
     <appSettings configBuilders="AzureKeyVault">
       <add key="webpages:Version" value="3.0.0.0" />
       <add key="webpages:Enabled" value="false" />
       <add key="ClientValidationEnabled" value="true" />
       <add key="UnobtrusiveJavaScriptEnabled" value="true" />
       <add key="MySecret" value="dummy1"/>
       <add key="Secrets--MySecret" value="dummy2"/>
     </appSettings>
   ```

1. 在 HomeController 的 [關於] 控制器方法中，加入下列程式碼行以擷取祕密，並將它儲存於 ViewBag。
 
   ```csharp
            var secret = ConfigurationManager.AppSettings["MySecret"];
            var secret2 = ConfigurationManager.AppSettings["Secrets--MySecret"];
            ViewBag.Secret = $"Secret: {secret}";
            ViewBag.Secret2 = $"Secret2: {secret2}";
   ```

1. 在 About.cshtml 檢視中，加入下列程式碼以顯示祕密的值 (僅限測試)。

   ```csharp
      <h3>@ViewBag.Secret</h3>
      <h3>@ViewBag.Secret2</h3>
   ```

恭喜，您現在已確認 Web 應用程式可以使用 Key Vault，安全地存取已儲存的祕密。

## <a name="clean-up-resources"></a>清除資源

不再需要資源群組時，請將它刪除。 這會刪除 Key Vault 及相關資源。 若要透過入口網站刪除資源群組：

1. 在入口網站頂端的 [搜尋] 方塊中，輸入資源群組的名稱。 當您在搜尋結果中看到此快速入門中使用的資源群組時，請選取它。
2. 選取 [刪除資源群組]。
3. 在 [輸入資源群組名稱:] 方塊中輸入資源群組的名稱，然後選取 [刪除]。

## <a name="next-steps"></a>後續步驟

若要深入了解 Key Vault 的開發，請閱讀[金鑰保存庫開發人員指南](key-vault-developers-guide.md)