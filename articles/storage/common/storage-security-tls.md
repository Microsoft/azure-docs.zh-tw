---
title: 為 Azure 儲存體用戶端啟用安全的 TLS | Microsoft Docs
description: 了解如何在 Azure 儲存體的用戶端中啟用 TLS 1.2。
services: storage
author: fhryo-msft
ms.service: storage
ms.topic: article
ms.date: 06/25/2018
ms.author: fryu
ms.component: common
ms.openlocfilehash: 86331f60e1be7d55c39e5d59f85b0f090a322587
ms.sourcegitcommit: e2ea404126bdd990570b4417794d63367a417856
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/14/2018
ms.locfileid: "45573853"
---
# <a name="enable-secure-tls-for-azure-storage-client"></a>為 Azure 儲存體用戶端啟用安全的 TLS

傳輸層安全性 (TLS) 和安全通訊端層 (SSL) 是密碼編譯通訊協定，可透過電腦網路提供通訊安全性。 目前已確認 SSL 1.0、2.0 和 3.0 易遭受攻擊。 RFC 已加以禁用。 TLS 1.0 因使用不安全的區塊編碼器 (DES CBC 和 RC2 CBC) 和資料流編碼器 (RC4) 而缺乏安全性。 PCI 委員會也建議移轉至更高的 TLS 版本。 如需詳細資訊，請參閱[傳輸層安全性 (TLS)](https://en.wikipedia.org/wiki/Transport_Layer_Security#SSL_1.0.2C_2.0_and_3.0)。

Azure 儲存體自 2015 年起即停止使用 SSL 3.0，並在公用 HTTPS 端點上使用 TLS 1.2，但仍支援 TLS 1.0 和 TLS 1.1 以維持回溯相容性。

為了確保安全且相容的 Azure 儲存體連線，您必須先在用戶端啟用 TLS 1.2 或更新版本，再傳送操作 Azure 儲存體服務的要求。

## <a name="enable-tls-12-in-net-client"></a>在 .NET 用戶端啟用 TLS 1.2

若要讓用戶端能夠交涉 TLS 1.2，作業系統和 .NET Framework 版本都必須支援 TLS 1.2。 請參閱 [TLS 1.2 的支援](https://docs.microsoft.com/dotnet/framework/network-programming/tls#support-for-tls-12)中的詳細資訊。

下列範例說明如何在 .NET 用戶端啟用 TLS 1.2。

```csharp

    static void EnableTls12()
    {
        // Enable TLS 1.2 before connecting to Azure Storage
        System.Net.ServicePointManager.SecurityProtocol = System.Net.SecurityProtocolType.Tls12;

        // Connect to Azure Storage
        CloudStorageAccount storageAccount = CloudStorageAccount.Parse("DefaultEndpointsProtocol=https;AccountName={yourstorageaccount};AccountKey={yourstorageaccountkey};EndpointSuffix=core.windows.net");
        CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();

        CloudBlobContainer container = blobClient.GetContainerReference("foo");
        container.CreateIfNotExists();
    }

```

## <a name="enable-tls-12-in-powershell-client"></a>在 PowerShell 用戶端啟用 TLS 1.2

下列範例說明如何在 PowerShell 用戶端啟用 TLS 1.2。

```powershell

# Enable TLS 1.2 before connecting to Azure Storage
[System.Net.ServicePointManager]::SecurityProtocol = [System.Net.SecurityProtocolType]::Tls12;

$resourceGroup = "{YourResourceGropuName}"
$storageAccountName = "{YourStorageAccountNme}"
$prefix = "foo"

# Connect to Azure Storage
$storageAccount = Get-AzureRmStorageAccount -ResourceGroupName $resourceGroup -Name $storageAccountName
$ctx = $storageAccount.Context
$listOfContainers = Get-AzureStorageContainer -Context $ctx -Prefix $prefix
$listOfContainers

```

## <a name="verify-tls-12-connection"></a>驗證 TLS 1.2 連線

您可以使用 Fiddler 來確認是否已使用 TLS 1.2。 開啟 Fiddler 以開始擷取用戶端網路流量，然後執行上述範例。 接著，您可以在範例所建立的連線中看到 TLS 版本。

下列螢幕擷取畫面是驗證的範例。

![在 Fiddler 中驗證 TLS 版本的螢幕擷取畫面](./media/storage-security-tls/storage-security-tls-verify-in-fiddler.png)

## <a name="see-also"></a>另請參閱

* [傳輸層安全性 (TLS)](https://en.wikipedia.org/wiki/Transport_Layer_Security#SSL_1.0.2C_2.0_and_3.0)
* [TLS 上的 PCI 合規性](https://blog.pcisecuritystandards.org/migrating-from-ssl-and-early-tls)
* [在 Java 用戶端中啟用 TLS](https://www.java.com/en/configure_crypto.html)
