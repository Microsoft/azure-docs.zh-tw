---
title: 使用 HPC Pack 在 Linux VM 上執行 OpenFOAM | Microsoft Docs
description: 在 Azure 上部署 Microsoft HPC Pack 叢集，並且跨 RDMA 網路在多個 Linux 計算節點上執行 OpenFOAM 工作。
services: virtual-machines-linux
documentationcenter: ''
author: dlepow
manager: jeconnoc
editor: ''
tags: azure-service-management,azure-resource-manager,hpc-pack
ms.assetid: c0bb1637-bb19-48f1-adaa-491808d3441f
ms.service: virtual-machines-linux
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: big-compute
ms.date: 07/22/2016
ms.author: danlep
ms.openlocfilehash: 9032a0b68c4c8789010b0304b64a63d4924521fb
ms.sourcegitcommit: 744747d828e1ab937b0d6df358127fcf6965f8c8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/16/2018
ms.locfileid: "42141889"
---
# <a name="run-openfoam-with-microsoft-hpc-pack-on-a-linux-rdma-cluster-in-azure"></a>在 Azure 中的 Linux RDMA 叢集以 Microsoft HPC Pack 執行 OpenFoam
本文說明一個在 Azure 虛擬機器中執行 OpenFoam 的方式。 在這裡，您將在 Azure 上部署一個具有 Linux 計算節點的 Microsoft HPC Pack 叢集，並使用 Intel MPI 來執行 [OpenFoam](http://openfoam.com/) 作業。 您可以使用支援 RDMA 的 Azure VM 作為計算節點，讓計算節點能夠透過 Azure RDMA 網路進行通訊。 其他在 Azure 中執行 OpenFoam 的選項還包括 Marketplace 中所提供已完整設定的市售映像 (例如 UberCloud 的 [OpenFoam 2.3 on CentOS 6](https://azuremarketplace.microsoft.com/marketplace/apps/cfd-direct.cfd-direct-from-the-cloud))，以及藉由在 [Azure Batch](https://blogs.technet.microsoft.com/windowshpc/2016/07/20/introducing-mpi-support-for-linux-on-azure-batch/) 上執行。 

[!INCLUDE [learn-about-deployment-models](../../../../includes/learn-about-deployment-models-both-include.md)]

OpenFOAM (代表 Open Field Operation and Manipulation) 是一個開放原始碼計算流體力學 (CFD) 軟體套件，廣泛運用在商業和學術組織的工程及科學領域中。 其中包含網格處理工具，特別是 snappyHexMesh，這是一種用於複雜 CAD 幾何的前置和後置處理的平行化網格處理器。 幾乎所有的程序皆以平行方式執行，讓使用者能夠依其需求充分利用電腦硬體。  

Microsoft HPC Pack 提供可在 Microsoft Azure 虛擬機器叢集上執行大規模 HPC 和平行應用程式 (包括的 MPI 應用程式) 的功能。 HPC Pack 也支援在 HPC Pack 叢集中部署的 Linux 計算節點 VM 上，執行 Linux HPC 應用程式。 如需搭配 HPC Pack 使用 Linux 計算節點的簡介，請參閱[開始在 Azure 中的 HPC Pack 叢集使用 Linux 計算節點](hpcpack-cluster.md)。

> [!NOTE]
> 本文說明如何使用 HPC Pack 來執行 Linux MPI 工作負載。 本文假設您對 Linux 系統管理及在 Linux 叢集上執行 MPI 工作負載，有某種程度的熟悉。 如果您使用的 MPI 和 OpenFOAM 版本與本文中所顯示的版本不同，您可能必須修改一些安裝和組態步驟。 
> 
> 

## <a name="prerequisites"></a>必要條件
* **具有支援 RDMA 之 Linux 計算節點的 HPC Pack 叢集** - 使用 [Azure Resource Manager 範本](https://azure.microsoft.com/marketplace/partners/microsofthpc/newclusterlinuxcn/)或 [Azure PowerShell 指令碼](hpcpack-cluster-powershell-script.md)，部署具有 A8、A9、H16r 或 H16rm 大小之 Linux 計算節點的 HPC Pack 叢集。 如需了解任一選項的必要條件與步驟，請參閱 [開始使用 Azure 中 HPC Pack 叢集內的 Linux 計算節點](hpcpack-cluster.md) 。 如果您選擇 PowerShell 指令碼部署選項，請參閱本文結尾範例檔案中的範例組態檔。 您可以使用此組態來部署 Azure 型 HPC Pack 叢集，其中包含一個 A8 大小的 Windows Server 2012 R2 前端節點，以及兩個 A8 大小的 SUSE Linux Enterprise Server 12 計算節點。 請將您的訂用帳戶和服務名稱取代為適當的值。 
  
  **其他應該知道的事項**
  
  * 如需 Azure 的 Linux RDMA 網路必要條件，請參閱[高效能運算 VM 大小](../../windows/sizes-hpc.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)。
  * 如果使用 PowerShell 指令碼部署選項，請將所有 Linux 計算節點部署在一個雲端服務內，以使用 RDMA 網路連線。
  * 部署 Linux 節點之後，請透過 SSH 進行連線來執行任何額外的系統管理工作。 您可以在 Azure 入口網站中找到每個 Linux VM 的 SSH 連線詳細資料。  
* **Intel MPI** - 若要在 Azure 中的 SLES 12 HPC 計算節點上執行 OpenFOAM，您必須從 [Intel.com 網站](https://software.intel.com/en-us/intel-mpi-library/)安裝 Intel MPI Library 5 執行階段。 (Intel MPI 5 已預先安裝在 CentOS 型 HPC 映像上)。在稍後的步驟中，請視需要在 Linux 計算節點上安裝 Intel MPI。 若要為此步驟做準備，請在向 Intel 註冊之後，依循確認電子郵件中的連結前往相關的網頁。 然後，複製適當 Intel MPI 版本之 .tgz 檔案的下載連結。 這篇文章根據 Intel MPI 5.0.3.048 版。
* **OpenFOAM Source Pack** - 從 [OpenFOAM Foundation 網站](http://openfoam.org/download/2-3-1-source/)下載適用於 Linux 的OpenFOAM Source Pack 軟體。 本文是依據 Source Pack 2.3.1 版 (可透過 OpenFOAM-2.3.1.tgz 的形式下載) 而撰寫的。 請依照本文稍後的指示，在 Linux 計算節點上解壓縮並編譯 OpenFOAM。
* **EnSight** (選擇性) - 若要查看 OpenFOAM 模擬的結果，請下載並安裝 [EnSight](https://ensighttransfe.wpengine.com/direct-access-downloads/) 視覺效果與分析程式。 授權和下載資訊請見 EnSight 網站。

## <a name="set-up-mutual-trust-between-compute-nodes"></a>設定運算節點之間的相互信任
在多個 Linux 節點上執行跨節點作業，需要節點相互信任 (藉由 **rsh** 或 **ssh**)。 當您使用 Microsoft HPC Pack IaaS 部署指令碼建立 HPC Pack 叢集時，指令碼會自動為您指定的系統管理員帳戶設定永久相互信任。 針對您在叢集的網域中建立的非系統管理員使用者，您必須在將工作配置給他們時，設定節點間的暫時相互信任，並且在工作完成之後終結關聯性。 若要為每位使用者建立信任，請將 HPC Pack 用於信任關係的 RSA 金鑰組提供給叢集。

### <a name="generate-an-rsa-key-pair"></a>產生 RSA 金鑰組
產生 RSA 金鑰組很容易，其中包含公開金鑰和私密金鑰，方法是執行 Linux **ssh-keygen** 命令。

1. 登入 Linux 電腦。
2. 執行以下命令：
   
   ```
   ssh-keygen -t rsa
   ```
   
   > [!NOTE]
   > 按 **Enter** 以使用預設設定，直到完成命令。 請勿在這裡輸入複雜密碼；當系統提示您輸入密碼時，只要按 **Enter**鍵。
   > 
   > 
   
   ![產生 RSA 金鑰組][keygen]
3. 將目錄變更為 ~/.ssh 目錄。 私密金鑰會儲存在 id_rsa，而公開金鑰會儲存在 id_rsa.pub。
   
   ![私密和公開金鑰][keys]

### <a name="add-the-key-pair-to-the-hpc-pack-cluster"></a>將金鑰組新增至 HPC Pack 叢集
1. 使用您的 HPC Pack 系統管理員帳戶與您的前端節點進行遠端桌面連線 (當您執行部署指令碼時設定的系統管理員帳戶)。
2. 您可以使用標準的 Windows Server 程序在叢集的 Active Directory 網域中建立網域使用者帳戶。 例如，在前端節點上使用 Active Directory 使用者和電腦工具。 本文中的範例假設您建立名為 hpclab\hpcuser 的網域使用者。
3. 建立名為 C:\cred.xml 的檔案，並且將 RSA 金鑰資料複製到其中。 本文結尾有提供一個範例 cred.xml 檔案。
   
   ```
   <ExtendedData>
     <PrivateKey>Copy the contents of private key here</PrivateKey>
     <PublicKey>Copy the contents of public key here</PublicKey>
   </ExtendedData>
   ```
4. 開啟命令提示字元並輸入下列命令，以設定 hpclab\hpcuser 帳戶的認證資料。 您使用 **extendeddata** 參數來傳遞您針對金鑰資料建立之 C:\cred.xml 檔案的名稱。
   
   ```
   hpccred setcreds /extendeddata:c:\cred.xml /user:hpclab\hpcuser /password:<UserPassword>
   ```
   
   這個命令會成功完成而沒有輸出。 為您執行工作所需的使用者帳戶設定認證之後，將 cred.xml 檔案儲存在安全的位置，或將它刪除。
5. 如果您在其中一個 Linux 節點上產生 RSA 金鑰組，請記得在您完成使用後刪除金鑰。 如果 HPC Pack 找到現有的 id_rsa 檔案或 id_rsa.pub 檔案，它就不會設定相互信任。

> [!IMPORTANT]
> 我們不建議以共用叢集上的叢集系統管理員身分執行 Linux 工作，因為系統管理員提交的工作會在 Linux 節點上的根帳戶底下執行。 不過，非系統管理員使用者所提交的作業會在具有與作業使用者相同名稱的本機 Linux 使用者帳戶下執行。 在此情況下，HPC Pack 會在配置給該作業的所有節點上，為此 Linux 使用者設定相互信任。 您可以在執行工作之前，手動在 Linux 節點上設定 Linux 使用者，否則 HPC Pack 會在工作提交時自動建立使用者。 如果 HPC Pack 建立使用者，HPC Pack 會在工作完成之後刪除它。 為了降低安全性威脅，HPC Pack 會在作業完成之後移除金鑰。
> 
> 

## <a name="set-up-a-file-share-for-linux-nodes"></a>為 Linux 節點設定檔案共用
現在，請在前端節點的資料夾上設定一個標準 SMB 共用。 若要允許 Linux 節點存取具有共用路徑的應用程式檔案，請將共用資料夾掛接在 Linux 節點上。 如果您想，您可以使用其他檔案共用選項 (例如 Azure 檔案共用，在許多案例中皆建議使用此選項) 或 NFS 共用。 請參閱[開始在 Azure 中的 HPC Pack 叢集使用 Linux 計算節點](hpcpack-cluster.md)中的檔案共用資訊和詳細步驟。

1. 在前端節點上建立資料夾，並藉由設定讀取/寫入權限與每個人共用。 例如，在前端節點上將 C:\OpenFOAM 共用為 \\\\SUSE12RDMA-HN\OpenFOAM。 在這裡， *SUSE12RDMA-HN* 是前端節點的主機名稱。
2. 開啟 Windows PowerShell 視窗並執行下列命令：
   
   ```
   clusrun /nodegroup:LinuxNodes mkdir -p /openfoam
   
   clusrun /nodegroup:LinuxNodes mount -t cifs //SUSE12RDMA-HN/OpenFOAM /openfoam -o vers=2.1`,username=<username>`,password='<password>'`,dir_mode=0777`,file_mode=0777
   ```

第一個命令會在 LinuxNodes 群組中的所有節點上建立名為 /openfoam 的資料夾。 第二個命令會將共用資料夾 //SUSE12RDMA-HN/OpenFOAM 掛接在 Linux 節點上，且 dir_mode 和 file_mode 位元會設為 777。 命令中的 *username* 和 *password* 應為前端節點上使用者的認證。

> [!NOTE]
> 第二個命令中的 “\`” 符號是 PowerShell 的逸出符號。 “\`,” 表示 “,” (逗號) 是命令的一部分。
> 
> 

## <a name="install-mpi-and-openfoam"></a>安裝 MPI 和 OpenFOAM
若要在 RDMA 網路上以 MPI 工作的形式執行 OpenFOAM，您必須使用 Intel MPI Library 編譯 OpenFOAM。 

首先，執行數個 **clusrun** 命令，以在您的 Linux 節點上安裝 Intel MPI 程式庫 (如果尚未安裝) 和 OpenFOAM。 請使用先前設定的前端節點共用，在 Linux 節點之間共用安裝檔案。

> [!IMPORTANT]
> 這些安裝和編譯步驟是範例。 您必須具備一些 Linux 系統管理知識，以確保正確安裝相依的編譯器和程式庫。 您可能需要修改一些特定的環境變數，或 Intel MPI 與 OpenFOAM 版本的其他設定。 如需詳細資料，請參閱適用於您環境的 [Intel MPI Library for Linux 安裝指南](http://registrationcenter-download.intel.com/akdlm/irc_nas/1718/INSTALL.html?lang=en&fileExt=.html)和 [OpenFOAM Source Pack 安裝](http://openfoam.org/download/2-3-1-source/)。
> 
> 

### <a name="install-intel-mpi"></a>安裝 Intel MPI
將下載的 Intel MPI 安裝套件 (在此範例中為 l_mpi_p_5.0.3.048.tgz) 儲存在前端節點的 C:\OpenFoam 中，使 Linux 節點能夠從 /openfoam 存取此檔案。 然後，執行 **clusrun** ，以在所有 Linux 節點上安裝 Intel MPI Library。

1. 下列命令會複製安裝套件，並將其解壓縮到每個節點的 /opt/intel 上。
   
   ```
   clusrun /nodegroup:LinuxNodes mkdir -p /opt/intel
   
   clusrun /nodegroup:LinuxNodes cp /openfoam/l_mpi_p_5.0.3.048.tgz /opt/intel/
   
   clusrun /nodegroup:LinuxNodes tar -xzf /opt/intel/l_mpi_p_5.0.3.048.tgz -C /opt/intel/
   ```
2. 若要以無訊息方式安裝 Intel MPI Library，請使用 silent.cfg 檔案。 您可以在本文結尾處的範例檔案中找到範例。 將此檔案放在共用資料夾 /openfoam 中。 如需 silent.cfg 檔案的詳細資訊，請參閱 [Intel MPI Library for Linux 安裝指南 - 無訊息安裝](http://registrationcenter-download.intel.com/akdlm/irc_nas/1718/INSTALL.html?lang=en&fileExt=.html#silentinstall)。
   
   > [!TIP]
   > 請確實將您的 silent.cfg 檔案儲存為具有 Linux 行尾結束符號 (只有 LF，不是 CR LF) 的文字檔。 此步驟可確保它在 Linux 節點上正常執行。
   > 
   > 
3. 以無訊息模式安裝 Intel MPI Library。
   
   ```
   clusrun /nodegroup:LinuxNodes bash /opt/intel/l_mpi_p_5.0.3.048/install.sh --silent /openfoam/silent.cfg
   ```

### <a name="configure-mpi"></a>設定 MPI
若要進行測試，您應在每個 Linux 節點的 /etc/security/limits.conf 中加入以下幾行：

    clusrun /nodegroup:LinuxNodes echo "*               hard    memlock         unlimited" `>`> /etc/security/limits.conf
    clusrun /nodegroup:LinuxNodes echo "*               soft    memlock         unlimited" `>`> /etc/security/limits.conf


更新 limits.conf 檔案之後，請重新啟動 Linux 節點。 例如，使用下列 **clusrun** 命令：

```
clusrun /nodegroup:LinuxNodes systemctl reboot
```

重新啟動之後，請確定共用資料夾已掛接為 /openfoam。

### <a name="compile-and-install-openfoam"></a>編譯和安裝 OpenFOAM
將下載的 OpenFOAM Source Pack 安裝套件 (在此範例中為 OpenFOAM-2.3.1.tgz) 儲存至前端節點的 C:\OpenFoam，使 Linux 節點能夠從 /openfoam 存取此檔案。 然後，執行 **clusrun** 命令，以在所有 Linux 節點上編譯 OpenFOAM。

1. 在每個 Linux 節點上建立資料夾 /opt/OpenFOAM、將來源封裝複製到此資料夾，並在該處加以解壓縮。
   
   ```
   clusrun /nodegroup:LinuxNodes mkdir -p /opt/OpenFOAM
   
   clusrun /nodegroup:LinuxNodes cp /openfoam/OpenFOAM-2.3.1.tgz /opt/OpenFOAM/
   
   clusrun /nodegroup:LinuxNodes tar -xzf /opt/OpenFOAM/OpenFOAM-2.3.1.tgz -C /opt/OpenFOAM/
   ```
2. 若要使用 Intel MPI Library 編譯 OpenFOAM，請先設定 Intel MPI 和 OpenFOAM 的某些環境變數。 請使用名為 settings.sh 的 Bash 指令碼來設定變數。 您可以在本文結尾處的範例檔案中找到範例。 將此檔案 (使用 Linux 行尾結束符號儲存的) 放在共用資料夾 /openfoam 中。 此檔案也包含您後續用來執行 OpenFOAM 工作的 MPI 和 OpenFOAM 執行階段的設定。
3. 安裝編譯 OpenFOAM 所需的相依封裝。 根據您的 Linux 散發套件，您可能需要先新增儲存機制。 執行類似下列的 **clusrun** 命令：
   
    ```
    clusrun /nodegroup:LinuxNodes zypper ar http://download.opensuse.org/distribution/13.2/repo/oss/suse/ opensuse
   
    clusrun /nodegroup:LinuxNodes zypper -n --gpg-auto-import-keys install --repo opensuse --force-resolution -t pattern devel_C_C++
    ```
   
    如有需要，可透過 SSH 連線到每個 Linux 節點來執行命令，以確認它們是否正常執行。
4. 執行下列命令以編譯 OpenFOAM。 編譯程序需要一些時間才能完成，並且會在標準輸出中產生大量的記錄資訊，因此，請使用 **/interleaved** 選項以便以交錯方式顯示輸出。
   
   ```
   clusrun /nodegroup:LinuxNodes /interleaved source /openfoam/settings.sh `&`& /opt/OpenFOAM/OpenFOAM-2.3.1/Allwmake
   ```
   
   > [!NOTE]
   > 命令中的 “\`” 符號是 PowerShell 的逸出符號。 “\`&” 表示 “&” 是命令的一部分。
   > 
   > 

## <a name="prepare-to-run-an-openfoam-job"></a>準備執行 OpenFOAM 工作
現在，請準備在兩個 Linux 節點上執行名為 sloshingTank3D 的 MPI 作業，這是其中一個 OpenFoam 範例。 

### <a name="set-up-the-runtime-environment"></a>設定執行階段環境
若要在 Linux 節點上設定 MPI 和 OpenFOAM 的執行階段環境，請在前端節點上的 Windows PowerShell 視窗中執行下列命令。 (此命令僅適用於 SUSE Linux。)

```
clusrun /nodegroup:LinuxNodes cp /openfoam/settings.sh /etc/profile.d/
```

### <a name="prepare-sample-data"></a>準備範例資料
請使用您先前設定的前端節點共用，在 Linux 節點之間共用檔案 (掛接為 /openfoam)。

1. 透過 SSH 連接到您其中一個 Linux 計算節點。
2. 執行下列命令以設定 OpenFOAM 執行階段環境 (如果您尚未執行此作業)。
   
   ```
   $ source /openfoam/settings.sh
   ```
3. 將 sloshingTank3D 範例複製到共用資料夾，並瀏覽到該資料夾。
   
   ```
   $ cp -r $FOAM_TUTORIALS/multiphase/interDyMFoam/ras/sloshingTank3D /openfoam/
   
   $ cd /openfoam/sloshingTank3D
   ```
4. 如果您使用此範例的預設參數，其執行時間可能需要數十分鐘，因此您可能會想要修改某些參數，讓它執行得更快。 一個簡單的選擇是修改 system/controlDict 檔案中的時間步階變數 deltaT 和 writeInterval。 此檔案儲存了所有與控制時間以及讀取和寫入解決方案資料有關的輸入資料。 例如，您可以將 deltaT 的值從 0.05 變更為 0.5，以及將 writeInterval 的值從 0.05 變更為 0.5。
   
   ![修改步階變數][step_variables]
5. 在 system/decomposeParDict 檔案中指定所要的變數值。 此範例使用兩個分別具有 8 個核心的 Linux 節點，因此，請將 numberOfSubdomains 設為 16，以及將 hierarchicalCoeffs 的 n 設為 (1 1 16)，這表示以 16 個處理程序平行執行 OpenFOAM。 如需詳細資訊，請參閱 [OpenFOAM User Guide: 3.4 Running applications in parallel (OpenFOAM 使用者指南：3.4 平行執行應用程式)](http://cfd.direct/openfoam/user-guide/running-applications-parallel/#x12-820003.4)。
   
   ![分解程序][decompose]
6. 從 sloshingTank3D 目錄執行下列命令，以準備範例資料。
   
   ```
   $ . $WM_PROJECT_DIR/bin/tools/RunFunctions
   
   $ m4 constant/polyMesh/blockMeshDict.m4 > constant/polyMesh/blockMeshDict
   
   $ runApplication blockMesh
   
   $ cp 0/alpha.water.org 0/alpha.water
   
   $ runApplication setFields  
   ```
7. 在前端節點上，您應該會看到範例資料檔案複製到 C:\OpenFoam\sloshingTank3D 中。 (C:\OpenFoam 是前端節點上的共用資料夾。)
   
   ![前端節點上的資料檔案][data_files]

### <a name="host-file-for-mpirun"></a>mpirun 的主機檔案
在此步驟中，您會建立 **mpirun** 命令所使用的主機檔案 (計算節點清單)。

1. 在其中一個 Linux 節點上的 /openfoam 下，建立名為 hostfile 的檔案，以便在所有 Linux 節點上的 /openfoam/hostfile 都可存取此檔案。
2. 將您的 Linux 節點名稱寫入此檔案中。 在此範例中，檔案會包含下列名稱：
   
   ```       
   SUSE12RDMA-LN1
   SUSE12RDMA-LN2
   ```
   
   > [!TIP]
   > 您也可以在前端節點的 C:\OpenFoam\hostfile 上建立此檔案。 如果您選擇這個選項，請將其儲存為具有 Linux 行尾結束符號 (只有 LF，不是 CR LF) 的文字檔。 這可確保它在 Linux 節點上正常運作。
   > 
   > 
   
   **Bash 指令碼包裝函式**
   
   如果您有許多 Linux 節點，而您想要讓您的作業只在其中一些節點上執行，則不建議使用固定的主機檔案，因為您不知道系統會將哪些節點配置給您的作業。 在此情況下，請為 **mpirun** 撰寫 Bash 指令碼包裝函式，以自動建立主機檔案。 您可以在本文結尾找到名為 hpcimpirun.sh 的範例 Bash 指令碼包裝函式，並將其儲存為 /openfoam/hpcimpirun.sh。此範例指令碼會執行下列動作：
   
   1. 設定 **mpirun**的環境變數，和透過 RDMA 網路執行 MPI 工作時所使用的某些附加命令參數。 在此範例中，它會設定下列變數：
      
      * I_MPI_FABRICS=shm:dapl
      * I_MPI_DAPL_PROVIDER=ofa-v2-ib0
      * I_MPI_DYNAMIC_CONNECTION=0
   2. 根據環境變數 $CCP_NODES_CORES 建立主機檔案，該變數會在工作啟動時由 HPC 前端節點設定。
      
      $CCP_NODES_CORES 的格式會遵循下列模式：
      
      ```
      <Number of nodes> <Name of node1> <Cores of node1> <Name of node2> <Cores of node2>...`
      ```
      
      其中
      
      * `<Number of nodes>` - 配置給此工作的節點數目。  
      * `<Name of node_n_...>` - 配置給此工作的各節點名稱。
      * `<Cores of node_n_...>` - 配置給此工作的節點上核心數目。
      
      例如，如果作業需要兩個節點才能執行，則 $CCP_NODES_CORES 會類似於：
      
      ```
      2 SUSE12RDMA-LN1 8 SUSE12RDMA-LN2 8
      ```
   3. 呼叫 **mpirun** 命令，並將兩個參數附加到命令列。
      
      * `--hostfile <hostfilepath>: <hostfilepath>` - 指令碼所建立的主機檔案路徑
      * `-np ${CCP_NUMCPUS}: ${CCP_NUMCPUS}` - HPC Pack 前端節點所設定的環境變數，會儲存配置給此工作的總核心數目。 在此案例中，它會指定 **mpirun**的處理程序數目。

## <a name="submit-an-openfoam-job"></a>提交 OpenFOAM 工作
現在，您可以在 HPC 叢集管理員中提交工作。 您必須在命令列中，針對某些作業工作傳遞指令碼 hpcimpirun.sh。

1. 連接至您的叢集前端節點並且啟動 HPC 叢集管理員。
2. 在 [資源管理] 中，確定 Linux 計算節點處於 [線上] 狀態。 如果不是，請選取這些節點，然後按一下 [ **上線**]。
3. 在 [工作管理] 中，按一下 [新增工作]。
4. 輸入作業的名稱，例如 *sloshingTank3D*。
   
   ![工作詳細資料][job_details]
5. 在 [ **工作資源**] 中，選取 [節點] 做為資源類型，並將 [最小值] 設為 2。 在此範例中，此組態會在兩個分別具有 8 個核心的 Linux 節點上執行作業。
   
   ![工作資源][job_resources]
6. 按一下左導覽窗格中的 [編輯工作]，然後按一下 [加入] 來將工作加入到作業中。 請使用下列命令列與設定，將四個工作新增到作業中。
   
   > [!NOTE]
   > 執行 `source /openfoam/settings.sh` 會設定 OpenFOAM 和 MPI 執行階段環境，因此下列每個作業會在 OpenFOAM 命令之前加以呼叫。
   > 
   > 
   
   * **工作 1**。 執行 **decomposePar**，產生以平行方式執行 **interDyMFoam** 的資料檔案。
     
     * 將一個節點指派給工作
     * **命令列** - `source /openfoam/settings.sh && decomposePar -force > /openfoam/decomposePar${CCP_JOBID}.log`
     * **工作目錄** - /openfoam/sloshingTank3D
     
     請參閱下圖。 以同樣的方式設定其餘的工作。
     
     ![作業 1 詳細資料][task_details1]
   * **工作 2**。 以平行方式執行 **interDyMFoam** ，以計算範例。
     
     * 將兩個節點指派給工作
     * **命令列** - `source /openfoam/settings.sh && /openfoam/hpcimpirun.sh interDyMFoam -parallel > /openfoam/interDyMFoam${CCP_JOBID}.log`
     * **工作目錄** - /openfoam/sloshingTank3D
   * **工作 3**。 執行 **reconstructPar**，以將來自每個 processor_N_ 目錄的數組時間目錄合併成一組目錄。
     
     * 將一個節點指派給工作
     * **命令列** - `source /openfoam/settings.sh && reconstructPar > /openfoam/reconstructPar${CCP_JOBID}.log`
     * **工作目錄** - /openfoam/sloshingTank3D
   * **工作  4**。 以平行方式執行 **foamToEnsight** ，以將 OpenFOAM 結果檔案轉換成 EnSight 格式，並將 EnSight 檔案放在案例目錄中名為 Ensight 的目錄內。
     
     * 將兩個節點指派給工作
     * **命令列** - `source /openfoam/settings.sh && /openfoam/hpcimpirun.sh foamToEnsight -parallel > /openfoam/foamToEnsight${CCP_JOBID}.log`
     * **工作目錄** - /openfoam/sloshingTank3D
7. 以遞增作業順序，將相依性新增至這些作業。
   
   ![作業相依性][task_dependencies]
8. 按一下 [ **提交** ] 以執行此工作。
   
   根據預設，HPC Pack 會以您目前登入的使用者帳戶提交工作。 按一下 [ **提交**] 之後可能會出現一個對話方塊，提示您輸入使用者名稱和密碼。
   
   ![工作認證][creds]
   
   在某些情況下，HPC Pack 會記住您以前輸入的使用者資訊，而不會顯示此對話方塊。 若要讓 HPC Pack 再次顯示此對話方塊，請在命令提示字元中輸入下列命令，然後提交作業。
   
   ```
   hpccred delcreds
   ```
9. 執行工作可能需要數十分鐘到數小時不等，視您為範例設定的參數而定。 在熱度圖中，您會看到作業在 Linux 節點上執行。 
   
   ![熱度圖][heat_map]
   
   在每個節點上會啟動 8 個處理程序。
   
   ![Linux 程序][linux_processes]
10. 工作完成時，請 C:\OpenFoam\sloshingTank3D 下的資料夾中找出工作結果，記錄檔則位於 C:\OpenFoam 上。

## <a name="view-results-in-ensight"></a>在 EnSight 中檢視結果
您可以選擇性地使用 [EnSight](http://www.ensight.com/) 來視覺化和分析 OpenFOAM 工作的結果。 如需 EnSight 中的視覺效果和動畫的詳細資訊，請參閱此 [視訊指南](http://www.ensight.com/ensight.com/envideo/)。

1. 在前端節點上安裝 EnSight 之後，請加以啟動。
2. 開啟 C:\OpenFoam\sloshingTank3D\EnSight\sloshingTank3D.case。
   
   您會在檢視器中看見一個儲存槽。
   
   ![EnSight 中的儲存槽][tank]
3. 從 **internalMesh** 建立 **Isosurface**，然後選擇變數 **alpha_water**。
   
   ![建立 isosurface][isosurface]
4. 為先前的步驟中建立的 **Isosurface_part** 設定色彩。 例如，將它設為水藍色。
   
   ![編輯 isosurface 色彩][isosurface_color]
5. 在 [組件] 面板中選取 [圖表牆]，以從 [圖表牆] 建立 **Iso-volume**，然後按一下工具列中的 **Isosurfaces** 按鈕。
6. 在對話方塊中，選取 **Isovolume** 做為 [類型]，然後將 [Isovolume 範圍] 的最小值設為 0.5。 若要建立 isovolume，請按一下 [使用選取的組件建立] 。
7. 為先前的步驟中建立的 **Iso_volume_part** 設定色彩。 例如，將它設為深藍色。
8. 設定 [ **圖表牆**] 的色彩。 例如，將它設為透明的白色。
9. 現在，按一下 [ **播放** ] 以檢視模擬的結果。
   
    ![儲存槽結果][tank_result]

## <a name="sample-files"></a>範例檔案
### <a name="sample-xml-configuration-file-for-cluster-deployment-by-powershell-script"></a>PowerShell 指令碼叢集部署的 XML 組態檔範例
 ```
<?xml version="1.0" encoding="utf-8" ?>
<IaaSClusterConfig>
  <Subscription>
    <SubscriptionName>Subscription-1</SubscriptionName>
    <StorageAccount>allvhdsje</StorageAccount>
  </Subscription>
  <Location>Japan East</Location>  
  <VNet>
    <VNetName>suse12rdmavnet</VNetName>
    <SubnetName>SUSE12RDMACluster</SubnetName>
  </VNet>
  <Domain>
    <DCOption>HeadNodeAsDC</DCOption>
    <DomainFQDN>hpclab.local</DomainFQDN>
  </Domain>
  <Database>
    <DBOption>LocalDB</DBOption>
  </Database>
  <HeadNode>
    <VMName>SUSE12RDMA-HN</VMName>
    <ServiceName>suse12rdma-je</ServiceName>
    <VMSize>A8</VMSize>
    <EnableRESTAPI />
    <EnableWebPortal />
  </HeadNode>
  <LinuxComputeNodes>
    <VMNamePattern>SUSE12RDMA-LN%1%</VMNamePattern>
    <ServiceName>suse12rdma-je</ServiceName>
    <VMSize>A8</VMSize>
    <NodeCount>2</NodeCount>
      <ImageName>b4590d9e3ed742e4a1d46e5424aa335e__suse-sles-12-hpc-v20150708</ImageName>
  </LinuxComputeNodes>
</IaaSClusterConfig>
```

### <a name="sample-credxml-file"></a>範例 cred.xml 檔案
```
<ExtendedData>
  <PrivateKey>-----BEGIN RSA PRIVATE KEY-----
MIIEpQIBAAKCAQEAxJKBABhnOsE9eneGHvsjdoXKooHUxpTHI1JVunAJkVmFy8JC
qFt1pV98QCtKEHTC6kQ7tj1UT2N6nx1EY9BBHpZacnXmknpKdX4Nu0cNlSphLpru
lscKPR3XVzkTwEF00OMiNJVknq8qXJF1T3lYx3rW5EnItn6C3nQm3gQPXP0ckYCF
Jdtu/6SSgzV9kaapctLGPNp1Vjf9KeDQMrJXsQNHxnQcfiICp21NiUCiXosDqJrR
AfzePdl0XwsNngouy8t0fPlNSngZvsx+kPGh/AKakKIYS0cO9W3FmdYNW8Xehzkc
VzrtJhU8x21hXGfSC7V0ZeD7dMeTL3tQCVxCmwIDAQABAoIBAQCve8Jh3Wc6koxZ
qh43xicwhdwSGyliZisoozYZDC/ebDb/Ydq0BYIPMiDwADVMX5AqJuPPmwyLGtm6
9hu5p46aycrQ5+QA299g6DlF+PZtNbowKuvX+rRvPxagrTmupkCswjglDUEYUHPW
05wQaNoSqtzwS9Y85M/b24FfLeyxK0n8zjKFErJaHdhVxI6cxw7RdVlSmM9UHmah
wTkW8HkblbOArilAHi6SlRTNZG4gTGeDzPb7fYZo3hzJyLbcaNfJscUuqnAJ+6pT
iY6NNp1E8PQgjvHe21yv3DRoVRM4egqQvNZgUbYAMUgr30T1UoxnUXwk2vqJMfg2
Nzw0ESGRAoGBAPkfXjjGfc4HryqPkdx0kjXs0bXC3js2g4IXItK9YUFeZzf+476y
OTMQg/8DUbqd5rLv7PITIAqpGs39pkfnyohPjOe2zZzeoyaXurYIPV98hhH880uH
ZUhOxJYnlqHGxGT7p2PmmnAlmY4TSJrp12VnuiQVVVsXWOGPqHx4S4f9AoGBAMn/
vuea7hsCgwIE25MJJ55FYCJodLkioQy6aGP4NgB89Azzg527WsQ6H5xhgVMKHWyu
Q1snp+q8LyzD0i1veEvWb8EYifsMyTIPXOUTwZgzaTTCeJNHdc4gw1U22vd7OBYy
nZCU7Tn8Pe6eIMNztnVduiv+2QHuiNPgN7M73/x3AoGBAOL0IcmFgy0EsR8MBq0Z
ge4gnniBXCYDptEINNBaeVStJUnNKzwab6PGwwm6w2VI3thbXbi3lbRAlMve7fKK
B2ghWNPsJOtppKbPCek2Hnt0HUwb7qX7Zlj2cX/99uvRAjChVsDbYA0VJAxcIwQG
TxXx5pFi4g0HexCa6LrkeKMdAoGAcvRIACX7OwPC6nM5QgQDt95jRzGKu5EpdcTf
g4TNtplliblLPYhRrzokoyoaHteyxxak3ktDFCLj9eW6xoCZRQ9Tqd/9JhGwrfxw
MS19DtCzHoNNewM/135tqyD8m7pTwM4tPQqDtmwGErWKj7BaNZARUlhFxwOoemsv
R6DbZyECgYEAhjL2N3Pc+WW+8x2bbIBN3rJcMjBBIivB62AwgYZnA2D5wk5o0DKD
eesGSKS5l22ZMXJNShgzPKmv3HpH22CSVpO0sNZ6R+iG8a3oq4QkU61MT1CfGoMI
a8lxTKnZCsRXU1HexqZs+DSc+30tz50bNqLdido/l5B4EJnQP03ciO0=
-----END RSA PRIVATE KEY-----</PrivateKey>
  <PublicKey>ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDEkoEAGGc6wT16d4Ye+yN2hcqigdTGlMcjUlW6cAmRWYXLwkKoW3WlX3xAK0oQdMLqRDu2PVRPY3qfHURj0EEellpydeaSekp1fg27Rw2VKmEumu6Wxwo9HddXORPAQXTQ4yI0lWSerypckXVPeVjHetbkSci2foLedCbeBA9c/RyRgIUl227/pJKDNX2Rpqly0sY82nVWN/0p4NAyslexA0fGdBx+IgKnbU2JQKJeiwOomtEB/N492XRfCw2eCi7Ly3R8+U1KeBm+zH6Q8aH8ApqQohhLRw71bcWZ1g1bxd6HORxXOu0mFTzHbWFcZ9ILtXRl4Pt0x5Mve1AJXEKb username@servername;</PublicKey>
</ExtendedData>
```
### <a name="sample-silentcfg-file-to-install-mpi"></a>安裝 MPI 的 silent.cfg 檔案範例
```
# Patterns used to check silent configuration file
#
# anythingpat - any string
# filepat     - the file location pattern (/file/location/to/license.lic)
# lspat       - the license server address pattern (0123@hostname)
# snpat       - the serial number pattern (ABCD-01234567)

# accept EULA, valid values are: {accept, decline}
ACCEPT_EULA=accept

# optional error behavior, valid values are: {yes, no}
CONTINUE_WITH_OPTIONAL_ERROR=yes

# install location, valid values are: {/opt/intel, filepat}
PSET_INSTALL_DIR=/opt/intel

# continue with overwrite of existing installation directory, valid values are: {yes, no}
CONTINUE_WITH_INSTALLDIR_OVERWRITE=yes

# list of components to install, valid values are: {ALL, DEFAULTS, anythingpat}
COMPONENTS=DEFAULTS

# installation mode, valid values are: {install, modify, repair, uninstall}
PSET_MODE=install

# directory for non-RPM database, valid values are: {filepat}
#NONRPM_DB_DIR=filepat

# Serial number, valid values are: {snpat}
#ACTIVATION_SERIAL_NUMBER=snpat

# License file or license server, valid values are: {lspat, filepat}
#ACTIVATION_LICENSE_FILE=

# Activation type, valid values are: {exist_lic, license_server, license_file, trial_lic, serial_number}
ACTIVATION_TYPE=trial_lic

# Path to the cluster description file, valid values are: {filepat}
#CLUSTER_INSTALL_MACHINES_FILE=filepat

# Intel(R) Software Improvement Program opt-in, valid values are: {yes, no}
PHONEHOME_SEND_USAGE_DATA=no

# Perform validation of digital signatures of RPM files, valid values are: {yes, no}
SIGNING_ENABLED=yes

# Select yes to enable mpi-selector integration, valid values are: {yes, no}
ENVIRONMENT_REG_MPI_ENV=no

# Select yes to update ld.so.conf, valid values are: {yes, no}
ENVIRONMENT_LD_SO_CONF=no


```

### <a name="sample-settingssh-script"></a>範例 settings.sh 指令碼
```
#!/bin/bash

# impi
source /opt/intel/impi/5.0.3.048/bin64/mpivars.sh
export MPI_ROOT=$I_MPI_ROOT
export I_MPI_FABRICS=shm:dapl
export I_MPI_DAPL_PROVIDER=ofa-v2-ib0
export I_MPI_DYNAMIC_CONNECTION=0

# openfoam
export FOAM_INST_DIR=/opt/OpenFOAM
source /opt/OpenFOAM/OpenFOAM-2.3.1/etc/bashrc
export WM_MPLIB=INTELMPI
```


### <a name="sample-hpcimpirunsh-script"></a>範例 hpcimpirun.sh 指令碼
```
#!/bin/bash

# The path of this script
SCRIPT_PATH="$( dirname "${BASH_SOURCE[0]}" )"

# Set mpirun runtime evironment
source /opt/intel/impi/5.0.3.048/bin64/mpivars.sh
export MPI_ROOT=$I_MPI_ROOT
export I_MPI_FABRICS=shm:dapl
export I_MPI_DAPL_PROVIDER=ofa-v2-ib0
export I_MPI_DYNAMIC_CONNECTION=0

# mpirun command
MPIRUN=mpirun
# Argument of "--hostfile"
NODELIST_OPT="--hostfile"
# Argument of "-np"
NUMPROCESS_OPT="-np"

# Get node information from ENVs
NODESCORES=(${CCP_NODES_CORES})
COUNT=${#NODESCORES[@]}

if [ ${COUNT} -eq 0 ]
then
    # CCP_NODES_CORES is not found or is empty, just run the mpirun without hostfile arg.
    ${MPIRUN} $*
else
    # Create the hostfile file
    NODELIST_PATH=${SCRIPT_PATH}/hostfile_$$

    # Get every node name and write into the hostfile file
    I=1
    while [ ${I} -lt ${COUNT} ]
    do
        echo "${NODESCORES[${I}]}" >> ${NODELIST_PATH}
        let "I=${I}+2"
    done

    # Run the mpirun with hostfile arg
    ${MPIRUN} ${NUMPROCESS_OPT} ${CCP_NUMCPUS} ${NODELIST_OPT} ${NODELIST_PATH} $*

    RTNSTS=$?
    rm -f ${NODELIST_PATH}
fi

exit ${RTNSTS}

```





<!--Image references-->
[keygen]:media/hpcpack-cluster-openfoam/keygen.png
[keys]:media/hpcpack-cluster-openfoam/keys.png
[step_variables]:media/hpcpack-cluster-openfoam/step_variables.png
[data_files]:media/hpcpack-cluster-openfoam/data_files.png
[decompose]:media/hpcpack-cluster-openfoam/decompose.png
[job_details]:media/hpcpack-cluster-openfoam/job_details.png
[job_resources]:media/hpcpack-cluster-openfoam/job_resources.png
[task_details1]:media/hpcpack-cluster-openfoam/task_details1.png
[task_dependencies]:media/hpcpack-cluster-openfoam/task_dependencies.png
[creds]:media/hpcpack-cluster-openfoam/creds.png
[heat_map]:media/hpcpack-cluster-openfoam/heat_map.png
[tank]:media/hpcpack-cluster-openfoam/tank.png
[tank_result]:media/hpcpack-cluster-openfoam/tank_result.png
[isosurface]:media/hpcpack-cluster-openfoam/isosurface.png
[isosurface_color]:media/hpcpack-cluster-openfoam/isosurface_color.png
[linux_processes]:media/hpcpack-cluster-openfoam/linux_processes.png
