---
title: Azure DNS 疑難排解指南 | Microsoft Docs
description: 如何對使用 Azure DNS 的常見問題進行疑難排解
services: dns
documentationcenter: na
author: genlin
manager: cshepard
editor: ''
ms.assetid: 95b01dc3-ee69-4575-a259-4227131e4f9c
ms.service: dns
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 01/20/2017
ms.author: genli
ms.openlocfilehash: 816ad6c0079e9272286cdc072ff71bf15313eb8e
ms.sourcegitcommit: c52123364e2ba086722bc860f2972642115316ef
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/11/2018
ms.locfileid: "34069094"
---
# <a name="azure-dns-troubleshooting-guide"></a>Azure DNS 疑難排解指南

此頁面提供 Azure DNS 常見問題的疑難排解資訊。

如果這些步驟未能解決問題，您也可以在 [MSDN 上的社群支援論壇](https://social.msdn.microsoft.com/Forums/en-US/home?forum=WAVirtualMachinesVirtualNetwork)搜尋或張貼您的問題。 或者，開啟 Azure 支援要求。


## <a name="i-cant-create-a-dns-zone"></a>我無法建立 DNS 區域

若要解決常見的問題，請嘗試下列步驟：

1.  檢閱 Azure DNS 稽核記錄檔以判斷失敗原因。
2.  每個 DNS 區域名稱都必須是其資源群組中唯一的。 也就是說，兩個名稱相同的 DNS 區域無法共用一個資源群組。 請嘗試使用不同的區域名稱，或不同的資源群組。
3.  您會看到「您已達到或超過訂用帳戶 {subscription id} 中的區域數目上限。」的錯誤訊息。 請使用其他 Azure 訂用帳戶、刪除相同的區域或聯絡 Azure 支援中心以提高您的訂用帳戶限制。
4.  您會看到「區域 '{zone name}' 無法使用。」的錯誤訊息。 此錯誤表示 Azure DNS 無法為此 DNS 區域配置名稱伺服器。 請嘗試使用不同的區域名稱。 或者，如果您是網域名稱擁有者，請連絡 Azure 支援中心，他們可以為您配置名稱伺服器。


### <a name="recommended-documents"></a>**建議的文件**

[DNS 區域和記錄](dns-zones-records.md)
<br>
[建立 DNS 區域](dns-getstarted-create-dnszone-portal.md)

## <a name="i-cant-create-a-dns-record"></a>我無法建立 DNS 記錄

若要解決常見的問題，請嘗試下列步驟：

1.  檢閱 Azure DNS 稽核記錄檔以判斷失敗原因。
2.  記錄集是否已經存在？  Azure DNS 是使用記錄*集*管理記錄，記錄集是名稱與類型相同之記錄的集合。 如果已經有名稱與類型相同的記錄存在，此時若要新增其他此類型記錄，您應該編輯現有的記錄集。
3.  您是否正嘗試在 DNS 區域頂點 (區域的 [根]) 建立記錄？ 如果是，DNS 慣例是使用 ‘@’ 字元做為記錄名稱。 也請注意，DNS 標準不允許在區域頂點使用 CNAME 記錄。
4.  您有 CNAME 衝突嗎？  DNS 標準不允許 CNAME 記錄的名稱和任何其他類型記錄的名稱相同。 如果您有現有的 CNAME，建立不同類型但名稱相同的記錄將會失敗。  同樣地，如果名稱和不同類型的現有記錄相同，建立 CNAME 將會失敗。 請透過移除其他記錄或選擇不同的記錄名稱來排除衝突。
5.  您已達到 DNS 區域中允許的記錄集數目限制嗎？ Azure 入口網站中，區域的 [內容] 底下會顯示目前的記錄集數目和記錄集數目上限。 如果您已經達到此限制，請刪除部分記錄集或連絡 Azure 支援中心以提高您在此區域的記錄集上限，然後再試一次。 


### <a name="recommended-documents"></a>**建議的文件**

[DNS 區域和記錄](dns-zones-records.md)
<br>
[建立 DNS 區域](dns-getstarted-create-dnszone-portal.md)



## <a name="i-cant-resolve-my-dns-record"></a>我無法解析我的 DNS 記錄

DNS 名稱解析是多步驟程序，可能會因為許多原因而失敗。 下面的步驟將可協助您調查 Azure DNS 中裝載之區域中 DNS 記錄的 DNS 解析為何會失敗。

1.  確認已經在 Azure DNS 中正確設定 DNS 記錄。 在 Azure 入口網站中檢閱 DNS 記錄、檢查區域名稱、記錄名稱及記錄類型是否正確。
2.  確認 DNS 記錄已在 Azure DNS 名稱伺服器上正確解析。
    - 如果您從本機電腦執行 DNS 查詢，您可能會看到沒有反映名稱伺服器目前狀態的快取結果。  而且，公司網路通常是使用會防止 DNS 查詢被導向到特定名稱伺服器的 DNS Proxy 伺服器。  為避免發生這些問題，請使用網路型名稱解析服務，例如 [digwebinterface](http://digwebinterface.com)。
    - 請務必為您的 DNS 區域指定正確的名稱伺服器，如 Azure 入口網站中所顯示的。
    - 檢查 DNS 名稱是否正確 (您將必須指定包括區域名稱在內的完整名稱) 和記錄類型是否正確
3.  確認 DNS 網域名稱已經正確[委派給 Azure DNS 名稱伺服器](dns-domain-delegation.md)。 有[許多第三方網站可提供 DNS 委派驗證](https://www.bing.com/search?q=dns+check+tool)。 這是一個*區域*委派測試，所以您應該只輸入 DNS 區域名稱，而不是完整的記錄名稱。
4.  完成上述步驟之後，現在應該可以正確解析您的 DNS 記錄。 若要確認，您可以再使用一次 [digwebinterface](http://digwebinterface.com)，這一次使用預設名稱伺服器設定。


### <a name="recommended-documents"></a>**建議的文件**

[將網域委派給 Azure DNS](dns-domain-delegation.md)



## <a name="how-do-i-specify-the-service-and-protocol-for-an-srv-record"></a>如何指定 SRV 記錄的 [服務] 和 [通訊協定]？

Azure DNS 是以記錄集的方式管理 DNS 記錄—記錄集是名稱與類型相同之記錄的集合。 對於 SRV 記錄集，需要在記錄集名稱中指定 [服務] 和 [通訊協定]。 系統已經為記錄集內每筆記錄個別指定其他 SRV 參數 ([優先順序]、[權數]、[連接埠] 及 [目標])。

例如，SRV 記錄名稱 (服務名稱 'sip'，通訊協定 'tcp')：

- \_sip.\_tcp (在區域頂點建立記錄集)
- \_sip.\_tcp.sipservice (建立名為 'sipservice' 的記錄集)

### <a name="recommended-documents"></a>**建議的文件**

[DNS 區域和記錄](dns-zones-records.md)
<br>
[使用 Azure 入口網站建立 DNS 記錄集和記錄](dns-getstarted-create-recordset-portal.md)
<br>
[SRV 記錄類型 (Wikipedia)](https://en.wikipedia.org/wiki/SRV_record) \(英文\)


## <a name="next-steps"></a>後續步驟

* 了解 [DNS 區域和記錄](dns-zones-records.md)
* 若要開始使用 Azure DNS，請學習如何[建立 DNS 區域](dns-getstarted-create-dnszone-portal.md)和[建立 DNS 記錄](dns-getstarted-create-recordset-portal.md)。
* 若要移轉現有的 DNS 區域，請學習如何[匯入和匯出 DNS 區域檔案](dns-import-export.md)。

