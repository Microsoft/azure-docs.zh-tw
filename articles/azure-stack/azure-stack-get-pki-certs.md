---
title: 為 Azure Stack 整合式系統部署產生 Azure Stack 公開金鑰基礎結構憑證 | Microsoft Docs
description: 說明 Azure Stack 整合式系統的 Azure Stack PKI 憑證部署程序。
services: azure-stack
documentationcenter: ''
author: mattbriggs
manager: femila
editor: ''
ms.assetid: ''
ms.service: azure-stack
ms.workload: na
pms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/18/2018
ms.author: mabrigg
ms.reviewer: ppacent
ms.openlocfilehash: b5adc1bb5a5aae96f37cc312588aa71e57d8342e
ms.sourcegitcommit: ebb460ed4f1331feb56052ea84509c2d5e9bd65c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/24/2018
ms.locfileid: "42916346"
---
# <a name="azure-stack-certificates-signing-request-generation"></a>Azure Stack 憑證簽署要求產生

[從 PowerShell 資源庫](https://aka.ms/AzsReadinessChecker)可取得本文中所述的 Azure Stack 整備檢查工具。 此工具會建立適用於 Azure Stack 部署的憑證簽署要求 (CSR)。 應在部署前要求、產生及驗證憑證，才有足夠的時間進行測試。

Azure Stack 整備檢查工具 (AzsReadinessChecker) 會執行下列憑證要求：

 - **標準憑證要求**  
    根據[產生適用於 Azure Stack 部署的 PKI 憑證](azure-stack-get-pki-certs.md)提出要求。
 - **平台即服務**  
    對 [Azure Stack 公開金鑰基礎結構憑證要求 - 選擇性 PaaS 憑證](azure-stack-pki-certs.md#optional-paas-certificates)中所指定的憑證，選擇性地要求平台即服務 (PaaS) 名稱。



## <a name="prerequisites"></a>必要條件

在產生 Azure Stack 部署的 PKI 憑證 CSR 之前，您的系統應該符合下列必要條件：

 - Microsoft Azure Stack 整備檢查工具
 - 憑證屬性：
    - 區域名稱
    - 外部完整網域名稱 (FQDN)
    - 主體
 - Windows 10 或 Windows Server 2016
 
  > [!NOTE]
  > 當您收到從憑證授權單位返回的憑證時，您需要在相同系統中完成[準備 Azure Stack PKI 憑證](azure-stack-prepare-pki-certs.md)中的步驟！

## <a name="generate-certificate-signing-requests"></a>產生憑證簽署要求

使用下列步驟來準備及驗證 Azure Stack PKI 憑證： 

1.  執行下列 Cmdlet，以從 PowerShell (5.1 或更新版本) 提示字元安裝 AzsReadinessChecker：

    ````PowerShell  
        Install-Module Microsoft.AzureStack.ReadinessChecker
    ````

2.  宣告**主體** 作為已排序的字典。 例如︰ 

    ````PowerShell  
    $subjectHash = [ordered]@{"OU"="AzureStack";"O"="Microsoft";"L"="Redmond";"ST"="Washington";"C"="US"} 
    ````
    > [!note]  
    > 如果提供一般名稱 (CN)，這將會由憑證要求的第一個 DNS 名稱所覆寫。

3.  宣告輸出目錄已經存在。 例如︰

    ````PowerShell  
    $outputDirectory = "$ENV:USERPROFILE\Documents\AzureStackCSR"
    ````
4.  宣告識別系統

    Azure Active Directory

    ```PowerShell
    $IdentitySystem = "AAD"
    ````

    Active Directory Federation Services

    ```PowerShell
    $IdentitySystem = "ADFS"
    ````

5. 宣告預定用於 Azure Stack 部署的**區域名稱**和**外部 FQDN**。

    ```PowerShell
    $regionName = 'east'
    $externalFQDN = 'azurestack.contoso.com'
    ````

    > [!note]  
    > `<regionName>.<externalFQDN>` 構成 Azure Stack 中所有外部 DNS 名稱據以建立的基礎，在此範例中，入口網站會是 `portal.east.azurestack.contoso.com`。  

6. 若要針對每個 DNS 名稱產生憑證簽署要求：

    ```PowerShell  
    Start-AzsReadinessChecker -RegionName $regionName -FQDN $externalFQDN -subject $subjectHash -OutputRequestPath $OutputDirectory -IdentitySystem $IdentitySystem
    ````

    若要包含 PaaS 服務，請指定 ```-IncludePaaS``` 參數

7. 或者，在開發/測試環境中。 若要產生具有多個主體別名的單一憑證要求，請新增 **-RequestType SingleCSR** 參數和值 (**不**建議用於生產環境)：

    ```PowerShell  
    Start-AzsReadinessChecker -RegionName $regionName -FQDN $externalFQDN -subject $subjectHash -RequestType SingleCSR -OutputRequestPath $OutputDirectory -IdentitySystem $IdentitySystem
    ````

    若要包含 PaaS 服務，請指定 ```-IncludePaaS``` 參數
    
8. 檢閱輸出：

    ````PowerShell  
    AzsReadinessChecker v1.1803.405.3 started
    Starting Certificate Request Generation

    CSR generating for following SAN(s): dns=*.east.azurestack.contoso.com&dns=*.blob.east.azurestack.contoso.com&dns=*.queue.east.azurestack.contoso.com&dns=*.table.east.azurestack.cont
    oso.com&dns=*.vault.east.azurestack.contoso.com&dns=*.adminvault.east.azurestack.contoso.com&dns=portal.east.azurestack.contoso.com&dns=adminportal.east.azurestack.contoso.com&dns=ma
    nagement.east.azurestack.contoso.com&dns=adminmanagement.east.azurestack.contoso.com
    Present this CSR to your Certificate Authority for Certificate Generation: C:\Users\username\Documents\AzureStackCSR\wildcard_east_azurestack_contoso_com_CertRequest_20180405233530.req
    Certreq.exe output: CertReq: Request Created

    Finished Certificate Request Generation

    AzsReadinessChecker Log location: C:\Program Files\WindowsPowerShell\Modules\Microsoft.AzureStack.ReadinessChecker\1.1803.405.3\AzsReadinessChecker.log
    AzsReadinessChecker Completed
    ````

9.  將所產生的 **.REQ** 檔案提交至您的 CA (內部或公用 CA)。  **Start-AzsReadinessChecker** 的輸出目錄包含必須提交給憑證授權單位的 CSR。  它也包含一個子目錄作為參考，其中包含在憑證要求產生期間所使用的 INF 檔案。 確定您的 CA 會使用所產生的要求來產生符合 [Azure Stack PKI 需求](azure-stack-pki-certs.md)的憑證。

## <a name="next-steps"></a>後續步驟

[準備 Azure Stack PKI 憑證](azure-stack-prepare-pki-certs.md)
