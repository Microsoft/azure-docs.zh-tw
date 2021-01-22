---
title: Azure 應用程式設定搭配 .NET Core 的快速入門 | Microsoft Docs
description: 在本快速入門中，您會建立 .NET Core 應用程式搭配 Azure 應用程式組態，以集中儲存和管理應用程式設定 (與您的程式碼分開)。
services: azure-app-configuration
author: AlexandraKemperMS
ms.service: azure-app-configuration
ms.topic: quickstart
ms.custom: devx-track-csharp
ms.date: 09/28/2020
ms.author: alkemper
ms.openlocfilehash: 0ff80287971365b1477be319dc7a04760687f6a9
ms.sourcegitcommit: 52e3d220565c4059176742fcacc17e857c9cdd02
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/21/2021
ms.locfileid: "98663395"
---
# <a name="quickstart-create-a-net-core-app-with-app-configuration"></a>快速入門：使用應用程式設定建立 .NET Core 應用程式

在本快速入門中，您會將 Azure 應用程式組態納入 .NET Core 主控台應用程式中，以集中儲存和管理應用程式設定 (與您的程式碼分開)。

## <a name="prerequisites"></a>Prerequisites

- Azure 訂用帳戶 - [建立免費帳戶](https://azure.microsoft.com/free/dotnet)
- [.NET Core SDK](https://dotnet.microsoft.com/download) - 也適用於 [Azure Cloud Shell](https://shell.azure.com)。

## <a name="create-an-app-configuration-store"></a>建立應用程式組態存放區

[!INCLUDE [azure-app-configuration-create](../../includes/azure-app-configuration-create.md)]

7. 選取 [組態總管]   > [建立]   > [索引鍵/值]  以新增下列索引鍵/值組：

    | Key | 值 |
    |---|---|
    | TestApp:Settings:Message | Azure 應用程式設定的值 |

    目前先讓 [標籤]  和 [內容類型]  保持空白。

8. 選取 [套用]  。

## <a name="create-a-net-core-console-app"></a>建立 .NET Core 主控台應用程式

您會使用 [.NET Core 命令列介面 (CLI)](/dotnet/core/tools/) 來建立新的 .NET Core 主控台應用程式專案。 使用 .NET Core CLI 而非 Visual Studio 的好處，在於 .NET Core CLI 可同時於 Windows、macOS 及 Linux 平台上取得。  或者，使用 [Azure Cloud Shell](https://shell.azure.com) 中提供的預先安裝工具。

1. 為您的專案建立新資料夾。

2. 在新的資料夾中，執行下列命令以建立新的 .NET Core 主控台應用程式專案：

    ```dotnetcli
    dotnet new console
    ```

## <a name="connect-to-an-app-configuration-store"></a>連線至應用程式組態存放區

1. 透過執行下列命令，將參考新增至 `Microsoft.Extensions.Configuration.AzureAppConfiguration` NuGet 套件：

    ```dotnetcli
    dotnet add package Microsoft.Extensions.Configuration.AzureAppConfiguration
    ```

2. 執行下列命令以還原您專案的套件：

    ```dotnetcli
    dotnet restore
    ```

3. 開啟 Program.cs  ，並將參考新增至 .NET Core 應用程式組態提供者。

    ```csharp
    using Microsoft.Extensions.Configuration;
    using Microsoft.Extensions.Configuration.AzureAppConfiguration;
    ```

4. 藉由呼叫 `builder.AddAzureAppConfiguration()` 方法將 `Main` 方法更新為使用應用程式設定。

    ```csharp
    static void Main(string[] args)
    {
        var builder = new ConfigurationBuilder();
        builder.AddAzureAppConfiguration(Environment.GetEnvironmentVariable("ConnectionString"));

        var config = builder.Build();
        Console.WriteLine(config["TestApp:Settings:Message"] ?? "Hello world!");
    }
    ```

## <a name="build-and-run-the-app-locally"></a>於本機建置並執行應用程式

1. 設定名為 **ConnectionString** 的環境變數，並將其設定為應用程式組態存放區的存取金鑰。 在命令列中執行下列命令：

    ```cmd
    setx ConnectionString "connection-string-of-your-app-configuration-store"
    ```

    如果您使用 Windows PowerShell，請執行下列命令：

    ```azurepowershell
    $Env:ConnectionString = "connection-string-of-your-app-configuration-store"
    ```

    如果您使用 macOS 或 Linux，請執行下列命令：

    ```console
    export ConnectionString='connection-string-of-your-app-configuration-store'
    ```

    重新啟動命令提示字元，讓變更生效。 列印環境變數的值，以驗證是否已正確設定。

2. 執行下列命令來建置主控台應用程式：

    ```dotnetcli
    dotnet build
    ```

3. 建置成功完成後，請執行下列命令以於本機執行應用程式：

    ```dotnetcli
    dotnet run
    ```

## <a name="clean-up-resources"></a>清除資源

[!INCLUDE [azure-app-configuration-cleanup](../../includes/azure-app-configuration-cleanup.md)]

## <a name="next-steps"></a>後續步驟

在本快速入門中，您已建立新的應用程式組態存放區，並透過[應用程式組態提供者](/dotnet/api/Microsoft.Extensions.Configuration.AzureAppConfiguration)將其與 .NET Corek 主控台應用程式搭配使用。 若要了解如何將 .NET Core 應用程式設定為以動態方式重新整理組態設定，請繼續進行下一個教學課程。

> [!div class="nextstepaction"]
> [啟用動態組態](./enable-dynamic-configuration-dotnet-core.md)