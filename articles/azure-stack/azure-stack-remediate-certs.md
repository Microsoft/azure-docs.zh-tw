---
title: 修復 Azure Stack 的憑證問題 | Microsoft Docs
description: 使用 Azure Stack 整備檢查程式來檢閱及修復憑證問題。
services: azure-stack
documentationcenter: ''
author: sethmanheim
manager: femila
editor: ''
ms.assetid: ''
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: get-started-article
ms.date: 05/08/2018
ms.author: sethm
ms.reviewer: ''
ms.openlocfilehash: 5e96c731496d79ca081091e2059a35545f963bd6
ms.sourcegitcommit: 4b1083fa9c78cd03633f11abb7a69fdbc740afd1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/10/2018
ms.locfileid: "49078627"
---
# <a name="remediate-common-issues-for-azure-stack-pki-certificates"></a>修復 Azure Stack PKI 憑證的一般問題
本文中的資訊可協助您了解並解決 Azure Stack PKI 憑證的一般問題。 當您使用 Azure Stack 整備檢查程式工具來[驗證 Azure Stack PKI 憑證](azure-stack-validate-pki-certs.md)時，可以探索到問題。 此工具會進行檢查，以確保憑證符合 Azure Stack 部署和 Azure Stack 祕密修復的 PKI 需求，並將結果記錄在 [report.json 檔案](azure-stack-validation-report.md)中。  

## <a name="pfx-encryption"></a>PFX 加密
**失敗** - PFX 加密不是 TripleDES-SHA1。   
**補救** - 匯出採用 **TripleDES-SHA1** 加密的 PFX 檔案。 從憑證嵌入式管理單元匯出或使用 Export-PFXCertificate 時，這是所有 Windows 10 用戶端的預設值。 

## <a name="read-pfx"></a>讀取 PFX
**警告** - 密碼只會保護憑證中的私人資訊。  
**修復** - 建議您使用 [啟用憑證隱私權] 的選用設定將 PFX 檔案匯出。  

**失敗** - PFX 檔案無效。  
**修復** - 使用[準備 Azure Stack PKI 憑證以進行部署](azure-stack-prepare-pki-certs.md)中的步驟，將憑證重新匯出。

## <a name="signature-algorithm"></a>簽章演算法
**失敗** - 簽章演算法是 SHA1。    
**修復** - 使用「Azure Stack 憑證簽署要求產生」中的步驟，重新產生具有 SHA256 簽章演算法的憑證簽署要求 (CSR)。 然後將 CSR 重新提交至憑證授權單位，以重新發行憑證。

## <a name="private-key"></a>私密金鑰
**失敗** - 私密金鑰遺失或不包含本機電腦屬性。  
**修復** - 從產生 CSR 的電腦中，使用「準備 Azure Stack PKI 憑證以進行部署」中的步驟，將憑證重新匯出。 這些步驟包括從本機電腦憑證存放區匯出。

## <a name="certificate-chain"></a>憑證鏈結
**失敗** - 憑證鏈結不完整。  
**修復** - 憑證應該包含完整的憑證鏈結。  使用[準備 Azure Stack PKI 憑證以進行部署](azure-stack-prepare-pki-certs.md)中的步驟將憑證重新匯出，並選取 [如果可能的話，包含憑證路徑中的所有憑證] 選項。

## <a name="dns-names"></a>DNS 名稱
**失敗** - 憑證上的 DNSNameList 不包含 Azure Stack 服務端點名稱或有效的萬用字元比對。  萬用字元比對僅對 DNS 名稱最左邊的命名空間有效。 例如，\*.region.domain.com 僅對 portal.region.domain.com 有效，對 \*.table.region.domain.com 則無效。  
**修復** - 使用「Azure Stack 憑證簽署要求產生」中的步驟，重新產生具有正確 DNS 名稱可支援 Azure Stack 端點的 CSR。 將 CSR 重新提交給憑證授權單位，然後遵循[準備 Azure Stack PKI 憑證以進行部署](azure-stack-prepare-pki-certs.md)中的步驟，從產生 CSR 的電腦匯出憑證。  

## <a name="key-usage"></a>金鑰使用方式
**失敗** - 金鑰使用方式遺失數位簽章或金鑰加密、或增強金鑰使用方式遺失伺服器驗證或用戶端驗證。  
**修復** - 使用 [Azure Stack 憑證簽署要求產生](azure-stack-get-pki-certs.md)中的步驟，重新產生具有正確金鑰使用方式屬性的 CSR。  將 CSR 重新提交至憑證授權單位，並確認憑證範本不會覆寫要求中的金鑰使用方式。

## <a name="key-size"></a>金鑰大小
**失敗** - 金鑰大小小於 2048    
**修復** - 使用 [Azure Stack 憑證簽署要求產生](azure-stack-get-pki-certs.md)中的步驟，重新產生具有正確金鑰長度 (2048) 的 CSR，然後將 CSR 重新提交至憑證授權單位。

## <a name="chain-order"></a>鏈結順序
**失敗** - 憑證鏈結的順序不正確。  
**修復** - 使用[準備 Azure Stack PKI 憑證以進行部署](azure-stack-prepare-pki-certs.md)中的步驟將憑證重新匯出，並選取 [如果可能的話，包含憑證路徑中的所有憑證] 選項。 確定僅選取分葉憑證以進行匯出。 

## <a name="other-certificates"></a>其他憑證
**失敗** - PFX 套件包含的憑證不是分葉憑證或不屬於憑證鏈結。  
**修復** - 使用[準備 Azure Stack PKI 憑證以進行部署](azure-stack-prepare-pki-certs.md)中的步驟將憑證重新匯出，並選取 [如果可能的話，包含憑證路徑中的所有憑證] 選項。 確定僅選取分葉憑證以進行匯出。

## <a name="fix-common-packaging-issues"></a>修正常見的封裝問題
AzsReadinessChecker 可以匯入 PFX 檔案然後再加以匯出，來修正常見封裝問題，包括： 
 - [PFX 加密] 不是 TripleDES-SHA1
 - 「私密金鑰」遺失本機電腦屬性。
 - 「憑證鏈結」不完整或錯誤。 (如果 PFX 套件不包含憑證鏈結，則本機電腦必須包含憑證鏈結)。 
 - 「其他憑證」。
不過，如果您需要產生新的 CSR，然後重新發出憑證，AzsReadinessChecker 就幫不上忙。 

### <a name="prerequisites"></a>必要條件
執行此工具的電腦上必須先將下列先決條件準備就緒： 
 - Windows 10 或 Windows Server 2016，具有網際網路連線能力。
 - PowerShell 5.1 或更新版本。 若要檢查版本，請執行下列 PowerShell 命令，然後再檢閱「主要」版本和「次要」版本。

   > `$PSVersionTable.PSVersion`
 - 設定[適用於 Azure Stack 的 PowerShell](azure-stack-powershell-install.md)。 
 - 下載最新版的 [Microsoft Azure Stack 整備檢查程式](https://aka.ms/AzsReadinessChecker)工具。

### <a name="import-and-export-an-existing-pfx-file"></a>匯入和匯出現有的 PFX 檔案
1. 在符合先決條件的電腦上，開啟系統管理 PowerShell 提示字元，然後執行下列命令以安裝 AzsReadinessChecker  
   > `Install-Module Microsoft.AzureStack.ReadinessChecker- Force`

2. 從 PowerShell 提示字元中，執行下列命令以設定 PFX 密碼。 將 PFXpassword 取代為實際的密碼。 
   > `$password = Read-Host -Prompt PFXpassword -AsSecureString`

3. 從 PowerShell 提示字元中，執行下列命令以匯出新的 PFX 檔案。
   - 針對 -PfxPath，指定前往您正在使用的 PFX 檔案的路徑。  在下列範例中，路徑是 .\certificates\ssl.pfx。
   - 針對 -ExportPFXPath，指定要進行匯出的 PFX 檔案的位置和名稱。  在下列範例中，路徑是 .\certificates\ssl_new.pfx

   > `Start-AzsReadinessChecker -PfxPassword $password -PfxPath .\certificates\ssl.pfx -ExportPFXPath .\certificates\ssl_new.pfx`  

4. 此工具完成之後，檢閱輸出確認是否成功：![結果](./media/azure-stack-remediate-certs/remediate-results.png)

## <a name="next-steps"></a>後續步驟
[深入了解 Azure Stack 安全性](azure-stack-rotate-secrets.md)
