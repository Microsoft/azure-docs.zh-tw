---
title: 將工作提交至 Azure 中的 HPC Pack 叢集 | Microsoft Docs
description: 了解如何設定內部部署電腦，以將工作提交至 Azure 中的 HPC Pack 叢集
services: virtual-machines-windows
documentationcenter: ''
author: dlepow
manager: jeconnoc
editor: ''
tags: azure-resource-manager,azure-service-management,hpc-pack
ms.assetid: 78f6833c-4aa6-4b3e-be71-97201abb4721
ms.service: virtual-machines-windows
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-multiple
ms.workload: big-compute
ms.date: 05/14/2018
ms.author: danlep
ms.openlocfilehash: f2cf26bc9f980729e74c4a4e0b4e3f4b311fd754
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39421129"
---
# <a name="submit-hpc-jobs-from-an-on-premises-computer-to-an-hpc-pack-cluster-deployed-in-azure"></a>將 HPC 工作從內部部署電腦提交至在 Azure 中部署的 HPC Pack 叢集
[!INCLUDE [learn-about-deployment-models](../../../includes/learn-about-deployment-models-both-include.md)]

設定內部部署用戶端電腦以將作業提交至 Azure 中的 [Microsoft HPC Pack](https://technet.microsoft.com/library/cc514029) 叢集。 本文說明如何使用用戶端工具設定本機電腦，以透過 HTTPS 將工作提交至 Azure 中的叢集。 如此一來，數個叢集使用者可以將工作提交至雲端型 HPC Pack 叢集，但是不會直接連接到前端節點 VM，或存取 Azure 訂用帳戶。

![將工作提交至 Azure 中的叢集][jobsubmit]

## <a name="prerequisites"></a>必要條件
* **在 Azure VM 中部署的 HPC Pack 前端節點** - 建議您使用 [Azure 快速入門範本](https://azure.microsoft.com/documentation/templates/)等自動化工具來部署前端節點和叢集。 您必須要有前端節點的 DNS 名稱和叢集系統管理員的認證，才能完成本文中的步驟。
* **用戶端電腦** - 您必須要有可執行 HPC Pack 用戶端公用程式的 Windows 或 Windows Server 用戶端電腦 (請參閱[系統需求](https://technet.microsoft.com/library/dn535781.aspx))。 如果您只想要使用 HPC Pack Web 入口網站或 REST API 來提交工作，您可以使用自行選擇的任何用戶端電腦。
* **HPC Pack 安裝媒體** - 若要安裝 HPC Pack 用戶端公用程式，可從 [Microsoft 下載中心](https://www.microsoft.com/download/details.aspx?id=56360)取得最新版 HPC Pack 的免費安裝套件。 請確定您下載的是安裝在前端節點 VM 上的相同 HPC Pack 版本。

## <a name="step-1-install-and-configure-the-web-components-on-the-head-node"></a>步驟 1：在前端節點上安裝及設定 Web 元件
若要啟用透過 HTTPS 以 REST 介面將工作提交至叢集的功能，請確定在 HPC Pack 前端節點上設定 HPC Pack Web 元件。 如果尚未安裝，您必須先執行 HpcWebComponents.msi 安裝檔案，以安裝 Web 元件。 然後，請執行 HPC PowerShell 指令碼 **Set-HPCWebComponents.ps1**，以設定元件。

如需詳細程序，請參閱 [安裝 Microsoft HPC Pack Web 元件](http://technet.microsoft.com/library/hh314627.aspx)。

> [!TIP]
> HPC Pack 叢集的某些 Azure 快速入門範本會自動安裝和設定 Web 元件。
> 
> 

**安裝 Web 元件**

1. 使用叢集系統管理員的認證連接到前端節點 VM。
1. 從 HPC Pack 安裝程式資料夾，在前端節點上執行 HpcWebComponents.msi。
1. 依照精靈中的步驟安裝 Web 元件

**設定 Web 元件**

1. 在前端節點上，以系統管理員身分啟動 HPC PowerShell。
1. 若要將目錄切換至組態指令碼的位置，請輸入下列命令：
   
    ```powershell
    cd $env:CCP_HOME\bin
    ```
1. 若要設定 REST 介面並啟動 HPC Web 服務，輸入下列命令：
   
    ```powershell
    .\Set-HPCWebComponents.ps1 –Service REST –enable
    ```
1. 當系統提示您選取憑證時，請選擇與前端節點的公用 DNS 名稱相對應的憑證。 例如，如果您使用傳統部署模型來部署前端節點 VM，憑證名稱的格式就會是 CN=&lt;*HeadNodeDnsName*&gt;.cloudapp.net。 如果您使用 Resource Manager 部署模型，則憑證名稱的格式會是 CN=&lt;*HeadNodeDnsName*&gt;.&lt;*region*&gt;.cloudapp.azure.com。
   
   > [!NOTE]
   > 您稍後將作業從內部部署電腦提交至前端節點時，選取此憑證。 請勿選取或設定與 Active Directory 網域中前端節點的電腦名稱對應的憑證 (例如 CN=*MyHPCHeadNode.HpcAzure.local*)。
   > 
   > 
1. 若要設定 Web 入口網站以提交工作，請輸入下列命令：
   
    ```powershell
    .\Set-HPCWebComponents.ps1 –Service Portal -enable
    ```
1. 指令碼完成之後，請輸入下列命令，以停止並重新啟動 HPC 工作排程器服務：
   
    ```powershell
    net stop hpcscheduler
    net start hpcscheduler
    ```

## <a name="step-2-install-the-hpc-pack-client-utilities-on-an-on-premises-computer"></a>步驟 2：在內部部署電腦上安裝 HPC Pack 用戶端公用程式
如果您想要在電腦上安裝 HPC Pack 用戶端公用程式，請從 [Microsoft 下載中心](https://www.microsoft.com/download/details.aspx?id=56360) 下載 HPC Pack 安裝程式檔案 (完整安裝)。 當您開始安裝時，請選擇安裝 **HPC Pack 用戶端公用程式**。

若要使用 HPC Pack 用戶端工具將工作提交至前端節點 VM，您也必須從前端節點匯出憑證，並將它安裝在用戶端電腦上。 憑證必須是 .CER 格式。

**從前端節點匯出憑證**

1. 在前端節點上，將 [憑證] 嵌入式管理單元新增至本機電腦帳戶的 Microsoft 管理主控台。 如需新增嵌入式管理單元的步驟，請參閱 [將憑證嵌入式管理單元新增至 MMC](https://technet.microsoft.com/library/cc754431.aspx)。
1. 在主控台樹狀目錄中，展開 [憑證 - 本機電腦]  >  [個人]，然後按一下 [憑證]。
1. 找出您在[步驟 1：在前端節點上安裝及設定 Web 元件](#step-1-install-and-configure-the-web-components-on-the-head-node)中為 HPC Pack Web 元件設定的憑證 (例如 CN=&lt;*HeadNodeDnsName*&gt;.cloudapp.net)。
1. 在憑證上按一下滑鼠右鍵，並選取 [所有工作]  >  [匯出]。
1. 在 [憑證匯出精靈] 中按 [下一步]，然後確定已選取 [否，不要匯出私密金鑰]。
1. 依照精靈中的其餘步驟，以 DER 編碼的二進位 X.509 (.CER) 格式匯出憑證。

**在用戶端電腦上匯入憑證**

1. 將您從前端節點中匯出的憑證複製到用戶端電腦上的資料夾。
1. 在用戶端電腦上執行 certmgr.msc。
1. 在 [憑證管理員] 中，展開 [憑證 - 目前的使用者]  >  [受信任的根憑證授權單位]，在 [憑證] 上按一下滑鼠右鍵，然後按一下 [所有工作]  >  [匯入]。
1. 在 [憑證匯入精靈] 中按 [下一步]  ，然後依照步驟，將您從前端節點匯出的憑證匯入到「受信任的根憑證授權單位」存放區。

> [!TIP]
> 您可能會看到安全性警告，因為用戶端電腦無法辨識前端節點上的憑證授權單位。 基於測試目的，您可以忽略此警告並完成憑證匯入。
> 
> 

## <a name="step-3-run-test-jobs-on-the-cluster"></a>步驟 3：在叢集上執行測試工作
若要確認您的設定，請嘗試從內部部署的電腦在 Azure 中的叢集上執行工作。 例如，您可以使用 HPC Pack GUI 工具或命令列命令，將工作提交至叢集。 您也可以使用 Web 型入口網站來提交工作。

**在用戶端電腦上執行工作提交命令**

1. 在已安裝 HPC Pack 用戶端公用程式的用戶端電腦上，啟動「命令提示字元」。
1. 輸入範例命令。 例如，若要列出叢集上的所有工作，請根據前端節點的完整 DNS 名稱輸入類似下列其中一項的命令：
   
    ```command
    job list /scheduler:https://<HeadNodeDnsName>.cloudapp.net /all
    ```
   
    或
   
    ```command
    job list /scheduler:https://<HeadNodeDnsName>.<region>.cloudapp.azure.com /all
    ```
   
   > [!TIP]
   > 在排程器 URL 中請使用前端節點的完整 DNS 名稱，而不是 IP 位址。 如果您指定 IP 位址，則會顯示如下的錯誤：「伺服器憑證必須具有有效的信任鏈結，或放在受信任的根存放區」。
   > 
   > 
1. 出現提示時，請輸入使用者名稱 (格式為 &lt;DomainName&gt;\\&lt;UserName&gt;) 和 HPC 叢集系統管理員或您所設定之其他叢集使用者的密碼。 您可以選擇將認證儲存在本機，以供更多工作運用。
   
    工作清單隨即出現。

**在用戶端電腦上使用 HPC 工作管理員**

1. 如果您先前未儲存叢集使用者的網域認證，在提交作業時，您可以在 [認證管理員] 中新增認證。
   
    a. 在用戶端電腦的控制台中，啟動 [認證管理員]。
   
    b. 按一下 [Windows 認證] >  [新增一般認證]。
   
    c. 指定網際網路位址 (例如， https://&lt;HeadNodeDnsName&gt;.cloudapp.net/HpcScheduler 或 https://&lt;HeadNodeDnsName&gt;.&lt;region&gt;.cloudapp.azure.com/HpcScheduler)，和您設定的叢集系統管理員或其他叢集使用者的使用者名稱 (&lt;DomainName&gt;\\&lt;UserName&gt;) 和密碼。
1. 在用戶端電腦上，啟動 [HPC 工作管理員]。
1. 在 [選取前端節點] 對話方塊中，輸入 Azure 中前端節點的 URL (例如， https://&lt;HeadNodeDnsName&gt;.cloudapp.net 或 https://&lt;HeadNodeDnsName&gt;.&lt;region&gt;.cloudapp.azure.com)。
   
    [HPC 工作管理員] 隨即開啟，並顯示前端節點上的工作清單。

**使用在前端節點上執行的 Web 入口網站**

1. 在用戶端電腦上啟動網頁瀏覽器，並根據前端節點的完整 DNS 名稱輸入下列其中一個位址：
   
    ```
    https://<HeadNodeDnsName>.cloudapp.net/HpcPortal
    ```
   
    或
   
    ```
    https://<HeadNodeDnsName>.<region>.cloudapp.azure.com/HpcPortal
    ```
1. 在出現的安全性對話方塊中，輸入 HPC 叢集系統管理員的網域認證。 (您也可以在不同的角色中新增其他叢集使用者。 請參閱[管理叢集使用者](https://technet.microsoft.com/library/ff919335.aspx)。)
   
    Web 入口網站會開啟並顯示作業清單檢視。
1. 若要提交會從叢集傳回字串 "Hello World" 的範例工作，請按一下左側導覽列中的 [新增工作]  。
1. 在 [新增作業] 頁面的 [來源提交頁面] 下，按一下 **HelloWorld**。 工作提交頁面隨即出現。
1. 按一下 [提交] 。 出現提示時，請提供 HPC 叢集系統管理員的網域認證。 作業提交後，作業 ID 會出現在 [我的作業]  頁面上。
1. 若要檢視您所提交之作業的結果，請按一下作業 ID，然後按一下 [檢視工作] 以檢視命令輸出 (在 [輸出] 下)。

## <a name="next-steps"></a>後續步驟
* 您也可以使用 [HPC Pack REST API](http://social.technet.microsoft.com/wiki/contents/articles/7737.creating-and-submitting-jobs-by-using-the-rest-api-in-microsoft-hpc-pack-windows-hpc-server.aspx)將工作提交至 Azure 叢集。
* 如果您要從 Linux 用戶端提交叢集工作，請參閱 [HPC Pack 2012 R2 SDK and Sample Code (HPC Pack 2012 R2 SDK 和範例程式碼)](https://www.microsoft.com/download/details.aspx?id=41633)中的 Python 範例。

<!--Image references-->
[jobsubmit]: ./media/virtual-machines-windows-hpcpack-cluster-submit-jobs/jobsubmit.png
