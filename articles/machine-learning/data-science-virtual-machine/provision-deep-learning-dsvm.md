---
title: 在 Azure 上佈建深入學習資料科學虛擬機器 | Microsoft Docs
description: 在 Azure 上設定和建立深入學習資料科學虛擬機器以進行分析和機器學習。
services: machine-learning
documentationcenter: ''
author: gopitk
manager: cgronlun
ms.assetid: e1467c0f-497b-48f7-96a0-7f806a7bec0b
ms.service: machine-learning
ms.component: data-science-vm
ms.workload: data-services
ms.devlang: na
ms.topic: conceptual
ms.date: 03/16/2018
ms.author: gokuma
ms.openlocfilehash: 2172acfd61188c09693b9c7dacf37ff7a638d7f4
ms.sourcegitcommit: e2adef58c03b0a780173df2d988907b5cb809c82
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/28/2018
---
# <a name="provision-a-deep-learning-virtual-machine-on-azure"></a>在 Azure 上佈建深入學習虛擬機器 

深入學習虛擬機器 (DLVM) 是熱門[資料科學虛擬機器](http://aka.ms/dsvm) (DSVM) 的特別設定變體，讓使用 GPU 型 VM 執行個體以快速定型深入訓練模型更加容易。 它支援 Windows 2016 或 Ubuntu DSVM 作為基底。 DLVM 會共用相同的核心 VM 映像，因此也共用可以在 DSVM 取得的所有豐富工具組。 

DLVM 包含數個 AI 的工具，包括 GPU 版本的熱門深入學習架構，像是 Microsoft Cognitive Toolkit、TensorFlow、Keras、Caffe2、Chainer；用來取得和預先處理映像、文字資料的工具，用於資料科學模型化和開發活動的工具，例如 Microsoft R Server Developer Edition、Anaconda Python、Python 和 R 適用的 Jupyter Notebook、Python 和 R 適用的 IDE、SQL 資料庫和其他許多資料科學和 ML 工具。 

## <a name="create-your-deep-learning-virtual-machine"></a>建立深入學習虛擬機器
建立深入學習虛擬機器執行個體的步驟如下： 

1. 瀏覽至 [Azure 入口網站](https://portal.azure.com/#create/microsoft-ads.dsvm-deep-learningtoolkit
)上的虛擬機器清單。
2. 選取底部的 [建立] 按鈕以進入精靈。![create-dlvm](./media/dlvm-provision-wizard.PNG)
3. 針對此圖右邊列舉的**四個步驟**，用來建立 DLVM 的精靈會要求**輸入**每個步驟。 以下是設定每個步驟所需的輸入：
   
   1. **基本概念**
      
      1. **名稱**：您建立的資料科學伺服器名稱。
      2. **選取深入學習 VM 的作業系統類型**：選擇 Windows 或 Linux (針對 Windows 2016 和 Ubuntu Linux 基底 DSVM)
      2. **使用者名稱**：系統管理員帳戶登入識別碼。
      3. **密碼**：系統管理員帳戶密碼。
      4. **訂用帳戶**：如果您有多個訂用帳戶，請選取要用來建立機器和開立帳單的訂用帳戶。
      5. **資源群組**：您可以建立新的群組，或使用訂用帳戶中的**空白**現有 Azure 資源群組。
      6. **位置**：選取最適合的資料中心。 它通常是擁有您大部分的資料或者是最接近您的實際位置以取得最快速度的網路存取的資料中心。 
      
> [!NOTE]
> DLVM 支援所有 NC 和 ND 系列 GPU VM 執行個體。 佈建 DLVM 時，您必須在 Azure 中選擇其中一個具有 GPU 的位置。 請參閱[依區域的 Azure 產品頁面](https://azure.microsoft.com/regions/services/)以取得可用位置，並在 [計算] 下方尋找 [NC 系列]、[NCv2 系列]、[NCv3 系列] 或 [ND 系列]。 

   2. **設定**：選取符合功能性需求和成本條件約束的其中一個 NC 系列 (NC、NCv2、NCv3) 或 ND 系列 GPU 虛擬機器大小。 為您的 VM 建立儲存體帳戶。  ![dlvm-settings](./media/dlvm-provision-step-2.PNG)
   
   3. **摘要**：請確認您輸入的所有資訊都正確無誤。
   5. **購買**：按一下 [購買] 以開始佈建。 會提供一個交易條款的連結。 VM 除了計算您在 [大小]  步驟中所選擇的伺服器大小之外，不會收取任何其他費用。 

> [!NOTE]
> 佈建大約 10-20 分鐘。 在 Azure 入口網站中會顯示佈建的狀態。
> 


## <a name="how-to-access-the-deep-learning-virtual-machine"></a>如何存取深入學習虛擬機器

### <a name="windows-edition"></a>Windows 版本
建立 VM 之後，您可以使用您在前面的 **基本** 區段中設定的系統管理員帳戶認證從遠端桌面登入 VM。 

### <a name="linux-edition"></a>Linux 版本

建立 VM 之後，您就可以使用 SSH 登入。 針對文字殼層介面，使用您在步驟 3 的 **基本** 區段中建立的帳戶認證。 在 Windows 用戶端上，您可以下載 SSH 用戶端工具，例如 [Putty](http://www.putty.org)。 如果您偏好圖形化桌面 (X Windows 系統)，您可以在 Putty 上使用 X11 轉寄或安裝 X2Go 用戶端。

> [!NOTE]
> 在測試中，X2Go 用戶端的效能優於 X11 轉寄。 我們建議您使用 X2Go 用戶端作為圖形化桌面介面。
> 
> 

#### <a name="installing-and-configuring-x2go-client"></a>安裝和設定 X2Go 用戶端
Linux DLVM 已經佈建了 X2Go 伺服器，並準備接受用戶端連接。 若要連線到 Linux VM 圖形化桌面，請在用戶端上完成下列程序：

1. 從 [X2Go](http://wiki.x2go.org/doku.php/doc:installation:x2goclient)下載並安裝您用戶端平台適用的 X2Go 用戶端。    
2. 執行 X2Go 用戶端，然後選取 [新增工作階段] 。 會開啟具有多個索引標籤的組態視窗。 輸入下列組態參數︰
   * **[工作階段] 索引標籤**：
     * **主機**︰Linux 資料科學 VM 的主機名稱或 IP 位址。
     * **登入**︰Linux VM 上的使用者名稱。
     * **SSH 連接埠**︰保留預設值 22。
     * **工作階段類型**︰將值變更為 **XFCE**。 Linux DSVM 目前僅支援 XFCE 桌面。
   * **[媒體] 索引標籤**︰您可以關閉聲音支援和用戶端列印，如果不需要使用的話。
   * **共用資料夾**︰如果您想要用戶端機器的目錄掛接在 Linux VM 上，請在此索引標籤上加入要與 VM 分享的目錄。

當您透過 X2Go 用戶端使用 SSH 用戶端或 XFCE 圖形化桌面登入 VM 之後，便可開始使用已安裝並設定於 VM 上的工具。 在 XFCE 上，您可以看到許多工具的應用程式功能表捷徑和桌面圖示。

一旦建立並佈建您的 VM，您已準備好開始使用在上面安裝及設定的工具。 我們為許多工具提供了開始功能表磚和桌面圖示。 
