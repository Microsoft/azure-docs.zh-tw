---
title: 使用 IPv6 部署網際網路對向負載平衡器 - Azure 範本 | Microsoft Docs
description: 如何部署 Azure Load Balancer 和負載平衡 VM 的 IPv6 支援。
services: load-balancer
documentationcenter: na
author: KumudD
manager: timlt
editor: ''
tags: azure-resource-manager
keywords: ipv6, azure load balancer, 雙重堆疊, 公用 ip, 原生 ipv6, 行動, iot
ms.assetid: 2998e943-13fc-4ea9-a68c-875e53a08db3
ms.service: load-balancer
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/25/2017
ms.author: kumud
ms.openlocfilehash: 431b43979ac364d943c58c40b4199b7f30f9acf6
ms.sourcegitcommit: c3d53d8901622f93efcd13a31863161019325216
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/29/2018
ms.locfileid: "30263073"
---
# <a name="deploy-an-internet-facing-load-balancer-solution-with-ipv6-using-a-template"></a>使用範本部署配置有 IPv6 的網際網路面向負載平衡器解決方案

> [!div class="op_single_selector"]
> * [PowerShell](load-balancer-ipv6-internet-ps.md)
> * [Azure CLI](load-balancer-ipv6-internet-cli.md)
> * [範本](load-balancer-ipv6-internet-template.md)



Azure 負載平衡器是第 4 層 (TCP、UDP) 負載平衡器。 此負載平衡器可藉由在負載平衡器集合中，將連入流量分散於雲端服務或虛擬機器中狀況良好的服務執行個體之間，來提供高可用性。 Azure Load Balancer 也會在多個連接埠、多個 IP 位址或兩者上顯示這些服務。

## <a name="example-deployment-scenario"></a>範例部署案例

下圖說明使用本文所述範例範本部署的負載平衡解決方案。

![負載平衡器案例](./media/load-balancer-ipv6-internet-template/lb-ipv6-scenario.png)

在此案例中，您將建立下列 Azure 資源：

* 虛擬網路介面，用於每個已指派 IPv4 和 IPv6 位址的 VM
* 配置有 IPv4 和 IPv6 公用 IP 位址的網際網路面向負載平衡器
* 兩個負載平衡規則，用以對應公用 VIP 至私人端點
* 包含兩個 VM 的可用性設定組
* 兩部虛擬機器 (VM)

## <a name="deploying-the-template-using-the-azure-portal"></a>使用 Azure 入口網站部署範本

這份文件參考 [Azure 快速入門範本](https://azure.microsoft.com/documentation/templates/201-load-balancer-ipv6-create/)資源庫中發佈的範本。 您可以從資源庫下載範本，或在 Azure 中直接從資源庫啟動部署。 本文假設您已下載範本至本機電腦。

1. 開啟 Azure 入口網站，並使用有權限在 Azure 訂用帳戶內建立 VM 和網路資源的帳戶登入。 此外，若您不是使用現有資源，該帳戶就必須具有建立資源群組和儲存體帳戶的權限。
2. 按一下功能表中的 [+ 新增]，然後在搜尋方塊中輸入「範本」。 選取搜尋結果中的 [範本部署]。

    ![lb-ipv6-portal-step2](./media/load-balancer-ipv6-internet-template/lb-ipv6-portal-step2.png)

3. 在 [所有項目] 刀鋒視窗中，按一下 [範本部署]。

    ![lb-ipv6-portal-step3](./media/load-balancer-ipv6-internet-template/lb-ipv6-portal-step3.png)

4. 按一下 [建立]。

    ![lb-ipv6-portal-step4](./media/load-balancer-ipv6-internet-template/lb-ipv6-portal-step4.png)

5. 按一下 [編輯範本]。 刪除現有的內容，複製/貼上範本檔案的整個內容 (包括開始和結束的 {})，然後按一下 [儲存]。

    > [!NOTE]
    > 如果您使用 Microsoft Internet Explorer，當您貼上內容時，會看到對話方塊，詢問是否要允許存取 Windows 剪貼簿。 按一下 [允許存取]。

    ![lb-ipv6-portal-step5](./media/load-balancer-ipv6-internet-template/lb-ipv6-portal-step5.png)

6. 按一下 [編輯參數]。 在 [參數] 刀鋒視窗中，依照指引指定 [範本] 參數區段中的值，然後按一下 [儲存] 關閉 [參數] 刀鋒視窗。 在 [自訂部署] 刀鋒視窗中，選取您的訂用帳戶，選取現有資源群組或建立一個。 如果建立資源群組，請選取資源群組的位置。 接著，按一下 [法律條款]，然後按一下法律條款的 [購買]。 Azure 會開始部署資源。 部署所有資源需要幾分鐘。

    ![lb-ipv6-portal-step6](./media/load-balancer-ipv6-internet-template/lb-ipv6-portal-step6.png)

    如需有關這些參數的詳細資訊，請參閱本文稍後的 [範本參數和變數](#template-parameters-and-variables) 章節。

7. 若要查看範本建立的資源，按一下 [瀏覽]，將清單向下捲動直到您看到 [資源群組]，然後按一下它。

    ![lb-ipv6-portal-step7](./media/load-balancer-ipv6-internet-template/lb-ipv6-portal-step7.png)

8. 在 [資源群組] 刀鋒視窗上，按一下您在步驟 6 中指定的資源群組名稱。 您會看到所有已部署資源的清單。 如果一切順利，[上次部署] 下應該會顯示 [成功]。 如果沒有，請確定您使用的帳戶具有建立所需資源的權限。

    ![lb-ipv6-portal-step8](./media/load-balancer-ipv6-internet-template/lb-ipv6-portal-step8.png)

    > [!NOTE]
    > 如果在完成步驟 6 之後立即瀏覽資源群組，[上次部署] 會在資源部署期間顯示部署的狀態。

9. 按一下資源清單中的 [myIPv6PublicIP]。 您會看到它在 IP 位址下有個 IPv6 位址，其 DNS 名稱是您在步驟 6 中為 dnsNameforIPv6LbIP 參數指定的值。 此資源是公用 IPv6 位址和主機名稱，且可以存取網際網路用戶端。

    ![lb-ipv6-portal-step9](./media/load-balancer-ipv6-internet-template/lb-ipv6-portal-step9.png)

## <a name="validate-connectivity"></a>驗證連線能力

成功部署範本後，您可以完成下列工作來驗證連線︰

1. 登入 Azure 入口網站，連接到範本部署建立的每個 VM。 如果您部署的是 Windows Server VM，請從命令提示字元執行 ipconfig /all。 您會看到 VM 同時有 IPv4 和 IPv6 位址。 如果您部署的是 Linux VM，必須依據 Linux 散發套件提供的指示，將 Linux 作業系統設定為接收動態 IPv6 位址。
2. 從 IPv6 網際網路連接的用戶端，初始化連接至負載平衡器公用 IPv6 位址的連線。 若要確認負載平衡器在兩個 VM 之間達到平衡，您可以在每個 VM 上安裝網頁伺服器，像是 Microsoft 網際網路資訊服務 (IIS)。 在每部伺服器的預設網頁上可能會有 "Server0" 或 "Server1" 文字，可用來唯一識別它。 然後，在 IPv6 網際網路連接的用戶端上開啟網際網路瀏覽器，瀏覽至您為負載平衡器的 dnsNameforIPv6LbIP 參數指定的主機名稱，即可確認每個 VM 的端對端 IPv6 連線能力。 如果您只只在其中一部伺服器看到網頁，可能需要清除瀏覽器的快取。 開啟多個私人瀏覽工作階段。 您應該會看來自每部伺服器的回應。
3. 從 IPv4 網際網路連接的用戶端，初始化連接至負載平衡器公用 IPv4 位址的連線。 若要確認負載平衡器在兩個 VM 之間達到平衡，您可以使用 IIS 進行測試，步驟 2 中所述。
4. 從每個 VM 初始化傳出連線，連至 IPv6 或 IPv4 連接的網際網路裝置。 在這兩個案例中，目的地裝置看到的來源 IP 是負載平衡器的公用 IPv4 或 IPv6 位址。

> [!NOTE]
> IPv4 和 IPv6 的 ICMP 在 Azure 網路中會被封鎖。 因此，像 ping 這類的 ICMP 工具一定會失敗。 若要測試連線，使用 TCP 替代方法，例如 TCPing 或 PowerShell Test-NetConnection Cmdlet。 請注意，圖中顯示的 IP 位址是您可能會看到的範例值。 因為 IPv6 位址為動態指派，您收到的位址會不同，且可能因地區而異。 此外，負載平衡器上公用 IPv6 位址與後端集區中私人 IPv6 位址的起始前置詞不同很常見。

## <a name="template-parameters-and-variables"></a>範本參數和變數

Azure Resource Manager 範本可包含多個變數和參數，讓您自訂自己的需求。 變數用於不想讓使用者變更的固定值。 參數用於部署範本時要由使用者提供的值。 範例範本是針對本文所述案例而設定。 您可自訂此範本以符合您的環境需求。

本文中所用的範例範本包含下列變數和參數︰

| 參數 / 變數 | 注意 |
| --- | --- |
| adminUsername |指定用來登入虛擬機器的系統管理員帳戶名稱。 |
| adminPassword |指定用來登入虛擬機器的系統管理員帳戶密碼。 |
| dnsNameforIPv4LbIP |指定您想要指派為負載平衡器公用名稱的 DNS 主機名稱。 這個名稱會解析為負載平衡器的公用 IPv4 位址。 此名稱必須是小寫，並符合規則運算式：^ [a-z][a-z0-9-]{1,61}[a-z0-9]$。 |
| dnsNameforIPv6LbIP |指定您想要指派為負載平衡器公用名稱的 DNS 主機名稱。 這個名稱會解析為負載平衡器的公用 IPv6 位址。 此名稱必須是小寫，並符合規則運算式：^ [a-z][a-z0-9-]{1,61}[a-z0-9]$。 這可以和 IPv4 位址的名稱相同。 當名稱相同時，若用戶端傳送此名稱的 DNS 查詢，Azure 將傳回 A 和 AAAA 記錄。 |
| vmNamePrefix |指定 VM 名稱前置詞。 建立 VM 時，範本會在名稱上附加一個數字 (0、1 等等)。 |
| nicNamePrefix |指定網路介面名稱前置詞。 建立網路介面時，範本會在名稱上附加一個數字 (0、1 等等)。 |
| storageAccountName |輸入現有儲存體帳戶的名稱，或指定範本所建立的新儲存體帳戶的名稱。 |
| availabilitySetName |輸入 VM 使用的可用性設定組稱。 |
| addressPrefix |用來定義虛擬網路位址範圍的位址首碼。 |
| subnetName |為 VNet 建立的子網路名稱。 |
| subnetPrefix |用來定義子網路位址範圍的位址首碼。 |
| vnetName |指定 VM 所使用的 VNet 名稱。 |
| ipv4PrivateIPAddressType |私人 IP 位址使用的配置方法 (Static 或 Dynamic)。 |
| ipv6PrivateIPAddressType |私人 IP 位址使用的配置方法 (Dynamic)。 IPv6 只支援 Dynamic (動態) 配置。 |
| numberOfInstances |範本部署的負載平衡執行個體的數目。 |
| ipv4PublicIPAddressName |指定您想要用來與負載平衡器的公用 IPv4 位址進行通訊的 DNS 名稱。 |
| ipv4PublicIPAddressType |公用 IP 位址使用的配置方法 (Static 或 Dynamic)。 |
| Ipv6PublicIPAddressName |指定您想要用來與負載平衡器的公用 IPv6 位址進行通訊的 DNS 名稱。 |
| ipv6PublicIPAddressType |公用 IP 位址使用的配置方法 (Dynamic)。 IPv6 只支援 Dynamic (動態) 配置。 |
| lbName |指定負載平衡器的名稱。 這個名稱會顯示在入口網站，或在以 CLI 或 PowerShell 命令參考它時用到。 |

範本中的其餘變數是 Azure 建立資源時指派的衍生值。 請勿變更這些變數。
