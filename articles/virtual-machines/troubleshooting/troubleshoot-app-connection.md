---
title: 在 Azure 中針對 VM 應用程式存取進行疑難排解 | Microsoft Docs
description: 使用這些詳細的疑難排解步驟，來隔離連線至在 Azure 中虛擬機器上執行之應用程式時所發生的問題。
services: virtual-machines
documentationcenter: ''
author: genlin
manager: jeconnoc
editor: ''
tags: top-support-issue,azure-service-management,azure-resource-manager
keywords: 無法啟動應用程式, 程式無法開啟, 接聽連接埠被封鎖, 無法啟動程式, 接聽連接埠被封鎖
ms.assetid: b9ff7cd0-0c5d-4c3c-a6be-3ac47abf31ba
ms.service: virtual-machines
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: na
ms.topic: troubleshooting
ms.date: 05/11/2018
ms.author: genli
ms.openlocfilehash: ae5a720ce6ddd6b773229d0968bc283aa5cfa5ba
ms.sourcegitcommit: b7e5bbbabc21df9fe93b4c18cc825920a0ab6fab
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47411817"
---
# <a name="troubleshoot-application-connectivity-issues-on-virtual-machines-in-azure"></a>針對 Azure 中 虛擬機器上的應用程式連線問題進行疑難排解

有各種原因會造成無法啟動或連接至在 Azure 虛擬機器 (VM) 上執行的應用程式。 原因包括應用程式並未執行，或未在預期的連接埠上接聽，接聽連接埠遭到封鎖，或網路規則未正確將流量傳遞至應用程式。 本文說明條理式方法，以找出並更正問題。

如果您在使用 RDP 或 SSH 連接到 VM 時發生問題，請先參閱下列其中一篇文章︰

* [針對以 Windows 為基礎的 Azure 虛擬機器的遠端桌面連線進行疑難排解](troubleshoot-rdp-connection.md)
* [針對以 Linux 為基礎的 Azure 虛擬機器的安全殼層 (SSH) 連線進行疑難排解](troubleshoot-ssh-connection.md)。

如果在本文章中有任何需要協助的地方，您可以連絡 [MSDN Azure 和 Stack Overflow 論壇](https://azure.microsoft.com/support/forums/)上的 Azure 專家。 或者，您也可以提出 Azure 支援事件。 請移至 [Azure 支援網站](https://azure.microsoft.com/support/options/)，然後選取 [取得支援]。

## <a name="quick-start-troubleshooting-steps"></a>快速入門的疑難排解步驟
如果您在連接到應用程式時發生問題，請嘗試下列一般疑難排解步驟。 請在每個步驟之後，嘗試重新連接到您的應用程式︰

* 重新啟動虛擬機器
* 重新建立端點/防火牆規則/網路安全性群組 (NSG) 規則
  * [資源管理員模型 - 管理網路安全性群組](../../virtual-network/manage-network-security-group.md)
  * [傳統模型 - 管理雲端服務端點](../../cloud-services/cloud-services-enable-communication-role-instances.md)
* 從不同的位置 (例如不同的 Azure 虛擬網路) 進行連線
* 重新部署虛擬機器
  * [重新部署 Windows VM](redeploy-to-new-node-windows.md)
  * [重新部署 Linux VM](redeploy-to-new-node-linux.md)
* 重新建立虛擬機器

如需詳細資訊，請參閱 [疑難排解端點連接能力 (RDP/SSH/HTTP 等失敗問題)](https://social.msdn.microsoft.com/Forums/azure/en-US/538a8f18-7c1f-4d6e-b81c-70c00e25c93d/troubleshooting-endpoint-connectivity-rdpsshhttp-etc-failures?forum=WAVirtualMachinesforWindows)。

## <a name="detailed-troubleshooting-overview"></a>詳細疑難排解概觀
有四個主要區域來疑難排解在 Azure 虛擬機器執行上之應用程式的存取。

![疑難排解無法啟動應用程式](./media/virtual-machines-common-troubleshoot-app-connection/tshoot_app_access1.png)

1. 在 Azure 虛擬機器上執行的應用程式。
   * 應用程式本身是否正確地執行？
2. Azure 虛擬機器
   * VM 本身是否正確地執行並回應要求？
3. Azure 網路端點。
   * 傳統部署模型中虛擬機器的雲端服務端點。
   * 資源管理員部署模型中虛擬機器的網路安全性群組和輸入 NAT 規則。
   * 流量是否可以透過預期的連接埠從使用者流向 VM/應用程式？
4. 您的網際網路邊緣裝置。
   * 是否已經有防火牆規則會防止流量正確地流動？

針對正透過站對站 VPN 或 ExpressRoute 連線存取應用程式的用戶端電腦，可能造成問題的主要區域是應用程式和 Azure 虛擬機器。

若要判斷問題的來源並更正，請遵循下列步驟。

## <a name="step-1-access-application-from-target-vm"></a>步驟 1︰從目標 VM 存取應用程式
嘗試使用適當的用戶端程式，從執行所在的 VM 存取應用程式。 使用本機主機名稱、本機 IP 位址或迴路位址 (127.0.0.1)。

![直接從 VM 啟動應用程式](./media/virtual-machines-common-troubleshoot-app-connection/tshoot_app_access2.png)

例如，如果應用程式是網頁伺服器，則在 VM 上開啟瀏覽器，並嘗試存取在 VM 上託管的網頁。

如果您可以存取應用程式，請移至 [步驟 3](#step2)。

如果您無法存取應用程式，請檢查下列設定：

* 應用程式正在目標虛擬機器上執行。
* 應用程式正在接聽預期的 TCP 和 UDP 連接埠。

在 Windows 和 Linux 虛擬機器兩者上，使用 **netstat -a** 命令顯示作用中的接聽連接埠。 檢查應用程式應該接聽之預期連接埠的輸出。 重新啟動應用程式，或視需要將它設定成使用預期的連接埠，然後嘗試在本機重新存取應用程式。

## <a id="step2"></a>步驟 2︰從相同虛擬網路中的其他 VM 存取應用程式
使用 VM 的主機名稱或其 Azure 指派的公用、私人或提供者 IP 位址，嘗試存取不同 VM，但相同虛擬網路中的應用程式。 若為使用傳統部署模型建立的虛擬機器，請勿使用雲端服務的公用 IP 位址。

![從其他 VM 啟動應用程式](./media/virtual-machines-common-troubleshoot-app-connection/tshoot_app_access3.png)

例如，如果應用程式是網頁伺服器，請嘗試在相同虛擬網路中的其他 VM 上使用瀏覽器存取網頁。

如果您可以存取應用程式，請移至 [步驟 3](#step3)。

如果您無法存取應用程式，請檢查下列設定：

* 目標 VM 上的主機防火牆允許輸入要求與輸出回應的流量。
* 在目標 VM 上執行的入侵偵測或網路監視軟體允許流量。
* 雲端服務端點或網路安全性群組允許流量：
  * [傳統模型 - 管理雲端服務端點](../../cloud-services/cloud-services-enable-communication-role-instances.md)
  * [資源管理員模型 - 管理網路安全性群組](../../virtual-network/manage-network-security-group.md)
* 在您的 VM 中，測試 VM 與 VM 間的路徑執行的個別元件 (例如負載平衡器或防火牆) 允許流量。

在 Windows 虛擬機器上，請使用「具有進階安全性的 Windows 防火牆」判斷防火牆規則是否排除了您應用程式的輸入與輸出流量。

## <a id="step3"></a>步驟 3︰ 從虛擬網路外部存取應用程式
嘗試從位於虛擬網路外部的電腦存取在該虛擬網路內 VM 上執行的應用程式。 使用與您的原始用戶端電腦不同的網路。

![從位於虛擬網路之外的電腦啟動應用程式](./media/virtual-machines-common-troubleshoot-app-connection/tshoot_app_access4.png)

例如，如果應用程式是 Web 伺服器，請嘗試從虛擬網路外的電腦執行瀏覽器來存取網頁。

如果您無法存取應用程式，請檢查下列設定：

* 對於使用傳統部署模型建立的 VM：
  
  * 確認 VM 的端點組態允許連入流量，特別是通訊協定 (TCP 或 UDP) 和公用與私人連接埠號碼。
  * 確認端點上的存取控制清單 (ACL) 不會阻擋來自網際網路的連入流量。
  * 如需詳細資訊，請參閱[如何設定虛擬機器的端點](../windows/classic/setup-endpoints.md)。
* 對於使用資源管理員部署模型建立的 VM：
  
  * 確認 VM 的輸入 NAT 規則組態允許連入流量，特別是通訊協定 (TCP 或 UDP) 和公用與私人連接埠號碼。
  * 確認網路安全性群組允許輸入要求與輸出回應的流量。
  * 如需詳細資訊，請參閱[什麼是網路安全性群組？](../../virtual-network/security-overview.md)

如果虛擬機器或端點是負載平衡集的成員：

* 請確認探查通訊協定 (TCP 或 UDP) 和連接埠號碼正確。
* 如果探查通訊協定和連接埠與負載平衡集通訊協定和連接埠不同：
  * 請確認應用程式正在接聽探查通訊協定 (TCP 或 UDP) 和連接埠號碼 (在目標 VM 上使用 **netstat –a** )。
  * 確認目標 VM 上的主機防火牆允許輸入探查要求與輸出探查回應的流量。

如果您可以存取應用程式，請確定您的網際網路邊緣裝置允許：

* 從您的用戶端電腦輸出到 Azure 虛擬機器的應用程式要求流量。
* 來自 Azure 虛擬機器的輸入應用程式回應流量。

## <a name="step-4-if-you-cannot-access-the-application-use-ip-verify-to-check-the-settings"></a>步驟 4 如果您無法存取應用程式，請使用 IP 驗證來檢查設定。 

如需詳細資訊，請參閱 [Azure 網路監視概觀](https://docs.microsoft.com/azure/network-watcher/network-watcher-monitoring-overview)。 

## <a name="additional-resources"></a>其他資源
[針對以 Windows 為基礎的 Azure 虛擬機器的遠端桌面連線進行疑難排解](troubleshoot-rdp-connection.md)

[疑難排解以 Linux 為基礎之 Azure 虛擬機器的安全殼層 (SSH) 連線](troubleshoot-ssh-connection.md)


