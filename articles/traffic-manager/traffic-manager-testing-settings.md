---
title: 驗證 Azure 流量管理員設定 | Microsoft Docs
description: 此文章將協助您驗證流量管理員設定
services: traffic-manager
documentationcenter: ''
author: kumudd
manager: timlt
editor: ''
ms.assetid: 2180b640-596e-4fb2-be59-23a38d606d12
ms.service: traffic-manager
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/16/2017
ms.author: kumud
ms.openlocfilehash: aadff1806a7cb22347283143563467366e857569
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "23110716"
---
# <a name="verify-traffic-manager-settings"></a>驗證流量管理員設定

若要測試您的流量管理員設定，您需要有在不同位置的多個用戶端，讓您可以從中執行測試。 然後，在流量管理員設定檔中一次關閉一個端點。

* 設定低的 DNS TTL，方便快速傳播變更，例如 30 秒。
* 知道正在測試的設定檔中您 Azure 雲端服務和網站的 IP 位址。
* 使用可讓您解析 IP 位址之 DNS 名稱的工具並顯示該位址。

您要查看設定檔中 DNS 名稱是否解析成端點的 IP 位址。 名稱的解析方法應與流量管理員設定檔中定義的流量路由方法一致。 您可以使用像是 **nslookup** 或 **dig** 等工具來解析 DNS 名稱。

下列範例可協助您測試流量管理員設定檔。

### <a name="check-traffic-manager-profile-using-nslookup-and-ipconfig-in-windows"></a>使用 Windows 中的 nslookup 和 ipconfig 檢查流量管理員設定檔

1. 以系統管理員的身分開啟命令或 Windows PowerShell 命令提示字元。
2. 輸入 `ipconfig /flushdns` 以排清 DNS 解析程式快取。
3. 輸入 `nslookup <your Traffic Manager domain name>`。 例如，下列命令會檢查包含 myapp.contoso 前置詞的網域名稱

        nslookup myapp.contoso.trafficmanager.net

    典型的結果會顯示下列資訊：

    + 正在存取 DNS 伺服器的 DNS 名稱和 IP 位址，藉此解析此流量管理員網域名稱。
    + 您在命令行上輸入在 "nslookup" 之後的流量管理員網域名稱，和流量管理員網域名稱解析的 IP 位址。 第二個 IP 位址是要檢查的重點。 它應符合正在測試流量管理員設定檔中其中一個雲端服務或網站的 虛擬 IP (VIP)。

## <a name="how-to-test-the-failover-traffic-routing-method"></a>如何測試容錯移轉流量路由方法

1. 保持所有端點運作。
2. 在單一用戶端，使用 nslookup 或類似的公用程式要求公司網域名稱的 DNS 解析。
3. 確定解析後的 IP 位址與您的主要端點相符。
4. 關閉您的主要端點或移除監視檔案，所以流量管理員會認為應用程式已關閉。
5. 等待時間為流量管理員設定檔的 DNS 存留時間 (TTL) 再加上 2 分鐘。 例如，若您的 DNS TTL 為 300 秒 (5 分鐘)，則您必須等待 7 分鐘。
6. 排清您的 DNS 用戶端快取，使用 nslookup 要求 DNS 解析。 在 Windows 中，您可以使用 ipconfig/flushdns 命令排清 DNS 快取。
7. 確定解析後的 IP 位址與您的次要端點相符。
8. 重複此程序，依序關閉每個端點。 請確定 DNS 傳回清單中下一個端點的 IP 位址。 關閉所有端點之後，您應可再次取得主要端點的 IP 位址。

## <a name="how-to-test-the-weighted-traffic-routing-method"></a>如何測試加權流量路由方法

1. 保持所有端點運作。
2. 在單一用戶端，使用 nslookup 或類似的公用程式要求公司網域名稱的 DNS 解析。
3. 確定解析後的 IP 位址與您的端點之一相符。
4. 排清 DNS 用戶端快取，然後對每個端點重複步驟 2 和 3。 您應該可看到為每個端點傳回不同的 IP 位址。

## <a name="how-to-test-the-performance-traffic-routing-method"></a>如何測試效能流量路由方法

若要有效地測試效能流量路由方法，您必須有個在位於世界上不同位置的用戶端。 您可以在不同 Azure 區域中建立用戶端以用來測試您的服務。 如果您有全域網路，可以從世界上的其他地方遠端登入用戶端，並從該處執行測試。

或者，您可以找到免費的 Web 型 DNS 查閱和挖掘服務。 其中一些工具提供您從全球不同位置檢查 DNS 名稱解析的能力。 例如，搜尋 "DNS lookup" 相關資料。 協力廠商服務 (如 Gomez、Keynote) 可用於確認您的設定檔正如預期般分配流量。

## <a name="next-steps"></a>後續步驟

* [關於流量管理員流量路由方法](traffic-manager-routing-methods.md)
* [流量管理員的效能考量](traffic-manager-performance-considerations.md)
* [疑難排解流量管理員的已降級狀態](traffic-manager-troubleshooting-degraded.md)
