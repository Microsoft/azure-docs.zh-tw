---
title: Microsoft Azure StorSimple Data Manager UI | Microsoft Docs
description: 說明如何使用 StorSimple 資料管理員服務 UI
services: storsimple
documentationcenter: NA
author: alkohli
manager: jeconnoc
editor: ''
ms.assetid: ''
ms.service: storsimple
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 01/16/2018
ms.author: alkohli
ms.openlocfilehash: d704cf8e6840c6a7b0a637c404d421f9f1497c46
ms.sourcegitcommit: 7edfa9fbed0f9e274209cec6456bf4a689a4c1a6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/17/2018
ms.locfileid: "27862252"
---
# <a name="manage-the-storsimple-data-manager-service-in-azure-portal"></a>在 Azure 入口網站中管理 StorSimple 資料管理員服務

本文說明如何使用 StorSimple 資料管理員 UI 來轉換 StorSimple 8000 系列裝置上的資料。 然後，轉換後的資料就可供其他 Azure 服務取用，例如 Azure 媒體服務、Azure HDInsight、Azure Machine Learning 和 Azure 搜尋服務。


## <a name="use-storsimple-data-transformation"></a>使用 StorSimple 資料轉換

您可以在 StorSimple 資料管理員這個資源內具現化資料轉換。 資料轉換服務可讓您將資料從 StorSimple 格式轉換為 blob 或 Azure 檔案服務中的原生格式。 若要轉換 StorSimple 原生格式資料，您必須指定 StorSimple 8000 系列裝置的詳細資料，以及您想要轉換的資料。

### <a name="create-a-storsimple-data-manager-service"></a>建立 StorSimple Data Manager 服務

執行下列步驟來建立 StorSimple Data Manager 服務。

1. 使用您的 Microsoft 帳戶認證以登入 [Azure 入口網站](https://portal.azure.com/)。

2. 按一下 [+ 建立資源]，然後搜尋 StorSimple 資料管理員。

    ![建立 StorSimple 資料管理員服務 1](./media/storsimple-data-manager-ui/create-service-1.png)

3. 按一下 [StorSimple 資料管理員]，然後按一下 [建立]。
    
    ![建立 StorSimple 資料管理員服務 2](./media/storsimple-data-manager-ui/create-service-3.png)

3. 對於新的服務，請指定下列項目：

    1. 為您的 StorSimple 資料管理員提供唯一的 [服務名稱]。 這是可以用來識別服務的易記名稱。 名稱長度可介於 3 到 24 個字元之間，且可以是字母、數字和連字號。 名稱必須以字母或數字為開頭或結尾。

    2. 從下拉式清單中選擇 [訂用帳戶]。 訂用帳戶會連結到您的帳單帳戶。 如果您只有一個訂用帳戶，此欄位會自動填入該訂用帳戶 (且無法供您選取)。

    3. 選取現有資源群組或建立新的群組。 如需詳細資訊，請參閱 [Azure 資源群組](https://azure.microsoft.com/documentation/articles/virtual-machines-windows-infrastructure-resource-groups-guidelines/)。

    4. 為您的服務指定 [位置]，以供存放您的儲存體帳戶和 StorSimple 資料管理員服務。 StorSimple 裝置管理員服務、資料管理員服務和相關聯的儲存體帳戶應該全都位於支援的區域內。
    
    5. 若要在儀表板上取得此服務的連結，請選取 [釘選到儀表板]。
    
    6. 按一下頁面底部的 [新增] 。

    ![建立 StorSimple 資料管理員服務 3](./media/storsimple-data-manager-ui/create-service-4.png)

服務建立需要幾分鐘的時間。 成功建立服務之後，您將會看到通知，且新的服務隨即顯示出來。

### <a name="create-a-data-transformation-job-definition"></a>建立資料轉換作業定義

在 StorSimple Data Manager 服務中，您需要建立資料轉換作業定義。 針對您想要以原生格式移至儲存體帳戶的 StorSimple 資料，作業定義會指定其詳細資料。 建立作業定義後，您就可以使用不同的執行階段設定重新執行此作業。

執行下列步驟來建立作業定義。

1. 瀏覽至您所建立的服務。 移至 [管理] > [作業定義]。

2. 按一下 [+ 作業定義]。

    ![按一下 [+ 作業定義]](./media/storsimple-data-manager-ui/create-job-definition-1.png)

3. 提供作業定義的名稱。 此名稱可介於 3 到 63 個字元之間。 此名稱可包含大寫和小寫字母、數字與連字號。

4. 指定用來執行作業的位置。 此位置可與服務部署所在的位置不同。

5. 按一下 [來源] 以指定來源資料存放庫。

    ![設定來源資料存放庫](./media/storsimple-data-manager-ui/create-job-definition-2.png)

6. 由於這是新的資料管理員服務，尚未設定任何資料儲存機制。 在 [設定資料來源] 中，指定 StorSimple 8000 系列裝置的詳細資料和感興趣的資料。

   若要新增 StorSimple 資料管理員作為資料存放庫，請在資料存放庫下拉式清單中按一下 [新增]，然後按一下 [新增資料存放庫]。

    ![新增資料存放庫](./media/storsimple-data-manager-ui/create-job-definition-3.png)
  
    1. 選擇 [StorSimple 8000 系列管理員] 作為資料存放庫類型。
    
    2. 為來源資料存放庫輸入易記名稱。
    
    3. 從下拉式清單中，選擇與 StorSimple 裝置管理員服務相關聯的訂用帳戶。
    
    4. 在 [資源] 中提供 StorSimple 裝置管理員的名稱。

    5. 輸入 StorSimple 裝置管理員服務的 [服務資料加密金鑰]。 

    ![設定來源資料存放庫 1](./media/storsimple-data-manager-ui/create-job-definition-4.png)

    完成時按一下 [確定]。 這會儲存您的資料存放庫。 在其他作業定義中重複使用此 StorSimple 裝置管理員，而不需重新輸入這些參數。 按一下 [確定] 後，只要幾秒鐘的時間，新建立的來源資料存放庫就會出現在下拉式清單中。

7. 從 [資料存放庫] 的下拉式清單中，選取您建立的資料存放庫。 

    1. 輸入包含感興趣資料的 StorSimple 8000 系列裝置名稱。

    2. 指定感興趣資料所在之 StorSimple 裝置上的磁碟區名稱。

    3. 在 [篩選] 子區段中，以 \MyRootDirectory\Data 格式輸入感興趣資料所在的根目錄。 系統不支援 \C:\Data 之類的磁碟機代號。 您也可以在這裡新增任何檔案篩選。

    4. 資料轉換服務會處理透過快照推送至 Azure 的資料。 執行此作業時，您可以選擇在每次執行此作業 (以處理最新資料) 時進行備份，或使用雲端中存在的最新備份 (如果您要處理一些封存資料)。

    5. 按一下 [SERVICEPRINCIPAL] 。

    ![設定來源資料存放庫 2](./media/storsimple-data-manager-ui/create-job-definition-8.png)

8. 接下來，必須設定目標資料存放庫。 選擇儲存體帳戶，將檔案放入該帳戶中的 blob。 在下拉式清單中，選取 [新增]，然後選取 [進行設定]。

9. 選取想要新增的目標存放庫類型，以及與存放庫相關聯的其他參數。

    如果您選取 [儲存體帳戶] 類型的目標，則可以指定易記名稱、訂用帳戶 (選擇與服務相同的訂用帳戶或其他訂用帳戶) 和儲存體帳戶。
        ![設定目標資料存放庫 1](./media/storsimple-data-manager-ui/create-job-definition-10.png)

    當作業執行時，系統就會建立儲存體佇列。 此佇列中會填入已轉換的 blob 備妥時的相關訊息。 此佇列的名稱與作業定義的名稱相同。
    
10. 在新增資料存放庫後，請等候幾分鐘的時間。
    
    1. 從 [目標帳戶名稱] 的下拉式清單中，選取您建立的存放庫作為目標。

    2. 選擇 [Blob] 或 [檔案] 儲存體類型。 指定已轉換資料所在的儲存體容器名稱。 按一下 [SERVICEPRINCIPAL] 。

        ![設定目標資料存放庫儲存體帳戶](./media/storsimple-data-manager-ui/create-job-definition-16.png)

11. 您也可以核取此選項，以在執行作業前顯示預估的作業持續時間。 按一下 [確定] 來建立作業定義。 作業定義現已設定完成。 您可以透過 UI，使用不同的執行階段設定來多次使用此作業定義。

    ![完成作業定義](./media/storsimple-data-manager-ui/create-job-definition-13.png)

    新建立的作業定義會新增至此服務的作業定義清單。

### <a name="run-the-job-definition"></a>執行作業定義

每當您需要將資料從 StorSimple 移至您在作業定義中指定的儲存體帳戶時，您就必須執行作業定義。 在執行階段，可以某些參數指定不同的值。 步驟如下：

1. 選取 StorSimple 資料管理員服務，然後移至 [管理] > [作業定義]。 選取並按一下您要執行的作業定義。
     
     ![啟動作業執行 1](./media/storsimple-data-manager-ui/start-job-run1.png)

2. 按一下 [立即執行]。
     
     ![啟動作業執行 2](./media/storsimple-data-manager-ui/start-job-run2.png)

3. 按一下 [回合設定]，修改您可能想針對這次作業執行來變更的任何設定。 按一下 [確定]，然後按一下 [執行] 來啟動您的作業。

    ![啟動作業執行 3](./media/storsimple-data-manager-ui/start-job-run3.png)

4. 若要監視此作業，請移至 StorSimple 資料管理員中的 [作業]。 除了在 [作業] 刀鋒視窗中監視，您也可以聆聽儲存體佇列，每次有檔案從 StorSimple 移至儲存體帳戶時就會加入訊息。

    ![啟動作業執行 4](./media/storsimple-data-manager-ui/start-job-run4.png)


## <a name="next-steps"></a>後續步驟

[使用 .NET SDK 啟動 StorSimple Data Manager 作業](storsimple-data-manager-dotnet-jobs.md)。