---
title: 建立包含 Web 應用程式防火牆的應用程式閘道 - Azure 入口網站 | Microsoft Docs
description: 了解如何使用 Azure 入口網站建立具有 Web 應用程式防火牆的應用程式閘道。
services: application-gateway
author: vhorne
manager: jpconnock
editor: tysonn
tags: azure-resource-manager
ms.service: application-gateway
ms.topic: article
ms.workload: infrastructure-services
ms.date: 01/26/2018
ms.author: victorh
ms.openlocfilehash: 9967813b193159b68aa0f008dae4440aa6e533dc
ms.sourcegitcommit: c47ef7899572bf6441627f76eb4c4ac15e487aec
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/04/2018
ms.locfileid: "33207386"
---
# <a name="create-an-application-gateway-with-a-web-application-firewall-using-the-azure-portal"></a>使用 Azure 入口網站建立包含 Web 應用程式防火牆的應用程式閘道

您可以使用 Azure 入口網站建立包含 [Web 應用程式防火牆](application-gateway-web-application-firewall-overview.md) (WAF) 的[應用程式閘道](application-gateway-introduction.md)。 WAF 會使用 [OWASP](https://www.owasp.org/index.php/Category:OWASP_ModSecurity_Core_Rule_Set_Project) 規則來保護您的應用程式。 這些規則包括防禦諸如 SQL 插入攻擊、跨網站指令碼攻擊，以及工作階段劫持等攻擊。

在本文中，您將了解：

> [!div class="checklist"]
> * 建立已啟用 WAF 的應用程式閘道
> * 建立用來作為後端伺服器的虛擬機器
> * 建立儲存體帳戶和設定診斷

![Web 應用程式防火牆範例](./media/application-gateway-web-application-firewall-portal/scenario-waf.png)

## <a name="log-in-to-azure"></a>登入 Azure

在 [http://portal.azure.com](http://portal.azure.com) 上登入 Azure 入口網站

## <a name="create-an-application-gateway"></a>建立應用程式閘道

需要虛擬網路，才能在您所建立的資源之間進行通訊。 這個範例中會建立兩個子網路：一個用於應用程式閘道，另一個用於後端伺服器。 您建立應用程式閘道時，可以同時建立虛擬網路。

1. 按一下 Azure 入口網站左上角的 [新增]。
2. 在 [精選] 清單中選取 [網路]，然後選取 [應用程式閘道]。
3. 針對應用程式閘道輸入這些值：

    - myAppGateway - 作為應用程式閘道的名稱。
    - myResourceGroupAG - 作為新的資源群組。
    - 選取 [WAF] 作為應用程式閘道層。

    ![建立新的應用程式閘道](./media/application-gateway-web-application-firewall-portal/application-gateway-create.png)

4. 接受其他設定的預設值，然後按一下 [確定]。
5. 按一下 [選擇虛擬網路]，按一下 [新建]，然後針對虛擬網路輸入這些值：

    - myVNet - 作為虛擬網路的名稱。
    - 10.0.0.0/16 - 作為虛擬網路位址空間。
    - myAGSubnet - 作為子網路名稱。
    - 10.0.0.0/24 - 作為子網路位址空間。

    ![建立虛擬網路](./media/application-gateway-web-application-firewall-portal/application-gateway-vnet.png)

6. 按一下 [確定] 以建立虛擬網路和子網路。
7. 依序按一下 [選擇公用 IP 位址]、[新建]，然後輸入公用 IP 位址的名稱。 在此範例中，公用 IP 位址名為 myAGPublicIPAddress。 接受其他設定的預設值，然後按一下 [確定]。
8. 接受接聽程式設定的預設值，將 Web 應用程式防火牆保持為停用，然後按一下 [確定]。
9. 檢閱 [摘要] 頁面上的設定，然後按一下 [確定] 以建立網路資源和應用程式閘道。 建立應用程式閘道可能需要幾分鐘的時間，請等候部署成功完成後，再繼續進行至下一節。

### <a name="add-a-subnet"></a>新增子網路

1. 按一下左側功能表中的 [所有資源]，然後從 [資源] 清單中按一下 [myVNet]。
2. 按一下 [子網路]，然後按一下 [子網路]。

    ![建立子網路](./media/application-gateway-web-application-firewall-portal/application-gateway-subnet.png)

3. 輸入 myBackendSubnet 作為子網路的名稱，然後按一下 [確定]。

## <a name="create-backend-servers"></a>建立後端伺服器

在此範例中，您要建立兩個虛擬機器，作為應用程式閘道的後端伺服器。 您也可以在虛擬機器上安裝 IIS，以確認成功建立應用程式閘道。

### <a name="create-a-virtual-machine"></a>建立虛擬機器

1. 按一下 [新增] 。
2. 按一下 [計算]，然後選取 [精選] 清單中的 [Windows Server 2016 Datacenter]。
3. 針對虛擬機器，請輸入這些值：

    - myVM - 作為虛擬機器的名稱。
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

    ![安裝自訂延伸模組](./media/application-gateway-web-application-firewall-portal/application-gateway-extension.png)

2. 執行下列命令以在虛擬機器上安裝 IIS： 

    ```azurepowershell-interactive
    Set-AzureRmVMExtension `
      -ResourceGroupName myResourceGroupAG `
      -ExtensionName IIS `
      -VMName myVM `
      -Publisher Microsoft.Compute `
      -ExtensionType CustomScriptExtension `
      -TypeHandlerVersion 1.4 `
      -SettingString '{"commandToExecute":"powershell Add-WindowsFeature Web-Server; powershell Add-Content -Path \"C:\\inetpub\\wwwroot\\Default.htm\" -Value $($env:computername)"}' `
      -Location EastUS
    ```

3. 建立第二個虛擬機器，並使用您剛完成的步驟來安裝 IIS。 輸入 myVM2 作為其名稱，及作為 Set-AzureRmVMExtension 中的 VMName。

### <a name="add-backend-servers"></a>新增後端伺服器

1. 按一下 [所有資源]，然後按一下 [myAppGateway]。
2. 按一下 [後端集區]。 已使用應用程式閘道自動建立預設集區。 按一下 [appGateayBackendPool]。
3. 按一下 [新增目標]，將您所建立的每個虛擬機器新增至後端集區。

    ![新增後端伺服器](./media/application-gateway-web-application-firewall-portal/application-gateway-backend.png)

4. 按一下 [檔案] 。

## <a name="create-a-storage-account-and-configure-diagnostics"></a>建立儲存體帳戶並設定診斷

## <a name="create-a-storage-account"></a>建立儲存體帳戶

在本教學課程中，應用程式閘道會使用儲存體帳戶來儲存資料，以達到偵測與預防的目的。 您也可以使用 Log Analytics 或事件中樞來記錄資料。

1. 按一下 Azure 入口網站左上角的 [新增]。
2. 選取 [儲存體]，然後選取 [儲存體帳戶 - blob、檔案、資料表、佇列]。
3. 輸入儲存體帳戶的名稱，針對資源群組選取 [使用現有的]，然後選取 [myResourceGroupAG]。 在此範例中，儲存體帳戶名稱是 myagstore1。 接受其他設定的預設值，然後按一下 [建立]。

## <a name="configure-diagnostics"></a>設定診斷

設定診斷以將資料記錄到 ApplicationGatewayAccessLog、ApplicationGatewayPerformanceLog 和 ApplicationGatewayFirewallLog 記錄。

1. 在左側功能表中，按一下 [所有資源]，然後選取 [myAppGateway]。
2. 在 [監視] 底下，按一下 [診斷記錄]。
3. 按一下 [新增診斷設定]。
4. 輸入 myDiagnosticsSettings 作為診斷設定的名稱。
5. 選取 [保存到儲存體帳戶]，然後按一下 [設定] 以選取您先前建立的 myagstore1 儲存體帳戶。
6. 選取要收集的應用程式閘道記錄並加以保留。
7. 按一下 [檔案] 。

    ![設定診斷](./media/application-gateway-web-application-firewall-portal/application-gateway-diagnostics.png)

## <a name="test-the-application-gateway"></a>測試應用程式閘道

1. 在 [概觀] 畫面上尋找應用程式閘道的公用 IP 位址。 按一下 [所有資源]，然後按一下 [myAGPublicIPAddress]。

    ![記錄應用程式閘道公用 IP 位址](./media/application-gateway-web-application-firewall-portal/application-gateway-record-ag-address.png)

2. 將公用 IP 位址複製並貼到您瀏覽器的網址列。

    ![測試應用程式閘道](./media/application-gateway-web-application-firewall-portal/application-gateway-iistest.png)

## <a name="next-steps"></a>後續步驟

在本文中，您已了解如何：

> [!div class="checklist"]
> * 已啟用建立包含 WAF 的應用程式閘道
> * 建立用來作為後端伺服器的虛擬機器
> * 建立儲存體帳戶和設定診斷

若要深入了解應用程式閘道和其相關聯的資源，請繼續進行操作說明文章。