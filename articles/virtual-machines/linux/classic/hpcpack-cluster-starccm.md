---
title: 使用 HPC Pack 在 Linux VM 上執行 STAR-CCM+ | Microsoft Docs
description: 在 Azure 上部署 Microsoft HPC Pack 叢集，並且跨 RDMA 網路在多個 Linux 計算節點上執行 STAR-CCM+ 作業。
services: virtual-machines-linux
documentationcenter: ''
author: xpillons
manager: jeconnoc
editor: ''
tags: azure-service-management,azure-resource-manager,hpc-pack
ms.assetid: 75523406-d268-4623-ac3e-811c7b74de4b
ms.service: virtual-machines-linux
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: big-compute
ms.date: 09/13/2016
ms.author: xpillons
ms.openlocfilehash: 8689d7abfd5ab45277df3b5672a1f6e7e874d88e
ms.sourcegitcommit: 6fcd9e220b9cd4cb2d4365de0299bf48fbb18c17
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/05/2018
ms.locfileid: "30841482"
---
# <a name="run-star-ccm-with-microsoft-hpc-pack-on-a-linux-rdma-cluster-in-azure"></a>在 Azure 中的 Linux RDMA 叢集以 Microsoft HPC Pack 執行 STAR-CCM+
本文將說明如何在 Azure 上部署 Microsoft HPC Pack 叢集，並在與 InfiniBand 相互連接的多個 Linux 計算節點上執行 [CD-adapco STAR-CCM+](http://www.cd-adapco.com/products/star-ccm%C2%AE) 作業。

[!INCLUDE [learn-about-deployment-models](../../../../includes/learn-about-deployment-models-both-include.md)]

Microsoft HPC Pack 提供功能來執行各種大規模 HPC 和平行應用程式，包括 Microsoft Azure 虛擬機器的叢集上的 MPI 應用程式。 HPC Pack 也支援在 HPC Pack 叢集中部署的 Linux 計算節點 VM 上，執行 Linux HPC 應用程式。 如需搭配使用 Linux 計算節點與 HPC Pack 的簡介，請參閱[開始在 Azure 中的 HPC Pack 叢集使用 Linux 計算節點](hpcpack-cluster.md)。

## <a name="set-up-an-hpc-pack-cluster"></a>設定 HPC Pack 叢集
從[下載中心](https://www.microsoft.com/en-us/download/details.aspx?id=44949)下載 HPC Pack IaaS 部署指令碼，並將它們解壓縮到本機。

Azure PowerShell 是必要條件。 如果未在本機電腦上設定 PowerShell，請閱讀本文： [如何安裝和設定 Azure PowerShell](/powershell/azure/overview)。

撰寫本文時，來自 Azure Marketplace 的 Linux 映像 (其包含 Azure InfiniBand 驅動程式) 適用於 SLES 12、CentOS 6.5 和 CentOS 7.1。 這篇文章根據 SLES 12 的使用方式。 若要擷取 Marketplace 中支援 HPC 的所有 Linux 映像的名稱，您可以執行下列 PowerShell 命令︰

```
    get-azurevmimage | ?{$_.ImageName.Contains("hpc") -and $_.OS -eq "Linux" }
```

輸出會列出提供這些映像的位置以及稍後會在部署範本中使用的映像名稱 (**ImageName**)。

部署叢集之前，必須建置 HPC Pack 部署範本檔案。 因為我們的目標是小型叢集，所以前端節點是網域控制站，並裝載本機 SQL Database。

下列範本會部署這類前端節點、建立名為 **MyCluster.xml** 的 XML 檔案，並將 **SubscriptionId**、**StorageAccount**、**Location**、**VMName** 和 **ServiceName** 的值取代成您的值。

    <?xml version="1.0" encoding="utf-8" ?>
    <IaaSClusterConfig>
      <Subscription>
        <SubscriptionId>99999999-9999-9999-9999-999999999999</SubscriptionId>
        <StorageAccount>mystorageaccount</StorageAccount>
      </Subscription>
      <Location>North Europe</Location>
      <VNet>
        <VNetName>hpcvnetne</VNetName>
        <SubnetName>subnet-hpc</SubnetName>
      </VNet>
      <Domain>
        <DCOption>HeadNodeAsDC</DCOption>
        <DomainFQDN>hpc.local</DomainFQDN>
      </Domain>
      <Database>
        <DBOption>LocalDB</DBOption>
      </Database>
      <HeadNode>
        <VMName>myhpchn</VMName>
        <ServiceName>myhpchn</ServiceName>
        <VMSize>Standard_D4</VMSize>
      </HeadNode>
      <LinuxComputeNodes>
        <VMNamePattern>lnxcn-%0001%</VMNamePattern>
        <ServiceNamePattern>mylnxcn%01%</ServiceNamePattern>
        <MaxNodeCountPerService>20</MaxNodeCountPerService>
        <StorageAccountNamePattern>mylnxstorage%01%</StorageAccountNamePattern>
        <VMSize>A9</VMSize>
        <NodeCount>0</NodeCount>
        <ImageName>b4590d9e3ed742e4a1d46e5424aa335e__suse-sles-12-hpc-v20150708</ImageName>
      </LinuxComputeNodes>
    </IaaSClusterConfig>

在提高權限的命令提示字元中執行 PowerShell 命令，開始建立前端節點︰

```
    .\New-HPCIaaSCluster.ps1 -ConfigFile MyCluster.xml
```

20 到 30 分鐘之後，前端節點應該會就緒。 從 Azure 入口網站上按一下虛擬機器的 [連接]  圖示，即可連接到它。

您最終可能必須修正 DNS 轉寄站。 若要這樣做，請啟動 [DNS 管理員]。

1. 在 [DNS 管理員] 中以滑鼠右鍵按一下伺服器名稱，並選取 [內容]，然後按一下 [轉寄站] 索引標籤。
2. 按一下 [編輯] 按鈕來移除任何轉寄站，然後按一下 [確定]。
3. 確定已選取 [如果沒有可用的轉寄站，則使用根目錄提示] 核取方塊，然後按一下 [確定]。

## <a name="set-up-linux-compute-nodes"></a>設定 Linux 計算節點
部署 Linux 計算節點，方法是使用用來建立前端節點的相同部署範本。

將檔案 **MyCluster.xml** 從本機電腦的複製到前端節點，並以您想要部署的節點數目 (<= 20) 更新 **NodeCount** 標籤。 您的 Azure 配額中務必要有足夠的可用核心，因為每個 A9 執行個體將會使用您訂用帳戶中的 16 個核心。 如果您想要在相同的預算中使用更多的 VM，您可以使用 A8 執行個體 (8 核心) 而不是 A9。

在前端節點上，複製 HPC Pack IaaS 部署指令碼。

在提高權限的命令提示字元中，執行下列 Azure PowerShell 命令：

1. 執行 **Add-AzureAccount** 以連接到您的 Azure 訂用帳戶。
2. 如果您有多個訂用帳戶，請執行 **Get-AzureSubscription** 來列出這些訂用帳戶。
3. 執行 **Select-AzureSubscription -SubscriptionName xxxx -Default** 命令來設定預設訂用帳戶。
4. 執行 **.\New-HPCIaaSCluster.ps1 -ConfigFile MyCluster.xml** 以開始部署 Linux 計算節點。
   
   ![前端節點部署實務][hndeploy]

開啟 [HPC Pack 叢集管理員] 工具。 幾分鐘後，Linux 計算節點會定期出現在叢集計算節點的清單中。 使用傳統部署模式，會循序建立 IaaS VM。 因此，如果節點數目十分重要，則部署所有節點可能需要很長的時間。

![HPC Pack 叢集管理員中的 Linux 節點][clustermanager]

現在，叢集中的所有節點都已啟動並執行，還有其他可進行的基礎結構設定。

## <a name="set-up-an-azure-file-share-for-windows-and-linux-nodes"></a>為 Windows 和 Linux 節點設定 Azure 檔案共用
您可以使用 Azure 檔案服務來儲存指令碼、應用程式封裝和資料檔案。 Azure 檔案提供 Azure Blob 儲存體的 CIFS 功能，以當成持續性存放區。 請注意，這不是最具調整性的解決方案，但它是最簡單且不需要專用 VM 的解決方案。

依照文章[在 Windows 上開始使用 Azure 檔案儲存體](../../../storage/files/storage-dotnet-how-to-use-files.md)的指示來建立 Azure 檔案共用。

將儲存體帳戶名稱保持為 **saname**、將檔案共用名稱保持為 **sharename** 以及將儲存體帳戶金鑰保持為 **sakey**。

### <a name="mount-the-azure-file-share-on-the-head-node"></a>在前端節點上裝載 Azure 檔案共用
開啟提升權限的命令提示字元，然後執行下列命令，將認證儲存在本機電腦保存庫中：

```
    cmdkey /add:<saname>.file.core.windows.net /user:<saname> /pass:<sakey>
```

然後，若要裝載 Azure 檔案共用，請執行：

```
    net use Z: \\<saname>.file.core.windows.net\<sharename> /persistent:yes
```

### <a name="mount-the-azure-file-share-on-linux-compute-nodes"></a>在 Linux 計算節點上裝載 Azure 檔案共用
HPC Pack 隨附的一個實用工具是 clusrun 工具。 您可以使用此命令列工具，在一組計算節點上同時執行相同的命令。 在本例中，它用來裝載 Azure 檔案共用，然後將它保存以避免重新開機。
在前端節點的提高權限命令提示字元中，執行下列命令。

建立裝載目錄：

```
    clusrun /nodegroup:LinuxNodes mkdir -p /hpcdata
```

裝載 Azure 檔案共用：

```
    clusrun /nodegroup:LinuxNodes mount -t cifs //<saname>.file.core.windows.net/<sharename> /hpcdata -o vers=2.1,username=<saname>,password='<sakey>',dir_mode=0777,file_mode=0777
```

保存掛接共用：

```
    clusrun /nodegroup:LinuxNodes "echo //<saname>.file.core.windows.net/<sharename> /hpcdata cifs vers=2.1,username=<saname>,password='<sakey>',dir_mode=0777,file_mode=0777 >> /etc/fstab"
```

## <a name="install-star-ccm"></a>安裝 STAR-CCM+
Azure VM 執行個體 A8 和 A9 提供 InfiniBand 支援與 RDMA 功能。 啟用這些功能的核心驅動程式都適用於 Azure Marketplace 中的 Windows Server 2012 R2、SUSE 12、CentOS 6.5 和 CentOS 7.1 映像。 Microsoft MPI 和 Intel MPI (5.x 版) 是 Azure 中可支援這些驅動程式的兩個 MPI 程式庫。

CD-adapco STAR-CCM+ 發行 11.x 和更新版本隨附 Intel MPI 5.x 版，因此含有 Azure 的 InfiniBand 支援。

從 [CD-adapco 入口網站](https://steve.cd-adapco.com)取得 Linux64 STAR-CCM+ 封裝。 在我們的案例中，以混合精確度使用版本 11.02.010。

在前端節點上，於 **/hpcdata** Azure 檔案共用中，使用下列內容建立名為 **setupstarccm.sh** 的殼層指令碼。 此指令碼將會在每個計算節點上執行，以在本機上設定 STAR-CCM+。

#### <a name="sample-setupstarcmsh-script"></a>範例 setupstarcm.sh 指令碼
```
    #!/bin/bash
    # setupstarcm.sh to set up STAR-CCM+ locally

    # Create the CD-adapco main directory
    mkdir -p /opt/CD-adapco

    # Copy the STAR-CCM package from the file share to the local directory
    cp /hpcdata/StarCCM/STAR-CCM+11.02.010_01_linux-x86_64.tar.gz /opt/CD-adapco/

    # Extract the package
    tar -xzf /opt/CD-adapco/STAR-CCM+11.02.010_01_linux-x86_64.tar.gz -C /opt/CD-adapco/

    # Start a silent installation of STAR-CCM without the FLEXlm component
    /opt/CD-adapco/starccm+_11.02.010/STAR-CCM+11.02.010_01_linux-x86_64-2.5_gnu4.8.bin -i silent -DCOMPUTE_NODE=true -DNODOC=true -DINSTALLFLEX=false

    # Update memory limits
    echo "*               hard    memlock         unlimited" >> /etc/security/limits.conf
    echo "*               soft    memlock         unlimited" >> /etc/security/limits.conf
```
現在，若要在所有 Linux 計算節點上設定 STAR-CCM+，請開啟提升權限的命令提示字元，並執行下列命令：

```
    clusrun /nodegroup:LinuxNodes bash /hpcdata/setupstarccm.sh
```

執行命令時，您可以使用叢集管理員的熱圖來監視 CPU 使用量。 幾分鐘後，所有節點應該都會正確設定。

## <a name="run-star-ccm-jobs"></a>執行 STAR-CCM+ 作業
HPC Pack 用於其工作排程器功能，以便執行 STAR-CCM+ 作業。 若要這樣做，我們需要用來開始作業和執行 STAR-CCM+ 的一些指令碼的支援。 為了簡單起見，輸入資料會先保留在 Azure 檔案共用上。

下列 PowerShell 指令碼用於將 STAR-CCM+ 作業放入佇列。 它會採用三個引數︰

* 模型名稱
* 要使用的節點數目
* 要在每個節點上使用的核心數目

因為 STAR-CCM+ 可以填入記憶體頻寬，一般來說，最好每個計算節點使用較少的核心，並新增新節點。 每個節點的確切核心數目取決於處理器系列和相互連接速度。

會對節點配置專用的作業，而且不能與其他作業共用。 作業未直接啟動為 MPI 作業。 **runstarccm.sh** 殼層指令碼會啟動 MPI 啟動器。

輸入模型和 **runstarccm.sh** 指令碼會儲存在先前掛接的 **/hpcdata** 共用中。

記錄檔會以工作識別碼來命名，並與 STAR-CCM+ 輸出檔案一起儲存在 **/hpcdata 共用**中。

#### <a name="sample-submitstarccmjobps1-script"></a>範例 SubmitStarccmJob.ps1 指令碼
```
    Add-PSSnapin Microsoft.HPC -ErrorAction silentlycontinue
    $scheduler="headnodename"
    $modelName=$args[0]
    $nbCoresPerNode=$args[2]
    $nbNodes=$args[1]

    #---------------------------------------------------------------------------------------------------------
    # Create a new job; this will give us the job ID that's used to identify the name of the uploaded package in Azure
    #
    $job = New-HpcJob -Name "$modelName $nbNodes $nbCoresPerNode" -Scheduler $scheduler -NumNodes $nbNodes -NodeGroups "LinuxNodes" -FailOnTaskFailure $true -Exclusive $true
    $jobId = [String]$job.Id

    #---------------------------------------------------------------------------------------------------------
    # Submit the job     
    $workdir =  "/hpcdata"
    $execName = "$nbCoresPerNode runner.java $modelName.sim"

    $job | Add-HpcTask -Scheduler $scheduler -Name "Compute" -stdout "$jobId.log" -stderr "$jobId.err" -Rerunnable $false -NumNodes $nbNodes -Command "runstarccm.sh $execName" -WorkDir "$workdir"


    Submit-HpcJob -Job $job -Scheduler $scheduler
```
將 **runner.java** 取代成您偏好的 STAR-CCM+ Java 模型啟動程式與記錄程式碼。

#### <a name="sample-runstarccmsh-script"></a>範例 runstarccm.sh 指令碼
```
    #!/bin/bash
    echo "start"
    # The path of this script
    SCRIPT_PATH="$( dirname "${BASH_SOURCE[0]}" )"
    echo ${SCRIPT_PATH}
    # Set the mpirun runtime environment
    export CDLMD_LICENSE_FILE=1999@flex.cd-adapco.com

    # mpirun command
    STARCCM=/opt/CD-adapco/STAR-CCM+11.02.010/star/bin/starccm+

    # Get node information from ENVs
    NODESCORES=(${CCP_NODES_CORES})
    COUNT=${#NODESCORES[@]}
    NBCORESPERNODE=$1

    # Create the hostfile file
    NODELIST_PATH=${SCRIPT_PATH}/hostfile_$$
    echo ${NODELIST_PATH}

    # Get every node name and write into the hostfile file
    I=1
    NBNODES=0
    while [ ${I} -lt ${COUNT} ]
    do
        echo "${NODESCORES[${I}]}" >> ${NODELIST_PATH}
        let "I=${I}+2"
        let "NBNODES=${NBNODES}+1"
    done
    let "NBCORES=${NBNODES}*${NBCORESPERNODE}"

    # Run STAR-CCM with the hostfile argument
    #  
    ${STARCCM} -np ${NBCORES} -machinefile ${NODELIST_PATH} \
        -power -podkey "<yourkey>" -rsh ssh \
        -mpi intel -fabric UDAPL -cpubind bandwidth,v \
        -mppflags "-ppn $NBCORESPERNODE -genv I_MPI_DAPL_PROVIDER=ofa-v2-ib0 -genv I_MPI_DAPL_UD=0 -genv I_MPI_DYNAMIC_CONNECTION=0" \
        -batch $2 $3
    RTNSTS=$?
    rm -f ${NODELIST_PATH}

    exit ${RTNSTS}
```

在我們的測試中，我們使用的是 Power-One-Demand 授權權杖。 針對該權杖，您必須將 **$CDLMD_LICENSE_FILE** 環境變數設定為 **1999@flex.cd-adapco.com**，以及命令列 **-podkey** 選項中的索引鍵。

在進行一些初始化之後，指令碼會從 HPC Pack 設定的 **$CCP_NODES_CORES** 環境變數中，擷取節點清單來建置 MPI 啟動器使用的 hostfile。 此 hostfile 將包含用於作業的計算節點名稱的清單，一行一個名稱。

**$CCP_NODES_CORES** 的格式會遵循下列模式：

```
<Number of nodes> <Name of node1> <Cores of node1> <Name of node2> <Cores of node2>...`
```

其中：

* `<Number of nodes>` 是配置給此作業的節點數目。
* `<Name of node_n_...>` 是配置給此作業的每個節點名稱。
* `<Cores of node_n_...>` 是配置給此作業的節點上核心數目。

核心數目 (**$NBCORES**) 也是根據節點的數目 (**$NBNODES**) 和每個節點的核心數目 (以 **$NBCORESPERNODE** 參數提供) 來計算。

針對 MPI 選項，在 Azure 上與 Intel MPI 搭配使用的選項如下︰

* `-mpi intel` 可指定 Intel MPI。
* `-fabric UDAPL` 可使用 Azure InfiniBand 動詞。
* `-cpubind bandwidth,v` 可使用 STAR-CCM+ 將 MPI 的頻寬最佳化。
* `-mppflags "-ppn $NBCORESPERNODE -genv I_MPI_DAPL_PROVIDER=ofa-v2-ib0 -genv I_MPI_DAPL_UD=0 -genv I_MPI_DYNAMIC_CONNECTION=0"` 可讓 Intel MPI 與 Azure InfiniBand 搭配運作，並可設定每個節點的必要核心數目。
* `-batch` 可以用批次模式啟動 STAR-CCM+ (無使用者介面)。

最後，若要開始作業，請確定您的節點啟動並執行，並且在叢集管理員中處於線上狀態。 然後從 PowerShell 命令提示字元中，執行此作業︰

```
    .\ SubmitStarccmJob.ps1 <model> <nbNodes> <nbCoresPerNode>
```

## <a name="stop-nodes"></a>停止節點
之後，當您完成測試之後，您可以使用下列 HPC Pack PowerShell 命令來停止和啟動節點︰

```
    Stop-HPCIaaSNode.ps1 -Name <prefix>-00*
    Start-HPCIaaSNode.ps1 -Name <prefix>-00*
```

## <a name="next-steps"></a>後續步驟
嘗試執行其他 Linux 工作負載。 例如，請參閱︰

* [在 Azure 中的 Linux 運算節點以 Microsoft HPC Pack 執行 NAMD](hpcpack-cluster-namd.md)
* [在 Azure 中的 Linux RDMA 叢集以 Microsoft HPC Pack 執行 OpenFOAM](hpcpack-cluster-openfoam.md)

<!--Image references-->
[hndeploy]:media/hpcpack-cluster-starccm/hndeploy.png
[clustermanager]:media/hpcpack-cluster-starccm/ClusterManager.png
