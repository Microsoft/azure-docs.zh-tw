---
title: 在 Visual Studio 中開始使用 Key Vault 連線服務 (ASP.NET Core 專案) | Microsoft Docs
description: 使用本教學課程，以協助您了解如何在 ASP.NET 或 ASP.NET Core Web 應用程式中新增 Key Vault 支援。
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
ms.openlocfilehash: e591771ee9c9cb12d9ec2ff61ec7f5a76691c8c7
ms.sourcegitcommit: 30c7f9994cf6fcdfb580616ea8d6d251364c0cd1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/18/2018
ms.locfileid: "42141516"
---
# <a name="get-started-with-key-vault-connected-service-in-visual-studio-aspnet-core-projects"></a>在 Visual Studio 中開始使用 Key Vault 連線服務 (ASP.NET Core 專案)

> [!div class="op_single_selector"]
> - [開始使用](vs-key-vault-aspnet-core-get-started.md)
> - [發生什麼情形](vs-key-vault-aspnet-core-what-happened.md)

本文提供您在 Visual Studio 中透過 [新增連線服務] 命令，將 Key Vault 新增至 ASP.NET Core 專案之後所需的其他指引。 如果您尚未將服務新增至專案，則隨時都可遵循[使用 Visual Studio 連線服務在 Web 應用程式中新增 Key Vault](vs-key-vault-add-connected-service.md)中的指示來執行此動作。

請參閱[我的 ASP.NET Core 專案發生什麼情形？](vs-key-vault-aspnet-core-what-happened.md)，以了解您的專案在新增連線服務時所做的變更。

## <a name="after-you-connect"></a>連線之後

1. 在 Azure 的 Key Vault 中新增祕密。 若要到達入口網站中的正確位置，按一下 [管理儲存在此 Key Vault 中的祕密] 連結。 如果您關閉了頁面或專案，可以在 [Azure 入口網站](https://portal.azure.com)中，選擇 [安全性] 下方的 [所有服務]、選擇 [Key Vault]，然後選擇您剛才建立的 Key Vault，就能瀏覽到該頁面或專案。

   ![瀏覽至入口網站](media/vs-key-vault-add-connected-service/manage-secrets-link.jpg)

1. 在您所建立 Key Vault 的 [Key Vault] 區段中，依序選擇 [祕密] 和 [產生/匯入]。

   ![產生/匯入祕密](media/vs-key-vault-add-connected-service/generate-secrets.jpg)

1. 輸入祕密 (例如 "MySecret")，並為它指定任何字串值作為測試，然後選擇 [建立] 按鈕。

   ![建立祕密](media/vs-key-vault-add-connected-service/create-a-secret.jpg)
 
1. (選擇性) 輸入其他祕密，但此時請將它命名為 **Secrets--MySecret**，藉以將它放置於分類中。 此語法會指定 **Secrets** 分類，其中包含 **MySecret** 祕密。
1. 在您的 Visual Studio 專案中，您現在可以藉由在程式碼中使用下列運算式來參考這些祕密：
 
   ```csharp
      config["MySecret"] // Access a secret without a section
      config["Secrets:MySecret"] // Access a secret in a section
      config.GetSection("Secrets")["MySecret"] // Get the configuration section and access a secret in it.
   ```

恭喜，您現在已啟用 Web 應用程式來使用 Key Vault，安全地存取已儲存的祕密。

## <a name="clean-up-resources"></a>清除資源

不再需要資源群組時，請加以刪除。 這會刪除 Key Vault 及相關資源。 若要透過入口網站刪除資源群組：

1. 在入口網站頂端的 [搜尋] 方塊中，輸入資源群組的名稱。 當您在搜尋結果中看到本快速入門中使用的資源群組時，請加以選取。
2. 選取 [刪除資源群組]。
3. 在 [輸入資源群組名稱:] 方塊中輸入資源群組的名稱，然後選取 [刪除]。

# <a name="next-steps"></a>後續步驟

在[金鑰保存庫開發人員指南](key-vault-developers-guide.md)中深入了解如何使用 Key Vault 進行開發