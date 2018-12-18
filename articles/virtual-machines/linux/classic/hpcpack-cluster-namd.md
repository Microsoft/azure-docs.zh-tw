---
title: Linux VM 上的 NAMD 與 Microsoft HPC Pack | Microsoft Docs
description: 在 Azure 上部署 Microsoft HPC Pack 叢集，並執行在多個 Linux 運算節點上具有 charmrun 的 NAMD 模擬
services: virtual-machines-linux
documentationcenter: ''
author: dlepow
manager: jeconnoc
editor: ''
tags: azure-service-management,azure-resource-manager,hpc-pack
ms.assetid: 76072c6b-ac35-4729-ba67-0d16f9443bd7
ms.service: virtual-machines-linux
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: big-compute
ms.date: 10/13/2016
ms.author: danlep
ms.openlocfilehash: 61dd49d4bd3183b6b9a78036d6d7d01798e4dc89
ms.sourcegitcommit: 6fcd9e220b9cd4cb2d4365de0299bf48fbb18c17
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/05/2018
ms.locfileid: "30842008"
---
# <a name="run-namd-with-microsoft-hpc-pack-on-linux-compute-nodes-in-azure"></a>在 Azure 中的 Linux 運算節點以 Microsoft HPC Pack 執行 NAMD
本文將說明在 Azure 虛擬機器上執行 Linux 高效能運算 (HPC) 工作負載的方法。 在這裡，您會在 Azure 上使用多個 Linux 計算節點設定 [Microsoft HPC Pack](https://technet.microsoft.com/library/cc514029) 叢集以及執行 [NAMD](http://www.ks.uiuc.edu/Research/namd/) 模擬，以計算和視覺化大型生物分子系統的結構。  

[!INCLUDE [learn-about-deployment-models](../../../../includes/learn-about-deployment-models-both-include.md)]

* **NAMD** (適用於奈米分子動力程式) 是專為高效能模擬大型生物分子系統而設計的平行分子動力套件，包含多達數百萬個原子。 這些系統的範例包括病毒、儲存格結構和大型蛋白質。 NAMD 會針對典型模擬縮放到數百個核心，以及針對最大型的模擬縮放至超過 500,000 個核心。
* **Microsoft HPC Pack** 提供在內部部署電腦或 Azure 虛擬機器的叢集中執行大規模 HPC 和平行應用程式 (包括的 MPI 應用程式) 的功能。 HPC Pack 的開發前身為 Windows HPC 工作負載的解決方案，其現已支援於部署在 HPC Pack 叢集中的 Linux 計算節點 VM 上，執行 Linux HPC 應用程式。 如需簡介，請參閱 [在 Azure 的 HPC Pack 叢集中開始使用 Linux 運算節點](hpcpack-cluster.md) 。

## <a name="prerequisites"></a>先決條件
* **具備 Linux 計算節點的 HPC Pack 叢集** - 使用 [Azure Resource Manager 範本](https://azure.microsoft.com/marketplace/partners/microsofthpc/newclusterlinuxcn/)或 [Azure PowerShell 指令碼](hpcpack-cluster-powershell-script.md)，在 Azure 上部署具備 Linux 計算節點的 HPC Pack 叢集。 如需了解任一選項的必要條件與步驟，請參閱 [開始使用 Azure 中 HPC Pack 叢集內的 Linux 計算節點](hpcpack-cluster.md) 。 如果您選擇 PowerShell 指令碼部署選項，請參閱本文結尾範例檔案中的範例組態檔。 此檔案會設定 Windows Server 2012 R2 的前端節點和四個大型 CentOS 6.6 計算節點組成之以 Azure 為基礎的 HPC Pack 叢集。 請視環境需要自訂此檔案。
* **NAMD 軟體與教學課程檔案** - 從 [NAMD](http://www.ks.uiuc.edu/Research/namd/) 網站下載 Linux 的 NAMD 軟體 (需要註冊)。 本文是以 NAMD 2.10 版為基礎，並使用 [Linux-x86_64 (64 位元 Intel/AMD 乙太網路)](http://www.ks.uiuc.edu/Development/Download/download.cgi?UserID=&AccessCode=&ArchiveID=1310) 封存。 另請下載 [NAMD 教學課程檔案](http://www.ks.uiuc.edu/Training/Tutorials/#namd)。 下載的是 .tar 檔案，而您需要 Windows 工具以解壓縮叢集前端節點上的檔案。 若要解壓縮檔案，請遵循本文稍後的指示。 
* **VMD** (選擇性) - 若要查看 NAMD 工作的結果，請在您選擇的電腦上，下載和安裝分子視覺化程式 [VMD](http://www.ks.uiuc.edu/Research/vmd/) 。 目前版本為 1.9.2。 請參閱 VMD 下載網站以開始作業。  

## <a name="set-up-mutual-trust-between-compute-nodes"></a>設定運算節點之間的相互信任
在多個 Linux 節點上執行跨節點作業，需要節點相互信任 (藉由 **rsh** 或 **ssh**)。 當您使用 Microsoft HPC Pack IaaS 部署指令碼建立 HPC Pack 叢集時，指令碼會自動為您指定的系統管理員帳戶設定永久相互信任。 針對您在叢集的網域中建立的非系統管理員使用者，您必須在將工作配置給他們時，設定節點間的暫時相互信任。 然後，在工作完成之後終結關聯性。 若要為每個使用者執行此動作，提供 HPC Pack 用來建立信任關係的 RSA 金鑰組給叢集。 指示如下。

### <a name="generate-an-rsa-key-pair"></a>產生 RSA 金鑰組
產生 RSA 金鑰組很容易，其中包含公開金鑰和私密金鑰，方法是執行 Linux **ssh-keygen** 命令。

1. 登入 Linux 電腦。
2. 執行以下命令：
   
   ```bash
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
1. 請使用您部署叢集時所提供的網域認證 (例如，hpc\clusteradmin) [由遠端桌面連線](../../windows/connect-logon.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)到前端節點 VM。 您可以從前端節點管理叢集。
2. 您可以使用標準的 Windows Server 程序在叢集的 Active Directory 網域中建立網域使用者帳戶。 例如，在前端節點上使用 Active Directory 使用者和電腦工具。 本文中的範例假設您在 hpclab 網域中建立名為 hpcuser 的網域使用者 (hpclab\hpcuser)。
3. 將網域使用者以叢集使用者身分加入 HPC Pack 叢集中。 如需指示，請參閱 [(Add or Remove Cluster Users) 新增或移除叢集使用者](https://technet.microsoft.com/library/ff919330.aspx)。
4. 建立名為 C:\cred.xml 的檔案，並且將 RSA 金鑰資料複製到其中。 您可以在本文結尾處的範例檔案中找到範例。
   
   ```
   <ExtendedData>
     <PrivateKey>Copy the contents of private key here</PrivateKey>
     <PublicKey>Copy the contents of public key here</PublicKey>
   </ExtendedData>
   ```
5. 開啟命令提示字元並輸入下列命令，以設定 hpclab\hpcuser 帳戶的認證資料。 您使用 **extendeddata** 參數來傳遞您針對金鑰資料建立的 C:\cred.xml 檔案名稱。
   
   ```command
   hpccred setcreds /extendeddata:c:\cred.xml /user:hpclab\hpcuser /password:<UserPassword>
   ```
   
   這個命令會成功完成而沒有輸出。 為您執行工作所需的使用者帳戶設定認證之後，將 cred.xml 檔案儲存在安全的位置，或將它刪除。
6. 如果您在其中一個 Linux 節點上產生 RSA 金鑰組，請記得在您完成使用後刪除金鑰。 如果 HPC Pack 找到現有的 id_rsa 檔案或 id_rsa.pub 檔案，則它不會設定相互信任。

> [!IMPORTANT]
> 我們不建議以共用叢集上的叢集系統管理員身分執行 Linux 工作，因為系統管理員提交的工作會在 Linux 節點上的根帳戶底下執行。 非系統管理員使用者所提交的作業會在具有與作業使用者相同名稱的本機 Linux 使用者帳戶下執行。 在此情況下，HPC Pack 會在配置給該作業的所有節點上，為此 Linux 使用者設定相互信任。 您可以在執行工作之前，手動在 Linux 節點上設定 Linux 使用者，否則 HPC Pack 會在工作提交時自動建立使用者。 如果 HPC Pack 建立使用者，HPC Pack 會在工作完成之後刪除它。 若要降低安全性威脅，金鑰會在工作完成之後於節點上移除。
> 
> 

## <a name="set-up-a-file-share-for-linux-nodes"></a>為 Linux 節點設定檔案共用
現在請設定 SMB 檔案共用，並在所有 Linux 節點上裝載共用資料夾，允許 Linux 節點存取具有共用路徑的 NAMD 檔案。 以下是在前端節點上裝載共用資料夾的步驟。 建議共用分配，例如目前不支援 Azure 檔案服務的 CentOS 6.6。 如果您的 Linux 節點支援 Azure 檔案共用，請參閱[如何搭配使用 Azure 檔案儲存體與 Linux](../../../storage/files/storage-how-to-use-files-linux.md)。 如需 HPC Pack 的其他檔案共用選項，請參閱 [開始在 Azure 中的 HPC Pack 叢集使用 Linux 運算節點](hpcpack-cluster.md)。

1. 在前端節點上建立資料夾，並藉由設定讀取/寫入權限與每個人共用。 在此範例中，\\\\CentOS66HN\Namd 是資料夾的名稱，其中CentOS66HN 是前端節點的主機名稱。
2. 在共用資料夾中建立名為 namd2 的子資料夾。 在 namd2 中，建立另一個名為 namdsample 的子資料夾。
3. 在資料夾中解壓縮 NAMD 檔案，方法是使用 Windows 的 **tar** 版本，或其他可以操作 .tar 封存的 Windows 公用程式。 
   
   * 將 NAMD tar 封存解壓縮到 \\\\CentOS66HN\Namd\namd2。
   * 解壓縮 \\\\CentOS66HN\Namd\namd2\namdsample 之下的教學課程檔案。
4. 開啟 Windows PowerShell 視窗並執行下列命令來將共用資料夾裝載在 Linux 節點上。
   
    ```command
    clusrun /nodegroup:LinuxNodes mkdir -p /namd2
   
    clusrun /nodegroup:LinuxNodes mount -t cifs //CentOS66HN/Namd/namd2 /namd2 -o vers=2.1`,username=<username>`,password='<password>'`,dir_mode=0777`,file_mode=0777
    ```

第一個命令會在 LinuxNodes 群組中的所有節點上建立名為 /namd2 的資料夾。 第二個命令會將共用資料夾 //CentOS66HN/Namd/namd2 掛接至此資料夾，其 dir_mode 和 file_mode 位元設為 777。 命令中的 *username* 和 *password* 應為前端節點上使用者的認證。

> [!NOTE]
> 第二個命令中的 “\`” 符號是 PowerShell 的逸出符號。 “\`,” 表示 “,” (逗號) 是命令的一部分。
> 
> 

## <a name="create-a-bash-script-to-run-a-namd-job"></a>建立執行 NAMD 作業的 Bash 指令碼
您的 NAMD 作業需要 *nodelist* 檔案，如此 **charmrun** 才可決定啟動 NAMD 程序時要使用的節點數目。 您會使用 Bash 指令碼，產生 nodelist 檔案，並在執行 **charmrun** 搭配此 nodelist 檔案。 然後您就可以在會呼叫此指令碼的 HPC 叢集管理員中提交 NAMD 工作。

使用您選擇的文字編輯器，在包含 NAMD 程式檔案的 /namd2 資料夾中建立 Bash 指令碼，並將它命名為 hpccharmrun.sh。如需快速概念證明，請複製本文結尾所提供的範例 hpccharmrun.sh 指令碼並移至[提交 NAMD 工作](#submit-a-namd-job)。

> [!TIP]
> 將您的指令碼儲存為具有 Linux 行尾結束符號 (只有 LF，不是 CR LF) 的文字檔案。 這可確保它在 Linux 節點上正常運作。
> 
> 

以下是這個 bash 指令碼的詳細作業內容。 

1. 定義一些變數。
   
   ```bash
   #!/bin/bash
   
   # The path of this script
   SCRIPT_PATH="$( dirname "${BASH_SOURCE[0]}" )"
   # Charmrun command
   CHARMRUN=${SCRIPT_PATH}/charmrun
   # Argument of ++nodelist
   NODELIST_OPT="++nodelist"
   # Argument of ++p
   NUMPROCESS="+p"
   ```
2. 從環境變數取得節點資訊。 $NODESCORES 會儲存來自 $CCP_NODES_CORES 的分割單字清單。 $COUNT 是 $NODESCORES 的大小。
   
   ```
   # Get node information from the environment variables
   NODESCORES=(${CCP_NODES_CORES})
   COUNT=${#NODESCORES[@]}
   ```    
   
   $CCP_NODES_CORES 變數的格式如下所示：
   
   ```
   <Number of nodes> <Name of node1> <Cores of node1> <Name of node2> <Cores of node2>…
   ```
   
   此變數會列出節點總數、節點名稱和配置給工作之每個節點上的核心數目。 例如，如果工作需要 10 個核心以執行，$CCP_NODES_CORES 的值會類似：
   
   ```
   3 CENTOS66LN-00 4 CENTOS66LN-01 4 CENTOS66LN-03 2
   ```
3. 如果未設定 $CCP_NODES_CORES 變數，請直接啟動 **charmrun**。 (這應該只有當您在 Linux 節點上直接執行這個指令碼時才會發生。)
   
   ```
   if [ ${COUNT} -eq 0 ]
   then
     # CCP_NODES is_CORES is not found or is empty, so just run charmrun without nodelist arg.
     #echo ${CHARMRUN} $*
     ${CHARMRUN} $*
   ```
4. 或者為 **charmrun**建立節點清單檔案。
   
   ```
   else
     # Create the nodelist file
     NODELIST_PATH=${SCRIPT_PATH}/nodelist_$$
   
     # Write the head line
     echo "group main" > ${NODELIST_PATH}
   
     # Get every node name and number of cores and write into the nodelist file
     I=1
     while [ ${I} -lt ${COUNT} ]
     do
         echo "host ${NODESCORES[${I}]} ++cpus ${NODESCORES[$(($I+1))]}" >> ${NODELIST_PATH}
         let "I=${I}+2"
     done
   ```
5. 搭配節點清單檔案執行 **charmrun** ，取得其傳回狀態，並且在結束時移除節點清單檔案。
   
   ${CCP_NUMCPUS} 是 HPC Pack 前端節點所設定的另一個環境變數。 它會儲存配置給這項工作的總核心數目。 我們可以用它來為 charmrun 指定處理序數目。
   
   ```
   # Run charmrun with nodelist arg
   #echo ${CHARMRUN} ${NUMPROCESS}${CCP_NUMCPUS} ${NODELIST_OPT} ${NODELIST_PATH} $*
   ${CHARMRUN} ${NUMPROCESS}${CCP_NUMCPUS} ${NODELIST_OPT} ${NODELIST_PATH} $*
   
   RTNSTS=$?
   rm -f ${NODELIST_PATH}
   fi
   
   ```
6. 當 **charmrun** 傳回狀態時結束。
   
   ```
   exit ${RTNSTS}
   ```

以下是指令碼產生的節點清單檔案中的資訊：

```
group main
host <Name of node1> ++cpus <Cores of node1>
host <Name of node2> ++cpus <Cores of node2>
…
```

例如︰

```
group main
host CENTOS66LN-00 ++cpus 4
host CENTOS66LN-01 ++cpus 4
host CENTOS66LN-03 ++cpus 2
```

## <a name="submit-a-namd-job"></a>提交 NAMD 作業
現在您已準備就緒在 HPC 叢集管理員中提交 NAMD 工作。

1. 連接至您的叢集前端節點並且啟動 HPC 叢集管理員。
2. 在 [Resource Management] 中，確定 Linux 計算節點處於 [線上] 狀態。 如果不是，請選取這些節點，然後按一下 [ **上線**]。
3. 在 [工作管理] 中，按一下 [新增工作]。
4. 輸入工作的名稱，例如 *hpccharmrun*。
   
   ![新的 HPC 工作][namd_job]
5. 在 [工作詳細資料] 頁面的 [工作資源] 底下，選取 [節點] 做為資源類型，並且將 [最小值] 設為 3。 ，我們在三個 Linux 節點上執行工作，且每個節點都有四個核心。
   
   ![工作資源][job_resources]
6. 按一下左導覽窗格中的 [編輯工作]，然後按一下 [加入] 來將工作加入到作業中。    
7. 在 [工作詳細資料和 I/O 重新導向] 頁面上，設定下列值：
   
   * **命令列** -
     `/namd2/hpccharmrun.sh ++remote-shell ssh /namd2/namd2 /namd2/namdsample/1-2-sphere/ubq_ws_eq.conf > /namd2/namd2_hpccharmrun.log`
     
     > [!TIP]
     > 前面的命令列是不含分行符號的單一命令。 它會換行以出現在**命令列**的數行之下。
     > 
     > 
   * **工作目錄** - /namd2
   * **最小值** - 3
     
     ![作業詳細資料][task_details]
     
     > [!NOTE]
     > 您在這裡設定工作目錄，因為 **charmrun** 嘗試瀏覽至每個節點上相同的工作目錄。 如果未設定工作目錄，HPC Pack 會在其中一個 Linux 節點上建立的隨機命名資料夾中啟動命令。 這會在其他節點上導致下列錯誤：`/bin/bash: line 37: cd: /tmp/nodemanager_task_94_0.mFlQSN: No such file or directory.` 若要避免這個問題，指定所有節點可存取為工作目錄的資料夾路徑。
     > 
     > 
8. 按一下 [確定]，然後按一下 [提交] 以執行此作業。
   
   根據預設，HPC Pack 會以您目前登入的使用者帳戶提交工作。 對話方塊可能會在您按一下 [提交] 之後提示您輸入使用者名稱和密碼。
   
   ![工作認證][creds]
   
   在某些情況下，HPC Pack 會記住您以前輸入的使用者資訊，而不會顯示此對話方塊。 若要讓 HPC Pack 再次顯示此對話方塊，請在命令提示字元中輸入下列命令，然後提交作業。
   
   ```command
   hpccred delcreds
   ```
9. 工作需要數分鐘的時間才能完成。
10. 在 \\<headnodeName>\Namd\namd2\namd2_hpccharmrun.log 中尋找工作記錄檔，在 \\<headnodeName>\Namd\namd2\namdsample\1-2-sphere\. 中尋找輸出檔案
11. 選擇性啟動 VMD 以檢視您的工作結果。 用來視覺化 NAMD 輸出檔案 (在此案例中，水圈中的泛素蛋白質分子) 的步驟已超出本文的範圍。 如需詳細資訊，請參閱 [NAMD 教學課程](http://www.life.illinois.edu/emad/biop590c/namd-tutorial-unix-590C.pdf) 。
    
    ![工作結果][vmd_view]

## <a name="sample-files"></a>範例檔案
### <a name="sample-xml-configuration-file-for-cluster-deployment-by-powershell-script"></a>PowerShell 指令碼叢集部署的 XML 組態檔範例
```
<?xml version="1.0" encoding="utf-8" ?>
<IaaSClusterConfig>
  <Subscription>
    <SubscriptionName>Subscription-1</SubscriptionName>
    <StorageAccount>mystorageaccount</StorageAccount>
  </Subscription>
      <Location>West US</Location>  
  <VNet>
    <VNetName>MyVNet</VNetName>
    <SubnetName>Subnet-1</SubnetName>
  </VNet>
  <Domain>
    <DCOption>HeadNodeAsDC</DCOption>
    <DomainFQDN>hpclab.local</DomainFQDN>
  </Domain>
  <Database>
    <DBOption>LocalDB</DBOption>
  </Database>
  <HeadNode>
    <VMName>CentOS66HN</VMName>
    <ServiceName>MyHPCService</ServiceName>
    <VMSize>Large</VMSize>
    <EnableRESTAPI />
    <EnableWebPortal />
  </HeadNode>
  <LinuxComputeNodes>
    <VMNamePattern>CentOS66LN-%00%</VMNamePattern>
    <ServiceName>MyLnxCNService</ServiceName>
     <VMSize>Large</VMSize>
     <NodeCount>4</NodeCount>
    <ImageName>5112500ae3b842c8b9c604889f8753c3__OpenLogic-CentOS-66-20150325</ImageName>
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

### <a name="sample-hpccharmrunsh-script"></a>範例 hpccharmrun.sh 指令碼
```
#!/bin/bash

# The path of this script
SCRIPT_PATH="$( dirname "${BASH_SOURCE[0]}" )"
# Charmrun command
CHARMRUN=${SCRIPT_PATH}/charmrun
# Argument of ++nodelist
NODELIST_OPT="++nodelist"
# Argument of ++p
NUMPROCESS="+p"

# Get node information from ENVs
# CCP_NODES_CORES=3 CENTOS66LN-00 4 CENTOS66LN-01 4 CENTOS66LN-03 4
NODESCORES=(${CCP_NODES_CORES})
COUNT=${#NODESCORES[@]}

if [ ${COUNT} -eq 0 ]
then
    # If CCP_NODES_CORES is not found or is empty, just run the charmrun without nodelist arg.
    #echo ${CHARMRUN} $*
    ${CHARMRUN} $*
else
    # Create the nodelist file
    NODELIST_PATH=${SCRIPT_PATH}/nodelist_$$

    # Write the head line
    echo "group main" > ${NODELIST_PATH}

    # Get every node name & cores and write into the nodelist file
    I=1
    while [ ${I} -lt ${COUNT} ]
    do
        echo "host ${NODESCORES[${I}]} ++cpus ${NODESCORES[$(($I+1))]}" >> ${NODELIST_PATH}
        let "I=${I}+2"
    done

    # Run the charmrun with nodelist arg
    #echo ${CHARMRUN} ${NUMPROCESS}${CCP_NUMCPUS} ${NODELIST_OPT} ${NODELIST_PATH} $*
    ${CHARMRUN} ${NUMPROCESS}${CCP_NUMCPUS} ${NODELIST_OPT} ${NODELIST_PATH} $*

    RTNSTS=$?
    rm -f ${NODELIST_PATH}
fi

exit ${RTNSTS}
```

 

<!--Image references-->
[keygen]:media/hpcpack-cluster-namd/keygen.png
[keys]:media/hpcpack-cluster-namd/keys.png
[namd_job]:media/hpcpack-cluster-namd/namd_job.png
[job_resources]:media/hpcpack-cluster-namd/job_resources.png
[creds]:media/hpcpack-cluster-namd/creds.png
[task_details]:media/hpcpack-cluster-namd/task_details.png
[vmd_view]:media/hpcpack-cluster-namd/vmd_view.png
