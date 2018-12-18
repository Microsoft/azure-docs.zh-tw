---
title: 在 Azure 中設定混合式 HPC Pack 叢集 | Microsoft Docs
description: 了解如何使用 Microsoft HPC Pack 和 Azure 設定一個小型的混合式高效能運算 (HPC) 叢集
services: cloud-services
documentationcenter: ''
author: dlepow
manager: timlt
editor: ''
tags: hpc-pack
ms.assetid: ''
ms.service: cloud-services
ms.workload: big-compute
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/22/2017
ms.author: danlep
ms.openlocfilehash: ad5c13723eef352148a40e3e7f4f2ff616867296
ms.sourcegitcommit: 3df3fcec9ac9e56a3f5282f6c65e5a9bc1b5ba22
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/04/2017
ms.locfileid: "23985301"
---
# <a name="set-up-a-hybrid-high-performance-computing-hpc-cluster-with-microsoft-hpc-pack-and-on-demand-azure-compute-nodes"></a>使用 Microsoft HPC Pack 和隨選 Azure 計算節點設定混合式高效能計算 (HPC) 叢集
使用 Microsoft HPC Pack 2012 R2 和 Azure 設定小型的混合式高效能運算 (HPC) 叢集。 本文中所示的叢集包含一個內部部署 HPC Pack 前端節點，以及一些您視需要部署在 Azure 雲端服務中的計算節點。 然後，您便可以在混合式叢集上執行計算作業。

![Hybrid HPC cluster][Overview] 

本教學課程示範一個有時稱為「將量擴大到雲端」的方法，此方法使用隨選 Azure 資源來執行大量計算的應用程式。

本教學課程假設您先前沒有使用計算叢集或 HPC Pack 的經驗。 其只是要協助您快速部署一個示範性質的混合式計算叢集。 如需有關使用 HPC Pack 2016，或有關在生產環境中以較大規模部署混合式 HPC Pack 叢集的考量和步驟，請參閱[詳細指引 (英文)](http://go.microsoft.com/fwlink/p/?LinkID=200493)。 如需使用 HPC Pack 的其他案例，包括 Azure 虛擬機器中的自動化叢集部署，請參閱[使用 HPC Pack 在 Azure 中建立及管理 Windows HPC 叢集的選項](../virtual-machines/windows/hpcpack-cluster-options.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)。

## <a name="prerequisites"></a>先決條件
* **Azure 訂用帳戶** - 如果您沒有 Azure 訂用帳戶，只需要幾分鐘就可以建立 [免費帳戶](https://azure.microsoft.com/free/) 。
* **一部執行 Windows Server 2012 R2 或 Windows Server 2012 的內部部署電腦** - 使用這部電腦作為 HPC 叢集的前端節點。 如果您目前執行的不是 Windows Server，可以下載並安裝 [評估版](https://www.microsoft.com/evalcenter/evaluate-windows-server-2012-r2)。
  
  * 電腦必須加入 Active Directory 網域。 基於測試目的，您可以將前端節點電腦設定為網域控制站。 若要新增 Active Directory 網域服務伺服器角色，並將前端節點電腦升級為網域控制站，請參閱 Windows Server 的文件。
  * 為了支援 HPC Pack，作業系統必須以下列其中一種語言安裝：英文、日文或簡體中心。
  * 確認已安裝重要及重大更新。
* **HPC Pack 2012 R2** - 免費[下載](http://go.microsoft.com/fwlink/p/?linkid=328024)最新版本的安裝套件，並將檔案複製到前端節點電腦。 選擇與安裝的 Windows Server 語言相同語言的安裝檔。

    >[!NOTE]
    > 如果您想要使用 HPC Pack 2016 而不是 HPC Pack 2012 R2，則需要額外設定。 請參閱[詳細指引](http://go.microsoft.com/fwlink/p/?LinkID=200493)。
    > 
* **網域帳戶** - 必須在前端節點上以本機系統管理員權限設定此帳戶，才能安裝 HPC Pack。
* **連接埠 443 上的 TCP 連線** 。

## <a name="install-hpc-pack-on-the-head-node"></a>在前端節點安裝 HPC Pack
您必須先在執行 Windows Server 的內部部署電腦上安裝 Microsoft HPC Pack。 此電腦會成為叢集的前端節點。

1. 使用具備本機系統管理員權限的網域帳戶登入前端節點。

2. 執行 HPC Pack 安裝檔中的 Setup.exe 來啟動 HPC Pack 安裝精靈。

3. 在 [HPC Pack 2012 R2 設定] 畫面上，按一下 [全新安裝或將新功能新增至現有安裝]。

    ![HPC Pack 2012 Setup][install_hpc1]

4. 在 [Microsoft 軟體使用者合約] 頁面上，按 [下一步]。

5. 在 [選取安裝類型] 頁面上，按一下 [藉由建立前端節點來建立新 HPC 叢集]，然後按 [下一步]。

6. 精靈會執行數項安裝前的測試。 如果所有測試皆通過，請在 [安裝規則]  on the  。 否則，請檢閱系統提供的資訊，並在您的環境中進行任何必要的變更。 然後重新執行測試，或是視需要重新啟動安裝精靈。
7. 在 [HPC DB 設定] 頁面上，確定已為所有 HPC 資料庫選取 [前端節點]，然後按 [下一步]。 

    ![DB Configuration][install_hpc4]

8. 接受精靈剩餘頁面上的預設選項。 在 [安裝必要元件] 頁面上，按一下 [安裝]。
   
    ![Install][install_hpc6]

9. 安裝完成之後，請取消勾選 [啟動 HPC 叢集管理員]，然後按一下 [完成]。 (您將在稍後的步驟中啟動 HPC 叢集管理員)。
   
    ![完成][install_hpc7]

## <a name="prepare-the-azure-subscription"></a>準備 Azure 訂閱
在 [Azure 入口網站](https://portal.azure.com)中對您的 Azure 訂用帳戶執行下列步驟。 完成這些步驟之後，您就可以從內部部署前端節點部署 Azure 節點。 
  
  > [!NOTE]
  > 請一併記下您的 Azure 訂用帳戶識別碼，稍後需要用到。 您可以在入口網站的 [訂用帳戶] 中找到此識別碼。
  > 

### <a name="upload-the-default-management-certificate"></a>上傳預設管理憑證
HPC Pack 會在前端節點安裝一個自我簽署憑證 (稱為 Default Microsoft HPC Azure Management 憑證)，您可以將它上傳作為 Azure 管理憑證。 這個憑證是為了方便進行測試及概念證明部署而提供，可保護前端節點與 Azure 之間的連線安全。

1. 從前端節點電腦登入 [Azure 入口網站](https://portal.azure.com)。

2. 按一下 [訂用帳戶] > 您的訂用帳戶名稱。

3. 按一下 [管理憑證] > [上傳]。

4. 瀏覽前端節點以找出 C:\Program Files\Microsoft HPC Pack 2012\Bin\hpccert.cer 檔案。 然後，按一下 [上傳]。

   
[Default HPC Azure Management] \(預設 HPC Azure 管理) 憑證會顯示在管理憑證清單中。

### <a name="create-an-azure-cloud-service"></a>建立 Azure 雲端服務
> [!NOTE]
> 為了獲得最佳效能，請在稍後的步驟中，將雲端服務和儲存體帳戶建立在同一個地理區域中。
> 
> 

1. 在入口網站中，按一下 [雲端服務 (傳統)] > [+新增]。

2.  鍵入服務的 DNS 名稱，選擇資源群組和位置，然後按一下 [建立]。


### <a name="create-an-azure-storage-account"></a>建立 Azure 儲存體帳戶
1. 在入口網站中，按一下 [儲存體帳戶 (傳統)] > [+新增]。

2. 鍵入帳戶的名稱，然後選取 [傳統] 部署模型。

3. 選擇資源群組和位置，並保留其他設定的預設值。 接著，按一下 [建立]。

## <a name="configure-the-head-node"></a>設定前端節點
若要使用 HPC 叢集管理員來部署 Azure 節點及提交工作，請先執行一些必要的叢集設定步驟。

1. 在前端節點上，啟動 HPC 叢集管理員。 如果顯示 [選取前端節點] 對話方塊，請按一下 [本機電腦]。 [Deployment To-do List]  隨即出現。

2. 在 [必要部署工作] 底下，按一下 [設定您的網路]。
   
    ![Configure Network][config_hpc2]

3. 在 [網路設定精靈] 中，選取 [All nodes only on an enterprise network]  \(拓撲 5)。 這是最簡單的網路設定，僅供示範之用。
   
    ![Topology 5][config_hpc3]
   
4. 按 [下一步]  以接受精靈剩餘頁面上的預設值。 然後，在 [檢閱] 索引標籤上，按一下 [設定] 以完成網路設定。

5. 在 [部署待辦事項清單] 中，按一下 [提供安裝認證]。

6. 在 [Installation Credentials]  對話方塊中，輸入您用來安裝 HPC Pack 之網域帳戶的認證。 然後按一下 [確定] 。 
   
    ![Installation Credentials][config_hpc6]
   
7. 在 [部署待辦事項清單] 中，按一下 [設定新節點的命名]。

8. 在 [指定節點命名序列] 對話方塊中，接受預設的命名序列，然後按一下 [確定]。 請完成這個步驟，即使您在本教學課程中新增的 Azure 節點會自動命名也一樣。
   
    ![Node Naming][config_hpc8]
   
9. 在 [部署待辦事項清單] 中，按一下 [建立節點範本]。 在本教學課程稍後，您可以使用節點範本將 Azure 節點新增至叢集。

10. 在 [Create Node Template Wizard] 中，執行下列動作：
    
    a. 在 [選擇節點範本類型] 頁面上，按一下 [Windows Azure 節點範本]，然後按一下 [下一步]。
    
    ![Node Template][config_hpc10]
    
    b. 按 [下一步] 以接受預設範本名稱。
    
    c. 然後，在 [管理憑證]  。 然後，在 [管理憑證] 中，瀏覽 [預設 Microsoft HPC Azure 管理]。 然後按 [下一步] 。
    
    ![Node Template][config_hpc12]
    
    d. 在 [提供服務資訊] 頁面上，選取您在先前步驟中建立的雲端服務和儲存體帳戶。 然後按 [下一步] 。
    
    ![Node Template][config_hpc13]
    
    e. 按 [下一步]  以接受精靈剩餘頁面上的預設值。 然後，在 [檢閱] 索引標籤上，按一下 [建立] 以建立節點範本。
    
    > [!NOTE]
    > 根據預設，Azure 節點範本包含可讓您使用 HPC 叢集管理員手動啟動 (佈建) 和停止節點的設定。 您可以選擇性地設定排程來自動啟動和停止 Azure 節點。
    > 
    > 

## <a name="add-azure-nodes-to-the-cluster"></a>將 Azure 節點新增至叢集
現在使用節點範本將 Azure 節點新增至叢集。 將節點新增至叢集會儲存其設定資訊，讓您可以隨時在雲端服務中啟動 (佈建) 這些節點。 在執行個體於雲端服務中執行之後，您的訂用帳戶才須為 Azure 節點付費。

請遵循下列步驟來新增兩個小型節點。

1. 在 HPC 叢集管理員中，按一下 [節點管理] \(在最新版本的 HPC Pack 中稱為 [資源管理]) > [新增節點]。
   
    ![Add Node][add_node1]

2. 在「新增節點精靈」中的 [選取部署方法] 頁面上，按一下 [新增 Windows Azure 節點]，然後按 [下一步]。
   
    ![Add Azure Node][add_node1_1]

3. 在 [指定新節點] 頁面上，選取您先前建立的 Azure 節點範本 (預設稱為 [預設 Azure 節點範本])。 接著，指定 **2** 個大小為 [小型] 的節點，然後按 [下一步]。
   
    ![Specify Nodes][add_node2]
   
4. 在 [完成新增節點精靈] 頁面上，按一下 [完成]。
    
     HPC 叢集管理員中現在會出現兩個 Azure 節點，名為 **AzureCN-0001** 和 **AzureCN-0002**。 兩者皆處於 [未部署]  狀態。
   
    ![Added Nodes][add_node3]

## <a name="start-the-azure-nodes"></a>啟動 Azure 節點
當您想要使用 Azure 中的叢集資源時，請使用 HPC 叢集管理員來啟動 (佈建) Azure 節點並讓節點上線。

1. 在 HPC 叢集管理員中，按一下 [節點管理] \(在最新版本的 HPC Pack 中稱為 [資源管理])，然後選取 Azure 節點。

2. 按一下 [開始]，然後按一下 [確定]。
   
   ![Start Nodes][add_node4]
   
    節點會轉換至 [正在佈建]  狀態。 檢視佈建記錄檔以追蹤佈建進度。
   
    ![Provision Nodes][add_node6]

3. 幾分鐘之後，Azure 節點就會完成佈建並處於 [離線]  狀態。 在此狀態下，角色執行個體已在執行，但還沒準備要接受叢集作業。

4. 若要確認角色執行個體已在執行，請在 Azure 入口網站中按一下 [雲端服務 (傳統)] > 您的雲端服務名稱。
   
   您應該看到兩個 **HpcWorkerRole** 執行個體 (節點) 已在服務中執行。 HPC Pack 也會自動部署兩個 **HpcProxy** 執行個體 (中型大小) 以處理前端節點與 Azure 之間的通訊。

   ![Running Instances][view_instances1]

5. 若要讓 Azure 節點上線以執行叢集工作，請選取節點，按一下滑鼠右鍵，然後按一下 [上線] 。
   
    ![Offline Nodes][add_node7]
   
    HPC 叢集管理員會指出節點處於 [線上]  狀態。

## <a name="run-a-command-across-the-cluster"></a>在叢集執行命令
若要追蹤安裝，請使用 HPC Pack **clusrun** 命令在一或多個叢集節點上執行命令或應用程式。 其中一個簡單的範例就是使用 **clusrun** 來取得 Azure 節點的 IP 設定。

1. 在前端節點上，以系統管理員身分開啟命令提示字元。

2. 輸入以下命令：
   
    `clusrun /nodes:azurecn* ipconfig`

3. 出現提示時，輸入您的叢集系統管理員密碼。 您應該會看到如下所示的命令輸出。
   
    ![clusrun][clusrun1]

## <a name="run-a-test-job"></a>執行測試工作
現在提交一個在混合式叢集上執行的測試作業。 這個範例是簡單的參數式掃掠作業 (一種本質平行計算)。 本例會執行使用 **set /a** 命令將整數加入自己本身的子工作。 叢集中的所有節點皆參與完成從 1 到 100 之整數的子工作。

1. 在 HPC 叢集管理員中，按一下 [作業管理] > [New Parametric Sweep Job] \(新增參數式掃掠作業)。
   
    ![New Job][test_job1]

2. 在 [新增參數式掃掠作業] 對話方塊中，於 [命令列] 中輸入 `set /a *+*` (覆寫出現的預設命令列)。 保留其餘設定的預設值，然後按一下 [提交]  提交工作。
   
    ![Parametric Sweep][param_sweep1]

3. 當工作完成時，按兩下 [My Sweep Task]  工作。

4. 按一下 [檢視工作] ，然後按一下子工作以檢視該子工作的計算結果輸出。
   
    ![Task Results][view_job361]

5. 若要查看是哪個節點執行該子工作的計算，請按一下 [已配置的節點]。 (您的叢集可能會顯示不同的節點名稱。)
   
    ![Task Results][view_job362]

## <a name="stop-the-azure-nodes"></a>停止 Azure 節點
試驗完叢集之後，請停止 Azure 節點，以避免給您的帳戶產生不必要的費用。 這個步驟會停止雲端服務並移除 Azure 角色執行個體。

1. 在 HPC 叢集管理員中，於 [節點管理] \(在舊版的 HPC Pack 中稱為 [資源管理]) 中，選取這兩個 Azure 節點。 然後，按一下 [停止]。
   
    ![Stop Nodes][stop_node1]

2. 在 [停止 Windows Azure 節點] 對話方塊中，按一下 [停止]。 
   
3. 節點會轉換至 [正在停止]  狀態。 幾分鐘之後，HPC 叢集管理員就會顯示這些節點為 [未部署] 狀態。
   
    ![Not Deployed Nodes][stop_node4]

4. 若要確認角色執行個體已不再於 Azure 中執行，請在 Azure 入口網站中按一下 [雲端服務 (傳統)] > 您的雲端服務名稱。 不會有任何執行個體部署於生產環境中。 
   
    這樣就完成了教學課程。

## <a name="next-steps"></a>後續步驟
* 瀏覽 [HPC Pack](https://technet.microsoft.com/library/cc514029) 文件。
* 若要設定較大規模的混合式 HPC Pack 叢集部署，請參閱 [Burst to Azure with Microsoft HPC Pack (使用 Microsoft HPC Pack 高載至 Azure 背景工作角色執行個體)](http://go.microsoft.com/fwlink/p/?LinkID=200493)。
* 如需在 Azure 中建立 HPC Pack 叢集的其他方式，包括使用 Azure Resource Manager 範本，請參閱[使用 HPC Pack 在 Azure 中建立及管理 Windows HPC 叢集的選項](../virtual-machines/windows/hpcpack-cluster-options.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)。


[Overview]: ./media/cloud-services-setup-hybrid-hpcpack-cluster/hybrid_cluster_overview.png
[install_hpc1]: ./media/cloud-services-setup-hybrid-hpcpack-cluster/install_hpc1.png
[install_hpc4]: ./media/cloud-services-setup-hybrid-hpcpack-cluster/install_hpc4.png
[install_hpc6]: ./media/cloud-services-setup-hybrid-hpcpack-cluster/install_hpc6.png
[install_hpc7]: ./media/cloud-services-setup-hybrid-hpcpack-cluster/install_hpc7.png
[install_hpc10]: ./media/cloud-services-setup-hybrid-hpcpack-cluster/install_hpc10.png
[config_hpc2]: ./media/cloud-services-setup-hybrid-hpcpack-cluster/config_hpc2.png
[config_hpc3]: ./media/cloud-services-setup-hybrid-hpcpack-cluster/config_hpc3.png
[config_hpc6]: ./media/cloud-services-setup-hybrid-hpcpack-cluster/config_hpc6.png
[config_hpc8]: ./media/cloud-services-setup-hybrid-hpcpack-cluster/config_hpc8.png
[config_hpc10]: ./media/cloud-services-setup-hybrid-hpcpack-cluster/config_hpc10.png
[config_hpc12]: ./media/cloud-services-setup-hybrid-hpcpack-cluster/config_hpc12.png
[config_hpc13]: ./media/cloud-services-setup-hybrid-hpcpack-cluster/config_hpc13.png
[add_node1]: ./media/cloud-services-setup-hybrid-hpcpack-cluster/add_node1.png
[add_node1_1]: ./media/cloud-services-setup-hybrid-hpcpack-cluster/add_node1_1.png
[add_node2]: ./media/cloud-services-setup-hybrid-hpcpack-cluster/add_node2.png
[add_node3]: ./media/cloud-services-setup-hybrid-hpcpack-cluster/add_node3.png
[add_node4]: ./media/cloud-services-setup-hybrid-hpcpack-cluster/add_node4.png
[add_node6]: ./media/cloud-services-setup-hybrid-hpcpack-cluster/add_node6.png
[add_node7]: ./media/cloud-services-setup-hybrid-hpcpack-cluster/add_node7.png
[view_instances1]: ./media/cloud-services-setup-hybrid-hpcpack-cluster/view_instances1.png
[clusrun1]: ./media/cloud-services-setup-hybrid-hpcpack-cluster/clusrun1.png
[test_job1]: ./media/cloud-services-setup-hybrid-hpcpack-cluster/test_job1.png
[param_sweep1]: ./media/cloud-services-setup-hybrid-hpcpack-cluster/param_sweep1.png
[view_job361]: ./media/cloud-services-setup-hybrid-hpcpack-cluster/view_job361.png
[view_job362]: ./media/cloud-services-setup-hybrid-hpcpack-cluster/view_job362.png
[stop_node1]: ./media/cloud-services-setup-hybrid-hpcpack-cluster/stop_node1.png
[stop_node4]: ./media/cloud-services-setup-hybrid-hpcpack-cluster/stop_node4.png
[view_instances2]: ./media/cloud-services-setup-hybrid-hpcpack-cluster/view_instances2.png
