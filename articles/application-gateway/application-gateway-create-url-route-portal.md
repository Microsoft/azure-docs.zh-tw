---
title: 建立包含 URL 路徑型路由規則的應用程式閘道 - Azure 入口網站
description: 了解如何使用 Azure 入口網站來建立應用程式閘道和虛擬機器擴展集的 URL 路徑型路由規則。
services: application-gateway
author: vhorne
manager: jpconnock
tags: azure-resource-manager
ms.service: application-gateway
ms.topic: article
ms.workload: infrastructure-services
ms.date: 3/26/2018
ms.author: victorh
ms.openlocfilehash: 5bec7be5f7ad744960d2602aaf24fec51d869267
ms.sourcegitcommit: 7208bfe8878f83d5ec92e54e2f1222ffd41bf931
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/14/2018
ms.locfileid: "39056244"
---
# <a name="create-an-application-gateway-with-path-based-routing-rules-using-the-azure-portal"></a>使用 Azure 入口網站建立包含 URL 路徑型路由規則的應用程式閘道

您可以使用 Azure 入口網站，在建立[應用程式閘道](application-gateway-introduction.md)時設定 [URL 路徑型路由規則](application-gateway-url-route-overview.md)。 在本教學課程中，您可以使用虛擬機器建立後端集區。 然後，您可以建立路由規則，確保 Web 流量會抵達集區中的適當伺服器。

在本文中，您將了解：

> [!div class="checklist"]
> * 建立應用程式閘道
> * 建立後端伺服器的虛擬機器
> * 建立包含後端伺服器的後端集區
> * 建立後端接聽程式
> * 建立路徑型路由規則

![URL 路由範例](./media/application-gateway-create-url-route-portal/scenario.png)

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

## <a name="log-in-to-azure"></a>登入 Azure

在 [http://portal.azure.com](http://portal.azure.com) 上登入 Azure 入口網站

## <a name="create-an-application-gateway"></a>建立應用程式閘道

需要虛擬網路，才能在您所建立的資源之間進行通訊。 這個範例中會建立兩個子網路：一個用於應用程式閘道，另一個用於後端伺服器。 您建立應用程式閘道時，可以同時建立虛擬網路。

1. 按一下 Azure 入口網站左上角的 [新增]。
2. 在 [精選] 清單中選取 [網路]，然後選取 [應用程式閘道]。
3. 針對應用程式閘道輸入這些值：

    - myAppGateway - 作為應用程式閘道的名稱。
    - myResourceGroupAG - 作為新資源群組。

    ![建立新的應用程式閘道](./media/application-gateway-create-url-route-portal/application-gateway-create.png)

4. 接受其他設定的預設值，然後按一下 [確定]。
5. 按一下 [選擇虛擬網路]，按一下 [新建]，然後針對虛擬網路輸入這些值：

    - myVNet - 作為虛擬網路的名稱。
    - 10.0.0.0/16 - 作為虛擬網路位址空間。
    - myAGSubnet - 作為子網路名稱。
    - 10.0.0.0/24 - 作為子網路位址空間。

    ![建立虛擬網路](./media/application-gateway-create-url-route-portal/application-gateway-vnet.png)

6. 按一下 [確定] 以建立虛擬網路和子網路。
7. 依序按一下 [選擇公用 IP 位址]、[新建]，然後輸入公用 IP 位址的名稱。 在此範例中，公用 IP 位址名為 myAGPublicIPAddress。 接受其他設定的預設值，然後按一下 [確定]。
8. 接受接聽程式設定的預設值，將 Web 應用程式防火牆保持為停用，然後按一下 [確定]。
9. 檢閱 [摘要] 頁面上的設定，然後按一下 [確定] 以建立網路資源和應用程式閘道。 建立應用程式閘道可能需要幾分鐘的時間，請等候部署成功完成後，再繼續進行至下一節。

### <a name="add-a-subnet"></a>新增子網路

1. 按一下左側功能表中的 [所有資源]，然後從 [資源] 清單中按一下 [myVNet]。
2. 按一下 [子網路]，然後按一下 [子網路]。

    ![建立子網路](./media/application-gateway-create-url-route-portal/application-gateway-subnet.png)

3. 輸入 myBackendSubnet 作為子網路的名稱，然後按一下 [確定]。

## <a name="create-virtual-machines"></a>建立虛擬機器

在此範例中，您要建立三個虛擬機器，作為應用程式閘道的後端伺服器。 您也可以在虛擬機器上安裝 IIS，以確認成功建立應用程式閘道。

1. 按一下 [新增] 。
2. 按一下 [計算]，然後選取 [精選] 清單中的 [Windows Server 2016 Datacenter]。
3. 針對虛擬機器，請輸入這些值：

    - myVM1 - 作為虛擬機器的名稱。
    - azureuser - 作為系統管理員使用者名稱。
    - *Azure123456!* 作為密碼。
    - 選取 [使用現有的]，然後選取 [myResourceGroupAG]。

4. 按一下 [確定]。
5. 選取 [DS1_V2] 作為虛擬機器的大小，然後按一下 [選取]。
6. 確定您已選取 [myVNet] 作為虛擬網路，而且子網路是 [myBackendSubnet]。 
7. 按一下 [停用] 來停用開機診斷。
8. 按一下 [確定]，檢閱 [摘要] 頁面上的設定，然後按一下 [建立]。

### <a name="install-iis"></a>安裝 IIS

1. 開啟互動式殼層，並確定它是設定為 **PowerShell**。

    ![安裝自訂延伸模組](./media/application-gateway-create-url-route-portal/application-gateway-extension.png)

2. 執行下列命令以在虛擬機器上安裝 IIS： 

    ```azurepowershell-interactive
    $publicSettings = @{ "fileUris" = (,"https://raw.githubusercontent.com/Azure/azure-docs-powershell-samples/master/application-gateway/iis/appgatewayurl.ps1");  "commandToExecute" = "powershell -ExecutionPolicy Unrestricted -File appgatewayurl.ps1" }
    Set-AzureRmVMExtension `
      -ResourceGroupName myResourceGroupAG `
      -Location eastus `
      -ExtensionName IIS `
      -VMName myVM1 `
      -Publisher Microsoft.Compute `
      -ExtensionType CustomScriptExtension `
      -TypeHandlerVersion 1.4 `
      -Settings $publicSettings
    ```

3. 再多建立兩個虛擬機器，並使用您剛完成的步驟來安裝 IIS。 輸入 myVM2 和 myVM3 作為名稱，及作為 Set-AzureRmVMExtension 中的 VMName 值。

## <a name="create-backend-pools-with-the-virtual-machines"></a>建立包含虛擬機器的後端集區

1. 按一下 [所有資源]，然後按一下 [myAppGateway]。
2. 按一下 [後端集區]。 已自動建立具有應用程式閘道的預設集區。 按一下 [appGatewayBackendPool]。
3. 按一下 [新增目標] 將 myVM1 新增至 appGatewayBackendPool。

    ![新增後端伺服器](./media/application-gateway-create-url-route-portal/application-gateway-backend.png)

4. 按一下 [檔案] 。
5. 依序按一下 [後端集區] 和 [新增]。
6. 輸入 imagesBackendPool 的名稱，並使用 [新增目標] 來新增 myVM2。
7. 按一下 [確定]。
8. 再次按一下 [新增]，以 videoBackendPool 名稱新增另一個後端集區，並將 myVM3 新增給該後端集區。

## <a name="create-a-backend-listener"></a>建立後端接聽程式

1. 按一下 [接聽程式]，然後按一下 [基本]。
2. 依序輸入 myBackendListener 作為名稱、myFrontendPort 作為前端連接埠的名稱，以及 8080 作為接聽程式的通訊埠。
3. 按一下 [確定]。

## <a name="create-a-path-based-routing-rule"></a>建立路徑型路由規則

1. 按一下 [規則]，然後按一下 [路徑型]。
2. 輸入 rule2 作為名稱。
3. 輸入 Images 作為第一個路徑的名稱。 輸入 */images/*\* 作為路徑。 選取 **imagesBackendPool** 作為後端集區。
4. 輸入 Video 作為第二個路徑的名稱。 輸入 */video/*\* 作為路徑。 選取 **videoBackendPool** 作為後端集區。

    ![建立路徑型規則](./media/application-gateway-create-url-route-portal/application-gateway-route-rule.png)

5. 按一下 [確定]。

## <a name="test-the-application-gateway"></a>測試應用程式閘道

1. 按一下 [所有資源]，然後按一下 [myAGPublicIPAddress]。

    ![記錄應用程式閘道公用 IP 位址](./media/application-gateway-create-url-route-portal/application-gateway-record-ag-address.png)

2. 將公用 IP 位址複製並貼到您瀏覽器的網址列。 例如，http://http://40.121.222.19。

    ![在應用程式閘道中測試基底 URL](./media/application-gateway-create-url-route-portal/application-gateway-iistest.png)

3. 將 URL 變更為 http://&lt;ip-address&gt;:8080/images/test.htm，將 &lt;ip-address&gt; 取代為您的 IP 位址，然後您會看到類似下列的範例：

    ![在應用程式閘道中測試影像 URL](./media/application-gateway-create-url-route-portal/application-gateway-iistest-images.png)

4. 將 URL 變更為 http://&lt;ip-address&gt;:8080/video/test.htm，將 &lt;ip-address&gt; 取代為您的 IP 位址，然後您會看到類似下列的範例：

    ![在應用程式閘道中測試影片 URL](./media/application-gateway-create-url-route-portal/application-gateway-iistest-video.png)

## <a name="next-steps"></a>後續步驟

在本文中，您已了解如何

> [!div class="checklist"]
> * 建立應用程式閘道
> * 建立後端伺服器的虛擬機器
> * 建立包含後端伺服器的後端集區
> * 建立後端接聽程式
> * 建立路徑型路由規則

若要深入了解應用程式閘道和其相關聯的資源，請繼續進行說明文章。
