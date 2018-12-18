---
title: 驗證傳輸到 Microsoft Azure 虛擬網路的 VPN 輸送量 | Microsoft Docs
description: 本文件的目的是在協助使用者驗證從其內部部署資源到 Azure 虛擬機器的網路輸送量。
services: vpn-gateway
documentationcenter: na
author: chadmath
manager: jasmc
editor: ''
tags: azure-resource-manager,azure-service-management
ms.assetid: ''
ms.service: vpn-gateway
ms.devlang: na
ms.topic: troubleshooting
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 06/15/2018
ms.author: radwiv;chadmat;genli
ms.openlocfilehash: 7e6b3e7496c4a063156ff3b8feae1f5096efe55f
ms.sourcegitcommit: 04fc1781fe897ed1c21765865b73f941287e222f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/13/2018
ms.locfileid: "39035613"
---
# <a name="how-to-validate-vpn-throughput-to-a-virtual-network"></a>如何驗證傳輸到虛擬網路的 VPN 輸送量

VPN 閘道連線可讓您在 Azure 內的虛擬網路和內部部署 IT 基礎結構之間建立安全的跨單位連線。

本文章說明如何驗證從內部部署資源到 Azure 虛擬機器 (VM) 的網路輸送量。 本文章也提供疑難排解指引。

>[!NOTE]
>本文章旨在協助診斷和修正常見的問題。 如果您在使用下列資訊後仍無法解決問題，[請連絡支援人員](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade)。
>
>

## <a name="overview"></a>概觀

VPN 閘道連線涉及下列元件：

- 內部部署 VPN 裝置 (檢視[已經驗證的 VPN 裝置列表)](vpn-gateway-about-vpn-devices.md#devicetable)。
- 公用網際網路
- Azure VPN 閘道
- Azure VM

下圖顯示內部部署網路透過 VPN 到 Azure 虛擬網路的邏輯連線能力。

![客戶網路使用 VPN 到 MSFT 網路的邏輯連線能力](./media/vpn-gateway-validate-throughput-to-vnet/VPNPerf.png)

## <a name="calculate-the-maximum-expected-ingressegress"></a>計算最大預期輸入/輸出

1.  判斷您應用程式的基準輸送量需求。
2.  判斷您的 Azure VPN 閘道輸送量限制。 如需說明，請參閱 [規劃與設計 VPN 閘道](vpn-gateway-plan-design.md) 的「依 SKU 和 VPN 類型彙總輸送量」區段。
3.  判斷 VM 大小的 [Azure VM 輸送量指引](../virtual-machines/virtual-machines-windows-sizes.md)。
4.  決定您網際網路服務提供者 (ISP) 的頻寬。
5.  計算您預期的輸送量 - 最小頻寬的 (VM、閘道、ISP) * 0.8。

如果您算出的輸送量不符合您應用程式的基準輸送量需求，您需要增加您發現已成為瓶頸的資源頻寬。 如果要調整 Azure VPN 閘道的大小，請參閱 [變更閘道 SKU](vpn-gateway-about-vpn-gateway-settings.md#gwsku)。 如果調整虛擬機器的大小，請參閱 [調整 VM 的大小](../virtual-machines/virtual-machines-windows-resize-vm.md)。 如果您未能享有預期的網際網路頻寬，您可能也需要連絡您的 ISP。

## <a name="validate-network-throughput-by-using-performance-tools"></a>使用效能工具驗證網路輸送量

由於測試時的 VPN 通道輸送量飽和情況無法呈現準確的結果，因此應在非尖峰時段執行此驗證。

iPerf 是我們用於此測試的工作，分別在 Windows 與 Linux 上工作，並具有用戶端與伺服器模式。 Windows VM 限制在 3 Gbps。

此工具不會對磁碟執行任何讀取/寫入作業。 此工具只會產生從一端到另一端自我產生的 TCP 流量。 此工具然後根據測量用戶端與伺服器節點之間的可用頻寬實驗，產生統計資料。 在兩個節點之間測試時，一個節點作為伺服器，另一個節點作為用戶端。 一旦完成此項測試後，我們建議您對調角色，以測試雙方節點的上傳和下載輸送量。

### <a name="download-iperf"></a>下載 iPerf
下載 [iPerf](https://iperf.fr/download/iperf_3.1/iperf-3.1.2-win64.zip)。 如需詳細資訊，請參閱 [iPerf 文件](https://iperf.fr/iperf-doc.php)。

 >[!NOTE]
 >本文章討論的協力廠商產品是由獨立於 Microsoft 的公司製造。 Microsoft 對於這些產品的效能或可靠性不提供任何默示或其他保證。
 >
 >

### <a name="run-iperf-iperf3exe"></a>執行 iPerf (iperf3.exe)
1. 啟用允許流量的 NSG/ACL 規則 (適用於 Azure VM 上的公用 IP 位址測試)。

2. 在這兩個節點上，啟用連接埠 5001 的防火牆例外狀況。

    **Windows：** 以系統管理員身分執行下列命令。

    ```CMD
    netsh advfirewall firewall add rule name="Open Port 5001" dir=in action=allow protocol=TCP localport=5001
    ```

    如果要在測試完成時移除規則，請執行此命令：

    ```CMD
    netsh advfirewall firewall delete rule name="Open Port 5001" protocol=TCP localport=5001
    ```
     
    **Azure Linux：** Azure Linux 映像有寬鬆的防火牆。 如果有應用程式接聽連接埠，則允許通過流量。 受保護的自訂映像可能需要明確開啟連接埠。 常見的 Linux OS 層防火牆包括 `iptables`、`ufw` 或 `firewalld`。

3. 在伺服器節點上，請變更至將 iperf3.exe 解壓縮的目錄。 然後在伺服器模式中執行 iPerf，並以下列命令設為接聽連接埠 5001︰

     ```CMD
     cd c:\iperf-3.1.2-win65

     iperf3.exe -s -p 5001
     ```

4. 在用戶端節點上，變更至將 iperf 工具解壓縮的目錄，然後執行下列命令：

    ```CMD
    iperf3.exe -c <IP of the iperf Server> -t 30 -p 5001 -P 32
    ```

    用戶端引起從連接埠 5001 到伺服器的流量 30 秒。 旗標「-P」指出我們正使用伺服器節點的 32 條同時連線。

    下列畫面顯示此範例的輸出：

    ![輸出](./media/vpn-gateway-validate-throughput-to-vnet/06theoutput.png)

5. (選用) 如果要保留測試結果，請執行此命令：

    ```CMD
    iperf3.exe -c IPofTheServerToReach -t 30 -p 5001 -P 32  >> output.txt
    ```

6. 完成之前的步驟後，請在對調角色後執行相同的步驟，因此伺服器節點現在將是用戶端，反之亦然。

## <a name="address-slow-file-copy-issues"></a>處理檔案複製變慢的問題
使用 Windows 檔案總管或透過 RDP 工作階段拖放時，您可能會遇到檔案複製變慢的問題。 此問題一般是因為下列一個或多個因素造成︰

- 如 Windows 檔案總管與 RDP 的檔案應用程式，並未在複製檔案時使用多執行緒。 為了提升效能，請使用如 [Richcopy](https://technet.microsoft.com/magazine/2009.04.utilityspotlight.aspx) 等多執行緒的檔案複製應用程式，以 16 或 32 條執行緒複製檔案。 如果要在 Richcopy 內變更用於檔案複製的執行緒數量，請按一下 [動作]  >  [複製選項]  >  [檔案複製] 。<br><br>
![檔案複製變慢的問題](./media/vpn-gateway-validate-throughput-to-vnet/Richcopy.png)<br>
- VM 磁碟讀取/寫入速度不足。 如需詳細資訊，請參閱 [Azure 儲存體疑難排解](../storage/common/storage-e2e-troubleshooting.md)。

## <a name="on-premises-device-external-facing-interface"></a>內部部署裝置的外部對應介面
如果內部部署 VPN 裝置連結網際網路的 IP 位址包含在 Azure 中的 [區域網路](vpn-gateway-howto-site-to-site-resource-manager-portal.md#LocalNetworkGateway) 定義內，您可能會遇到無法呼叫 VPN、零星中斷連線或效能問題。

## <a name="checking-latency"></a>檢查延遲
使用 tracert 追蹤至 Microsoft Azure Edge 裝置，以判斷躍點之間是否有任何超過 100 ms 的延遲。

從內部部署網路，執行 *tracert* 以追蹤至 Azure Gateway 或 VM 的 VIP。 您只看到 * 傳回後，即表示您已觸達 Azure 邊緣。 您看到包括「MSN」的 DNS 名稱傳回時，即表示您已觸達 Microsoft 骨幹。<br><br>
![檢查延遲](./media/vpn-gateway-validate-throughput-to-vnet/08checkinglatency.png)

## <a name="next-steps"></a>後續步驟
如需詳細資訊或協助，請看看下列連結：

- [最佳化 Azure 虛擬機器的網路輸送量](../virtual-network/virtual-network-optimize-network-bandwidth.md)
- [Microsoft 支援服務](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade)
