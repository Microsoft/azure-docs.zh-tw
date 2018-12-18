---
title: 設定資料科學虛擬機器的通用身分識別 - Azure | Microsoft Docs
description: 在企業團隊的 DSVM 環境中設定通用身分識別。
keywords: 深度學習, 資料科學工具, 資料科學虛擬機器, 地理空間分析, 小組資料科學流程
services: machine-learning
documentationcenter: ''
author: gopitk
manager: cgronlun
ms.assetid: ''
ms.service: machine-learning
ms.component: data-science-vm
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/08/2018
ms.author: gokuma
ms.openlocfilehash: 25d40b6a72ab6da61feb1458f5930eb48ef1d900
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39436296"
---
# <a name="set-up-a-common-identity-on-the-data-science-virtual-machine"></a>在資料科學虛擬機器上設定通用身分識別

在 Azure 虛擬機器 (VM) (包括「資料科學虛擬機器」(DSVM)) 上，您可以在佈建 VM 時，建立本機使用者帳戶。 接著，使用者便可使用這些認證向 VM 進行驗證。 如果您有多個需要存取的 VM，這種方式很快就會在您管理認證時變得累贅。 透過標準型身分識別提供者的通用使用者帳戶和管理，可讓您使用一組認證來存取 Azure 上的多個資源，包括多個 DSVM。 

Active Directory 是一個熱門的身分識別提供者，在 Azure 上 (以服務的形式) 及在內部部署環境中都支援。 您可以使用 Azure Active Directory (Azure AD) 或內部部署 Active Directory，在獨立的 DSVM 上或 Azure 虛擬機器擴展集內的 DSVM 叢集上驗證使用者。 做法是將 DSVM 執行個體加入 Active Directory 網域。 

如果您已經有 Active Directory 來管理身分識別，便可以使用它作為通用身分識別提供者。 如果您沒有 Active Directory，則可以透過名為 [Azure Active Directory Domain Services](https://docs.microsoft.com/azure/active-directory-domain-services/) (Azure AD DS) 的服務，在 Azure 上執行受控 Active Directory 執行個體。 

[Azure AD](https://docs.microsoft.com/azure/active-directory/) 的文件提供詳細的[管理指示](https://docs.microsoft.com/azure/active-directory/choose-hybrid-identity-solution#synchronized-identity)，包括將 Azure AD 連線至您的內部部署目錄 (如果有的話)。 

本文說明使用 Azure AD DS 在 Azure 上設定完全受控 Active Directory 網域服務的步驟。 接著，您可以將 DSVM 加入受控 Active Directory 網域，以讓使用者能夠使用通用的使用者帳戶和認證來存取 DSVM 的集區 (以及其他 Azure 資源)。 

## <a name="set-up-a-fully-managed-active-directory-domain-on-azure"></a>在 Azure 上設定完全受控的 Active Directory 網域

Azure AD DS 可透過在 Azure 上提供完全受控的服務，讓您輕鬆地管理您的身分識別。 在這個 Active Directory 網域上，您會管理使用者和群組。 在您目錄中設定 Azure 裝載的 Active Directory 網域和使用者帳戶的步驟如下：

1. 在 Azure 入口網站中，將使用者新增至 Active Directory： 

   a. 使用具備目錄全域管理員身分的帳戶來登入 [Azure Active Directory 管理中心](https://aad.portal.azure.com)。
    
   b. 選取 [Azure Active Directory]，然後選取 [使用者和群組]。
    
   c. 在 [使用者和群組] 上，選取 [所有使用者]，然後選取 [新增使用者]。
   
      [使用者] 窗格隨即開啟。
      
      ![[使用者] 窗格](./media/add-user.png)
    
   d. 輸入使用者的詳細資料，例如「名稱」和「使用者名稱」。 使用者名稱的網域名稱部分必須是初始預設網域名稱 "[網域名稱].onmicrosoft.com"，或是已驗證的非同盟[自訂網域名稱](../../active-directory/add-custom-domain.md)，例如"contoso.com"。
    
   e. 複製或記下產生的使用者密碼，以便在此程序完成後，將它提供給使用者。
    
   f. (選擇性) 您可以開啟使用者的 [設定檔]、[群組] 或 [目錄角色]，然後在其中填入資訊。 
    
   g. 在 [使用者] 中，選取 [建立]。
    
   h. 將產生的密碼安全地散發給新使用者，以便讓使用者可以登入。

1. 建立 Azure AD DS 執行個體。 請依照[使用 Azure 入口網站啟用 Azure Active Directory Domain Services](https://docs.microsoft.com/azure/active-directory-domain-services/active-directory-ds-getting-started)一文 (工作 1 到 5) 中的指示進行操作。 請務必更新 Active Directory 中現有的使用者密碼，以便同步 Azure AD DS 中的密碼。 也請務必將 DNS 新增至 Azure AD DS，如上文的工作 4 所述。 

1. 在先前步驟工作 2 所建立的虛擬網路中，建立個別的 DSVM 子網路。
1. 在 DSVM 子網路中建立一或多個「資料科學 VM」執行個體。 
1. 依照[指示](https://docs.microsoft.com/azure/active-directory-domain-services/active-directory-ds-join-ubuntu-linux-vm )將 DSVM 新增至 Active Directory。 
1. 掛接「Azure 檔案服務」共用來裝載您的首頁或筆記本目錄，以便在任何電腦上掛接您的工作區。 (如果您需要嚴格的檔案層級權限，則需要在一或多個 VM 上執行 NFS)。

   a. [建立 Azure 檔案服務共用](../../storage/files/storage-how-to-create-file-share.md)。
    
   b. 在 Linux DSVM 上掛接它。 當您在 Azure 入口網站上的儲存體帳戶中，選取「Azure 檔案服務」共用的 [連線] 按鈕時，會顯示要在 Linux DSVM 的 Bash 殼層中執行的命令。 此命令看起來如下︰
   
   ```
   sudo mount -t cifs //[STORAGEACCT].file.core.windows.net/workspace [Your mount point] -o vers=3.0,username=[STORAGEACCT],password=[Access Key or SAS],dir_mode=0777,file_mode=0777,sec=ntlmssp
   ```
1. 例如，假設您已在 /data/workspace 中掛接「Azure 檔案服務」共用。 現在，請為共用中的每個使用者建立目錄：/data/workspace/user1、/data/workspace/user2 等。 在每個使用者的工作區中建立 `notebooks` 目錄。 
1. 在 `$HOME/userx/notebooks/remote` 中建立 `notebooks` 的符號連結。   

現在，您的使用者便會在裝載於 Azure 的 Active Directory 執行個體中。 藉由使用 Active Directory 認證，使用者將可登入任何已加入 Azure AD DS 的 DSVM (SSH 或 JupyterHub)。 由於使用者工作區位於「Azure 檔案服務」共用上，因此使用者在使用 JupyterHub 時，可以從任何 DSVM 存取其筆記本和其他工作。 

針對自動調整，您可以使用虛擬機器擴展集來建立 VM 的集區，這些 VM 都是以這種方式加入網域，而且已掛接共用磁碟。 使用者可以登入虛擬機器擴展集內的任何可用電腦，而且能夠存取儲存其筆記本的共用磁碟。 

## <a name="next-steps"></a>後續步驟

* [安全地儲存用來存取雲端資源的認證](dsvm-secure-access-keys.md)



