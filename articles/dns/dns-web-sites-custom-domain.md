---
title: 教學課程 - 建立 Web 應用程式的自訂 Azure DNS 記錄
description: 在本教學課程中，您會使用 Azure DNS 來建立 Web 應用程式的自訂網域 DNS 記錄。
services: dns
author: vhorne
ms.service: dns
ms.topic: tutorial
ms.date: 7/20/2018
ms.author: victorh
ms.openlocfilehash: b39c2c672869bb446e58134a85130d10491fe047
ms.sourcegitcommit: 35ceadc616f09dd3c88377a7f6f4d068e23cceec
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/08/2018
ms.locfileid: "39621108"
---
# <a name="tutorial-create-dns-records-in-a-custom-domain-for-a-web-app"></a>教學課程：在自訂網域中建立 Web 應用程式的 DNS 記錄 

您可以設定 Azure DNS 來裝載 Web 應用程式的自訂網域。 例如，您可以建立一個 Azure Web 應用程式，並讓使用者使用 www.contoso.com 或 contoso.com 作為完整網域名稱 (FQDN) 存取該應用程式。

> [!NOTE]
> 本教學課程會使用 Contoso.com 作為範例。 用您自己的網域名稱來取代 contoso.com。

若要這樣做，您必須建立三筆記錄︰

* 指向 contoso.com 的根 "A" 記錄
* 要進行驗證的根 "TXT" 記錄
* 指向 A 記錄之 www 名稱的 "CNAME" 記錄

請記住，如果您在 Azure 中建立 Web 應用程式的 A 記錄，如果 Web 應用程式的基礎 IP 位址變更，則您必須手動更新 A 記錄。

在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 建立自訂網域的 A 記錄和 TXT 記錄
> * 建立自訂網域的 CNAME 記錄
> * 測試新記錄
> * 新增 Web 應用程式的自訂主機名稱
> * 測試自訂主機名稱


如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

[!INCLUDE [cloud-shell-powershell.md](../../includes/cloud-shell-powershell.md)]

## <a name="prerequisites"></a>必要條件

- [建立 App Service 應用程式](../app-service/app-service-web-get-started-html.md)，或使用您針對另一個教學課程建立的應用程式。

- 在 Azure DNS 中建立 DNS 區域，並將註冊機構中的區域委派給 Azure DNS。

   1. 若要建立 DNS 區域，請依照 [建立 DNS 區域](dns-getstarted-create-dnszone.md)的步驟進行。
   2. 若要將區域委派給 Azure DNS，請依照 [DNS 網域委派](dns-domain-delegation.md)中的步驟進行。

建立區域並委派給 Azure DNS 之後，便可以為您的自訂網域建立記錄。

## <a name="create-an-a-record-and-txt-record"></a>建立 A 記錄和 TXT 記錄

A 記錄可用來將名稱對應到其 IP 位址。 在下列範例中，我們會使用您的 Web 應用程式 IPv4 位址將 「\@」指派為 A 記錄。 \@ 通常代表根網域。

### <a name="get-the-ipv4-address"></a>取得 IPV4 位址

在 Azure 入口網站的應用程式服務頁面左側導覽中，選取 [自訂網域]。 

![[自訂網域] 功能表](../app-service/./media/app-service-web-tutorial-custom-domain/custom-domain-menu.png)

在 [自訂網域] 頁面中，複製應用程式的 IPv4 位址：

![入口網站瀏覽至 Azure 應用程式](../app-service/./media/app-service-web-tutorial-custom-domain/mapping-information.png)

### <a name="create-the-a-record"></a>建立 A 記錄

```powershell
New-AzureRMDnsRecordSet -Name "@" -RecordType "A" -ZoneName "contoso.com" `
 -ResourceGroupName "MyAzureResourceGroup" -Ttl 600 `
 -DnsRecords (New-AzureRmDnsRecordConfig -IPv4Address "<your web app IP address>")
```

### <a name="create-the-txt-record"></a>建立 TXT 記錄

應用程式服務只會在設定時使用此記錄，以確認您擁有自訂網域。 系統驗證您的自訂網域並在 App Service 中設定之後，您就可以刪除此 TXT 記錄。

```powershell
New-AzureRMDnsRecordSet -ZoneName contoso.com -ResourceGroupName MyAzureResourceGroup `
 -Name `"@" -RecordType "txt" -Ttl 600 `
 -DnsRecords (New-AzureRmDnsRecordConfig -Value  "contoso.azurewebsites.net")
```

## <a name="create-the-cname-record"></a>建立 CNAME 記錄

如果您的網域已受 Azure DNS 管理 (請參閱 [DNS 網域委派](dns-domain-delegation.md))，您可以使用下列範例，建立 contoso.azurewebsites.net 的 CNAME 記錄。

開啟 Azure PowerShell 並建立新的 CNAME 記錄。 此範例會建立一個記錄集類型 CNAME，並在 DNS 區域中建立一個「存留期」為 600 秒的 "contoso.com"，其 Web 應用程式的別名為 contoso.azurewebsites.net。

### <a name="create-the-record"></a>建立記錄

```powershell
New-AzureRMDnsRecordSet -ZoneName contoso.com -ResourceGroupName "MyAzureResourceGroup" `
 -Name "www" -RecordType "CNAME" -Ttl 600 `
 -DnsRecords (New-AzureRmDnsRecordConfig -cname "contoso.azurewebsites.net")
```

以下是回應範例：

```
    Name              : www
    ZoneName          : contoso.com
    ResourceGroupName : myresourcegroup
    Ttl               : 600
    Etag              : 8baceeb9-4c2c-4608-a22c-229923ee185
    RecordType        : CNAME
    Records           : {contoso.azurewebsites.net}
    Tags              : {}
```

## <a name="test-the-new-records"></a>測試新記錄

如下所示，您可以使用 nslookup 來查詢 "www.contoso.com" 和 “contoso.com”，驗證所建立的記錄正確無誤：

```
PS C:\> nslookup
Default Server:  Default
Address:  192.168.0.1

> www.contoso.com
Server:  default server
Address:  192.168.0.1

Non-authoritative answer:
Name:    <instance of web app service>.cloudapp.net
Address:  <ip of web app service>
Aliases:  www.contoso.com
contoso.azurewebsites.net
<instance of web app service>.vip.azurewebsites.windows.net

> contoso.com
Server:  default server
Address:  192.168.0.1

Non-authoritative answer:
Name:    contoso.com
Address:  <ip of web app service>

> set type=txt
> contoso.com

Server:  default server
Address:  192.168.0.1

Non-authoritative answer:
contoso.com text =

        "contoso.azurewebsites.net"
```
## <a name="add-custom-host-names"></a>新增自訂主機名稱

現在您可以將自訂主機名稱新增至 Web 應用程式：

```powershell
set-AzureRmWebApp `
 -Name contoso `
 -ResourceGroupName MyAzureResourceGroup `
 -HostNames @("contoso.com","www.contoso.com","contoso.azurewebsites.net")
```
## <a name="test-the-custom-host-names"></a>測試自訂主機名稱

請開啟網頁瀏覽器，然後瀏覽至 `http://www.<your domainname>` 和 `http://<you domain name>`。

> [!NOTE]
> 請確定您納入了 `http://` 前置詞，否則瀏覽器可能會嘗試為您預測 URL！

這兩個 URL 應該會導向相同的頁面。 例如︰

![Contoso 應用程式服務](media/dns-web-sites-custom-domain/contoso-app-svc.png)


## <a name="clean-up-resources"></a>清除資源

若不再需要本教學課程中建立的資源，請刪除 **MyAzureResourceGroup** 資源群組。

## <a name="next-steps"></a>後續步驟

了解如何建立 Azure DNS 私人區域。

> [!div class="nextstepaction"]
> [利用 PowerShell 開始使用 Azure DNS 私人區域](private-dns-getstarted-powershell.md)
