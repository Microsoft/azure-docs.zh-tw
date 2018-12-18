---
title: 教學課程：使用流量管理員來建立 Auzre DNS 別名記錄以支援網域頂點名稱
description: 本教學課程說明如何使用「流量管理員」來設定 Auzre DNS 別名記錄，以支援使用網域頂點名稱。
services: dns
author: vhorne
ms.service: dns
ms.topic: tutorial
ms.date: 9/25/2018
ms.author: victorh
ms.openlocfilehash: 4a5d8968861f6f2938e605d2f7106529ca401df4
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46967433"
---
# <a name="tutorial-configure-an-alias-record-to-support-apex-domain-names-with-traffic-manager"></a>教學課程：使用流量管理員來設定別名記錄以支援頂點網域名稱 

您可以為網域名稱頂點 (例如 contoso.com) 建立別名記錄，以參考「流量管理員」設定檔。 因此，您可以不使用重新導向服務達到此目的，而是設定讓 Azure DNS 直接從您的區域參考「流量管理員」設定檔。 


在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 建立主機 VM 和網路基礎結構
> * 建立流量管理員設定檔
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
2. 在入口網站的左上方，按一下 [建立資源]，在搜尋方塊中輸入「資源群組」，然後建立一個名為 **RG-DNS-Alias-TM** 的資源群組。
3. 依序按一下 [建立資源]、[網路] 及 [虛擬網路]。
4. 建立名為 **VNet-Servers** 的虛擬網路，將它放在 **RG-DNS-Alias-TM** 資源群組中，然後將子網路命名為 **SN-Web**。

## <a name="create-two-web-server-virtual-machines"></a>建立兩部 Web 伺服器虛擬機器
1. 依序按一下 [建立資源] 和 [Windows Server 2016 VM]。
2. 輸入 **Web-01** 作為名稱，然後將 VM 放在 [RG-DNS-Alias-TM] 資源群組中。 輸入使用者名稱、密碼，然後按一下 [確定]。
3. 針對 [大小]，選擇具有 8 GB RAM 的 SKU。
4. 針對 [設定]，選取 [VNet-Servers] 虛擬網路、[SN-Web] 子網路。
5. 按一下 [公用 IP 位址]，然後在 [指派] 底下，按一下 [靜態]，再按一下 [確定]。
6. 針對公用輸入連接埠，選取 [HTTP]、[HTTPS]及 [RDP (3389)]，然後按一下 [確定]。
7. 在 [摘要] 頁面上，按一下 [建立]。

   這需要幾分鐘的時間來完成。
6. 重複此程序來建立名為 **Web-02** 的另一部虛擬機器。

### <a name="add-a-dns-label"></a>新增 DNS 標籤
公用 IP 位址需要有 DNS 標籤，才能與「流量管理員」搭配運作。
1. 在 [RG-DNS-Alias-TM] 資源群組中，按一下 [Web-01-ip] 公用 IP 位址。
2. 在 [設定] 底下，按一下 [設定]。
3. 在 DNS 名稱標籤文字方塊中，輸入 **web01pip**。
4. 按一下 [檔案] 。

針對 **Web-02-ip** 公用 IP 位址，使用 **web02pip** 作為 DNS 名稱標籤來重複此程序。

### <a name="install-iis"></a>安裝 IIS

在 **Web-01** 和 **Web-02** 上安裝 IIS。

1. 連線至 **Web-01** 並登入。
2. 從「伺服器管理員儀表板」中，按一下 [新增角色及功能]。
3. 按 [下一步] 三次，然後在 [伺服器角色] 頁面上，選取 [網頁伺服器 (IIS)]。
4. 按一下 [新增功能]，然後按 [下一步]。
5. 按 [下一步] 四次，然後按一下 [安裝]。

   這需要幾分鐘才能完成。
6. 當安裝完成時，按一下 [關閉]。
7. 開啟網頁瀏覽器，然後瀏覽至 **localhost** 以確認預設 IIS 網頁伺服器有出現。

重複此程序以在 **Web-02**上安裝 IIS。


## <a name="create-a-traffic-manager-profile"></a>建立流量管理員設定檔

到期 

1. 開啟 [RG-DNS-Alias-TM] 資源群組，然後按一下 [Web-01-ip] 公用 IP 位址。 記下 IP 位址以供稍後使用。 針對 **Web-02-ip** 公用 IP 位址重複此程序。
1. 依序按一下 [建立資源]、[網路] 及 [流量管理員設定檔]。
2. 針對名稱輸入 **TM-alias-test**，然後將它放在 [RG-DNS-Alias-TM] 資源群組中。
3. 按一下頁面底部的 [新增] 。
4. 部署完成時，按一下 [前往資源]。
5. 在流量管理員設定檔頁面上的 [設定] 底下，按一下 [端點]。
6. 按一下 [新增] 。
7. 針對 [類型]，選取 [外部端點]，針對 [名稱]，輸入 **EP-Web01**。
8. 在 [完整網域名稱 (FQDN) 或 IP] 文字方塊中，輸入您先前記下的 **Web-01-ip** IP 位址。
9. 選取與其他資源相同的 [位置]，然後按一下 [確定]。

使用您先前記下的 **Web-02-ip**IP 位址，重複此程序來新增 **Web-02** 端點。

## <a name="create-an-alias-record"></a>建立別名記錄

建立一個指向「流量管理員」設定檔的別名記錄。

1. 按一下您的 Azure DNS 區域以開啟該區域。
2. 按一下 [記錄集]。
3. 將 [名稱] 文字方塊保留空白以代表網域名稱頂點 (例如 contoso.com)。
4. 將 [類型] 保留為 [A] 記錄。
5. 按一下 [別名記錄集] 核取方塊。
6. 按一下 [選擇 Azure 服務]，然後選取 [TM-alias-test]「流量管理員」設定檔。

## <a name="test-the-alias-record"></a>測試別名記錄

1. 從網頁瀏覽器中，瀏覽至您的網域名稱頂點 (例如 contoso.com)。 您應該會看見 IIS 預設網頁。 關閉網頁瀏覽器。
2. 關閉 **Web-01** 虛擬機器，然後等候幾分鐘讓它完全關閉。
3. 開啟一個新的網頁瀏覽器，然後再次瀏覽至您的網域名稱頂點。
4. 您應該會再次看見預設的 IIS 頁面，因為「流量管理員」已處理情況並將流量導向到 **Web-02**。

## <a name="clean-up-resources"></a>清除資源

當不再需要 **RG-DNS-Alias-TM** 資源群組時，您可以將它刪除，以刪除為本教學課程建立的所有資源。

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已建立一個別名記錄以使用頂點網域名稱來參考「流量管理員」設定檔。 若要了解 Azure DNS 和 Web 應用程式，請繼續進行 Web 應用程式的教學課程。

> [!div class="nextstepaction"]
> [在自訂網域中建立 Web 應用程式的 DNS 記錄](./dns-web-sites-custom-domain.md)
