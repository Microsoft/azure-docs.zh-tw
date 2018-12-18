---
title: 為 Azure Stack 整合式系統部署準備 Azure Stack 公開金鑰基礎結構憑證 | Microsoft Docs
description: 說明如何準備 Azure Stack 整合式系統的 Azure Stack PKI 憑證。
services: azure-stack
documentationcenter: ''
author: mattbriggs
manager: femila
editor: ''
ms.service: azure-stack
ms.workload: na
pms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 09/10/2018
ms.author: mabrigg
ms.reviewer: ppacent
ms.openlocfilehash: ef9fe0e05343f9c99656634a075b1bd464a13c7e
ms.sourcegitcommit: 5a9be113868c29ec9e81fd3549c54a71db3cec31
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/11/2018
ms.locfileid: "44379582"
---
# <a name="prepare-azure-stack-pki-certificates-for-deployment"></a>準備 Azure Stack PKI 憑證以進行部署
必須使用與 Azure Stack 憑證需求符合的屬性，來匯入和匯出[取自您所選 CA 的](azure-stack-get-pki-certs.md)憑證檔案。


## <a name="prepare-certificates-for-deployment"></a>準備憑證以進行部署
使用下列步驟來準備及驗證 Azure Stack PKI 憑證： 

### <a name="import-the-certificate"></a>匯入憑證

1.  將[取自您所選 CA 的](azure-stack-get-pki-certs.md)原始憑證版本複製到部署主機的目錄中。 
  > [!WARNING]
  > 如果已透過任何方式匯入、匯出或修改從 CA 直接提供的檔案，則請勿複製該檔案。

1.  以滑鼠右鍵按一下憑證，然後根據從 CA 傳遞憑證的方式選取 [安裝憑證] 或 [安裝 PFX]。

1. 在 [憑證匯入精靈] 中，選取 [本機電腦] 作為匯入位置。 選取 [下一步] 。 在下列畫面上，再次按一下 [下一步]。

    ![本機電腦匯入位置](.\media\prepare-pki-certs\1.png)

1.  選擇 [將所有憑證放在下列存放區中]，然後選取 [企業信任] 作為位置。 按一下 [確定] 以關閉 [憑證存放區選取項目] 對話方塊，然後按 [下一步]。

    ![設定憑證存放區](.\media\prepare-pki-certs\3.png)

    a. 如果您要匯入 PFX，將會看到一個額外的對話方塊。 在 [私密金鑰保護] 頁面上，輸入您憑證檔案的密碼，然後啟用 [將此索引鍵標示為可匯出。這可讓您在稍後備份或傳輸索引鍵] 選項。 選取 [下一步] 。

    ![將索引鍵標示為可匯出](.\media\prepare-pki-certs\2.png)

1. 按一下 [完成] 完成匯入。

### <a name="export-the-certificate"></a>匯出憑證

開啟憑證管理員 MMC 主控台並連線到本機電腦憑證存放區。

1. 開啟 Microsoft Management Console：在 Windows 10 中，以滑鼠右鍵按一下 [開始] 功能表，然後按一下 [執行]。 鍵入 **mmc**，並按一下 [確定]。

1. 按一下 [檔案]、[新增/移除嵌入式管理單元]，然後選取 [憑證]，並按一下 [新增]。

    ![新增憑證嵌入式管理單元](.\media\prepare-pki-certs\mmc-2.png)
 
1. 選取 [電腦帳戶]，並按一下 [下一步]，然後依序選取 [本機電腦] 和 [完成]。 按一下 [確定] 以關閉 [新增/移除嵌入式管理單元] 頁面。

    ![新增憑證嵌入式管理單元](.\media\prepare-pki-certs\mmc-3.png)

1. 瀏覽至 [憑證] > [企業信任] > [憑證位置]。 請確認在右側看到您的憑證。

1. 從 [憑證管理員] 主控台的工作列中，選取 [動作] > [所有工作] > [匯出]。 選取 [下一步] 。

  > [!NOTE]
  > 視您有多少個 Azure Stack 憑證而定，可能需要多次完成此處理序。

1. 選取 [是，匯出私密金鑰]，然後按 [下一步]。

1. 在 [匯出檔案格式] 區段中，選取 [匯出所有延伸內容]，然後按 [下一步]。

1. 選取 [密碼] 並提供憑證的密碼。 請記住這個密碼，因為它要作為部署參數使用。 選取 [下一步] 。

1. 選擇要匯出之 pfx 檔案的檔案名稱和位置。 選取 [下一步] 。

1. 選取 [完成]。

## <a name="next-steps"></a>後續步驟
[驗證 PKI 憑證](azure-stack-validate-pki-certs.md)