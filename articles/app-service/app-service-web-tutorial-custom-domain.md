---
title: 將現有的自訂 DNS 名稱對應至 Azure Web Apps | Microsoft Docs
description: 學習如何將現有的自訂 DNS 網域名稱 (虛名網域) 新增至 Azure App Service 中的 Web 應用程式、行動裝置應用程式後端或 API 應用程式。
keywords: 應用程式服務, Azure 應用程式服務, 網域對應, 網域名稱, 現有的網域, 主機名稱
services: app-service\web
documentationcenter: nodejs
author: cephalin
manager: erikre
editor: ''
ms.assetid: dc446e0e-0958-48ea-8d99-441d2b947a7c
ms.service: app-service-web
ms.workload: web
ms.tgt_pltfrm: na
ms.devlang: nodejs
ms.topic: tutorial
ms.date: 06/18/2018
ms.author: cephalin
ms.custom: mvc
ms.openlocfilehash: 30199005db93f9a43a37d2c72bb34dd772265419
ms.sourcegitcommit: 31241b7ef35c37749b4261644adf1f5a029b2b8e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/04/2018
ms.locfileid: "43664882"
---
# <a name="tutorial-map-an-existing-custom-dns-name-to-azure-web-apps"></a>教學課程：將現有的自訂 DNS 名稱對應至 Azure Web Apps

[Azure Web Apps](app-service-web-overview.md) 提供可高度擴充、自我修復的 Web 主機服務。 本教學課程會示範如何將現有的自訂 DNS 名稱對應至 Azure Web 應用程式。

![入口網站瀏覽至 Azure 應用程式](./media/app-service-web-tutorial-custom-domain/app-with-custom-dns.png)

在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 使用 CNAME 記錄來對應子網域 (例如，`www.contoso.com`)
> * 使用 A 記錄來對應根網域 (例如，`contoso.com`)
> * 使用 CNAME 記錄來對應萬用字元網域 (例如，`*.contoso.com`)
> * 將預設 URL 重新導向至自訂目錄
> * 使用指令碼來自動對應網域

## <a name="prerequisites"></a>必要條件

若要完成本教學課程：

* [建立 App Service 應用程式](/azure/app-service/)，或使用您針對另一個教學課程建立的應用程式。
* 購買網域名稱，並確定您擁有網域提供者 (例如 GoDaddy) 之 DNS 登錄的存取權。

  例如，若要為 `contoso.com` 和 `www.contoso.com` 新增 DNS 項目，您必須有權設定 `contoso.com` 根網域的 DNS 設定。

  > [!NOTE]
  > 如果您沒有現有的網域名稱，請考慮[使用 Azure 入口網站購買網域](custom-dns-web-site-buydomains-web-app.md)。 

## <a name="prepare-the-app"></a>準備應用程式

若要將自訂 DNS 名稱對應至 Web 應用程式，Web 應用程式的 [App Service 方案](https://azure.microsoft.com/pricing/details/app-service/)必須是付費層 (**共用**、**基本**、**標準**或**進階**)。 在此步驟中，您要確定 App Service 應用程式位於支援的定價層。

[!INCLUDE [app-service-dev-test-note](../../includes/app-service-dev-test-note.md)]

### <a name="sign-in-to-azure"></a>登入 Azure

開啟 [Azure 入口網站](https://portal.azure.com)並使用您的 Azure 帳戶登入。

### <a name="navigate-to-the-app-in-the-azure-portal"></a>瀏覽至 Azure 入口網站中的應用程式

從左側功能表，選取 [App Service]，然後選取應用程式的名稱。

![入口網站瀏覽至 Azure 應用程式](./media/app-service-web-tutorial-custom-domain/select-app.png)

您看到 App Service 應用程式的管理分頁。  

<a name="checkpricing"></a>

### <a name="check-the-pricing-tier"></a>檢查定價層

在應用程式分頁的左側導覽中，捲動到 [設定] 區段，然後選取 [相應增加 (App Service 方案)]。

![相應增加功能表](./media/app-service-web-tutorial-custom-domain/scale-up-menu.png)

會以藍色框線醒目顯示應用程式目前的層。 請檢查以確定您的應用程式不是位於 **F1** 層。 **F1** 層不支援自訂 DNS。 

![檢查定價層](./media/app-service-web-tutorial-custom-domain/check-pricing-tier.png)

如果 App Service 方案不是位於 **F1** 層，請關閉 [相應增加] 分頁，然後跳至 [對應 CNAME 記錄](#cname)。

<a name="scaleup"></a>

### <a name="scale-up-the-app-service-plan"></a>相應增加 App Service 方案

選取任何非免費層 (**D1**、**B1**、**B2**、**B3** 或「生產」類別中的任何一層)。 如需其他選項，請按一下 [查看其他選項]。

按一下 [套用]。

![檢查定價層](./media/app-service-web-tutorial-custom-domain/choose-pricing-tier.png)

當您看見下列通知時，表示擴充作業已完成。

![擴充作業確認](./media/app-service-web-tutorial-custom-domain/scale-notification.png)

<a name="cname"></a>

## <a name="map-your-domain"></a>對應您的網域

您可以使用 **CNAME 記錄**或 **A 記錄**將自訂 DNS 名稱對應至 App Service。 請依照個別步驟操作：

- [對應 CNAME 記錄](#map-a-cname-record)
- [對應 A 記錄](#map-an-a-record)
- [對應萬用字元網域 (使用 CNAME 記錄)](#map-a-wildcard-domain)

> [!NOTE]
> 您應對所有自訂 DNS 名稱使用 CNAME 記錄，但根網域除外 (例如 `contoso.com`)。 對於根網域，請使用 A 記錄。

### <a name="map-a-cname-record"></a>對應 CNAME 記錄

在教學課程範例中，您新增 `www` 子網域 (例如，`www.contoso.com`) 的 CNAME 記錄。

#### <a name="access-dns-records-with-domain-provider"></a>存取網域提供者中的 DNS 記錄

[!INCLUDE [Access DNS records with domain provider](../../includes/app-service-web-access-dns-records-no-h.md)]

#### <a name="create-the-cname-record"></a>建立 CNAME 記錄

新增 CNAME 記錄以將子網域對應到應用程式的預設主機名稱 (`<app_name>.azurewebsites.net`，其中 `<app_name>` 是您的應用程式的名稱)。

對於 `www.contoso.com` 網域範例，新增將名稱 `www` 對應至 `<app_name>.azurewebsites.net` 的 CNAME 記錄。

新增 CNAME 之後，DNS 記錄分頁看起來如下列範例所示：

![入口網站瀏覽至 Azure 應用程式](./media/app-service-web-tutorial-custom-domain/cname-record.png)

#### <a name="enable-the-cname-record-mapping-in-azure"></a>在 Azure 中啟用 CNAME 記錄對應

在 Azure 入口網站之應用程式分頁的左側導覽中，選取 [自訂網域]。 

![[自訂網域] 功能表](./media/app-service-web-tutorial-custom-domain/custom-domain-menu.png)

在應用程式的 [自訂網域] 分頁中，在清單中新增自訂的完整 DNS 名稱 (`www.contoso.com`)。

選取 [新增主機名稱] 旁的 **+** 圖示。

![新增主機名稱](./media/app-service-web-tutorial-custom-domain/add-host-name-cname.png)

輸入您新增 CNAME 記錄的完整網域名稱，例如 `www.contoso.com`。 

選取 [驗證]。

[新增主機名稱] 頁面隨即顯示。 

請確定 [主機名稱記錄類型] 設為 [CNAME (www.example.com 或任何子網域)]。

選取 [新增主機名稱]。

![將 DNS 名稱新增至應用程式](./media/app-service-web-tutorial-custom-domain/validate-domain-name-cname.png)

可能需要一些時間，新的主機名稱才會反映在應用程式的 [自訂網域] 分頁中。 嘗試重新整理瀏覽器以更新資料。

![CNAME 記錄已新增](./media/app-service-web-tutorial-custom-domain/cname-record-added.png)

> [!NOTE]
> 若要新增 SSL 繫結，請參閱[將現有的自訂 SSL 憑證繫結至 Azure Web Apps](app-service-web-tutorial-custom-ssl.md)。

如果稍早錯過某個步驟，或在某處打錯字，您在頁面底部會看到驗證錯誤。

![驗證錯誤](./media/app-service-web-tutorial-custom-domain/verification-error-cname.png)

<a name="a"></a>

### <a name="map-an-a-record"></a>對應 A 記錄

在教學課程範例中，您新增根網域 (例如，`contoso.com`) 的 A 記錄。 

<a name="info"></a>

#### <a name="copy-the-apps-ip-address"></a>複製應用程式的 IP 位址

若要對應 A 記錄，您需要應用程式的外部 IP 位址。 您可以在 Azure 入口網站之應用程式的 [自訂網域] 分頁中找到這個 IP 位址。

在 Azure 入口網站之應用程式分頁的左側導覽中，選取 [自訂網域]。 

![[自訂網域] 功能表](./media/app-service-web-tutorial-custom-domain/custom-domain-menu.png)

在 [自訂網域] 頁面中，複製應用程式的 IP 位址。

![入口網站瀏覽至 Azure 應用程式](./media/app-service-web-tutorial-custom-domain/mapping-information.png)

#### <a name="access-dns-records-with-domain-provider"></a>存取網域提供者中的 DNS 記錄

[!INCLUDE [Access DNS records with domain provider](../../includes/app-service-web-access-dns-records-no-h.md)]

#### <a name="create-the-a-record"></a>建立 A 記錄

若要將 A 記錄對應至應用程式，App Service 需要**兩筆** DNS 記錄︰

- **A** 記錄對應至應用程式的 IP 位址。
- **TXT** 記錄對應至應用程式的預設主機名稱 `<app_name>.azurewebsites.net`。 App Service 只會在設定時使用此記錄，以確認您擁有自訂網域。 驗證您的自訂網域並且在 App Service 中設定之後，您就可以刪除此 TXT 記錄。 

對於 `contoso.com` 網域範例，請根據下表建立 A 和 TXT 記錄 (`@` 通常代表根網域)。 

| 記錄類型 | Host | 值 |
| - | - | - |
| 具有使用  | `@` | 來自[複製應用程式的 IP 位址](#info)的 IP 位址 |
| TXT | `@` | `<app_name>.azurewebsites.net` |

新增記錄時，DNS 記錄分頁看起來如下列範例所示：

![[DNS 記錄] 頁面](./media/app-service-web-tutorial-custom-domain/a-record.png)

<a name="enable-a"></a>

#### <a name="enable-the-a-record-mapping-in-the-app"></a>在應用程式中啟用 A 記錄對應

回到 Azure 入口網站中的應用程式 [自訂網域] 分頁，在清單中新增自訂的完整 DNS 名稱 (例如，`contoso.com`)。

選取 [新增主機名稱] 旁的 **+** 圖示。

![新增主機名稱](./media/app-service-web-tutorial-custom-domain/add-host-name.png)

輸入您設定 A 記錄的完整網域名稱，例如 `contoso.com`。

選取 [驗證]。

[新增主機名稱] 頁面隨即顯示。 

請確定 [主機名稱記錄類型] 設為 [A 記錄 (example.com)]。

選取 [新增主機名稱]。

![將 DNS 名稱新增至應用程式](./media/app-service-web-tutorial-custom-domain/validate-domain-name.png)

可能需要一些時間，新的主機名稱才會反映在應用程式的 [自訂網域] 分頁中。 嘗試重新整理瀏覽器以更新資料。

![A 記錄已新增](./media/app-service-web-tutorial-custom-domain/a-record-added.png)

> [!NOTE]
> 若要新增 SSL 繫結，請參閱[將現有的自訂 SSL 憑證繫結至 Azure Web Apps](app-service-web-tutorial-custom-ssl.md)。

如果稍早錯過某個步驟，或在某處打錯字，您在頁面底部會看到驗證錯誤。

![驗證錯誤](./media/app-service-web-tutorial-custom-domain/verification-error.png)

<a name="wildcard"></a>

### <a name="map-a-wildcard-domain"></a>對應萬用字元網域

在教學課程範例中，您藉由新增 CNAME 記錄，將[萬用字元 DNS 名稱](https://en.wikipedia.org/wiki/Wildcard_DNS_record) (例如，`*.contoso.com`) 對應至 App Service 應用程式。 

#### <a name="access-dns-records-with-domain-provider"></a>存取網域提供者中的 DNS 記錄

[!INCLUDE [Access DNS records with domain provider](../../includes/app-service-web-access-dns-records-no-h.md)]

#### <a name="create-the-cname-record"></a>建立 CNAME 記錄

新增 CNAME 記錄以將萬用字元名稱對應至應用程式的預設主機名稱 (`<app_name>.azurewebsites.net`)。

針對 `*.contoso.com` 網域範例，CNAME 記錄會將名稱 `*` 對應至 `<app_name>.azurewebsites.net`。

新增 CNAME 時，DNS 記錄分頁看起來如下列範例所示：

![入口網站瀏覽至 Azure 應用程式](./media/app-service-web-tutorial-custom-domain/cname-record-wildcard.png)

#### <a name="enable-the-cname-record-mapping-in-the-app"></a>在應用程式中啟用 CNAME 記錄對應

您現在可以將符合萬用字元名稱的任何子網域新增至應用程式 (例如，`sub1.contoso.com` 和 `sub2.contoso.com` 符合 `*.contoso.com`)。 

在 Azure 入口網站之應用程式分頁的左側導覽中，選取 [自訂網域]。 

![[自訂網域] 功能表](./media/app-service-web-tutorial-custom-domain/custom-domain-menu.png)

選取 [新增主機名稱] 旁的 **+** 圖示。

![新增主機名稱](./media/app-service-web-tutorial-custom-domain/add-host-name-cname.png)

輸入符合萬用字元網域的完整網域名稱 (例如，`sub1.contoso.com`)，然後選取 [驗證]。

[新增主機名稱] 按鈕會啟用。 

請確定 [主機名稱記錄類型] 設為 [CNAME 記錄 (www.example.com 或任何子網域)]。

選取 [新增主機名稱]。

![將 DNS 名稱新增至應用程式](./media/app-service-web-tutorial-custom-domain/validate-domain-name-cname-wildcard.png)

可能需要一些時間，新的主機名稱才會反映在應用程式的 [自訂網域] 分頁中。 嘗試重新整理瀏覽器以更新資料。

再次選取 **+** 圖示，以新增另一個與萬用字元網域相符的主機名稱。 例如，新增 `sub2.contoso.com`。

![CNAME 記錄已新增](./media/app-service-web-tutorial-custom-domain/cname-record-added-wildcard2.png)

> [!NOTE]
> 若要新增 SSL 繫結，請參閱[將現有的自訂 SSL 憑證繫結至 Azure Web Apps](app-service-web-tutorial-custom-ssl.md)。

## <a name="test-in-browser"></a>在瀏覽器中測試

瀏覽至您稍早設定的 DNS 名稱 (`contoso.com`、`www.contoso.com`、`sub1.contoso.com` 和 `sub2.contoso.com`)。

![入口網站瀏覽至 Azure 應用程式](./media/app-service-web-tutorial-custom-domain/app-with-custom-dns.png)

## <a name="resolve-404-not-found"></a>解析 404「找不到」

如果您在瀏覽自訂網域 URL 時收到 HTTP 404 (找不到) 錯誤，請使用 <a href="https://www.whatsmydns.net/" target="_blank">WhatsmyDNS.net</a> 確認網域是否能解析成應用程式的 IP 位址。 如果不行，這可能是下列其中一個原因所造成：

- 設定的自訂網域遺漏 A 記錄和/或 CNAME 記錄。
- 瀏覽器用戶端已將網域的舊 IP 位址加入快取。 請清除快取並再次測試 DNS 解析。 在 Windows 電腦上，您可以使用 `ipconfig /flushdns` 清除快取。

<a name="virtualdir"></a>

## <a name="migrate-an-active-domain"></a>移轉作用中的網域

若要將即時網站及其 DNS 網域名稱遷移至 App Service 而不要停機，請參閱[將作用中的 DNS 名稱遷移至 Azure App Service](app-service-custom-domain-name-migrate.md)。

## <a name="redirect-to-a-custom-directory"></a>重新導向至自訂目錄

根據預設，App Service 會將 Web 要求導向應用程式程式碼的根目錄。 不過，某些 Web 架構並非從根目錄開始。 例如，[Laravel](https://laravel.com/) 從`public` 子目錄開始。 若要繼續 `contoso.com` DNS 範例，這類應用程式在 `http://contoso.com/public` 上可以存取，不過實際上您會想要將 `http://contoso.com` 改為導向 `public` 目錄。 這個步驟與 DNS 解析無關，反而是與自訂虛擬目錄相關。

若要引導應用程式，請在 Web 應用程式頁面左側的導覽中選取 [應用程式設定]。 

在頁面底部，根虛擬目錄 `/` 預設指向 `site\wwwroot`，這是應用程式程式碼的根目錄。 例如，請將其變更為指向 `site\wwwroot\public`，然後再儲存變更。 

![自訂虛擬目錄](./media/app-service-web-tutorial-custom-domain/customize-virtual-directory.png)

當作業完成時，應用程式應該會傳回位於根路徑 (如 http://contoso.com)) 的正確頁面。

## <a name="automate-with-scripts"></a>使用指令碼進行自動化

您可以使用 [Azure CLI](/cli/azure/install-azure-cli) 或 [Azure PowerShell](/powershell/azure/overview)，透過指令碼將自訂網域的管理作業自動化。 

### <a name="azure-cli"></a>Azure CLI 

下列命令會在 App Service 應用程式中新增所設定的自訂 DNS 名稱。 

```bash 
az webapp config hostname add \
    --webapp-name <app_name> \
    --resource-group <resource_group_name> \ 
    --hostname <fully_qualified_domain_name> 
``` 

如需詳細資訊，請參閱[將自訂網域對應至 Web 應用程式](scripts/app-service-cli-configure-custom-domain.md)。 

### <a name="azure-powershell"></a>Azure PowerShell 

下列命令會在 App Service 應用程式中新增所設定的自訂 DNS 名稱。 

```PowerShell  
Set-AzureRmWebApp `
    -Name <app_name> `
    -ResourceGroupName <resource_group_name> ` 
    -HostNames @("<fully_qualified_domain_name>","<app_name>.azurewebsites.net") 
```

如需詳細資訊，請參閱[將自訂網域指派給 Web 應用程式](scripts/app-service-powershell-configure-custom-domain.md)。

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何：

> [!div class="checklist"]
> * 使用 CNAME 記錄來對應子網域
> * 使用 A 記錄來對應根網域
> * 使用 CNAME 記錄來對應萬用字元網域
> * 將預設 URL 重新導向至自訂目錄
> * 使用指令碼來自動對應網域

前進到下一個教學課程，以了解如何將自訂 SSL 憑證繫結至 Web 應用程式。

> [!div class="nextstepaction"]
> [將現有的自訂 SSL 憑證繫結至 Azure Web Apps](app-service-web-tutorial-custom-ssl.md)
