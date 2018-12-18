---
title: 對 Azure 中的網路虛擬設備問題進行疑難排解 | Microsoft Docs
description: 了解如何對 Azure 中的網路虛擬設備問題進行疑難排解。
services: virtual-network
documentationcenter: na
author: genlin
manager: cshepard
editor: ''
tags: azure-resource-manager
ms.service: virtual-network
ms.devlang: na
ms.topic: troubleshooting
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 08/02/2018
ms.author: genli
ms.openlocfilehash: 5f11129e562ec47f4c81790cf21c9a390cc697b8
ms.sourcegitcommit: 35ceadc616f09dd3c88377a7f6f4d068e23cceec
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/08/2018
ms.locfileid: "39621400"
---
#  <a name="network-virtual-appliance-issues-in-azure"></a>Azure 中的網路虛擬設備問題

在 Microsoft Azure 中使用第三方網路虛擬設備 (NVA) 時，您可能會遇到 VM 或 VPN 連線問題和錯誤。 本文提供可協助您驗證 NVA 組態的基本 Azure 平台需求的基本步驟。

第三方 NVA 及其與 Azure 平台的整合所需的技術支援，由 NVA 廠商所提供。 如果您有牽涉到 NVA 的連線或路由問題，您應直接[連絡 NVA 廠商](https://support.microsoft.com/help/2984655/support-for-azure-market-place-for-virtual-machines)。

[!INCLUDE [support-disclaimer](../../includes/support-disclaimer.md)]

## <a name="checklist-for-troubleshooting-with-nva-vendor"></a>NVA 廠商的疑難排解檢查清單

- NVA VM 軟體的軟體更新
- 服務帳戶設定和功能
- 虛擬網路子網路上將流量導向至 NVA 的使用者定義路由 (UDR)
- 虛擬網路子網路上從 NVA 導向流量的 UDR
- NVA 內的路由資料表和規則 (例如，從 NIC2 到 NIC1)
- 追蹤 NVA NIC 以確認是否接收和傳送網路流量

## <a name="basic-troubleshooting-steps"></a>基本疑難排解步驟

- 檢查基本組態
- 檢查 NVA 效能
- 進階網路疑難排解

## <a name="check-the-minimum-configuration-requirements-for-nvas-on-azure"></a>檢查 Azure 上的 NVA 最基本的組態需求

每個 NVA 都有在 Azure 上正常運作所需的基本組態需求。 下節將提供用來確認這些基本組態的步驟。 如需詳細資訊，請[連絡 NVA 廠商](https://support.microsoft.com/help/2984655/support-for-azure-market-place-for-virtual-machines)。

**檢查 NVA 上是否已啟用 IP 轉送**

使用 Azure 入口網站

1.  在 [Azure 入口網站](https://portal.azure.com)中找出 NVA 資源，選取 [網路功能]，然後選取 [網路介面]。
2.  在 [網路介面] 頁面上，選取 [IP 組態]。
3.  確定已啟用 IP 轉送。

使用 PowerShell

1. 開啟 PowerShell，然後登入您的 Azure 帳戶。
2. 執行下列命令 (請將括弧中的值取代為您的資訊)：

        Get-AzureRmNetworkInterface -ResourceGroupName <ResourceGroupName> -Name <NicName>  

3. 檢查 **EnableIPForwarding** 屬性。
 
4. 如果未啟用 IP 轉送，請執行下列命令加以啟用：

          $nic2 = Get-AzureRmNetworkInterface -ResourceGroupName <ResourceGroupName> -Name <NicName>
          $nic2.EnableIPForwarding = 1
          Set-AzureRmNetworkInterface -NetworkInterface $nic2
          Execute: $nic2 #and check for an expected output:
          EnableIPForwarding   : True
          NetworkSecurityGroup : null

**檢查流量是否可路由至 NVA**

1. 在 [Azure 入口網站](https://portal.azure.com)上開啟 [網路監看員]，然後選取 [下一個躍點]。
2. 指定要讓 NVA 作為下一個躍點的 VM，以及用來檢視下一個躍點的目的地 IP 位址。 
3. 如果 NVA 未列為**下一個躍點**，請檢查和更新 Azure 路由表。

**檢查流量是否可到達 NVA**

1.  在 [Azure 入口網站](https://portal.azure.com)中開啟 [網路監看員]，然後選取 [IP 流量驗證]。 
2.  指定 VM 和 NVA 的 IP 位址，然後檢查是否有任何網路安全性群組 (NSG) 會封鎖流量。
3.  如果有 NSG 規則會封鎖流量，請在**有效的安全性**規則中找出 NSG，然後將其更新為允許流量通過。 接著，再次執行 [IP 流量驗證]，並使用 [連線檢查] 測試從 VM 到您內部或外部 IP 位址的 TCP 通訊。

**檢查 NVA 和 VM 是否接聽預期的流量**

1.  使用 RDP 或 SSH 連線至 NVA，並執行下列命令：

    若為 Windows：

        netstat -an

    若為 Linux：

        netstat -an | grep -i listen
2.  如果您未看見結果中列出的 NVA 軟體所使用的 TCP 連接埠，則必須在 NVA 和 VM 上設定應用程式，以接聽並回應到達這些連接埠的流量。 [視需要連絡 NVA 廠商以尋求協助](https://support.microsoft.com/help/2984655/support-for-azure-market-place-for-virtual-machines)。

## <a name="check-nva-performance"></a>檢查 NVA 效能

### <a name="validate-vm-cpu"></a>驗證 VM CPU

如果 CPU 使用率接近 100%，則可能會發生影響到網路封包捨棄的問題。 您的 VM 會在 Azure 入口網站中報告特定時間範圍內的平均 CPU。 如果 CPU 用量暴增，請調查是客體 VM 上的哪一個處理序造成高 CPU 用量，且如果可能的話，請使其降低。 您可能需要將 VM 的大小調整為較大的 SKU，或針對虛擬機器擴展集增加執行個體計數，或設定為自動調整 CPU 使用量。 發生前述任一問題時，請視需要[連絡 NVA 廠商以取得協助](https://support.microsoft.com/help/2984655/support-for-azure-market-place-for-virtual-machines)。

### <a name="validate-vm-network-statistics"></a>驗證 VM 網路統計資料 

如果 VM 網路用量暴增或出現使用量偏高的期間，您可能也需要增加 VM 的 SKU 大小，以取得更高的輸送能力。 您也可以啟用加速網路，藉以重新部署 VM。 若要確認 NVA 是否支援加速網路功能，請視需要[連絡 NVA 廠商以取得協助](https://support.microsoft.com/help/2984655/support-for-azure-market-place-for-virtual-machines)。

## <a name="advanced-network-administrator-troubleshooting"></a>進階網路管理員疑難排解

### <a name="capture-network-trace"></a>擷取網路追蹤
在您執行 **[PsPing](https://docs.microsoft.com/sysinternals/downloads/psping)** 或 **Nmap** 時，在來源 VM、NVA 和目的地 VM 上擷取同時網路追踨，然後停止追蹤。

1. 若要擷取同時網路追蹤，請執行下列命令：

    若為 Windows：

        netsh trace start capture=yes tracefile=c:\server_IP.etl scenario=netconnection

    若為 Linux：

        sudo tcpdump -s0 -i eth0 -X -w vmtrace.cap

2. 使用從來源 VM 到目的地 VM 的 **PsPing** 或 **Nmap** (例如：`PsPing 10.0.0.4:80` 或 `Nmap -p 80 10.0.0.4`)。

3. 使用[網路監視器](https://www.microsoft.com/download/details.aspx?id=4865)或 tcpdump，開啟從目的地 VM 的網路追蹤。 對您執行 **PsPing** 或 **Nmap** 的來源 VM 套用 IP 的顯示篩選器，例如 `IPv4.address==10.0.0.4 (Windows netmon)` 或 `tcpdump -nn -r vmtrace.cap src or dst host 10.0.0.4` (Linux)。

### <a name="analyze-traces"></a>分析追蹤

如果您未看到傳入後端 VM 追蹤的封包，則有可能是受到 NSG 或 UDR 干擾，或 NVA 路由表不正確。

若您看到了傳入封包但沒有回應，則您可能需要解決 VM 應用程式或防火牆的問題。 發生前述任一問題時，請[視需要連絡 NVA 廠商以取得協助](Let's link customer to 3rd party vendor again here: https://support.microsoft.com/help/2984655/support-for-azure-market-place-for-virtual-machines)。

