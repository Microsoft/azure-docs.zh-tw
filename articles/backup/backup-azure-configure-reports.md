---
title: 針對 Azure 備份設定報告
description: 使用復原服務保存庫針對 Azure 備份設定 Power BI 報告。
services: backup
author: adiganmsft
manager: shivamg
ms.service: backup
ms.topic: conceptual
ms.date: 07/26/2018
ms.author: adigan
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: c59653bf3709f7798fd92a44fa420b99f2cbc6b6
ms.sourcegitcommit: 1b561b77aa080416b094b6f41fce5b6a4721e7d5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/17/2018
ms.locfileid: "45733548"
---
# <a name="configure-azure-backup-reports"></a>設定 Azure 備份報告
本文說明使用復原服務保存庫針對 Azure 備份設定報告，以及使用 Power BI 存取這些報告的步驟。 執行這些步驟後，您可以直接移至 Power BI 以檢閱所有報告，以及自訂和建立報告。 

## <a name="supported-scenarios"></a>支援的案例
1. Azure 備份報告支援使用 Azure 復原服務代理程式進行 Azure 虛擬機器備份，以及將檔案/資料夾備份到雲端。
2. 目前不支援針對 Azure SQL、DPM 和 Azure 備份伺服器的報告功能。
3. 如果針對每個保存庫皆設定相同的儲存體帳戶，則可以檢閱跨保存庫和跨訂用帳戶的報告。 選取的儲存體帳戶應與復原服務保存庫位於相同的區域。
4. Power BI 中報告的排程重新整理頻率為 24 小時。 您也可以在 Power BI 中執行報告的臨機操作重新整理，這會使用客戶儲存體帳戶中的最新資料來轉譯報告。 

## <a name="prerequisites"></a>必要條件
1. 建立 [Azure 儲存體帳戶](../storage/common/storage-quickstart-create-account.md)以針對報告進行設定。 這個儲存體帳戶是用來儲存與報告相關的資料。
2. [建立 Power BI 帳戶](https://powerbi.microsoft.com/landing/signin/)，以使用 Power BI 入口網站檢視、自訂和建立您自己的報告。
3. 如果尚未註冊，請註冊資源提供者 **Microsoft.insights**，並搭配儲存體帳戶及復原服務保存庫的訂用帳戶，來使報告資料能流入儲存體帳戶。 若要這麼做，您必須前往 Azure 入口網站 > [訂用帳戶] > [資源提供者]，並查看此提供者以註冊它。 

## <a name="configure-storage-account-for-reports"></a>針對報告設定儲存體帳戶
使用下列步驟，以使用 Azure 入口網站針對復原服務保存庫設定儲存體帳戶。 這是一次性設定，一旦設定好儲存體帳戶之後，您可以直接移至 Power BI 檢視內容套件並運用報告。
1. 如果您已開啟復原服務保存庫，請繼續下一個步驟。 如果您並未開啟復原服務保存庫，但位於 Azure 入口網站中，請按一下 [所有服務]。

   * 在資源清單中輸入 **復原服務**。
   * 當您開始輸入時，清單會根據您輸入的文字進行篩選。 當您看到 [復原服務保存庫] 時，請按一下它。

      ![建立復原服務保存庫的步驟 1](./media/backup-azure-vms-encryption/browse-to-rs-vaults.png) <br/>

     隨即會出現 [復原服務保存庫] 清單。 在 [復原服務保存庫] 清單中選取保存庫。

     選取的保存庫儀表板隨即開啟。
2. 從保存庫下的項目清單中，按一下 [監視和報告] 區段下的 [備份報告]，以針對報告設定儲存體帳戶。

      ![選取備份報告功能表項目步驟 2](./media/backup-azure-configure-reports/backup-reports-settings.PNG)
3. 在 [備份報告] 刀鋒視窗中，按一下 [診斷設定] 連結。 此選項可開啟診斷設定 UI，其可用來將資料推送到客戶儲存體帳戶。

      ![啟用診斷步驟 3](./media/backup-azure-configure-reports/backup-azure-configure-reports.png)
4. 按一下 [開啟診斷] 連結。 這會開啟 UI 以設定儲存體帳戶。 

      ![開啟診斷步驟 4](./media/backup-azure-configure-reports/enable-diagnostics.png)
5. 在 [名稱] 欄位中輸入設定名稱，並選取 [封存至儲存體帳戶] 核取方塊，讓報告資料可以開始流入儲存體帳戶。

      ![啟用診斷步驟 5](./media/backup-azure-configure-reports/select-setting-name.png)
6. 按一下 [儲存體帳戶] 選擇器，並從儲存報告資料的清單中選取相關訂用帳戶和儲存體帳戶，然後按一下 [確定]。

      ![選取儲存體帳戶步驟 6](./media/backup-azure-configure-reports/select-subscription-sa.png)
7. 在 [記錄] 區段下選取 [AzureBackupReport] 核取方塊，並移動滑桿以選取此報告資料的保留期限。 儲存體帳戶中的報告資料會在使用此滑桿選取的期限內保留。

      ![儲存儲存體帳戶步驟 7](./media/backup-azure-configure-reports/save-diagnostic-settings.png)
8. 檢閱所有變更，然後按一下頂端的 [儲存] 按鈕，如上圖所示。 此動作可確保所有變更都會儲存，且儲存體帳戶已針對儲存報告資料進行設定。

9. 診斷設定資料表目前應該顯示保存庫中啟用的新設定。 如果未顯示，請重新整理資料表，即可看見更新的設定。

      ![檢視診斷設定步驟 9](./media/backup-azure-configure-reports/diagnostic-setting-row.png)

> [!NOTE]
> 一旦您將儲存體帳戶儲存從而設定報告後，應該**等待 24 小時**，初始資料推送才會完成。 只有在該時間之後，您才需要將 Power BI 中的 Azure 備份內容套件匯入。 如需進一步的詳細資料，請參閱[常見問題集一節](#frequently-asked-questions)。 
>
>

## <a name="view-reports-in-power-bi"></a>在 Power BI 中檢視報告 
使用復原服務保存庫針對報告設定儲存體帳戶之後，報告資料大約需要 24 小時的時間才會開始流入。 在設定儲存體帳戶的 24 小時後，請使用下列步驟以在 Power BI 中檢視報告：
1. [登入](https://powerbi.microsoft.com/landing/signin/) Power BI。
2. 按一下 [取得資料]，然後在內容套件庫中的 [服務] 底下，按一下 [取得]。 使用[針對存取內容套件的 Power BI 文件](https://powerbi.microsoft.com/documentation/powerbi-content-packs-services/)中所述的步驟。

     ![匯入內容套件](./media/backup-azure-configure-reports/content-pack-import.png)
3. 在搜尋列中輸入 **Azure Backup**，並按一下 [立即取得]。

      ![取得內容套件](./media/backup-azure-configure-reports/content-pack-get.png)
4. 輸入上述步驟 5 中所設定的儲存體帳戶名稱，然後按一下 [下一步] 按鈕。

    ![輸入儲存體帳戶名稱](./media/backup-azure-configure-reports/content-pack-storage-account-name.png)    
5. 輸入此儲存體帳戶的儲存體帳戶金鑰。 您可以透過瀏覽至 Azure 入口網站中的儲存體帳戶，來[檢視並複製儲存體存取金鑰](../storage/common/storage-account-manage.md#access-keys)。 

     ![輸入儲存體帳戶](./media/backup-azure-configure-reports/content-pack-storage-account-key.png) <br/>
     
6. 按一下 [登入] 按鈕。 登入成功後，您會收到「匯入資料」的通知。

    ![匯入內容套件](./media/backup-azure-configure-reports/content-pack-importing-data.png) <br/>
    
    當匯入於一段時間後完成時，您會收到「成功」的通知。 如果儲存體帳戶中有大量資料，匯入內容套件可能需要較長時間。
    
    ![成功匯入內容套件](./media/backup-azure-configure-reports/content-pack-import-success.png) <br/>
    
7. 一旦成功匯入資料，[Azure 備份] 內容套件就可以在 [瀏覽] 窗格的 [應用程式] 中看到。 現在該清單會顯示以黃色星號表示為新匯入報告的 Azure 備份儀表板、報告和資料集。 

     ![Azure 備份內容套件](./media/backup-azure-configure-reports/content-pack-azure-backup.png) <br/>
     
8. 按一下 [儀表板] 底下的 [Azure 備份]，這會顯示一組已釘選的重要資訊報告。

      ![Azure 備份儀表板](./media/backup-azure-configure-reports/azure-backup-dashboard.png) <br/>
9. 若要檢視完整的報告組合，請按一下儀表板中的任何報告。

      ![Azure 備份作業健康情況](./media/backup-azure-configure-reports/azure-backup-job-health.png) <br/>
10. 按一下報告中的每個索引標籤，以檢視該區域中的報告。

      ![Azure 備份報告索引標籤](./media/backup-azure-configure-reports/reports-tab-view.png)


## <a name="frequently-asked-questions"></a>常見問題集
1. **我該如何確認報告資料是否已開始流入儲存體帳戶？**
    
    您可以前往已設定的儲存體帳戶，然後選取容器。 如果容器有 insights-logs-azurebackupreport 的項目，則表示報告資料已經開始流入。

2. **將資料推送到儲存體帳戶和 Power BI 中 Azure 備份內容套件的頻率為何？**

   針對使用時間尚未達一天的使用者，將資料推送到儲存體帳戶需要大約 24 小時。 完成此初始推送後，資料會以下圖所示的頻率重新整理。 
      * 與「作業、警示、備份項目、保存庫、受保護的伺服器，以及原則」相關的資料，將會在記錄資料的同時推送到客戶儲存體帳戶。
      * 與「儲存體」相關的資料，將會每 24 小時推送到客戶儲存體帳戶。
   
    ![Azure 備份報表資料推送頻率](./media/backup-azure-configure-reports/reports-data-refresh-cycle.png)

  Power BI 具有[一天一次的排程重新整理](https://powerbi.microsoft.com/documentation/powerbi-refresh-data/#what-can-be-refreshed)。 您可以手動重新整理 Power BI 中內容套件的資料。

3. **報告可以保留多久？** 

   在設定儲存體帳戶時，您可以選取儲存體帳戶中報告資料的保留期限 (使用上述＜針對報告設定儲存體帳戶＞一節中的步驟 6)。 除此之外，您可以根據需求[在 Excel 中分析報告](https://powerbi.microsoft.com/documentation/powerbi-service-analyze-in-excel/)並儲存它們，以延長保留期間。 

4. **我在設定儲存體帳戶後，是否能在報告中看到所有的資料？**

   在「設定儲存體帳戶」後所產生的所有資料都會被推送到儲存體帳戶，並會出現在報告中。 不過，「進行中的作業」將不會針對報告進行推送。 當作業完成或失敗之後，便會被傳送至報告。

5. **如果我已經設定儲存體帳戶以檢視報告，我是否可以變更設定以使用其他儲存體帳戶？** 

   是。您可以變更設定以指向不同的儲存體帳戶。 在連線到 Azure 備份內容套件時，您應該使用新設定的儲存體帳戶。 此外，在設定不同的儲存體帳戶之後，新的資料將會流入這個儲存體帳戶。 但是較舊的資料 (變更設定之前的資料) 仍會保留在較舊的儲存體帳戶中。

6. **我是否可以檢視跨保存庫及跨訂用帳戶的報告？** 

   是。您可以在不同的保存庫上設定相同的儲存體帳戶，以檢視跨保存庫的報告。 您也可以針對跨訂用帳戶的保存庫設定相同的儲存體帳戶。 您接著可以在連線至 Power BI 中的 Azure 備份內容套件時，使用這個儲存體帳戶來檢視報告。 不過，選取的儲存體帳戶應與復原服務保存庫位於相同的區域。
   
## <a name="troubleshooting-errors"></a>錯誤疑難排解
| 錯誤詳細資料 | 解決方案 |
| --- | --- |
| 設定備份報告的儲存體帳戶之後，[儲存體帳戶] 依然顯示 [未設定]。 | 如果您已成功設定儲存體帳戶，不論這個問題是否發生，報表資料仍然會流動。 若要解決此問題，請移至 Azure 入口網站 > [所有服務] > [診斷設定] > [RS 保存庫] > [編輯設定]。 刪除先前配置的設定，再從同一個刀鋒視窗建立新設定。 這次將 [名稱] 欄位設定為 [服務]。 這樣應該能顯示設定的儲存體帳戶。 |
|在 Power BI 中匯入 Azure 備份內容套件後，會發生 **404 - 找不到容器**錯誤。 | 如本文件所建議，在復原服務保存庫中設定報表後，您必須等待 24 小時才能在 Power BI 中正確查看。 如果您在 24 小時之前嘗試存取報表，便會發生這個錯誤，因為完整的資料還不存在，無法顯示有效的報表。 |

## <a name="next-steps"></a>後續步驟
現在您已經設定好儲存體帳戶並匯入 Azure 備份內容套件，下一個步驟是自訂這些報告，並使用報告資料模型建立報告。 如需詳細資料，請參閱下列文章。

* [使用 Azure 備份報告資料模型](backup-azure-reports-data-model.md)
* [在 Power BI 中篩選報告](https://powerbi.microsoft.com/documentation/powerbi-service-about-filters-and-highlighting-in-reports/)
* [在 Power BI 中建立報告](https://powerbi.microsoft.com/documentation/powerbi-service-create-a-new-report/)

