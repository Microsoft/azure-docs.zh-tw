---
title: 設定雲端服務的 SSL | Microsoft Docs
description: 了解如何為 Web 角色指定 HTTPS 端點，以及如何上傳 SSL 憑證來保護應用程式的安全。 這些範例使用 Azure 入口網站。
services: cloud-services
documentationcenter: .net
author: jpconnock
manager: timlt
editor: ''
ms.assetid: 371ba204-48b6-41af-ab9f-ed1d64efe704
ms.service: cloud-services
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/26/2017
ms.author: jeconnoc
ms.openlocfilehash: e3e7d271375cd9c3f49d8fedd963b5234dab7902
ms.sourcegitcommit: e0a678acb0dc928e5c5edde3ca04e6854eb05ea6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/13/2018
ms.locfileid: "39001519"
---
# <a name="configuring-ssl-for-an-application-in-azure"></a>在 Azure 設定應用程式的 SSL

安全通訊端層 (SSL) 加密是最常用來保護在網際網路上傳送之資料的方法。 此常見工作會討論如何為 Web 角色指定 HTTPS 端點，以及如何上傳 SSL 憑證來保護應用程式的安全。

> [!NOTE]
> 此工作的程序適用於 Azure 雲端服務，若為應用程式服務，請參閱 [此處](../app-service/app-service-web-tutorial-custom-ssl.md)。
>

此工作使用生產部署。 本主題最後將提供關於如何使用預備部署的資訊。

如果尚未建立雲端服務，請先閱讀 [這裡](cloud-services-how-to-create-deploy-portal.md) 。

## <a name="step-1-get-an-ssl-certificate"></a>步驟 1：取得 SSL 憑證
若要設定應用程式的 SSL，首先您需要取得已由憑證授權單位 (CA) (專門核發憑證的受信任第三方) 簽署的 SSL 憑證。 如果您還沒有此類憑證，則必須向銷售 SSL 憑證的公司取得。

憑證必須符合 Azure 中對於 SSL 憑證的下列要求：

* 憑證必須包含私密金鑰。
* 憑證必須是為了進行金鑰交換而建立，且可匯出成個人資訊交換檔 (.pfx)。
* 憑證的主體名稱必須符合用來存取雲端服務的網域。 您無法向憑證授權單位 (CA) 取得 cloudapp.net 網域的 SSL 憑證。 您必須取得要在存取您的服務時使用的自訂網域名稱。 當您向 CA 要求憑證時，憑證的主體名稱必須符合用來存取應用程式的自訂網域名稱。 例如，如果您的自訂網域名稱為 **contoso.com**，則您需向 CA 要求 ***.contoso.com** 或 **www.contoso.com** 憑證。
* 憑證至少必須以 2048 位元加密。

基於測試目的，您可以 [建立](cloud-services-certs-create.md) 並使用自我簽署憑證。 自我簽署憑證不是由 CA 驗證，因此可以使用 cloudapp.net 網域做為網站 URL。 例如，以下工作即使用自我簽署憑證，該憑證中使用的一般名稱 (CN) 為 **sslexample.cloudapp.net**。

接下來，您必須在服務定義檔與服務組態檔中加入憑證的相關資訊。

<a name="modify"> </a>

## <a name="step-2-modify-the-service-definition-and-configuration-files"></a>步驟 2：修改服務定義檔和組態檔
您的應用程式必須已設定為使用憑證，而且您必須新增 HTTPS 端點。 因此，您需要更新服務定義檔與服務組態檔。

1. 在開發環境中，開啟服務定義檔 (CSDEF)、在 [WebRole] 區段內新增 [Certificates] 區段，並新增下列憑證 (及中繼憑證) 相關資訊：

   ```xml
    <WebRole name="CertificateTesting" vmsize="Small">
    ...
        <Certificates>
            <Certificate name="SampleCertificate"
                        storeLocation="LocalMachine"
                        storeName="My"
                        permissionLevel="limitedOrElevated" />
            <!-- IMPORTANT! Unless your certificate is either
            self-signed or signed directly by the CA root, you
            must include all the intermediate certificates
            here. You must list them here, even if they are
            not bound to any endpoints. Failing to list any of
            the intermediate certificates may cause hard-to-reproduce
            interoperability problems on some clients.-->
            <Certificate name="CAForSampleCertificate"
                        storeLocation="LocalMachine"
                        storeName="CA"
                        permissionLevel="limitedOrElevated" />
        </Certificates>
    ...
    </WebRole>
    ```

   **Certificates** 區段定義憑證的名稱、位置，以及其所在的存放區名稱。

   權限 (`permisionLevel` 屬性) 可設為下列其中一個值：

   | 權限值 | 說明 |
   | --- | --- |
   | limitedOrElevated |**(預設值)** 所有角色處理序都可以存取私密金鑰。 |
   | elevated |只有較高權限的處理序可以存取私密金鑰。 |

2. 在服務定義檔中，於 **Endpoints** 區段內新增 **InputEndpoint** 元素，以啟用 HTTPS：

   ```xml
    <WebRole name="CertificateTesting" vmsize="Small">
    ...
        <Endpoints>
            <InputEndpoint name="HttpsIn" protocol="https" port="443"
                certificate="SampleCertificate" />
        </Endpoints>
    ...
    </WebRole>
    ```

3. 在服務定義檔中，於 **Sites** 區段內新增 **Binding** 元素。 此項目會新增 HTTPS 繫結，以將端點對應至您的站台：

   ```xml
    <WebRole name="CertificateTesting" vmsize="Small">
    ...
        <Sites>
            <Site name="Web">
                <Bindings>
                    <Binding name="HttpsIn" endpointName="HttpsIn" />
                </Bindings>
            </Site>
        </Sites>
    ...
    </WebRole>
    ```

   如此即已對服務定義檔完成所有必要變更，但是您仍然需要將憑證資訊新增至服務組態檔。
4. 在服務組態檔 (CSCFG) ServiceConfiguration.Cloud.cscfg 中，新增**憑證**值以取代您的憑證。 下列程式碼範例會提供 [憑證] 區段的詳細資料，指紋值除外。

   ```xml
    <Role name="Deployment">
    ...
        <Certificates>
            <Certificate name="SampleCertificate"
                thumbprint="9427befa18ec6865a9ebdc79d4c38de50e6316ff"
                thumbprintAlgorithm="sha1" />
            <Certificate name="CAForSampleCertificate"
                thumbprint="79d4c38de50e6316ff9427befa18ec6865a9ebdc"
                thumbprintAlgorithm="sha1" />
        </Certificates>
    ...
    </Role>
    ```

(本例針對指紋演算法使用 **sha1**。 請指定適當的值作為憑證的指紋演算法。)

您已更新服務定義檔與服務組態檔，現在請封裝您的部署，以上傳至 Azure。 如果您是使用 **cspack**，請確定未使用 **/generateConfigurationFile** 旗標，因為如此會將覆寫您剛插入的憑證資訊。

## <a name="step-3-upload-a-certificate"></a>步驟 3：上傳憑證
連線到 Azure 入口網站並...

1. 在入口網站的 [所有資源] 區段中，選取您的雲端服務。

    ![發佈您的雲端服務](media/cloud-services-configure-ssl-certificate-portal/browse.png)

2. 按一下 [憑證]。

    ![按一下憑證圖示](media/cloud-services-configure-ssl-certificate-portal/certificate-item.png)

3. 按一下憑證區上方的 [上傳]。

    ![按一下 [上傳] 功能表項目](media/cloud-services-configure-ssl-certificate-portal/Upload_menu.png)

4. 提供 [檔案]、[密碼]，然後按一下資料輸入區底部的 [上傳]。

## <a name="step-4-connect-to-the-role-instance-by-using-https"></a>步驟 4：使用 HTTPS 來連線至角色執行個體
您的部署已在 Azure 啟動並執行，現在您可以使用 HTTPS 來與其連線。

1. 按一下 [網站 URL] 開啟網頁瀏覽器。

   ![按一下網站 URL](media/cloud-services-configure-ssl-certificate-portal/navigate.png)

2. 在網頁瀏覽器中，將連結修改為使用 **https** 而非 **http**，然後造訪網頁。

   > [!NOTE]
   > 如果使用自我簽署憑證，則當您在瀏覽器中瀏覽至與該自我簽署憑證相關聯的 HTTPS 端點時，您會看見憑證錯誤。 使用信任的憑證授權單位所簽署的憑證，則不會有此問題；因此，您可以忽略該錯誤。 (另一個選項為新增自我簽署憑證至使用者的受信任憑證授權單位憑證存放區。)
   >
   >

   ![網站預覽](media/cloud-services-configure-ssl-certificate-portal/show-site.png)

   > [!TIP]
   > 若要對預備部署而不是對生產部署使用 SSL，首先您需要判定要在預備部署中使用的 URL。 部署雲端服務之後，預備環境的 URL 即由**部署 ID** GUID 決定，格式如下：`https://deployment-id.cloudapp.net/`  
   >
   > 建立一般名稱 (CN) 等於 GUID 型 URL (如 **328187776e774ceda8fc57609d404462.cloudapp.net**) 的憑證。 使用入口網站將憑證新增至預備的雲端服務。 接著，將憑證資訊新增至 CSDEF 和 CSCFG 檔案、重新封裝應用程式，然後更新預備的部署以使用新套件。
   >

## <a name="next-steps"></a>後續步驟
* [雲端服務的一般設定](cloud-services-how-to-configure-portal.md)。
* 了解如何 [部署雲端服務](cloud-services-how-to-create-deploy-portal.md)。
* 設定 [自訂網域名稱](cloud-services-custom-domain-name-portal.md)。
* [管理您的雲端服務](cloud-services-how-to-manage-portal.md)。
