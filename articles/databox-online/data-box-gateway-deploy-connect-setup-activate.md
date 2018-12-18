---
title: 在 Azure 入口網站中連線、設定和啟動 Azure 資料箱閘道 | Microsoft Docs
description: 部署資料箱閘道的第三個教學課程將說明如何連線、設定和啟動您的虛擬裝置。
services: databox
author: alkohli
ms.service: databox
ms.subservice: gateway
ms.topic: article
ms.date: 09/27/2018
ms.author: alkohli
ms.openlocfilehash: 2126871472b044f9b8c0df99c7cb14df348eab0e
ms.sourcegitcommit: c282021dbc3815aac9f46b6b89c7131659461e49
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/12/2018
ms.locfileid: "49166741"
---
# <a name="tutorial-connect-set-up-activate-azure-data-box-gateway-preview"></a>教學課程：連線、設定、啟用 Azure 資料箱閘道 (預覽) 

## <a name="introduction"></a>簡介

本教學課程說明如何使用本機 Web UI 連線、設定和啟動您的資料箱閘道裝置。 

設定和啟用程序可能需要約 10 分鐘才能完成。 

在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 連線至虛擬裝置
> * 設定並啟動虛擬裝置

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。


> [!IMPORTANT]
> - 資料箱閘道處於預覽狀態。 部署訂購並部署此解決方案之前，請檢閱 [Azure 預覽版使用條款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。 


## <a name="prerequisites"></a>必要條件

在您設定及安裝資料箱閘道之前，請先確定：

* 您已根據[在 Hyper-V 中佈建資料箱閘道](data-box-gateway-deploy-provision-hyperv.md)或[在 VMware 中佈建資料箱閘道](data-box-gateway-deploy-provision-vmware.md)中的詳細說明，佈建虛擬裝置並取得其連線的 URL。
* 您具有先前建立用以管理資料箱閘道裝置的資料箱閘道服務適用的啟用金鑰。 如需詳細資訊，請移至[準備部署 Azure 資料箱閘道](data-box-gateway-deploy-prep.md)。

<!--* If this is the second or subsequent virtual device that you are registering with an existing StorSimple Device Manager service, you should have the service data encryption key. This key was generated when the first device was successfully registered with this service. If you have lost this key, see [Get the service data encryption key](storsimple-ova-web-ui-admin.md#get-the-service-data-encryption-key) for your Data Box Gateway.-->

## <a name="connect-to-the-local-web-ui-setup"></a>連線至本機 Web UI 設定 

1. 開啟瀏覽器視窗，並連接至本機 Web UI。 輸入：
   
   [https://ip-address-of-network-interface](https://ip-address-of-network-interface)
   
   請使用您在上一個教學課程中記下的連線 URL。 您會看到錯誤指出網站的安全性憑證有問題。 請按一下 [繼續瀏覽此網頁] 。 (這些步驟可能會隨著使用的瀏覽器而不同。)
   
    ![](./media/data-box-gateway-deploy-connect-setup-activate/image2.png)

2. 登入虛擬裝置的 Web UI。 預設密碼為 *Password1*。 
   
    ![](./media/data-box-gateway-deploy-connect-setup-activate/image3.png)

3. 系統會提示您變更裝置系統管理員密碼。 請輸入包含 8 到 16 個字元的新密碼。 此密碼必須包含下列幾項中的 3 項：大寫、小寫、數字和特殊字元。

    ![](./media/data-box-gateway-deploy-connect-setup-activate/image4.png)

您現在位於裝置的**儀表板**上。

## <a name="set-up-and-activate-the-virtual-device"></a>設定並啟動虛擬裝置
 
1. 在儀表板中，您可以移至對資料箱閘道服務設定和註冊虛擬裝置所需的各種設定。 [網路設定]、[Web Proxy 設定] 及 [時間設定] 是可省略的。 只有 [裝置名稱] 和 [雲端設定] 是必要的設定。
   
    ![](./media/data-box-gateway-deploy-connect-setup-activate/image5.png)

2. 在 [裝置名稱] 頁面中，為您的裝置設定易記名稱。 易記名稱的長度可以是 1 至 15 個字元，並且可包含字母、數字和連字號。

    ![](./media/data-box-gateway-deploy-connect-setup-activate/image6.png)

3. (選擇性) 設定您的 [網路設定]。 您會看到 1 或多個網路介面，視您在基礎虛擬機器中設定的數量而定。 以下是已啟用一個網路介面的虛擬裝置所顯示的 [網路設定] 頁面。
    
    ![](./media/data-box-gateway-deploy-connect-setup-activate/image7.png)
   
    設定網路設定時，請留意下列事項：

    - 如果您的環境中啟用了 DHCP，則會自動設定網路介面。 因此，系統會自動指派 IP 位址、子網路、閘道 和 DNS。
    - 如果未啟用 DHCP，您可以視需要指派靜態 IP。
    - 您可以將網路介面設定為 IPv4。
   
4. (可省略) 設定 Web Proxy 伺服器。 雖然 Web Proxy 設定是選用的，但請注意，如果您使用 Web Proxy，就只能在此處設定它。
   
   ![](./media/data-box-gateway-deploy-connect-setup-activate/image8.png)
   
   在 [Web Proxy]  頁面中：
   
   1. 以下列格式提供 **Web Proxy URL**：「 http://&lt;host-IP 位址」或「FDQN&gt;:連接埠號碼」。 請注意，此處不支援 HTTPS URL。
   2. 將 [驗證] 指定為 [基本] 或 [無]。
   3. 如果要使用驗證功能，您也必須提供 [使用者名稱] 和 [密碼]。
   4. 按一下 [套用]。 這將會驗證並套用您設定的 Web Proxy 設定。

5. (可省略) 設定裝置的時間設定，例如時區，以及主要和次要 NTP 伺服器。 NTP 伺服器是必要的，因為您的裝置必須讓時間同步，才能與您的雲端服務提供者進行驗證。
    
    ![](./media/data-box-gateway-deploy-connect-setup-activate/image9.png)
    
    在 [時間設定]  頁面中：
    
    1. 根據裝置部署的地理位置，從下拉式清單中選取 [時區]  。 裝置的預設時區是太平洋標準時間。 裝置將針對所有排程的操作使用這個時區。
    2. 指定裝置的 [主要 NTP 伺服器]  ，或是接受預設值 time.windows.com。 請確定您的網路允許 NTP 流量從您的資料中心通過網際網路。
    3. (選擇性) 指定裝置的 [次要 NTP 伺服器]  。
    4. 按一下 [Apply (套用)] 。 這將會驗證並套用您設定的時間設定。

6. 在 [雲端設定] 頁面中，對 Azure 入口網站中的資料箱閘道服務啟動您的裝置。
    
    1. 請輸入您在[取得啟用金鑰](data-box-gateway-deploy-prep.md#get-the-activation-key)中為資料箱閘道取得的 [啟用金鑰]。

    2. 按一下 [啟用]。 
       
         ![](./media/data-box-gateway-deploy-connect-setup-activate/image10.png)
    
    3. 您可能需要等待一分鐘，裝置才會成功啟動。 啟用之後，此頁面會更新，並指出裝置已成功啟動。


## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解資料箱閘道的相關主題，像是：

> [!div class="checklist"]
> * 連線至虛擬裝置
> * 設定並啟動虛擬裝置


請繼續進行下一個教學課程，以了解如何使用資料箱閘道傳輸資料。

> [!div class="nextstepaction"]
> [使用資料箱閘道傳輸資料](./data-box-gateway-deploy-add-shares.md)。
