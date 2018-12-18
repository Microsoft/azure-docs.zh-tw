---
title: 教學課程 - 建立 Azure DNS 別名記錄來參考 Azure 公用 IP 位址。
description: 本教學課程說明如何設定 Azure DNS 別名記錄來參考 Azure 公用 IP 位址。
services: dns
author: vhorne
ms.service: dns
ms.topic: tutorial
ms.date: 9/25/2018
ms.author: victorh
ms.openlocfilehash: 842015d853790e3ac78214cca6a70becb7f9fadf
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46991216"
---
# <a name="tutorial-configure-an-alias-record-to-refer-to-an-azure-public-ip-address"></a>教學課程：設定別名記錄來參考 Azure 公用 IP 位址 

在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 建立網路基礎結構
> * 建立 Web 伺服器虛擬機器
> * 建立別名記錄
> * 測試別名記錄


如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

## <a name="prerequisites"></a>必要條件
您必須提供一個可在要測試的 Azure DNS 中裝載的網域名稱。 您必須能夠完全控制此網域，包括能夠設定網域的名稱伺服器 (NS) 記錄。

如需有關在 Azure DNS 中裝載網域的指示，請參閱[教學課程：在 Azure DNS 中裝載您的網域](dns-delegate-domain-azure-dns.md)。

本教學課程使用的範例網域是 contoso.com，但您應該使用自己的網域名稱。

## <a name="create-the-network-infrastructure"></a>建立網路基礎結構
首先，請建立一個可供放置您 Web 伺服器的 VNet 和子網域。
1. 在 http://portal.azure.com 登入 Azure 入口網站。
2. 在入口網站的左上方，按一下 [建立資源]，在搜尋方塊中輸入「資源群組」，然後建立一個名為 **RG-DNS-Alias-pip** 的資源群組。
3. 依序按一下 [建立資源]、[網路] 及 [虛擬網路]。
4. 建立名為 **VNet-Server** 的虛擬網路，將它放在 **RG-DNS-Alias-pip** 資源群組中，然後將子網路命名為 **SN-Web**。

## <a name="create-a-web-server-virtual-machine"></a>建立 Web 伺服器虛擬機器
1. 依序按一下 [建立資源] 和 [Windows Server 2016 VM]。
2. 輸入 **Web-01** 作為名稱，然後將 VM 放在 [RG-DNS-Alias-TM] 資源群組中。 輸入使用者名稱、密碼，然後按一下 [確定]。
3. 針對 [大小]，選擇具有 8 GB RAM 的 SKU。
4. 針對 [設定]，選取 [VNet-Servers] 虛擬網路、[SN-Web] 子網路。 針對公用輸入連接埠，選取 [HTTP]、[HTTPS]及 [RDP (3389)]，然後按一下 [確定]。
5. 在 [摘要] 頁面上，按一下 [建立]。

   這需要幾分鐘的時間來完成。

### <a name="install-iis"></a>安裝 IIS

在 **Web-01** 上安裝 IIS。

1. 連線至 **Web-01** 並登入。
2. 從「伺服器管理員儀表板」中，按一下 [新增角色及功能]。
3. 按 [下一步] 三次，然後在 [伺服器角色] 頁面上，選取 [網頁伺服器 (IIS)]。
4. 按一下 [新增功能]，然後按 [下一步]。
5. 按 [下一步] 四次，然後按一下 [安裝]。

   這需要幾分鐘才能完成。
6. 當安裝完成時，按一下 [關閉]。
7. 開啟網頁瀏覽器，然後瀏覽至 **localhost** 以確認預設 IIS 網頁伺服器有出現。

## <a name="create-an-alias-record"></a>建立別名記錄

建立一個指向公用 IP 位址的別名記錄。

1. 按一下您的 Azure DNS 區域以開啟該區域。
2. 按一下 [記錄集]。
3. 在 [名稱]**Name** 文字方塊中，輸入 **web01**。
4. 將 [類型] 保留為 [A] 記錄。
5. 按一下 [別名記錄集] 核取方塊。
6. 按一下 [選擇 Azure 服務]，然後選取 [Web-01-ip] 公用 IP 位址。

## <a name="test-the-alias-record"></a>測試別名記錄

1. 在 [RG-DNS-Alias-pip] 資源群組中，按一下 [Web-01] 虛擬機器。 記下公用 IP 位址。
1. 從網頁瀏覽器中，瀏覽至 Web01-01 虛擬機器的完整網域名稱。 例如：**web01.contoso.com**。 您應該會看見 IIS 預設網頁。
2. 關閉網頁瀏覽器。
3. 停止 **Web-01** 虛擬機器，然後將它重新啟動。
4. 當它重新啟動時，記下虛擬機器的新公用 IP 位址。
5. 開啟新瀏覽器，然後再次瀏覽至 Web01-01 虛擬機器的完整網域名稱。 例如：**web01.contoso.com**。
6. 這應該仍然會成功，因為您使用別名記錄來指向公用 IP 位址資源，而不是標準的 A 記錄。

## <a name="clean-up-resources"></a>清除資源

當不再需要 **RG-DNS-Alias-pip** 資源群組時，您可以將它刪除，以刪除為本教學課程建立的所有資源。


## <a name="next-steps"></a>後續步驟

在本教學課程中，您已建立別名記錄來參考 Azure 公用 IP 位址。 若要了解 Azure DNS 和 Web 應用程式，請繼續進行 Web 應用程式的教學課程。

> [!div class="nextstepaction"]
> [在自訂網域中建立 Web 應用程式的 DNS 記錄](./dns-web-sites-custom-domain.md)
