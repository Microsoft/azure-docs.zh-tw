---
title: 針對 Azure 儲存體帳戶設定自訂網域名稱 | Microsoft Docs
description: 使用 Azure 入口網站，將您專屬的正式名稱 (CNAME) 對應至 Azure 儲存體帳戶中的 Blob 或 Web 端點。
services: storage
author: tamram
ms.service: storage
ms.topic: article
ms.date: 06/26/2018
ms.author: tamram
ms.component: blobs
ms.openlocfilehash: beacb39752b2af51aba4ddeb41c9bff68803df1b
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46954767"
---
# <a name="configure-a-custom-domain-name-for-your-azure-storage-account"></a>針對 Azure 儲存體帳戶設定自訂網域名稱

您可以設定自訂網域名稱，以供存取 Azure 儲存體帳戶中的 Blob 資料。 Blob 儲存體的預設端點是 `<storage-account-name>.blob.core.windows.net`。 您也可以使用產生為[靜態網站功能 (預覽)](storage-blob-static-website.md) 一部份的 Web 端點。 若您將自訂網域和子網域 (如 **www.contoso.com**) 對應至儲存體帳戶的 Blob 或 Web 端點，則可讓使用者使用該網域存取您儲存體帳戶中的 Blob 資料。

> [!IMPORTANT]
> Azure 儲存體尚未以原生方式支援使用自訂網域的 HTTPS。 您目前可以[使用 Azure CDN 透過 HTTP 以自訂網域存取 blob](storage-https-custom-domain-cdn.md)。
>

> [!NOTE]  
> 儲存體帳戶目前支援每個帳戶只能有一個自訂網域名稱。 這表示您無法將自訂網域名稱同時對應至 Web 和 Blob 服務端點。

下表展示的幾個範例 URL，適用於名為 **mystorageaccount** 之儲存體帳戶中的 Blob 資料。 針對儲存體帳戶註冊的自訂網域為 **www.contoso.com**：

| 資源類型 | 預設 URL | 自訂網域 URL |
| --- | --- | --- | --- |
| 儲存體帳戶 | http://mystorageaccount.blob.core.windows.net | http://www.contoso.com |
| Blob |http://mystorageaccount.blob.core.windows.net/mycontainer/myblob | http://www.contoso.com/mycontainer/myblob |
| 根容器 | http://mystorageaccount.blob.core.windows.net/myblob 或 http://mystorageaccount.blob.core.windows.net/$root/myblob| http://www.contoso.com/myblob 或 http://www.contoso.com/$root/myblob |
| Web |  http://mystorageaccount.[zone].web.core.windows.net/$web/[indexdoc] 或 http://mystorageaccount.[zone].web.core.windows.net/[indexdoc] 或 http://mystorageaccount.[zone].web.core.windows.net/$web 或 http://mystorageaccount.[zone].web.core.windows.net/ | http://www.contoso.com/$web 或 http://www.contoso.com/ 或 http://www.contoso.com/$web/[indexdoc] 或  http://www.contoso.com/[indexdoc] |

> [!NOTE]  
> 以下 Blob 服務端點的所有範例也適用於 Web 服務端點。

## <a name="direct-vs-intermediary-domain-mapping"></a>直接與中繼網域對應

您可使用兩種方式，將自訂網域指向您儲存體帳戶的 Blob 端點：直接 CNAME 對應與使用 *asverify* 中繼子網域。

### <a name="direct-cname-mapping"></a>直接 CNAME 對應

第一種方法最為簡易，亦即建立正式名稱 (CNAME) 記錄將您的自訂網域與子網域直接對應至 Blob 端點。 CNAME 記錄是將來源網域對應至目的地網域的網域名稱系統 (DNS) 功能。 在此案例中，來源網域為您的專屬自訂網域和子網域，例如 *www.contoso.com*。 目的地網域為您的 Blob 服務端點，例如 *mystorageaccount.blob.core.windows.net*。

[註冊自訂網域](#register-a-custom-domain)涵蓋直接方法。

### <a name="intermediary-mapping-with-asverify"></a>使用 *asverify* 的中繼對應

第二種方法也會使用 CNAME 記錄，但會先運用由 Azure 辨識的特殊子網域以避免停機：**asverify**。

在將自訂網域對應至 Blob 端點的程序中，會由於您在 [Azure 入口網站](https://portal.azure.com)中註冊網域而導致短暫停機。 若自訂網域目前支援不得發生停機之服務等級協定 (SLA) 的應用程式，您可以使用 Azure *asverify* 子網域做為中繼註冊步驟。 此中繼步驟可確保讓使用者能夠在執行 DNS 對應時存取您的網域。

[使用 *asverify* 子網域註冊自訂網域](#register-a-custom-domain-using-the-asverify-subdomain)涵蓋中繼方法。

## <a name="register-a-custom-domain"></a>註冊自訂網域
若您不需要顧及使用者會短暫地無法存取網域的問題，或是自訂網域目前未主控應用程式，您可以使用此程序來註冊自訂網域。 您可以使用 Azure DNS 來設定 Azure Blob 存放區的自訂 DNS 名稱。 如需詳細資訊，請參閱[使用 Azure DNS 為 Azure 服務提供自訂網域設定](https://docs.microsoft.com/azure/dns/dns-custom-domain#blob-storage)。

若自訂網域目前支援不允許發生任何停機狀況的應用程式，請遵循[使用 *asverify* 子網域註冊自訂網域](#register-a-custom-domain-using-the-asverify-subdomain)中所述的程序操作。

若要設定自訂網域名稱，您必須在 DNS 中建立新的 CNAME 記錄。 CNAME 記錄會指定網域名稱的別名。 在此案例中，CNAME 記錄會將自訂網域的位址，對應至儲存體帳戶的 Blob 儲存體端點。

一般而言，您可在網域註冊機構的網站上管理網域的 DNS 設定。 各註冊機構指定 CNAME 記錄的方法都很類似，只是稍微不同，但概念都一樣。 部分基本網域註冊套件並未提供 DNS 組態，因此您可能需要先升級網域註冊套件，然後再建立 CNAME 記錄。

1. 在 [Azure 入口網站](https://portal.azure.com)中瀏覽至您的儲存體帳戶。
1. 在功能表刀鋒視窗的 [BLOB SERVICE] 下方，選取 [自訂網域] 以開啟 [自訂網域] 刀鋒視窗。
1. 登入網域註冊機構的網站，然後前往 DNS 管理頁面。 您可能會在 **Domain Name**、**DNS** 或 **Name Server Management** 等區段中發現此頁面。
1. 尋找管理 CNAME 的區段。 您可能需要移至進階設定頁面，並尋找 **CNAME**、**Alias** 或 **Subdomains** 單字。
1. 建立新的 CNAME 記錄並提供子網域別名，如 **www** 或 **photos**。 接著，以 **mystorageaccount.blob.core.windows.net** 格式 (其中 *mystorageaccount* 代表儲存體帳戶的名稱) 提供主機名稱 (即 Blob 服務端點)。 在 [Azure 入口網站](https://portal.azure.com)中，主機名稱會顯示於 [自訂網域] 刀鋒視窗的項目 #1。
1. 在 [Azure 入口網站](https://portal.azure.com) [自訂網域] 刀鋒視窗的文字方塊中，輸入包含子網域的自訂網域名稱。 例如，若您的網域為 **contoso.com**，且子網域別名為 **www**，請輸入 **www.contoso.com**。 若您的子網域為 **photos**，請輸入 **photos.contoso.com**。 子網域為*必要項目*。
1. 在 [自訂網域] 刀鋒視窗上選取 [儲存]，以註冊您的自訂網域。 如果註冊成功，您會看見指出已成功更新儲存體帳戶的入口網站通知。

一旦透過 DNS 傳播您的新 CNAME 記錄，使用者只要具備適當權限，即可透過您的自訂網域檢視 Blob 資料。

## <a name="register-a-custom-domain-using-the-asverify-subdomain"></a>使用 *asverify* 子網域註冊自訂網域
如果自訂網域目前需支援不得發生停機時間之 SLA 的應用程式，您可以使用此程序來註冊自訂網域。 您可藉由建立從 `asverify.<subdomain>.<customdomain>` 指向 `asverify.<storageaccount>.blob.core.windows.net` 的 CNAME，預先透過 Azure 註冊您的網域。 接著，您可以建立從 `<subdomain>.<customdomain>` 指向 `<storageaccount>.blob.core.windows.net` 的第二個 CNAME，將傳送至自訂網域的流量引導至 Blob 端點。

**asverify** 子網域是 Azure 認可的特殊子網域。 在自己的子網域前加上 `asverify`，代表您允許 Azure 在不修改網域之 DNS 記錄的情況下認可自訂網域。 一旦修改網域的 DNS 記錄，其將會在無停機的情況下對應至 Blob 端點。

1. 在 [Azure 入口網站](https://portal.azure.com)中瀏覽至您的儲存體帳戶。
1. 在功能表刀鋒視窗的 [BLOB SERVICE] 下方，選取 [自訂網域] 以開啟 [自訂網域] 刀鋒視窗。
1. 登入 DNS 提供者的網站，然後前往 DNS 管理頁面。 您可能會在 **Domain Name**、**DNS** 或 **Name Server Management** 等區段中發現此頁面。
1. 尋找管理 CNAME 的區段。 您可能需要移至進階設定頁面，並尋找 **CNAME**、**Alias** 或 **Subdomains** 單字。
1. 建立新的 CNAME 記錄，並提供包含 *asverify* 子網域的子網域別名。 例如 **asverify.www** 或 **asverify.photos**。 接著，以 **asverify.mystorageaccount.blob.core.windows.net** 格式 (其中 **mystorageaccount** 代表儲存體帳戶的名稱) 提供主機名稱 (即 Blob 服務端點)。 在 [Azure 入口網站](https://portal.azure.com)中，主機名稱會顯示於 [自訂網域] 刀鋒視窗的項目 #2。
1. 在 [Azure 入口網站](https://portal.azure.com) [自訂網域] 刀鋒視窗的文字方塊中，輸入包含子網域的自訂網域名稱。 不包含 *asverify*。 例如，若您的網域為 **contoso.com**，且子網域別名為 **www**，請輸入 **www.contoso.com**。 若您的子網域為 **photos**，請輸入 **photos.contoso.com**。 子網域為必要項目。
1. 選取 [使用間接 CNAME 驗證] 核取方塊。
1. 在 [自訂網域] 刀鋒視窗上選取 [儲存]，以註冊您的自訂網域。 如果註冊成功，您會看見指出已成功更新儲存體帳戶的入口網站通知。 此時，自訂網域已通過 Azure 的驗證，不過前往網域的流量尚未路由傳送到儲存體帳戶。
1. 返回 DNS 提供者的網站，然後建立將子網域對應至 Blob 服務端點的另一個 CNAME 記錄。 例如，將子網域指定為 **www** 或 **photos** (無 *asverify*)，並將主機名稱指定為 **mystorageaccount.blob.core.windows.net** (其中 **mystorageaccount** 為您儲存體帳戶的名稱)。 待這個步驟完成後，自訂網域的註冊作業也宣告完成。
1. 最後，您可刪除已建立的 CNAME 記錄 (包含 **asverify** 子網域)，因為只有在中繼步驟中才需要使用該記錄。

一旦透過 DNS 傳播您的新 CNAME 記錄，使用者只要具備適當權限，即可透過您的自訂網域檢視 Blob 資料。

## <a name="test-your-custom-domain"></a>測試自訂網域

若要確認自訂網域是否確實對應至 Blob 服務端點，請在儲存體帳戶內的公用容器中建立 Blob。 接著，在網頁瀏覽器中使用以下格式的 URI 存取 Blob：

`http://<subdomain.customdomain>/<mycontainer>/<myblob>`

例如，下列 URI 可讓您透過 **photos.contoso.com** 自訂子網域中的 **myforms** 容器，存取 Web 表單：

`http://photos.contoso.com/myforms/applicationform.htm`

## <a name="deregister-a-custom-domain"></a>取消註冊自訂網域

若要取消註冊您的 Blob 儲存體端點自訂網域，請使用下列其中一項程序。

### <a name="azure-portal"></a>Azure 入口網站

請在 Azure 入口網站中執行下列作業，以移除自訂的網域設定：

1. 在 [Azure 入口網站](https://portal.azure.com)中瀏覽至您的儲存體帳戶。
1. 在功能表刀鋒視窗的 [BLOB SERVICE] 下方，選取 [自訂網域] 以開啟 [自訂網域] 刀鋒視窗。
1. 清除包含自訂網域名稱的文字方塊中的內容。
1. 選取 [儲存] 按鈕。

順利移除自訂網域後，您會看到指出已成功更新儲存體帳戶的入口網站通知。

### <a name="azure-cli"></a>Azure CLI

使用 [az storage account update](https://docs.microsoft.com/cli/azure/storage/account#az_storage_account_update) CLI 命令，並指定 `--custom-domain`引數值的空字串 (`""`)，以移除自訂網域註冊。

* 命令格式︰

  ```azurecli
  az storage account update \
      --name <storage-account-name> \
      --resource-group <resource-group-name> \
      --custom-domain ""
  ```

* 命令範例︰

  ```azurecli
  az storage account update \
      --name mystorageaccount \
      --resource-group myresourcegroup \
      --custom-domain ""
  ```

### <a name="powershell"></a>PowerShell

使用 [Set-AzureRmStorageAccount](/powershell/module/azurerm.storage/set-azurermstorageaccount) PowerShell cmdlet，並指定 `-CustomDomainName`引數值的空字串 (`""`)，以移除自訂網域註冊。

* 命令格式︰

  ```powershell
  Set-AzureRmStorageAccount `
      -ResourceGroupName "<resource-group-name>" `
      -AccountName "<storage-account-name>" `
      -CustomDomainName ""
  ```

* 命令範例︰

  ```powershell
  Set-AzureRmStorageAccount `
      -ResourceGroupName "myresourcegroup" `
      -AccountName "mystorageaccount" `
      -CustomDomainName ""
  ```

## <a name="next-steps"></a>後續步驟
* [將自訂網域對應至 Azure 內容傳遞網路 (CDN) 端點](../../cdn/cdn-map-content-to-custom-domain.md)
* [使用 Azure CDN 透過 HTTP 以自訂網域存取 blob](storage-https-custom-domain-cdn.md)
* [Azure Blob 儲存體中的靜態網站託管 (預覽)](storage-blob-static-website.md)
