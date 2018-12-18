---
title: 針對來自 Mac OS X 用戶端的 Azure 點對站 VPN 連線問題進行疑難排解| Microsoft Docs
description: P2S Mac OS X VPN 用戶端連線問題的疑難排解步驟
services: vpn-gateway
documentationcenter: na
author: anzaman
manager: rossort
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: vpn-gateway
ms.devlang: na
ms.topic: troubleshooting
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/27/2018
ms.author: alzam
ms.openlocfilehash: 1cf8195cbf65f27c71a4db18c0c61c8a25673acd
ms.sourcegitcommit: d74657d1926467210454f58970c45b2fd3ca088d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/28/2018
ms.locfileid: "30247707"
---
# <a name="troubleshoot-point-to-site-vpn-connections-from-mac-os-x-vpn-clients"></a>針對來自 Mac OS X VPN 用戶端的點對站 VPN 連線問題進行疑難排解

本文將使用原生 VPN 用戶端和 IKEv2 來協助您針對來自 Mac OS X 的「點對站」連線問題進行疑難排解。 Mac 中 IKEv2 的 VPN 用戶端非常基本，不允許太多的自訂。 只有四個設定需要檢查：

* 伺服器位址
* 遠端識別碼
* 本機識別碼
* 驗證設定
* OS 版本 (10.11 或更新版本)


## <a name="VPNClient"></a> 針對憑證型驗證問題進行疑難排解
1. 檢查 VPN 用戶端設定。 按 Command 鍵 + Shift 鍵以移至 [網路設定]，然後輸入 "VPN" 以檢查 VPN 用戶端設定。 從清單中，按一下需要調查的 VPN 項目。

  ![IKEv2 憑證型驗證](./media/vpn-gateway-troubleshoot-point-to-site-osx-ikev2/ikev2cert1.jpg)
2. 確認 [伺服器位址] 是完整的 FQDN 且包含 cloudapp.net。
3. [遠端識別碼] 應該與「伺服器位址」(閘道 FQDN) 相同。
4. [本機識別碼] 應該與用戶端憑證的**主體**相同。
5. 按一下 [認證設定] 以開啟 [認證設定] 頁面。

  ![驗證設定](./media/vpn-gateway-troubleshoot-point-to-site-osx-ikev2/ikev2auth2.jpg)
6. 確認已從下拉式清單中選取 [憑證]。
7. 按一下 [選取] 按鈕，並確認已選取正確的憑證。 按一下 [好] 以儲存所有變更。

## <a name="ikev2"></a>針對使用者名稱和密碼驗證問題進行疑難排解

1. 檢查 VPN 用戶端設定。 按 Command 鍵 + Shift 鍵以移至 [網路設定]，然後輸入 "VPN" 以檢查 VPN 用戶端設定。 從清單中，按一下需要調查的 VPN 項目。

  ![IKEv2 使用者名稱密碼](./media/vpn-gateway-troubleshoot-point-to-site-osx-ikev2/ikev2user3.jpg)
2. 確認 [伺服器位址] 是完整的 FQDN 且包含 cloudapp.net。
3. [遠端識別碼] 應該與「伺服器位址」(閘道 FQDN) 相同。
4. [本機識別碼] 可以空白。
5. 按一下 [認證設定] 按鈕，然後確認已從下拉式清單中選取 [使用者名稱]。

  ![驗證設定](./media/vpn-gateway-troubleshoot-point-to-site-osx-ikev2/ikev2auth4.jpg)
6. 確認所輸入的認證正確。

## <a name="additional"></a>其他步驟

如果您嘗試上述步驟且所有項目都設定正確，請下載 [Wireshark](https://www.wireshark.org/#download) 並執行封包擷取。

1. 依據 *iskmp* 進行篩選，然後查看 **IKE_SA** 封包。 您應該能夠在 [Payload: Security Association] \(承載: 安全性關聯\) 底下查看 SA 提案詳細資料。 
2. 確認用戶端和伺服器有一個通用集合。

  ![封包](./media/vpn-gateway-troubleshoot-point-to-site-osx-ikev2/packet5.jpg)

## <a name="next-steps"></a>後續步驟
如需其他協助，請參閱 [Microsoft 支援服務](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade)。
