---
title: 教學課程 - 在 Azure CDN 自訂網域上設定 HTTPS | Microsoft Docs
description: 在本教學課程中，您將了解如何在 Azure CDN 端點自訂網域上啟用和停用 HTTPS。
services: cdn
documentationcenter: ''
author: mdgattuso
manager: danielgi
editor: ''
ms.assetid: 10337468-7015-4598-9586-0b66591d939b
ms.service: cdn
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 06/29/2018
ms.author: magattus
ms.custom: mvc
ms.openlocfilehash: c7540ed2715d13921f005ed9b217f7bfb9cd0a0a
ms.sourcegitcommit: 4047b262cf2a1441a7ae82f8ac7a80ec148c40c4
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2018
ms.locfileid: "49092078"
---
# <a name="tutorial-configure-https-on-an-azure-cdn-custom-domain"></a>教學課程：在 Azure CDN 自訂網域上設定 HTTPS

> [!IMPORTANT]
> 這項功能不適用於**來自 Akamai 的標準 Azure CDN** 產品。 如需 Azure 內容傳遞網路 (CDN) 功能的比較，請參閱[比較 Azure CDN 產品功能](cdn-features.md)。

本教學課程示範如何針對與 Azure CDN 端點相關聯的自訂網域，啟用 HTTPS 通訊協定。 在您的自訂網域上使用 HTTPS 通訊協定 (例如 https:\//www.contoso.com)，確保在網際網路上傳送機密資料時能經由 TLS/SSL 加密安全地傳遞。 當網頁瀏覽器透過 HTTPS 連線到網站時，它會驗證網站的安全性憑證，並確認憑證是由合法的憑證授權單位所核發的。 此程序可提供安全性，並讓 Web 應用程式免於遭受攻擊。

Azure CDN 預設支援 CDN 端點主機名稱上的 HTTPS。 舉例來說，當您建立 CDN 端點時 (例如 https:\//contoso.azureedge.net) ，會自動啟用 HTTPS。  

自訂 HTTPS 功能的一些重要特色如下：

- 不須額外付費：取得或續約憑證不須付費，且 HTTPS 流量不另外收費。 您只需支付 CDN 的 GB 輸出。

- 容易啟用：從 [Azure 入口網站](https://portal.azure.com)按一下就能佈建。 您也可以使用 REST API 或其他開發工具來啟用此功能。

- 可以使用完整憑證管理：為您處理所有憑證採購及管理。 憑證會在到期之前自動佈建並更新，進而消除因為憑證到期而中斷服務的風險。

在本教學課程中，您了解如何：
> [!div class="checklist"]
> - 在您的自訂網域上啟用 HTTPS 通訊協定。
> - 使用 CDN 管理的憑證 
> - 使用您自己的憑證
> - 驗證網域
> - 在您的自訂網域上停用 HTTPS 通訊協定。

## <a name="prerequisites"></a>必要條件

您必須先建立 CDN 設定檔和至少一個 CDN 端點，才能完成本教學課程中的步驟。 如需詳細資訊，請參閱[快速入門：建立 Azure CDN 設定檔和端點](cdn-create-new-endpoint.md)。

此外，您必須在您的 CDN 端點上讓 Azure CDN 自訂網域產生關聯。 如需詳細資訊，請參閱[教學課程：將自訂網域新增至 Azure CDN 端點](cdn-map-content-to-custom-domain.md)

---

## <a name="ssl-certificates"></a>SSL 憑證
若要啟用 HTTPS 通訊協定以在 Azure CDN 自訂網域上安全地傳遞內容，您必須使用 SSL 憑證。 您可以選擇使用 Azure CDN 所管理的憑證，或使用您自己的憑證。


# <a name="option-1-default-enable-https-with-a-cdn-managed-certificatetaboption-1-default-enable-https-with-a-cdn-managed-certificate"></a>[選項 1 (預設值)：使用 CDN 管理的憑證啟用 HTTPS](#tab/option-1-default-enable-https-with-a-cdn-managed-certificate)

使用 CDN 管理的憑證時，只要按幾下滑鼠就可以開啟 HTTPS 功能。 Azure CDN 會完全處理憑證管理工作，例如購買和更新。 啟用此功能之後，程序就會立即啟動。 如果自訂網域已對應到 CDN 端點，則不需要任何進一步的動作。 Azure CDN 會處理相關步驟並自動完成您的要求。 不過，如果您的自訂網域對應到其他位置，您就必須使用電子郵件來驗證您的網域擁有權。

依照下列步驟啟用自訂網域的 HTTPS︰

1. 在 [Azure 入口網站](https://portal.azure.com)中，瀏覽至**來自 Microsoft 的標準 Azure CDN**、**來自 Verizon 的標準 Azure CDN**或**來自 Verizon 的進階 Azure CDN** 設定檔。

2. 在 CDN 端點清單中，選取包含自訂網域的端點。

    ![端點清單](./media/cdn-custom-ssl/cdn-select-custom-domain-endpoint.png)

    [端點] 頁面隨即出現。

3. 在自訂網域清單中，選取您要啟用 HTTPS 的自訂網域。

    ![自訂網域清單](./media/cdn-custom-ssl/cdn-custom-domain.png)

    [自訂網域] 頁面隨即出現。

4. 在憑證管理類型底下，選取 [CDN 管理]。

5. 選取 [開啟] 以啟用 HTTPS。

    ![自訂網域 HTTPS 狀態](./media/cdn-custom-ssl/cdn-select-cdn-managed-certificate.png)

6. 繼續[驗證網域](#validate-the-domain)。


# <a name="option-2-enable-https-with-your-own-certificatetaboption-2-enable-https-with-your-own-certificate"></a>[選項 2：使用您自己的憑證啟用 HTTPS](#tab/option-2-enable-https-with-your-own-certificate)

> [!IMPORTANT]
> 此選項僅適用於**來自 Microsoft 的 Azure CDN 標準**設定檔。 
>
 
您可以使用自己的憑證啟用 HTTPS 功能。 此程序會透過與 Azure Key Vault 的整合來進行，因為此一整合可讓您安全地儲存憑證。 Azure CDN 使用此安全機制來取得您的憑證，為此您需要執行一些額外的步驟。 當您建立 SSL 憑證時，您必須使用允許的憑證授權單位 (CA) 來加以建立。 否則，如果您使用非允許的 CA，系統會拒絕您的要求。 如需允許 CA 的清單，請參閱[可在 Azure CDN 上啟用自訂 HTTPS 的允許憑證授權單位](cdn-troubleshoot-allowed-ca.md)。

### <a name="prepare-your-azure-key-vault-account-and-certificate"></a>準備您的 Azure Key Vault 帳戶和憑證
 
1. Azure Key Vault：在與您想要啟用自訂 HTTPS 的 Azure CDN 設定檔和 CDN 端點相同的訂用帳戶底下，您必須有執行中的 Azure Key Vault 帳戶。 建立 Azure Key Vault 帳戶 (如果您還沒有的話)。
 
2. Azure Key Vault 憑證：如果您已擁有憑證，您可以將它直接上傳至您的 Azure Key Vault 帳戶，也可以從與 Azure Key Vault 整合的其中一個合作夥伴 CA 透過 Azure Key Vault 建立新憑證。 

### <a name="register-azure-cdn"></a>註冊 Azure CDN

透過 PowerShell，將 Azure CDN 註冊為 Azure Active Directory 中的應用程式。

1. 如有需要，請在本機電腦的 PowerShell 中安裝 [Azure PowerShell](https://www.powershellgallery.com/packages/AzureRM/6.0.0)。

2. 在 PowerShell 中執行下列命令：

     `New-AzureRmADServicePrincipal -ApplicationId "205478c0-bd83-4e1b-a9d6-db63a3e1e1c8"`

    ![在 PowerShell 中註冊 Azure CDN](./media/cdn-custom-ssl/cdn-register-powershell.png)
              

### <a name="grant-azure-cdn-access-to-your-key-vault"></a>為 Azure CDN 授與金鑰保存庫的存取權
 
授與 Azure CDN 權限，以存取您 Azure Key Vault 帳戶中的憑證 (祕密)。

1. 在您的金鑰保存庫帳戶中，於 [設定] 之下選取 [存取原則]，然後選取 [新增] 以建立新原則。

    ![建立新的存取原則](./media/cdn-custom-ssl/cdn-new-access-policy.png)

2. 在 [選取主體] 中，搜尋 **205478c0-bd83-4e1b-a9d6-db63a3e1e1c8**，然後選擇 [Microsoft.Azure.Cdn]。 按一下 [選取] 。

    ![存取原則設定](./media/cdn-custom-ssl/cdn-access-policy-settings.png)

3. 在 [祕密權限] 中，選取 [取得] 允許 CDN 執行這些權限，以取得並列出憑證。 

4. 選取 [確定] 。 

    Azure CDN 現在可以存取此金鑰保存庫和此金鑰保存庫中儲存的憑證 (祕密)。
 
### <a name="select-the-certificate-for-azure-cdn-to-deploy"></a>選取讓 Azure CDN 進行部署的憑證
 
1. 返回 Azure CDN 入口網站，然後選取您要啟用自訂 HTTPS 的設定檔和 CDN 端點。 

2. 在自訂網域清單中，選取您要啟用 HTTPS 的自訂網域。

    [自訂網域] 頁面隨即出現。

3. 在憑證管理類型底下，選取 [使用我自己的憑證]。 

    ![設定您的憑證](./media/cdn-custom-ssl/cdn-configure-your-certificate.png)

4. 選取金鑰保存庫、憑證 (祕密) 和憑證版本。

    Azure CDN 列出下列資訊： 
    - 您的訂用帳戶識別碼的金鑰保存庫帳戶。 
    - 所選金鑰保存庫底下的憑證 (密碼)。 
    - 可用的憑證版本。 
 
5. 選取 [開啟] 以啟用 HTTPS。
  
6. 當您使用自己的憑證時，不需要進行網域驗證。 請繼續進行[等待傳播](#wait-for-propagation)。

---

## <a name="validate-the-domain"></a>驗證網域

如果您已經有使用中的自訂網域經由 CNAME 記錄對應至自訂端點，或使用自己的憑證，則請繼續進行  
[自訂網域已對應至您的 CDN 端點](#custom-domain-is-mapped-to-your-cdn-endpoint-by-a-cname-record)。 如果您端點的 CNAME 記錄項目已不存在或包含 cdnverify 子網域，請繼續進行[自訂網域未對應至您的 CDN 端點](#custom-domain-is-not-mapped-to-your-cdn-endpoint)。

### <a name="custom-domain-is-mapped-to-your-cdn-endpoint-by-a-cname-record"></a>自訂網域已經由 CNAME 記錄對應至您的 CDN 端點

當您為端點新增自訂網域時，便已在網域註冊機構的 DNS 表格中建立 CNAME 記錄，以將它對應至 CDN 端點主機名稱。 如果該 CNAME 記錄仍然存在且不包含 cdnverify 子網域，則 DigiCert CA 會使用它來自動驗證您自訂網域的擁有權。 

如果您使用自己的憑證，則不需要進行網域驗證。

您的 CNAME 記錄應該採用下列格式，其中「名稱」是您的自訂網域名稱，而「值」則是您的 CDN 端點主機名稱：

| 名稱            | 類型  | 值                 |
|-----------------|-------|-----------------------|
| www.contoso.com | CNAME | contoso.azureedge.net |

如需有關 CNAME 記錄的詳細資訊，請參閱[建立 CNAME DNS 記錄](https://docs.microsoft.com/azure/cdn/cdn-map-content-to-custom-domain#create-the-cname-dns-records)。

如果您的 CNAME 記錄格式正確，DigiCert 就會自動驗證您的自訂網域名稱，並且為您的網域名稱建立專用憑證。 DigitCert 不會傳送驗證電子郵件給您，因此您不需要核准您的要求。 憑證有效期限為一年，並且會在到期之前自動更新。 請繼續進行[等待傳播](#wait-for-propagation)。 

自動驗證通常只需要幾分鐘。 如果您沒有看到您的網域在一小時內進行驗證，請開啟支援票證。

>[!NOTE]
>如果您具有 DNS 提供者的憑證授權單位授權 (CAA) 記錄，它必須包括 DigiCert 作為有效的 CA。 CAA 記錄可讓網域擁有者透過其 DNS 提供者，指定哪些 CA 有權為其網域發行憑證。 如果 CA 收到具有 CAA 記錄之網域的憑證訂單，而且該 CA 未列出為授權簽發者，則禁止該 CA 將憑證發行給該網域或子網域。 如需管理 CAA 記錄的相關資訊，請參閱[管理 CAA 記錄](https://support.dnsimple.com/articles/manage-caa-record/)。 有關 CAA 記錄工具，請參閱 [CAA 記錄協助程式](https://sslmate.com/caa/)。

### <a name="custom-domain-is-not-mapped-to-your-cdn-endpoint"></a>自訂網域未對應至您的 CDN 端點

如果您端點的 CNAME 記錄項目已不存在或包含 cdnverify 子網域，請依照此步驟中的其餘指示進行操作。

在您的自訂網域上啟用 HTTPS 之後，DigiCert CA 會根據網域的 [WHOIS](http://whois.domaintools.com/) 註冊資訊連絡其註冊人，以驗證網域的所有權。 連絡方式為透過電子郵件地址 (預設) 或列示在 WHOIS 註冊資訊中的電話號碼。 必須先完成網域驗證才能在您的自訂網域上啟用 HTTPS。 您有六個工作天可以核准網域。 未在六個工作天內核准的要求將會自動遭到取消。 

![WHOIS 記錄](./media/cdn-custom-ssl/whois-record.png)

DigiCert 也會將驗證電子郵件傳送至其他電子郵件地址。 如果 WHOIS 註冊者資訊為私用，請確認您可以直接從下列其中一個地址核准：

admin@&lt;your-domain-name.com&gt;  
administrator@&lt;your-domain-name.com&gt;  
webmaster@&lt;your-domain-name.com&gt;  
hostmaster@&lt;your-domain-name.com&gt;  
postmaster@&lt;your-domain-name.com&gt;  

您應該會在幾分鐘之內收到邀請您核准要求的電子郵件，如以下範例所示。 如果您使用垃圾郵件篩選器，請將 admin@digicert.com 加入白名單。 如果您未在 24 小時內收到驗證電子郵件，請連絡 Microsoft 支援服務。
    
![網域驗證電子郵件](./media/cdn-custom-ssl/domain-validation-email.png)

當您按一下核准連結時，系統會將您導向以下的線上核准表單： 
    
![網域驗證表單](./media/cdn-custom-ssl/domain-validation-form.png)

遵循表單上的指示；您有兩個驗證選項：

- 您可以核准未來所有經相同帳戶針對同一個根網域 (如 contoso.com) 的所有訂購。 如果您計劃新增相同根網域的其他自訂網域，建議使用這種方法。

- 您可以只核准這次要求使用的特定主機名稱。 後續的要求需要另外核准。

核准之後，DigiCert 會針對您的自訂網域名稱完成憑證建立。 憑證有效期限為一年，並且會在到期之前自動更新。

## <a name="wait-for-propagation"></a>等待傳播

自訂網域 HTTPS 功能要在網域名稱通過驗證之後 6-8 小時才會啟用。 當流程完成時，Azure 入口網站中的自訂 HTTPS 狀態會設定為 [已啟用]，而且 [自訂網域] 對話方塊中的四個作業步驟會標示為完成。 您的自訂網域現在已準備好使用 HTTPS。

![[啟用 HTTPS] 對話方塊](./media/cdn-custom-ssl/cdn-enable-custom-ssl-complete.png)

### <a name="operation-progress"></a>作業進度

下表指出啟用 HTTPS 時的作業進度。 啟用 HTTPS 之後，四個作業步驟會出現在 [自訂網域] 對話方塊中。 隨著每個步驟變成作用中狀態，其他子步驟詳細資料會在步驟進行時顯示於下方。 並非所有這些子步驟都會發生。 當步驟成功完成時，旁邊會出現綠色的核取記號。 

| 作業步驟 | 作業子步驟詳細資料 | 
| --- | --- |
| 1 提交要求 | 提交要求 |
| | 正在提交您的 HTTPS 要求。 |
| | 已成功提交您的 HTTPS 要求。 |
| 2 網域驗證 | 如果網域經由 CNAME 對應到 CDN 端點，則會自動進行驗證。 否則，系統會將驗證要求傳送到您的網域註冊記錄 (WHOIS 登記者) 中所列的電子郵件。 請儘速驗證網域。 |
| | 已成功驗證您的網域擁有權。 |
| | 網域擁有權驗證要求已過期 (客戶可能未在 6 天內回應)。 您網域上的 HTTPS 將不會啟用。 * |
| | 客戶拒絕網域擁有權驗證要求。 您網域上的 HTTPS 將不會啟用。 * |
| 3 憑證佈建 | 憑證授權單位正在發行在網域上啟用 HTTPS 所需的憑證。 |
| | 憑證已發行，目前正在部署到 CDN 網路。 這可能需要 6 小時的時間。 |
| | 已成功將憑證部署到 CDN 網路。 |
| 4 完成 | 已成功在您的網域上啟用 HTTPS。 |

\* 除非發生錯誤，否則不會出現這則訊息。 

如果錯誤發生於提交要求之前，則會顯示下列錯誤訊息：

<code>
We encountered an unexpected error while processing your HTTPS request. Please try again and contact support if the issue persists.
</code>



## <a name="clean-up-resources---disable-https"></a>清除資源 - 停用 HTTPS

在先前步驟中，您已在自訂網域上啟用 HTTPS 通訊協定。 如果您不想再使用自訂網域搭配 HTTPS，您可以執行下列步驟來停用 HTTPS：

### <a name="disable-the-https-feature"></a>停用 HTTPS 功能 

1. 在 [Azure 入口網站](https://portal.azure.com)中，瀏覽至**來自 Microsoft 的標準 Azure CDN**、**來自 Verizon 的標準 Azure CDN**或**來自 Verizon 的進階 Azure CDN** 設定檔。

2. 在端點清單中，按一下包含自訂網域的端點。

3. 按一下您要停用 HTTPS 的自訂網域。

    ![自訂網域清單](./media/cdn-custom-ssl/cdn-custom-domain-HTTPS-enabled.png)

4. 按一下 [關閉] 以停用 HTTPS，然後按一下 [套用]。

    ![自訂 HTTPS 對話方塊](./media/cdn-custom-ssl/cdn-disable-custom-ssl.png)

### <a name="wait-for-propagation"></a>等待傳播

自訂網域 HTTPS 功能停用之後，可能需要 6-8 小時才會生效。 當流程完成時，Azure 入口網站中的自訂 HTTPS 狀態會設定為 [已停用]，而且 [自訂網域] 對話方塊中的三個作業步驟會標示為完成。 您的自訂網域無法再使用 HTTPS。

![[停用 HTTPS] 對話方塊](./media/cdn-custom-ssl/cdn-disable-custom-ssl-complete.png)

#### <a name="operation-progress"></a>作業進度

下表指出停用 HTTPS 時的作業進度。 停用 HTTPS 之後，三個作業步驟會出現在 [自訂網域] 對話方塊中。 隨著每個步驟變成作用中狀態，其他詳細資料會顯示在步驟下方。 當步驟成功完成時，旁邊會出現綠色的核取記號。 

| 作業進度 | 作業詳細資料 | 
| --- | --- |
| 1 提交要求 | 正在提交您的要求 |
| 2 憑證取消佈建 | 刪除憑證 |
| 3 完成 | 已憑證刪除 |

## <a name="frequently-asked-questions"></a>常見問題集

1. *憑證提供者是誰？使用的是哪一種憑證？*

    對於**來自 Verizon 的 Azure CDN** 與**來自 Microsoft 的 Azure CDN**，Digicert 均為您的自訂網域提供專用/單一憑證。 

2. *您使用 IP 型或 SNI TLS/SSL？*

    **來自 Verizon 的 Azure CDN** 與**來自 Microsoft 的標準 Azure CDN** 都會使用 SNI TLS/SSL。

3. *如果沒有收到 DigiCert 的驗證電子郵件該怎麼辦？*

    如果您自訂網域的 CNAME 項目直接指向端點主機名稱 (且您未使用 cdnverify 子網域名稱)，您就不會收到網域驗證電子郵件。 驗證會自動進行。 否則，如果您沒有 CNAME 項目且未在 24 小時內收到電子郵件，請連絡 Microsoft 支援服務。

4. *SAN 憑證的安全性是否比專用憑證來得低？*
    
    SAN 憑證遵循和專用憑證相同的加密與安全性標準。 所有發行的 SSL 憑證都使用 SHA-256 來加強伺服器安全性。

5. *是否可以搭配 Akamai 的 Azure CDN 使用自訂網域 HTTPS？*

    目前，這項功能不適用於**來自 Akamai 的標準 Azure CDN** 設定檔。 Microsoft 正在進行相關工作，以在未來的幾個月內支援這項功能。

6. *是否需要我的 DNS 提供者的憑證授權單位授權記錄？*

    否，目前不需要憑證授權單位授權記錄。 不過，如果您的確有一個授權記錄，它必須包含 DigiCert 作為有效的 CA。

7. *在 2018 年 6 月 20 日，來自 Verizon 的 Azure CDN 預設開始使用專用憑證搭配 SNI TLS/SSL。如果使用主體別名 (SAN) 憑證和以 IP 為基礎的 TLS/SSL ，我的現有自訂網域會發生什麼狀況？*

    如果 Microsoft 分析只有對您的應用程式進行 SNI 用戶端要求，則您現有的網域將在未來幾個月中逐漸移轉至單一憑證。 如果 Microsoft 偵測到有一些對您的應用程式進行的非 SNI 用戶端要求，您的網域會保持使用 SAN 憑證搭配以 IP 為基礎的 TLS/SSL。 在任何情況下，不論您的用戶端要求是 SNI 還是非 SNI，您對於這些要求的服務或支援都不會中斷。


## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何：

> [!div class="checklist"]
> - 在您的自訂網域上啟用 HTTPS 通訊協定。
> - 使用 CDN 管理的憑證 
> - 使用您自己的憑證
> - 驗證網域。
> - 在您的自訂網域上停用 HTTPS 通訊協定。

請前往下一個教學課程，以了解如何在 CDN 端點上設定快取。

> [!div class="nextstepaction"]
> [教學課程：設定 Azure CDN 快取規則](cdn-caching-rules-tutorial.md)

