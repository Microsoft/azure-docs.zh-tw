---
title: 將儲存體總管連線到 Azure Stack 訂用帳戶或儲存體帳戶 | Microsoft Docs
description: 了解如何將儲存體總管連線到 Azure Stack 訂用帳戶
services: azure-stack
documentationcenter: ''
author: mattbriggs
manager: femila
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: get-started-article
ms.date: 06/15/2018
ms.author: mabrigg
ms.reviewer: xiaofmao
ms.openlocfilehash: 2f974b7773e7a4cbc0eda32a267bb5ab939644d8
ms.sourcegitcommit: cc4fdd6f0f12b44c244abc7f6bc4b181a2d05302
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47095111"
---
# <a name="connect-storage-explorer-to-an-azure-stack-subscription-or-a-storage-account"></a>將儲存體總管連線到 Azure Stack 訂用帳戶或儲存體帳戶

*適用於：Azure Stack 整合系統和 Azure Stack 開發套件*

在本文中，您將了解如何使用儲存體總管連線到 Azure Stack 訂用帳戶和儲存體帳戶。 Azure 儲存體總管是一個獨立應用程式，可讓您在 Windows、macOS 和 Linux 上輕鬆使用 Azure Stack 儲存體資料。

> [!NOTE]  
> 有數個工具可協助您將資料移動至 Azure Stack 儲存體，以及從 Azure Stack 儲存體移動資料。 如需詳細資訊，請參閱 [Azure Stack 儲存體適用的資料傳輸工具](azure-stack-storage-transfer.md)。

如果您尚未安裝儲存體總管，請[下載儲存體總管](http://www.storageexplorer.com/)並加以安裝。

連線到 Azure Stack 訂用帳戶或儲存體帳戶之後，您可以使用 [Azure 儲存體總管文章](../../vs-azure-tools-storage-manage-with-storage-explorer.md)來處理 Azure Stack 資料。 

## <a name="prepare-for-connecting-to-azure-stack"></a>準備連線至 Azure Stack

您必須可以直接存取 Azure Stack 或 VPN 連線，儲存體總管才能存取 Azure Stack 訂用帳戶。 若要深入了解如何設定 Azure Stack 的 VPN 連線，請參閱[使用 VPN 連線到 Azure Stack](azure-stack-connect-azure-stack.md#connect-to-azure-stack-with-vpn)。

若是 Azure Stack 開發套件，您必須匯出 Azure Stack 授權單位根憑證。

### <a name="export-and-then-import-the-azure-stack-certificate"></a>匯出 Azure Stack 憑證後再匯入

1. 在 Azure Stack 主機電腦或可使用 VPN 連線到 Azure Stack 的本機電腦上開啟 `mmc.exe`。 

2. 在 [檔案] 中，選取 [新增/移除嵌入式管理單元]，然後新增 [憑證] 以管理 [我的使用者帳戶]。

3. 在 **Console Root\Certificated (Local Computer)\Trusted Root Certification Authorities\Certificates** 之下尋找 **AzureStackSelfSignedRootCert**。

    ![透過 mmc.exe 載入 Azure Stack 根憑證](./media/azure-stack-storage-connect-se/add-certificate-azure-stack.png)

4. 以滑鼠右鍵按一下憑證，選取 [所有工作] > [匯出]，然後依照指示，匯出 **Base-64 encoded X.509 (.CER)** 憑證。

    下一個步驟中會使用所匯出的憑證。

5. 啟動儲存體總管，如果您看到 [連線至 Azure 儲存體] 對話方塊，請將它取消。

6. 在 [編輯] 功能表上，指向 [SSL 憑證]，然後選取 [匯入憑證]。 使用檔案選擇器對話方塊來尋找和開啟您在上一個步驟中瀏覽的憑證。

    匯入憑證之後，系統會提示您重新啟動儲存體總管。

    ![將憑證匯入儲存體總管中](./media/azure-stack-storage-connect-se/import-azure-stack-cert-storage-explorer.png)

7. 儲存體總管重新啟動後，請選取 [編輯] 功能表，然後查看是否已選取 [目標 Azure Stack]。 若未選取，請選取 [目標 Azure Stack]，然後重新啟動儲存體總管，變更才會生效。 不需要進行此設定，即可相容於 Azure Stack 環境。

    ![確保已選取目標 Azure Stack](./media/azure-stack-storage-connect-se/target-azure-stack.png)

## <a name="connect-to-an-azure-stack-subscription-with-azure-ad"></a>使用 Azure AD 連線到 Azure Stack 訂用帳戶

使用下列步驟將儲存體總管連線到 Azure Stack 訂用帳戶，其屬於 Azure Active Directory (Azure AD) 帳戶。

1. 在儲存體總管的左側窗格中，選取 [管理帳戶]。 
    您已登入的所有 Microsoft 訂用帳戶隨即顯示。

2. 若要連線到 Azure Stack 訂用帳戶，請選取 [新增帳戶]。

    ![新增 Azure Stack 帳戶](./media/azure-stack-storage-connect-se/add-azure-stack-account.png)

3. 在 [連線到 Azure 儲存體] 對話方塊的 [Azure 環境] 之下，選取 [Azure] 或 [Azure China] (視正在使用的 Azure Stack 帳戶而定)，選取 [登入] 以使用與至少一個有效 Azure Stack 訂用帳戶相關聯的 Azure Stack 帳戶來登入。

    ![連線到 Azure 儲存體](./media/azure-stack-storage-connect-se/azure-stack-connect-to-storage.png)

4. 成功使用 Azure Stack 帳戶登入後，左窗格會填入與該帳戶相關聯的 Azure Stack 訂用帳戶。 選取您想要使用的 Azure Stack 訂用帳戶，然後選取 [套用]。 (選取或清除 [所有訂用帳戶] 核取方塊切換開關，可選取全部或不選取任何列出的 Azure Stack 訂用帳戶。)

    ![在填妥 [自訂雲端環境] 對話方塊後選取 Azure Stack 訂用帳戶](./media/azure-stack-storage-connect-se/select-accounts-azure-stack.png)

    左窗格會顯示與所選 Azure Stack 訂用帳戶相關聯的儲存體帳戶。

    ![包含 Azure Stack 訂用帳戶的儲存體帳戶清單](./media/azure-stack-storage-connect-se/azure-stack-storage-account-list.png)

## <a name="connect-to-an-azure-stack-subscription-with-ad-fs-account"></a>使用 AD FS 帳戶連線到 Azure Stack 訂用帳戶

> [!Note]  
> Azure 同盟服務 (AD FS) 登入體驗支援具有 Azure Stack 1804 或更新更新的儲存體總管 1.2.0 或更新版本。
使用下列步驟將儲存體總管連線到 Azure Stack 訂用帳戶，其屬於 AD FS 帳戶。

1. 選取 [管理帳戶]。 總管會列出您已登入的 Microsoft 訂用帳戶。
2. 選取 [新增帳戶] 以連線到 Azure Stack 訂用帳戶。

    ![新增帳戶](media/azure-stack-storage-connect-se/add-an-account.png)

3. 選取 [下一步] 。 在 [連線至 Azure 儲存體] 對話方塊的 [Azure 環境] 底下，選取 [使用自訂環境]，然後按 [下一步]。

    ![連線到 Azure 儲存體](media/azure-stack-storage-connect-se/connect-to-azure-storage.png)

4. 輸入 Azure Stack 自訂環境的必要資訊。 

    | 欄位 | 注意 |
    | ---   | ---   |
    | 環境名稱 | 使用者可以自訂此欄位。 |
    | Azure Resource Manager 端點 | Azure Stack 開發套件的 Azure Resource Manager 資源端點範例。<br>針對操作人員： https://adminmanagement.local.azurestack.external <br> 針對使用者： https://management.local.azurestack.external |

    如果您使用 Azure Stack 整合式系統且不知道您的管理端點，請連絡您的操作人員。

    ![新增帳戶](./media/azure-stack-storage-connect-se/custom-environments.png)

5. 選取 [登入]，以連線至與至少一個作用中 Azure Stack 訂用帳戶相關聯的 Azure Stack 帳戶。



6. 選取您想要使用的 Azure Stack 訂用帳戶。 選取 [套用] 。

    ![帳戶管理](./media/azure-stack-storage-connect-se/account-management.png)

    左窗格會顯示與所選 Azure Stack 訂用帳戶相關聯的儲存體帳戶。

    ![相關聯的訂用帳戶清單](./media/azure-stack-storage-connect-se/list-of-associated-subscriptions.png)

## <a name="connect-to-an-azure-stack-storage-account"></a>連線到 Azure Stack 儲存體帳戶

您也可以使用儲存體帳戶名稱和金鑰配對來連線到 Azure Stack 儲存體帳戶。

1. 在儲存體總管的左側窗格中，選取 [管理帳戶]。 您已登入的所有 Microsoft 帳戶隨即顯示。

    ![新增帳戶](./media/azure-stack-storage-connect-se/azure-stack-sub-add-an-account.png)

2. 若要連線到 Azure Stack 訂用帳戶，請選取 [新增帳戶]。

    ![新增帳戶](./media/azure-stack-storage-connect-se/azure-stack-use-a-storage-and-key.png)

3. 在 [連線到 Azure 儲存體] 對話方塊中，選取 [使用儲存體帳戶名稱和金鑰]。

4. 在 [帳戶名稱] 中輸入您的帳戶名稱，在 [帳戶金鑰] 文字方塊中貼上帳戶金鑰，並在 [儲存體端點網域] 中選取 [其他] \(在下方輸入\)，然後輸入 Azure Stack 端點。

    Azure Stack 端點包含兩個部分：區域的名稱和 Azure Stack 網域。 在 Azure Stack 開發套件中，預設端點是 **local.azurestack.external**。 如果不確定您的端點，請連絡您的雲端系統管理員。

    ![連結名稱和金鑰](./media/azure-stack-storage-connect-se/azure-stack-attach-name-and-key.png)

5. 選取 [ **連接**]。
6. 成功連結儲存體帳戶之後，顯示的儲存體帳戶會在其名稱後面附加 (**外部，其他**)。

    ![VMWINDISK](./media/azure-stack-storage-connect-se/azure-stack-vmwindisk.png)

## <a name="next-steps"></a>後續步驟

* [開始使用儲存體總管](../../vs-azure-tools-storage-manage-with-storage-explorer.md)
* [Azure Stack 儲存體：差異與考量](azure-stack-acs-differences.md)
* 若要深入了解 Azure 儲存體，請參閱 [Microsoft Azure 儲存體簡介](../../storage/common/storage-introduction.md)
