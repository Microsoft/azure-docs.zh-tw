---
title: HPC Pack 叢集與 Azure Active Directory |Microsoft Docs
description: 了解如何整合 Azure 中的 Microsoft HPC Pack 2016 叢集與 Azure Active Directory
services: virtual-machines-windows
documentationcenter: ''
author: dlepow
manager: jeconnoc
ms.assetid: 9edf9559-db02-438b-8268-a6cba7b5c8b7
ms.service: virtual-machines-windows
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-multiple
ms.workload: big-compute
ms.date: 11/16/2017
ms.author: danlep
ms.openlocfilehash: bb0e878c4e987d111a535603cede25c639087ca7
ms.sourcegitcommit: 1d8612a3c08dc633664ed4fb7c65807608a9ee20
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/20/2017
ms.locfileid: "25452567"
---
# <a name="manage-an-hpc-pack-cluster-in-azure-using-azure-active-directory"></a>使用 Azure Active Directory 管理 Azure 中的 HPC Pack 叢集
[Microsoft HPC Pack 2016](https://technet.microsoft.com/library/cc514029) 支援與 [Azure Active Directory](../../active-directory/index.md) (Azure AD) 整合，以便系統管理員在 Azure 中部署 HPC Pack 叢集。



依照本文中的步驟執行高階工作︰ 
* 手動整合 HPC Pack 叢集與 Azure AD 租用戶
* 在 Azure 中管理和排程 HPC Pack 叢集中的工作 

整合 HPC Pack 叢集解決方案與 Azure AD 時，請依循標準步驟來整合其他應用程式和服務。 本文假設您已熟悉 Azure AD 中的基本使用者管理。 如需詳細資訊及背景，請參閱 [Azure Active Directory 文件](../../active-directory/index.md)和下一節。

## <a name="benefits-of-integration"></a>整合的優點


Azure Active Directory (Azure AD) 是多租用戶雲端型目錄和身分識別管理服務，可提供雲端解決方案的單一登入 (SSO) 存取權。

HPC Pack 叢集與 Azure AD 的整合可協助您達成下列目標︰

* 移除 HPC Pack 叢集中的傳統 Active Directory 網域控制站。 這可以協助降低維護叢集 (如果您的企業不需這麼做) 的成本，以及加快部署程序。
* 運用 Azure AD 所提供的下列優點︰
    *   單一登入 
    *   在 Azure 中使用 HPC Pack 叢集的本機 AD 身分識別 

    ![Azure Active Directory 環境](./media/hpcpack-cluster-active-directory/aad.png)


## <a name="prerequisites"></a>先決條件
* **Azure 虛擬機器中部署的 HPC Pack 2016 叢集** - 如需相關步驟，請參閱[在 Azure 中部署 HPC Pack 2016 叢集](hpcpack-2016-cluster.md)。 您必須要有前端節點的 DNS 名稱和叢集系統管理員的認證，才能完成本文中的步驟。

  > [!NOTE]
  > HPC Pack 2016 之前的 HPC Pack 版本不支援 Azure Active Directory 整合。



* **用戶端電腦** - 您需要有可執行 HPC Pack 用戶端公用程式的 Windows 或 Windows Server 用戶端電腦。 如果您只想要使用 HPC Pack Web 入口網站或 REST API 來提交工作，您可以使用自行選擇的任何用戶端電腦。

* **HPC Pack 用戶端公用程式** - 使用從 Microsoft 下載中心取得的免費安裝套件，在用戶端電腦上安裝 HPC Pack 用戶端公用程式。


## <a name="step-1-register-the-hpc-cluster-server-with-your-azure-ad-tenant"></a>步驟 1︰向 Azure AD 租用戶註冊 HPC 叢集伺服器
1. 登入 [Azure 入口網站](https://portal.azure.com)。
2. 如果您的帳戶可讓您存取多個 Azure AD 帳戶，請在右上角按一下您的帳戶。 然後將您的入口網站工作階段設定為所需的租用戶。 您必須具備存取目錄中資源的權限。 
3. 按一下左側服務瀏覽窗格中的 [Azure Active Directory]，按一下 [使用者和群組]，並確定已經建立或設定使用者帳戶。
4. 在 **Azure Active Directory** 中，按一下 [應用程式註冊] > [新增應用程式註冊]。 輸入以下資訊：
    * **名稱** - HPCPackClusterServer
    * **應用程式類型** - 選取 [Web 應用程式/API]
    * **單入 URL**- 範例的基礎 URL，預設為 `https://hpcserver`
    * 按一下頁面底部的 [新增] 。
5. 新增應用程式之後，請在 [應用程式註冊] 清單中選取它。 然後按一下 [設定] > [內容]。 輸入以下資訊：
    * 針對 [多重租用戶] 選取 [是]。
    * 將 [應用程式識別碼 URI] 變更為 `https://<Directory_name>/<application_name>`。 以 Azure AD 租用戶的完整名稱 (例如 `hpclocal.onmicrosoft.com`) 取代 `<Directory_name`> ，並以您先前選擇的名稱取代 `<application_name>`。
6. 按一下 [檔案] 。 儲存完成時，在應用程式頁面上，按一下 [資訊清單]。 找出 `appRoles` 設定並新增下列應用程式角色，以編輯資訊清單，然後按一下 [儲存]：

  ```json
  "appRoles": [
     {
     "allowedMemberTypes": [
         "User",
         "Application"
     ],
     "displayName": "HpcAdminMirror",
     "id": "61e10148-16a8-432a-b86d-ef620c3e48ef",
     "isEnabled": true,
     "description": "HpcAdminMirror",
     "value": "HpcAdminMirror"
     },
     {
     "allowedMemberTypes": [
         "User",
         "Application"
     ],
     "description": "HpcUsers",
     "displayName": "HpcUsers",
     "id": "91e10148-16a8-432a-b86d-ef620c3e48ef",
     "isEnabled": true,
     "value": "HpcUsers"
     }
  ],
  ```
7. 在 [Azure Active Directory] 中，按一下 [企業應用程式] > [所有應用程式]。 從清單中選取 **HPCPackClusterServer**。
8. 按一下 [內容]，並將 [需要使用者指派] 變更為 [是]。 按一下 [檔案] 。
9. 按一下 [使用者和群組] > [新增使用者]。 選取使用者並選取角色，然後按一下 [指派]。 將其中一個可用的角色 (HpcUsers 或 HpcAdminMirror) 指派給使用者。 對目錄中的其他使用者重複此步驟。 如需叢集使用者的背景資訊，請參閱[管理叢集使用者](https://technet.microsoft.com/library/ff919335(v=ws.11).aspx)。


## <a name="step-2-register-the-hpc-cluster-client-with-your-azure-ad-tenant"></a>步驟 2︰向 Azure AD 租用戶註冊 HPC 叢集用戶端

1. 登入 [Azure 入口網站](https://portal.azure.com)。
2. 如果您的帳戶可讓您存取多個 Azure AD 帳戶，請在右上角按一下您的帳戶。 然後將您的入口網站工作階段設定為所需的租用戶。 您必須具備存取目錄中資源的權限。 
3. 在 **Azure Active Directory** 中，按一下 [應用程式註冊] > [新增應用程式註冊]。 輸入以下資訊：

    * **名稱** - HPCPackClusterClient    
    * **應用程式類型** - 選取 [原生]
    * **重新導向 URI** - `http://hpcclient`
    * 按一下 [建立] 

4. 新增應用程式之後，請在 [應用程式註冊] 清單中選取它。 複製**應用程式識別碼**值，並將其儲存。 您稍後在設定應用程式時需要此資訊。

5. 按一下 [設定] > [必要權限] > [新增] > [選取 API]。 搜尋並選取 HpcPackClusterServer 應用程式 (在步驟 1 中建立)。

6. 在 [啟用存取] 頁面上，選取 [存取 HpcClusterServer]。 然後按一下 [完成]。


## <a name="step-3-configure-the-hpc-cluster"></a>步驟 3︰設定 HPC 叢集

1. 連接到 Azure 中的 HPC Pack 2016 前端節點。

2. 啟動 HPC PowerShell。

3. 執行以下命令：

    ```powershell

    Set-HpcClusterRegistry -SupportAAD true -AADInstance https://login.microsoftonline.com/ -AADAppName HpcPackClusterServer -AADTenant <your AAD tenant name> -AADClientAppId <client ID> -AADClientAppRedirectUri http://hpcclient
    ```
    其中

    * `AADTenant` 指定 Azure AD 租用戶名稱，例如 `hpclocal.onmicrosoft.com`
    * `AADClientAppId` 為在步驟 2 中建立的應用程式指定應用程式識別碼。

4. 視前端節點組態而定，執行下列其中一項：

    * 在單一前端節點 HPC Pack 叢集中，重新啟動 HpcScheduler 服務。

    * 在具有多個前端節點的 HPC Pack 叢集中，在前端節點上執行下列 PowerShell 命令，以重新啟動 HpcSchedulerStateful 服務：

    ```powershell
    Connect-ServiceFabricCluster

    Move-ServiceFabricPrimaryReplica –ServiceName "fabric:/HpcApplication/SchedulerStatefulService"

    ```

## <a name="step-4-manage-and-submit-jobs-from-the-client"></a>步驟 4︰從用戶端管理和提交工作

若要在電腦上安裝 HPC Pack 用戶端公用程式，請從 Microsoft 下載中心下載 HPC Pack 2016 安裝程式檔案 (完整安裝)。 當您開始安裝時，請選擇安裝 **HPC Pack 用戶端公用程式**。

若要準備用戶端電腦，請在用戶端電腦上安裝 [HPC 叢集設定](hpcpack-2016-cluster.md)期間所使用的憑證。 使用標準 Windows 憑證管理程序，將公開憑證安裝至 [憑證 - 目前使用者] > [受信任的根憑證授權單位] 存放區。 

您現在可以執行 HPC Pack 命令或使用 HPC Pack 工作管理員 GUI，以便利用 Azure AD 帳戶提交和管理叢集工作。 如需工作提交選項，請參閱[在 Azure 中將 HPC 作業提交至 HPC Pack 叢集](hpcpack-cluster-submit-jobs.md#step-3-run-test-jobs-on-the-cluster)。

> [!NOTE]
> 當您第一次嘗試在 Azure 中連接 HPC Pack 叢集時，就會出現快顯視窗。 輸入您的 Azure AD 認證進行登入。 隨即會快取權杖。 除非驗證變更或已清除快取的權杖，否則連往 Azure 中叢集的後續連線會使用快取的權杖。
>
  
例如，完成先前的步驟之後，您可以從內部部署用戶端查詢作業，如下所示︰

```powershell 
Get-HpcJob –State All –Scheduler https://<Azure load balancer DNS name> -Owner <Azure AD account>
```

## <a name="useful-cmdlets-for-job-submission-with-azure-ad-integration"></a>透過 Azure AD 整合提交作業的實用 Cmdlet 

### <a name="manage-the-local-token-cache"></a>管理本機權杖快取

HPC Pack 2016 提供下列 HPC PowerShell Cmdlet 來管理本機權杖快取。 這些 Cmdlet 可用於非互動方式提交作業。 請參閱下列範例：

```powershell
Remove-HpcTokenCache

$SecurePassword = "<password>" | ConvertTo-SecureString -AsPlainText -Force

Set-HpcTokenCache -UserName <AADUsername> -Password $SecurePassword -scheduler https://<Azure load balancer DNS name> 
```

### <a name="set-the-credentials-for-submitting-jobs-using-the-azure-ad-account"></a>設定認證以便利用 Azure AD 帳戶提交作業 

有時候，您可能想要在 HPC 叢集使用者底下執行作業 (針對已加入網域的 HPC 叢集、以一個網域使用者身分執行；針對未加入網域的 HPC 叢集，以一個前端節點上的本機使用者身分執行)。

1. 使用下列命令來設定認證︰

    ```powershell
    $localUser = "<username>"

    $localUserPassword="<password>"

    $secpasswd = ConvertTo-SecureString $localUserPassword -AsPlainText -Force

    $mycreds = New-Object System.Management.Automation.PSCredential ($localUser, $secpasswd)

    Set-HpcJobCredential -Credential $mycreds -Scheduler https://<Azure load balancer DNS name>
    ```

2. 然後如下所示提交作業。 作業/工作會在計算節點上以 $localUser 身分執行。

    ```powershell
    $emptycreds = New-Object System.Management.Automation.PSCredential ($localUser, (new-object System.Security.SecureString))
    ...
    $job = New-HpcJob –Scheduler https://<Azure load balancer DNS name>

    Add-HpcTask -Job $job -CommandLine "ping localhost" -Scheduler https://<Azure load balancer DNS name>

    Submit-HpcJob -Job $job -Scheduler https://<Azure load balancer DNS name> -Credential $emptycreds
    ```
    
   如果未使用 `Submit-HpcJob` 指定 `–Credential`，則會以 Azure AD 帳戶的本機對應使用者身分執行作業或工作。 (HPC 叢集會使用與 Azure AD 帳戶相同的名稱建立本機使用者以執行工作。)
    
3. 設定 Azure AD 帳戶的擴充資料。 使用 Azure AD 帳戶在 Linux 節點上執行 MPI 作業時，這非常有用。

   * 設定 Azure AD 帳戶本身的擴充資料

      ```powershell
      Set-HpcJobCredential -Scheduler https://<Azure load balancer DNS name> -ExtendedData <data> -AadUser
      ```
      
   * 設定擴充資料並且以 HPC 叢集使用者身分執行
   
      ```powershell
      Set-HpcJobCredential -Credential $mycreds -Scheduler https://<Azure load balancer DNS name> -ExtendedData <data>
      ```

