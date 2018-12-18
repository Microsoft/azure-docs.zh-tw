---
title: 在 Azure 中設定反向對應區域用於 SMTP Banner 檢查 | Microsoft Docs
description: 說明如何在 Azure 中設定反向對應區域用於 SMTP Banner 檢查
services: virtual-network
documentationcenter: virtual-network
author: genlin
manager: WillChen
editor: ''
tags: azure-resource-manager
ms.service: virtual-network
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: virtual-network
ms.workload: infrastructure
ms.date: 02/06/2018
ms.author: genli
ms.custom: ''
ms.openlocfilehash: 1e95b00ea08105238a860265e46275c24ed7bfbd
ms.sourcegitcommit: 4723859f545bccc38a515192cf86dcf7ba0c0a67
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 02/11/2018
ms.locfileid: "29151815"
---
#  <a name="configure-reverse-lookup-zones-for-an-smtp-banner-check"></a>設定反向對應區域用於 SMTP Banner 檢查

本文說明如何在 Azure DNS 中使用反向對應區域，以及建立反向 DNS (PTR) 記錄以進行 SMTP Banner 檢查。 

## <a name="symptom"></a>徵狀

如果您在 Microsoft Azure 中裝載 SMTP 伺服器，從遠端郵件伺服器傳送或接收郵件時，可能會收到下列錯誤訊息：

**554：沒有 PTR 記錄** 

## <a name="solution"></a>解決方法

對於 Azure 中的虛擬 IP 位址，反向記錄建立在 Microsoft 擁有的網域區域中，而不是自訂網域區域。

若要設定Microsoft 所擁有區域中的 PTR 記錄，請使用 PublicIpAddress 資源上的 -ReverseFqdn 屬性。 如需詳細資訊，請參閱[為 Azure 裝載的服務設定反向 DNS](../dns/dns-reverse-dns-for-azure-services.md)。 

當您設定 PTR 記錄時，請確定訂用帳戶擁有 IP 位址和反向 FQDN。 如果您嘗試設定不屬於訂用帳戶的反向 FQDN，您會收到下列錯誤訊息：

    Set-AzureRmPublicIpAddress : ReverseFqdn mail.contoso.com that PublicIPAddress ip01 is trying to use does not belong to subscription <Subscription ID>. One of the following conditions need to be met to establish ownership: 
                        
    1) ReverseFqdn 符合訂用帳戶下之任何公用 IP 資源的 FQDN; 
    2) ReverseFqdn 可解析成訂用帳戶下任一公用 IP 資源的 FQDN (透過 CName 記錄鏈); 
    3) ReverseFqdn 可解析成訂用帳戶下之靜態公用 IP 資源的 IP 位址 (透過 CName 及 A 記錄鏈)。 

如果您手動變更 SMTP Banner 以符合預設的反向 FQDN，遠端郵件伺服器仍然可能失敗，因為它期望 SMTP Banner 主機符合網域的 MX 記錄。