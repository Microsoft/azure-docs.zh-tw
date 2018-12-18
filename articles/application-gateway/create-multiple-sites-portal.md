---
title: 建立裝載多個網站的應用程式閘道 - Azure 入口網站 | Microsoft Docs
description: 了解如何使用 Azure 入口網站建立裝載多個網站的應用程式閘道。
services: application-gateway
author: vhorne
manager: jpconnock
editor: tysonn
ms.service: application-gateway
ms.topic: article
ms.workload: infrastructure-services
ms.date: 12/28/2017
ms.author: victorh
ms.openlocfilehash: 8f2e2500c39f42ebd7fefb3cec941088edf126f2
ms.sourcegitcommit: 7208bfe8878f83d5ec92e54e2f1222ffd41bf931
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/14/2018
ms.locfileid: "39045014"
---
# <a name="create-and-configure-an-application-gateway-to-host-multiple-web-sites-using-the-azure-portal"></a>使用 Azure 入口網站建立和設定應用程式閘道以裝載多個網站

您可以使用 Azure 入口網站，在建立[應用程式閘道](overview.md)時[設定裝載多個站台](multiple-site-overview.md)。 在本教學課程中，您可以使用虛擬機器定義後端位址集區。 接著，您可以根據擁有的網域來設定接聽程式和規則，確保網路流量會抵達集區中的適當伺服器。 本教學課程假設您擁有多個網域，並使用 *www.contoso.com* 和 *www.fabrikam.com* 的範例。

在本文中，您將了解：

> [!div class="checklist"]
> * 建立應用程式閘道
> * 建立後端伺服器的虛擬機器
> * 建立包含後端伺服器的後端集區
> * 建立後端接聽程式
> * 建立路由規則
> * 在網域中建立 CNAME 記錄

![多站台路由範例](./media/create-multiple-sites-portal/scenario.png)

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

    ![建立新的應用程式閘道](./media/create-multiple-sites-portal/application-gateway-create.png)

4. 接受其他設定的預設值，然後按一下 [確定]。
5. 按一下 [選擇虛擬網路]，按一下 [新建]，然後針對虛擬網路輸入這些值：

    - myVNet - 作為虛擬網路的名稱。
    - 10.0.0.0/16 - 作為虛擬網路位址空間。
    - myAGSubnet - 作為子網路名稱。
    - 10.0.0.0/24 - 作為子網路位址空間。

    ![建立虛擬網路](./media/create-multiple-sites-portal/application-gateway-vnet.png)

6. 按一下 [確定] 以建立虛擬網路和子網路。
7. 依序按一下 [選擇公用 IP 位址]、[新建]，然後輸入公用 IP 位址的名稱。 在此範例中，公用 IP 位址名為 myAGPublicIPAddress。 接受其他設定的預設值，然後按一下 [確定]。
8. 接受接聽程式設定的預設值，將 Web 應用程式防火牆保持為停用，然後按一下 [確定]。
9. 檢閱 [摘要] 頁面上的設定，然後按一下 [確定] 以建立網路資源和應用程式閘道。 建立應用程式閘道可能需要幾分鐘的時間，請等候部署成功完成後，再繼續進行至下一節。

### <a name="add-a-subnet"></a>新增子網路

1. 按一下左側功能表中的 [所有資源]，然後從 [資源] 清單中按一下 [myVNet]。
2. 按一下 [子網路]，然後按一下 [子網路]。

    ![建立子網路](./media/create-multiple-sites-portal/application-gateway-subnet.png)

3. 輸入 myBackendSubnet 作為子網路的名稱，然後按一下 [確定]。

## <a name="create-virtual-machines"></a>建立虛擬機器

在此範例中，您要建立兩個虛擬機器，作為應用程式閘道的後端伺服器。 您也要在虛擬機器上安裝 IIS，確認會正確地路由流量。

1. 按一下 [新增] 。
2. 按一下 [計算]，然後選取 [精選] 清單中的 [Windows Server 2016 Datacenter]。
3. 針對虛擬機器，請輸入這些值：

    - *contosoVM* - 作為虛擬機器的名稱。
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

    ![安裝自訂延伸模組](./media/create-multiple-sites-portal/application-gateway-extension.png)

2. 執行下列命令以在虛擬機器上安裝 IIS： 

    ```azurepowershell-interactive
    $publicSettings = @{ "fileUris" = (,"https://raw.githubusercontent.com/Azure/azure-docs-powershell-samples/master/application-gateway/iis/appgatewayurl.ps1");  "commandToExecute" = "powershell -ExecutionPolicy Unrestricted -File appgatewayurl.ps1" }
    Set-AzureRmVMExtension `
      -ResourceGroupName myResourceGroupAG `
      -Location eastus `
      -ExtensionName IIS `
      -VMName contosoVM `
      -Publisher Microsoft.Compute `
      -ExtensionType CustomScriptExtension `
      -TypeHandlerVersion 1.4 `
      -Settings $publicSettings
    ```

3. 建立第二個虛擬機器，並使用您剛完成的步驟來安裝 IIS。 輸入 *fabrikamVM* 作為名稱，並作為 Set-AzureRmVMExtension 中的 VMName 值。

## <a name="create-backend-pools-with-the-virtual-machines"></a>建立包含虛擬機器的後端集區

1. 按一下 [所有資源]，然後按一下 [myAppGateway]。
2. 依序按一下 [後端集區] 和 [新增]。
3. 輸入 contosoPool 的名稱，然後使用 [新增目標] 新增 contosoVM。

    ![新增後端伺服器](./media/create-multiple-sites-portal/application-gateway-multisite-backendpool.png)

4. 按一下 [確定]。
5. 依序按一下 [後端集區] 和 [新增]。
6. 使用您剛才完成的步驟，建立含有 fabrikamVM 的 fabrikamPool。

## <a name="create-backend-listeners"></a>建立後端接聽程式

1. 按一下 [接聽程式]，然後按一下 [多站台]。
2. 為接聽程式輸入這些值：
    
    - contosoListener - 作為接聽程式的名稱。
    - www.contoso.com - 以您的網域名稱取代此主機名稱範例。

3. 按一下 [確定]。
4. 使用 fabrikamListener 的名稱建立第二個接聽程式，然後使用您的第二個網域名稱。 此範例中使用 www.fabrikam.com。

## <a name="create-routing-rules"></a>建立路由規則

會以規則列出的順序進行處理，而且不論精確性為何，都會使用相符的第一個規則將流量進行導向。 例如，如果您在相同的連接埠上同時使用基本接聽程式的規則和多站台接聽程式的規則，則必須將多站台接聽程式的規則列於基本接聽程式的規則之前，多站台規則才能如預期般運作。 

在此範例中，您會建立兩個新規則，並刪除您在建立應用程式閘道時所建立的預設規則。 

1. 按一下 [規則]，然後按一下 [基本]。
2. 輸入 *contosoRule* 作為名稱。
3. 選取 contosoListener 作為接聽程式。
4. 選取 contosoPool 作為後端集區。

    ![建立路徑型規則](./media/create-multiple-sites-portal/application-gateway-multisite-rule.png)

5. 按一下 [確定]。
6. 使用 fabrikamRule、fabrikamListener 及 *fabrikamPool* 的名稱，建立第二個規則。
7. 按一下名為 rule1 的預設規則，然後按一下 [刪除]，即可刪除。

## <a name="create-a-cname-record-in-your-domain"></a>在網域中建立 CNAME 記錄

在以公用 IP 位址建立應用程式閘道之後，您可以取得 DNS 位址並用以在網域中建立 CNAME 記錄。 不建議使用 A-records，因為重新啟動應用程式閘道時，可能會變更 VIP。

1. 按一下 [所有資源]，然後按一下 [myAGPublicIPAddress]。

    ![記錄應用程式閘道 DNS 位址](./media/create-multiple-sites-portal/application-gateway-multisite-dns.png)

2. 複製 DNS 位址，並用來作為網域中新 CNAME 記錄的值。

## <a name="test-the-application-gateway"></a>測試應用程式閘道

1. 在瀏覽器的網址列中輸入您的網域名稱。 例如，http://www.contoso.com。

    ![在應用程式閘道中測試 contoso 網站](./media/create-multiple-sites-portal/application-gateway-iistest.png)

2. 將位址變更為您的另一個網域，您應該會看到類似下列的範例：

    ![測試應用程式閘道中的 fabrikam 網站](./media/create-multiple-sites-portal/application-gateway-iistest2.png)

## <a name="next-steps"></a>後續步驟

在本文中，您已了解如何：

> [!div class="checklist"]
> * 建立應用程式閘道
> * 建立後端伺服器的虛擬機器
> * 建立包含後端伺服器的後端集區
> * 建立後端接聽程式
> * 建立路由規則
> * 在網域中建立 CNAME 記錄
