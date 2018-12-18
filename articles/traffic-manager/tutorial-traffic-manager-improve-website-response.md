---
title: 教學課程 - 使用 Azure 流量管理員路由傳送流量來改善網站回應 | Microsoft Docs
description: 本教學課程文章說明如何建立流量管理員設定檔，以建置高度回應的網站。
services: traffic-manager
documentationcenter: ''
author: kumudd
manager: jeconnoc
editor: ''
Customer intent: As an IT Admin, I want to route traffic so I can improve website response by choosing the endpoint with lowest latency.
ms.assetid: ''
ms.service: traffic-manager
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 07/23/2018
ms.author: kumud
ms.openlocfilehash: 087dcda5826d96ad064c472fc897be7e61133387
ms.sourcegitcommit: d1aef670b97061507dc1343450211a2042b01641
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47392498"
---
# <a name="tutorial-improve-website-response-using-traffic-manager"></a>教學課程：使用流量管理員來改善網站回應 

本教學課程說明如何使用流量管理員將使用者流量導向具有最低延遲的網站，進而建立高度回應的網站。 通常，具有最低延遲的資料中心即為最短地理距離的資料中心。

在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 在 IIS 上建立兩部執行基本網站的 VM
> * 建立兩部測試 VM 以檢視流量管理員的運作
> * 為執行 IIS 的 VM 設定 DNS 名稱
> * 建立流量管理員設定檔以改善網站效能
> * 在流量管理員設定檔中新增 VM 端點
> * 檢視流量管理員的運作

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

## <a name="prerequisites"></a>必要條件
若要查看流量管理員的運作，本教學課程會要求您部署下列項目：
- 在不同 Azure 區域中執行的兩個基本網站執行個體 - **美國東部**和**西歐**。
- 用於測試流量管理員的兩個測試 VM - 一個 VM 位於**美國東部**，另一個 VM 位於**西歐**。 測試 VM 用來說明流量管理員如何將使用者流量路由傳送至在相同區域中執行的網站，因為該網站可提供最低延遲。

### <a name="sign-in-to-azure"></a>登入 Azure 

在 https://portal.azure.com 登入 Azure 入口網站。

### <a name="create-websites"></a>建立網站

在本節中，您會建立兩個網站執行個體，這兩個執行個體可在兩個 Azure 區域中為流量管理員設定檔提供兩個服務端點。 請執行下列步驟來建立這兩個網站：
1. 建立兩個 VM 來執行基本網站 - 一個位於**美國東部**，另一個位於**西歐**。
2. 在每個 VM 上安裝 IIS 伺服器並更新預設網站頁面，該頁面描述使用者在造訪網站時所連線的 VM 名稱。

#### <a name="create-vms-for-running-websites"></a>建立 VM 以供執行網站
在本節中，您會在**美國東部**和**西歐** Azure 區域中建立 *myIISVMEastUS* 和 *myIISVMWEurope* 兩個 VM。

1. 在 Azure 入口網站的左上角，選取 [建立資源] > [計算] > [Windows Server 2016 VM]。
2. 針對 [基本資料] 輸入或選取下列資訊、接受其餘設定的預設值，然後選取 [建立]：

    |設定|值|
    |---|---|
    |名稱|myIISVMEastUS|
    |使用者名稱| 輸入您選擇的使用者名稱。|
    |密碼| 輸入您選擇的密碼。 密碼長度至少必須有 12 個字元，而且符合[定義的複雜度需求](../virtual-machines/windows/faq.md?toc=%2fazure%2fvirtual-network%2ftoc.json#what-are-the-password-requirements-when-creating-a-vm)。|
    |資源群組| 選取 [新增]，然後輸入 myResourceGroupTM1。|
    |位置| 選取 [美國東部]。|
    |||
4. 在 [選擇大小] 底下選取虛擬機器大小。
5. 針對 [設定] 選取下列值，然後選取 [確定]：
    
    |設定|值|
    |---|---|
    |虛擬網路| 選取 [虛擬網路]，在 [建立虛擬網路] 中，針對 [名稱] 輸入 *myVNet1*，針對子網路輸入 *mySubnet*。|
    |網路安全性群組|選取 [基本]，然後在 [選取公用輸入連接埠] 下拉式清單中，選取 **HTTP** 和 **RDP** |
    |開機診斷|選取 [已停用]。|
    |||
6. 在 [摘要] 的 [建立] 底下，選取 [建立] 來開始部署虛擬機器。

7. 以下列變更再次完成步驟 1-6：

    |設定|值|
    |---|---|
    |資源群組 | 選取 [新增]，然後輸入「myResourceGroupTM2」|
    |位置|西歐|
    |虛擬機器名稱 | myIISVMWEurope|
    |虛擬網路 | 選取 [虛擬網路]，在 [建立虛擬網路] 中，針對 [名稱] 輸入 *myVNet2*，針對子網路輸入 *mySubnet*。|
    |||
8. 可能需要數分鐘才會建立虛擬機器。 請等到這兩個虛擬機器都已建立，再繼續進行其餘步驟。

   ![建立 VM](./media/tutorial-traffic-manager-improve-website-response/createVM.png)

#### <a name="install-iis-and-customize-the-default-web-page"></a>安裝 IIS 和自訂預設網頁

在本節中，您會在兩部 VM (*myIISVMEastUS*  & *myIISVMWEurope*) 上安裝 IIS 伺服器，然後更新預設網站頁面。 自訂的網站頁面會顯示當您從網頁瀏覽器瀏覽網站時，您所連線的 VM 名稱。

1. 選取左側功能表中的 [所有資源]，然後從資源清單按一下 *myIISVMEastUS*，其位於 *myResourceGroupTM1* 資源群組中。
2. 在 [概觀] 頁面上，按一下 [連線]，然後在 [連線至虛擬機器] 中選取 [下載 RDP 檔案]。 
3. 開啟所下載的 rdp 檔案。 如果出現提示，請選取 [連接]。 輸入您在建立 VM 時指定的使用者名稱和密碼。 您可能需要選取 [其他選擇]，然後選取 [使用不同的帳戶]，以指定您在建立 VM 時輸入的認證。 
4. 選取 [確定] 。
5. 您可能會在登入過程中收到憑證警告。 如果您收到警告，請選取 [是] 或 [繼續] 以繼續進行連線。
6. 在伺服器桌面上，瀏覽至 [Windows 系統管理工具]>[伺服器管理員]。
7. 啟動 VM1 上的 Windows PowerShell 上，並使用下列命令來安裝 IIS 伺服器及更新預設 htm 檔。
    ```powershell-interactive
    # Install IIS
      Install-WindowsFeature -name Web-Server -IncludeManagementTools
    
    # Remove default htm file
     remove-item  C:\inetpub\wwwroot\iisstart.htm
    
    #Add custom htm file
     Add-Content -Path "C:\inetpub\wwwroot\iisstart.htm" -Value $("Hello World from " + $env:computername)
    ```

     ![安裝 IIS 和自訂網頁](./media/tutorial-traffic-manager-improve-website-response/deployiis.png)
8. 關閉與 *myIISVMEastUS* 的 RDP 連線。
9. 重複步驟 1-8，方法是與 *myResourceGroupTM2* 資源群組內的 *myIISVMWEurope* VM 建立 RDP 連線，以安裝 IIS 並自訂其預設網頁。

#### <a name="configure-dns-names-for-the-vms-running-iis"></a>為執行 IIS 的 VM 設定 DNS 名稱

流量管理員會根據服務端點的 DNS 名稱來路由傳送使用者流量。 在本節中，您會設定 IIS 伺服器的 DNS 名稱 - *myIISVMEastUS* 和 *myIISVMWEurope*。

1. 按一下左側功能表中的 [所有資源]，然後從資源清單選取 *myIISVMEastUS*，其位於 *myResourceGroupTM1* 資源群組中。
2. 在 [概觀] 頁面的 [DNS 名稱] 底下，選取 [設定]。
3. 在 [組態] 頁面的 [DNS 名稱] 標籤之下，新增唯一的名稱，然後選取 [儲存]。
4. 針對 *myResourceGroupTM1* 資源群組中名為 *myIISVMWEurope* 的 VM，重複步驟 1-3。

### <a name="create-test-vms"></a>建立測試 VM

在本節中，您會在每個 Azure 區域 (**美國東部**和**西歐**) 中建立 VM (*mVMEastUS* 和 *myVMWestEurope*)。 當您瀏覽至網站時，您將使用這些 VM 來測試流量管理員如何將流量路由至最接近的 IIS 伺服器。

1. 在 Azure 入口網站的左上角，選取 [建立資源] > [計算] > [Windows Server 2016 VM]。
2. 針對 [基本資料] 輸入或選取下列資訊、接受其餘設定的預設值，然後選取 [建立]：

    |設定|值|
    |---|---|
    |名稱|myVMEastUS|
    |使用者名稱| 輸入您選擇的使用者名稱。|
    |密碼| 輸入您選擇的密碼。 密碼長度至少必須有 12 個字元，而且符合[定義的複雜度需求](../virtual-machines/windows/faq.md?toc=%2fazure%2fvirtual-network%2ftoc.json#what-are-the-password-requirements-when-creating-a-vm)。|
    |資源群組| 選取 [現有]，然後選取 [myResourceGroupTM1]。|
    |||

4. 在 [選擇大小] 底下選取虛擬機器大小。
5. 針對 [設定] 選取下列值，然後選取 [確定]：
    |設定|值|
    |---|---|
    |虛擬網路| 選取 [虛擬網路]，在 [建立虛擬網路] 中，針對 [名稱] 輸入 *myVNet3*，針對子網路輸入 *mySubnet*。|
    |網路安全性群組|選取 [基本]，然後在 [選取公用輸入連接埠] 下拉式清單中，選取 **HTTP** 和 **RDP** |
    |開機診斷|選取 [已停用]。|
    |||

6. 在 [摘要] 的 [建立] 底下，選取 [建立] 來開始部署虛擬機器。

7. 以下列變更再次完成步驟 1-5：

    |設定|值|
    |---|---|
    |虛擬機器名稱 | *myVMWEurope*|
    |資源群組 | 選取 [現有]，然後輸入 *myResourceGroupTM2*|
    |虛擬網路 | 選取 [虛擬網路]，在 [建立虛擬網路] 中，針對 [名稱] 輸入 *myVNet4*，針對子網路輸入 *mySubnet*。|
    |||

8. 可能需要數分鐘才會建立虛擬機器。 請等到這兩個虛擬機器都已建立，再繼續進行其餘步驟。

## <a name="create-a-traffic-manager-profile"></a>建立流量管理員設定檔
建立流量管理員設定檔，其導向使用者流量的方式是將使用者流量傳送至具有最低延遲的端點。

1. 在畫面的左上方，選取 [建立資源] > [網路] > [流量管理員設定檔] > [建立]。
2. 在 [建立流量管理員設定檔] 中，輸入或選取下列資訊、接受其餘設定的預設值，然後選取 [建立]：
    | 設定                 | 值                                              |
    | ---                     | ---                                                |
    | 名稱                   | 此名稱在 trafficmanager.net 區域內必須是唯一的，而且會產生 DNS 名稱 trafficmanager.net，用以存取您的流量管理員設定檔。                                   |
    | 路由方法          | 選取 [效能] 路由方法。                                       |
    | 訂用帳戶            | 選取您的訂用帳戶。                          |
    | 資源群組          | 選取 [新建]，然後輸入 *myResourceGroupTM1*。 |
    | 位置                | 選取 [美國東部]。  這項設定是指資源群組的位置，完全不影響將部署到全球的流量管理員設定檔。                              |
    |
  
    ![建立流量管理員設定檔](./media/tutorial-traffic-manager-improve-website-response/traffic-manager-profile.png)

## <a name="add-traffic-manager-endpoints"></a>新增流量管理員端點

新增兩部執行 IIS 伺服器的 VM (*myIISVMEastUS*  & *myIISVMWEurope*)，以將使用者流量路由傳送至最接近使用者的端點。

1. 在入口網站的搜尋列中，搜尋您在上一節建立的流量管理員設定檔名稱，然後在顯示的結果中選取設定檔。
2. 在 [流量管理員設定檔] 的 [設定] 區段中，按一下 [端點]，然後按一下 [新增]。
3. 輸入或選取下列資訊、接受其餘設定的預設值，然後選取 [確定]：

    | 設定                 | 值                                              |
    | ---                     | ---                                                |
    | 類型                    | Azure 端點                                   |
    | 名稱           | myEastUSEndpoint                                        |
    | 目標資源類型           | 公用 IP 位址                          |
    | 目標資源          | **選擇公用 IP 位址**以顯示具有相同訂用帳戶下公用 IP 位址的資源清單。 在 [資源] 中，選取名為 *myIISVMEastUS-ip* 的公用 IP 位址。 這是美國東部 IIS 伺服器 VM 的公用 IP 位址。|
    |        |           |

4. 重複步驟 2 和 3，以針對與 IIS 伺服器 VM *myIISVMWEurope* 相關聯的公用 IP 位址 *myIISVMWEurope-ip*，新增另一個名為*myWestEuropeEndpoint* 的端點。
5.  這兩個端點新增完畢後，它們會顯示在 [流量管理員設定檔] 中，而且監視狀態是 [線上]。

    ![新增流量管理員端點](./media/tutorial-traffic-manager-improve-website-response/traffic-manager-endpoint.png)
  

## <a name="test-traffic-manager-profile"></a>測試流量管理員設定檔
在本節中，您會測試流量管理員如何將使用者流量路由傳送至執行網站的最接近 VM，以提供最低延遲。 若要查看運作中的流量管理員，請完成下列步驟：
1. 決定流量管理員設定檔的 DNS 名稱。
2. 檢視流量管理員的運作，如下所示：
    - 從位於**美國東部**區域的測試 VM (*myVMEastUS*)，在網頁瀏覽器中瀏覽至您流量管理員設定檔的 DNS 名稱。
    - 從位於**西歐**區域的測試 VM (*myVMEastUS*)，在網頁瀏覽器中瀏覽至您流量管理員設定檔的 DNS 名稱。

### <a name="determine-dns-name-of-traffic-manager-profile"></a>決定流量管理員設定檔的 DNS 名稱
在本教學課程中，為了簡單起見，您可使用流量管理員設定檔的 DNS 名稱來瀏覽網站。 

您可以決定流量管理員設定檔的 DNS 名稱，如下所示：

1.  在入口網站的搜尋列中，搜尋您在上一節建立的**流量管理員設定檔**名稱。 在顯示的結果中，按一下流量管理員設定檔。
1. 按一下 [概觀] 。
2. [流量管理員設定檔] 會顯示新建立之流量管理員設定檔的 DNS 名稱。 在生產部署中，您會使用 DNS CNAME 記錄，將虛名網域名稱設定為指向流量管理員網域名稱。

   ![流量管理員 DNS 名稱](./media/tutorial-traffic-manager-improve-website-response/traffic-manager-dns-name.png)

### <a name="view-traffic-manager-in-action"></a>檢視流量管理員的運作
在本節中，您可以看到流量管理員的運作。 

1. 選取左側功能表中的 [所有資源]，然後從資源清單按一下 *myVMEastUS*，其位於 *myResourceGroupTM1* 資源群組中。
2. 在 [概觀] 頁面上，按一下 [連線]，然後在 [連線至虛擬機器] 中選取 [下載 RDP 檔案]。 
3. 開啟所下載的 rdp 檔案。 如果出現提示，請選取 [連接]。 輸入您在建立 VM 時指定的使用者名稱和密碼。 您可能需要選取 [其他選擇]，然後選取 [使用不同的帳戶]，以指定您在建立 VM 時輸入的認證。 
4. 選取 [確定] 。
5. 您可能會在登入過程中收到憑證警告。 如果您收到警告，請選取 [是] 或 [繼續] 以繼續進行連線。 
1. 在 VM *myVMEastUS* 的網頁瀏覽器中，輸入流量管理員設定檔的 DNS 名稱，以檢視您的網站。 VM 位於**美國東部**，所以您會路由至在最接近 IIS 伺服器 *myIISVMEastUS* (位於**美國東部**) 上裝載的最接近網站。

   ![測試流量管理員設定檔](./media/tutorial-traffic-manager-improve-website-response/eastus-traffic-manager-test.png)

2. 接下來，使用步驟 1-5 來連線到位於**西歐**的 VM *myVMWestEurope*，並從這個 VM 瀏覽到流量管理員設定檔網域名稱。  VM 位於**西歐**，所以您現在會路由至在最接近 IIS 伺服器 *myIISVMWEurope* (位於**西歐**) 上裝載的網站。 

   ![測試流量管理員設定檔](./media/tutorial-traffic-manager-improve-website-response/westeurope-traffic-manager-test.png)
   
## <a name="delete-the-traffic-manager-profile"></a>刪除流量管理員設定檔
如果不再需要，請刪除資源群組 (**ResourceGroupTM1** 和 **ResourceGroupTM2**)。 若要這麼做，請選取資源群組 (**ResourceGroupTM1** 或 **ResourceGroupTM2**)，然後選取 [刪除]。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [將流量散發至一組端點](traffic-manager-configure-weighted-routing-method.md)


