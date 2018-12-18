---
title: Azure 中的 IP 位址 中的 IP 位址
description: 了解如何尋找函式應用程式的輸入和輸出 IP 位址，以及造成其變更的原因。
services: functions
documentationcenter: ''
author: ggailey777
manager: cfowler
editor: ''
ms.service: functions
ms.workload: na
ms.devlang: na
ms.topic: article
ms.date: 07/18/2018
ms.author: glenga
ms.openlocfilehash: ce520c5972bf27c30ecb175d58b6b6071bf804db
ms.sourcegitcommit: 58c5cd866ade5aac4354ea1fe8705cee2b50ba9f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/24/2018
ms.locfileid: "42818287"
---
# <a name="ip-addresses-in-azure-functions"></a>Azure 中的 IP 位址 中的 IP 位址

本文說明下列與函式應用程式 IP 位址相關的主題：

* 如何尋找函式應用程式目前使用的 IP 位址。
* 什麼原因導致函式應用程式的 IP 位址變更。
* 如何限制可以存取函式應用程式的 IP 位址。
* 如何取得函式應用程式專用的 IP 位址。

IP 位址與函式應用程式相關聯，而非與個別函式相關聯。 傳入 HTTP 要求無法使用輸入 IP 位址來呼叫個別的函式；其必須使用預設網域名稱 (functionappname.azurewebsites.net) 或自訂網域名稱。

## <a name="function-app-inbound-ip-address"></a>函式應用程式輸入 IP 位址

每個函式應用程式都有單一輸入 IP 位址。 若要尋找該 IP 位址：

1. 登入 [Azure 入口網站](https://portal.azure.com)。
2. 巡覽至函式應用程式。
3. 選取 [平台功能]。
4. 選取 [屬性]，以及 [虛擬 IP 位址] 之下的輸入 IP 位址。

## <a name="find-outbound-ip-addresses"></a>函式應用程式輸出 IP 位址

每個函式應用程式都有一組可用的輸出 IP 位址。 任何來自應用程式的輸出連線 (例如連往後端資料庫) 都會使用其中一個可用的輸出 IP 位址作為來源 IP 位址。 您事先不知道指定的連線會使用哪個 IP 位址。 基於這個理由，您的後端服務必須對函式應用程式的所有輸出 IP 位址開啟其防火牆。

若要找出函式應用程式可用的輸出 IP 位址：

1. 登入 [Azure 資源總管](https://resources.azure.com)。
2. 選取 **訂用帳戶 > {您的訂用帳戶} > 提供者 > Microsoft.Web > 網站**。
3. 在 JSON 面板中，尋找 `id` 屬性結尾是函式應用程式名稱的網站。
4. 請參閱 `outboundIpAddresses` 和 `possibleOutboundIpAddresses`。 

函式應用程式目前可用的一組 `outboundIpAddresses`。 這組 `possibleOutboundIpAddresses` 包含只有當函式應用程式[調整為其他定價層](#outbound-ip-address-changes)時才可用的 IP 位址。

尋找可用輸出 IP 位址的替代方式是使用 [Cloud Shell](../cloud-shell/quickstart.md)：

```azurecli-interactive
az webapp show --resource-group <group_name> --name <app_name> --query outboundIpAddresses --output tsv
az webapp show --resource-group <group_name> --name <app_name> --query possibleOutboundIpAddresses --output tsv
```

## <a name="data-center-outbound-ip-addresses"></a>資料中心輸出 IP 位址

如果您需要將函式應用程式所用的輸出 IP 位址列入允許清單，另一個選項是將函式應用程式的資料中心 (Azure 區域) 列入允許清單。 您可以[下載 XML 檔案，其中列出所有 Azure 資料中心的 IP 位址](https://www.microsoft.com/en-us/download/details.aspx?id=41653)。 然後尋找適用於函式應用程式執行所在區域的 XML 元素。

例如，西歐 XML 元素可能如下所示：

```
  <Region Name="europewest">
    <IpRange Subnet="13.69.0.0/17" />
    <IpRange Subnet="13.73.128.0/18" />
    <!-- Some IP addresses not shown here -->
    <IpRange Subnet="213.199.180.192/27" />
    <IpRange Subnet="213.199.183.0/24" />
  </Region>
```

 如需有關此檔案何時更新和 IP 位址何時變更的資訊，請展開[下載中心頁面](https://www.microsoft.com/en-us/download/details.aspx?id=41653)的 [詳細資料] 區段。

## <a name="inbound-ip-address-changes"></a>輸入 IP 位址變更

 當您執行下列動作時，輸入 IP 位址**可能**會變更：

- 刪除函式應用程式，並在不同資源群組中重建。
- 刪除資源群組和區域組合中的最後一個函式應用程式，並予以重建。
- 刪除 SSL 繫結，例如在[憑證更新](../app-service/app-service-web-tutorial-custom-ssl.md#renew-certificates)期間。

如果您尚未採取任何動作 (例如所列的動作)，輸入 IP 位址也可能會改變。

## <a name="outbound-ip-address-changes"></a>輸出 IP 位址變更

當您執行下列動作時，函式應用程式的這組可用輸出 IP 位址可能會變更：

* 採取任何可變更輸入 IP 位址的動作。
* 變更您的 App Service 方案定價層。 您的應用程式可使用的所有可能輸出 IP 位址清單 (適用於所有定價層) 位於 `possibleOutboundIPAddresses` 屬性中。 請參閱[尋找輸出 IP](#find-outbound-ip-addresses)。

如果您尚未採取任何動作 (例如所列的動作)，輸入 IP 位址也可能會改變。

若要故意強制輸出 IP 位址變更：

1. 在標準與進階 v2 定價層之間，將您的 App Service 方案調升或調降。
2. 等候 10 分鐘。
3. 調整回到您開始的位置。

## <a name="ip-address-restrictions"></a>IP 位址限制

您可以設定一份 IP 位址清單，其中包含您要允許或拒絕存取函式應用程式的 IP 位址。 如需詳細資訊，請參閱 [Azure App Service 靜態 IP 限制](../app-service/app-service-ip-restrictions.md)。

## <a name="dedicated-ip-addresses"></a>專用的 IP 位址

如果您需要靜態的專用 IP 位址，我們建議使用 [App Service 環境](../app-service/environment/intro.md) (App Service 方案的[隔離層](https://azure.microsoft.com/pricing/details/app-service/))。 如需詳細資訊，請參閱 [App Service 環境 IP 位址](../app-service/environment/network-info.md#ase-ip-addresses)和[如何控制對 App Service 環境的輸入流量](../app-service/environment/app-service-app-service-environment-control-inbound-traffic.md)。

若要找出您的函式應用程式是否在 App Service 環境中執行：

1. 登入 [Azure 入口網站](https://portal.azure.com)。
2. 巡覽至函式應用程式。
3. 選取 [概觀] 索引標籤。
4. App Service 方案層會出現在 **App Service 方案/定價層**之下。 App Service 環境定價層為 [隔離]。
 
或者，您可以使用 [Cloud Shell](../cloud-shell/quickstart.md)：

```azurecli-interactive
az webapp show --resource-group <group_name> --name <app_name> --query sku --output tsv
```

App Service 環境 `sku` 為 `Isolated`。

## <a name="next-steps"></a>後續步驟

IP 變更的常見原因是函式應用程式級別變更。 [深入了解函式應用程式調整](functions-scale.md)。
