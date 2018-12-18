---
title: Azure 反向 DNS 概觀 | Microsoft Docs
description: 了解反向 DNS 如何運作以及如何在 Azure 中使用
services: dns
documentationcenter: na
author: vhorne
manager: jeconnoc
ms.service: dns
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 05/29/2017
ms.author: victorh
ms.openlocfilehash: fa3798a35804998936e0ac166fceff02b01231a0
ms.sourcegitcommit: 4e5ac8a7fc5c17af68372f4597573210867d05df
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/20/2018
ms.locfileid: "39171505"
---
# <a name="overview-of-reverse-dns-and-support-in-azure"></a>Azure 反向 DNS 和支援概觀

本文提供反向 DNS 運作方式以及 Azure 支援之反向 DNS 案例的概觀。

## <a name="what-is-reverse-dns"></a>什麼是反向 DNS？

傳統 DNS 記錄可啟用從 DNS 名稱至 (例如 'www.contoso.com') IP 位址 (例如 64.4.6.100) 的對應。  反向 DNS 則可將 IP 位址 (64.4.6.100) 轉譯回名稱 ('www.contoso.com')。

反向 DNS 記錄使用於多種情況。 例如，透過驗證電子郵件訊息的寄件者，反向 DNS 記錄廣泛用於對抗垃圾電子郵件。  接收端郵件伺服器會擷取傳送端伺服器 IP 位址的反向 DNS 記錄，並確認該主機是否獲授權從原始網域傳送電子郵件。 

## <a name="how-reverse-dns-works"></a>反向 DNS 的運作方式

反向 DNS 記錄託管於特殊的 DNS 區域，稱為 'ARPA' 區域。  這些區域會與一般階層託管網域 (例如 'contoso.com') 同時形成個別的 DNS 階層。

例如，DNS 記錄 'www.contoso.com' 是使用 DNS 'A' 記錄名稱 'www' 在區域 'contoso.com' 中實作的。  此 A 記錄會指向對應的 IP 位址，在此情況下為 64.4.6.100。  反向對應會在區域 '6.4.64.in-addr.arpa' 中使用名為 '100' 的 'PTR' 記錄分別實作 (請注意，IP 位址會在 ARPA 區域中反轉)。此 PTR 記錄 (如果它已正確設定) 會指向名稱 'www.contoso.com'。

當組織獲指派 IP 位址區塊時，也會取得管理對應的 ARPA 區域的權限。 對應至 Azure 所使用 IP 位址區塊的 ARPA 區域會由 Microsoft 託管並管理。 您的 ISP 可能會為您裝載您自己 IP 位址的 ARPA 區域，或可能讓您在所選的 DNS 服務 (例如，Azure DNS) 中裝載 ARPA 區域。

> [!NOTE]
> 正向 DNS 對應與反向 DNS 對應會在個別的平行 DNS 階層中實作。 'www.contoso.com' 的反向對應**不是**在區域 'contoso.com' 中託管，而是在對應的 IP 位址區塊的 ARPA 區域中託管。 IPv4 和 IPv6 位址區塊使用不同的區域。

### <a name="ipv4"></a>IPv4

IPv4 反向對應區域的名稱格式應該如下：`<IPv4 network prefix in reverse order>.in-addr.arpa`。

例如，在建立反向區域託管 IP 前置詞為 192.0.2.0/24 的主機記錄時，會先取出位址的網路首碼 (192.0.2)，然後反轉順序 (2.0.192) 並加上尾碼 `.in-addr.arpa`，以這種方式建立區域名稱。

|子網路類別|網路首碼  |反轉的網路首碼  |標準後置詞  |反向區域名稱 |
|-------|----------------|------------|-----------------|---------------------------|
|類別 A|203.0.0.0/8     | 203        | .in-addr.arpa   | `203.in-addr.arpa`        |
|類別 B|198.51.0.0/16   | 51.198     | .in-addr.arpa   | `51.198.in-addr.arpa`     |
|類別 C|192.0.2.0/24    | 2.0.192    | .in-addr.arpa   | `2.0.192.in-addr.arpa`    |

### <a name="classless-ipv4-delegation"></a>無類別 IPv4 委派

在某些情況下，配置給組織的 IP 範圍會小於類別 C (/24) 範圍。 本例中，IP 範圍不是落在 `.in-addr.arpa` 區域階層的區域界限內，因此無法委派為子區域。

改用不同的機制將個別反向對應 (PTR) 記錄的控制權轉移到專用的 DNS 區域。 這項機制會為每個 IP 範圍委派子區域，然後分別將範圍中的每個 IP 位址對應至使用 CNAME 記錄的子區域。

例如，假設 ISP 授與組織的 IP 範圍是 192.0.2.128/26。 這代表 64 個 IP 位址，從 192.0.2.128 到 192.0.2.191。 此範圍的反向 DNS 實作如下：
- 組織建立名為 128-26.2.0.192.in-addr.arpa 的反向對應區域。 前置詞 '128-26' 代表指派給類別 C (/24) 範圍內組織的網路區段。
- ISP 會建立 NS 記錄，從類別 C 的上層區域設定上述區域的 DNS 委派。 它也會在上層 (類別 C) 反向對應區域中建立 CNAME 記錄，將 IP 範圍中的每個 IP 位址對應到組織建立的新區域：

```
$ORIGIN 2.0.192.in-addr.arpa
; Delegate child zone
128-26    NS       <name server 1 for 128-26.2.0.192.in-addr.arpa>
128-26    NS       <name server 2 for 128-26.2.0.192.in-addr.arpa>
; CNAME records for each IP address
129       CNAME    129.128-26.2.0.192.in-addr.arpa
130       CNAME    130.128-26.2.0.192.in-addr.arpa
131       CNAME    131.128-26.2.0.192.in-addr.arpa
; etc
```
- 然後組織管理自己子區域內的個別 PTR 記錄。

```
$ORIGIN 128-26.2.0.192.in-addr.arpa
; PTR records for each UIP address. Names match CNAME targets in parent zone
129      PTR    www.contoso.com
130      PTR    mail.contoso.com
131      PTR    partners.contoso.com
; etc
```
IP 位址 '192.0.2.129' 的反向對應會查詢名為 '129.2.0.192.in-addr.arpa' 的 PTR 記錄。 此查詢透過上層區域的 CNAME 解析成子區域的 PTR 記錄。

### <a name="ipv6"></a>IPv6

IPv6 反向對應區域的名稱格式應該如下：`<IPv6 network prefix in reverse order>.ip6.arpa`。

例如： 在建立反向區域託管 IP 前置詞為 2001:db8:1000:abdc::/64 的主機記錄時，會隔離位址的網路首碼 (2001:db8:abdc::) 來建立區域名稱。 接著展開 IPv6 網路首碼以移除[零壓縮](https://technet.microsoft.com/library/cc781672(v=ws.10).aspx)，如果它用來縮短 IPv6 位址首碼 (2001:0db8:abdc:0000::)。 反轉順序，前置詞中每個十六進位數字之間使用句點為分隔符號，建置反轉的網路前置詞 (`0.0.0.0.c.d.b.a.8.b.d.0.1.0.0.2`)，再加上尾碼 `.ip6.arpa`。


|網路首碼  |已展開並反轉的網路首碼 |標準後置詞 |反向區域名稱  |
|---------|---------|---------|---------|
|2001:db8:abdc::/64    | 0.0.0.0.c.d.b.a.8.b.d.0.1.0.0.2        | .ip6.arpa        | `0.0.0.0.c.d.b.a.8.b.d.0.1.0.0.2.ip6.arpa`       |
|2001:db8:1000:9102::/64    | 2.0.1.9.0.0.0.1.8.b.d.0.1.0.0.2        | .ip6.arpa        | `2.0.1.9.0.0.0.1.8.b.d.0.1.0.0.2.ip6.arpa`        |


## <a name="azure-support-for-reverse-dns"></a>Azure 的反向 DNS 支援

Azure 支援與反向 DNS 相關的兩種不同案例：

**託管對應至您 IP 位址區塊的反轉對應區域。**
Azure DNS 可以用來[託管反轉對應區域和管理每個反向 DNS 對應的 PTR 記錄](dns-reverse-dns-hosting.md)，IPv4 及 IPv6 皆可。  建立反轉對應 (ARPA) 區域、設定委派，以及設定 PTR 記錄的程序與一般的 DNS 區域相同。  唯一的差異是必須透過您的 ISP 設定委派，而不是 DNS 註冊機構，而且僅應該使用 PTR 記錄類型。

**設定指派給 Azure 服務之 IP 位址的反向 DNS 記錄**。 [Azure 可讓您設定配置給 Azure 服務之 IP 位址的反向對應](dns-reverse-dns-for-azure-services.md)。  此反向對應是由 Azure 設定作為對應 ARPA 區域中的 PTR 記錄。  對應至 Azure 所使用 IP 位址範圍的這些 ARPA 區域會由 Microsoft 託管

## <a name="next-steps"></a>後續步驟

如需反向 DNS 的詳細資訊，請參閱維基百科的 [reverse DNS lookup](http://en.wikipedia.org/wiki/Reverse_DNS_lookup) (反向 DNS 對應)。
<br>
了解如何[在 Azure DNS 中為您 ISP 指派的 IP 範圍託管反向對應區域](dns-reverse-dns-for-azure-services.md)。
<br>
了解如何[管理 Azure 服務的反向 DNS 記錄](dns-reverse-dns-for-azure-services.md)。

