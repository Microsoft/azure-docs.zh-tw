---
title: 指定服務組態檔中的 DNS 設定 | Microsoft Docs
description: 使用虛擬網路的服務組態檔指定自訂的 DNS 設定
services: virtual-network
documentationcenter: na
author: genlin
manager: cshepard
editor: tysonn
ms.assetid: 467a4b99-8691-40b3-b640-e25e49675c71
ms.service: virtual-network
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/24/2016
ms.author: genli
ms.openlocfilehash: 009206f1e0ba848538ed2c666032a63051d062e4
ms.sourcegitcommit: fa493b66552af11260db48d89e3ddfcdcb5e3152
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/23/2018
ms.locfileid: "31790741"
---
# <a name="specifying-dns-settings-in-a-service-configuration-file"></a>指定服務組態檔中的 DNS 設定
## <a name="dns-elements"></a>DNS 項目
服務組態檔可能包含 DnsServers 項目，以及服務會使用的網域名稱系統 (DNS) 伺服器的 IPv4 位址清單。 服務組態檔中的設定優先於網路組態檔中的設定。 如需詳細資訊，請參閱 [Azure 服務組態結構描述 (.cscfg 檔)](https://msdn.microsoft.com/library/azure/ee758710.aspx)。

**NetworkConfiguration 項目**

      <DnsServers>
        <DnsServer name="ID1" IPAddress="IPAddress1" />
        <DnsServer name="ID2" IPAddress="IPAddress2" />
        <DnsServer name="ID3" IPAddress="IPAddress3" />
      </DnsServers>

> [!WARNING]
> **DnsServer** 項目中的 **name** 屬性只做為參考名稱。 它不代表 DNS 伺服器的主機名稱。 每個 **DnsServer** 屬性值在整個 Microsoft Azure 訂用帳戶中必須是唯一的。
> 
> 

## <a name="see-also"></a>另請參閱
[Azure 服務組態結構描述 (.cscfg)](https://msdn.microsoft.com/library/windowsazure/ee758710)

[Azure 虛擬網路組態結構描述](http://go.microsoft.com/fwlink/?LinkId=248093)

[使用網路組態檔設定虛擬網路](http://go.microsoft.com/fwlink/?LinkId=248094)

[關於管理入口網站中的虛擬網路設定](http://go.microsoft.com/fwlink/?LinkId=248092)

