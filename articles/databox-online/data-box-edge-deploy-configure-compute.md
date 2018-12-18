---
title: 使用 Azure Data Box Edge 轉換資料 | Microsoft Docs
description: 了解如何設定 Data Box Edge 上的計算角色，並使用它在資料傳送至 Azure 之前先轉換資料。
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: tutorial
ms.date: 10/08/2018
ms.author: alkohli
Customer intent: As an IT admin, I need to understand how to configure compute on Data Box Edge so I can use it to transform the data before sending it to Azure.
ms.openlocfilehash: 4729e08399132243543c6f4e1cadd537d185e9e3
ms.sourcegitcommit: c282021dbc3815aac9f46b6b89c7131659461e49
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/12/2018
ms.locfileid: "49166248"
---
# <a name="tutorial-transform-data-with-azure-data-box-edge-preview"></a>教學課程：使用 Azure Data Box Edge 轉換資料 (預覽)

本教學課程說明如何設定 Data Box Edge 上的計算角色。 計算角色設定後，Data Box Edge 即可在資料傳送至 Azure 之前先轉換資料。

此程序可能需要約 30-45 分鐘才能完成。 

在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 建立 IoT 中樞資源
> * 設定計算角色
> * 新增計算模組
> * 確認資料轉換和傳輸

> [!IMPORTANT]
> Data Box Edge 目前處於預覽狀態。 部署訂購並部署此解決方案之前，請檢閱 [Azure 預覽版使用條款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。 
 
## <a name="prerequisites"></a>必要條件

在 Data Box Edge 上設定計算節點之前，請確定：

* 您的 Data Box Edge 裝置已依照[連線並啟用您的 Azure Data Box Edge](data-box-edge-deploy-connect-setup-activate.md) 中的詳細說明啟動。


## <a name="create-an-iot-hub-resource"></a>建立 IoT 中樞資源

在 Data Box Edge 上設定計算節點之前，您必須先建立 IoT 中樞資源。

如需詳細指示，請移至[建立 IoT 中樞](https://docs.microsoft.com/azure/iot-hub/iot-hub-create-through-portal#create-an-iot-hub)。 請使用您用於 Data Box Edge 資源的相同訂用帳戶和資源群組。

![建立 IoT 中樞資源](./media/data-box-edge-deploy-configure-compute/create-iothub-resource-1.png)

當 Edge 計算角色未設定時，請注意 
- IoT 中樞資源沒有任何 IoT 裝置或 IoT Edge 裝置。
- 您無法建立 Edge 本機共用。 當您新增共用時，為 Edge 計算建立本機共用的選項不會啟用。


## <a name="set-up-compute-role"></a>設定計算角色

已在 Edge 裝置上設定 Edge 計算角色時，該角色會建立兩個裝置：一個是 IoT 裝置，另一個是 IoT Edge 裝置。 這兩個裝置都可以在 IoT 中樞資源中檢視。

若要在裝置上設定計算角色，請執行下列步驟。

1. 移至 Data Box Edge 資源，然後移至 [概觀]，再按一下 [設定計算角色]。 

    ![設定計算角色](./media/data-box-edge-deploy-configure-compute/setup-compute-1.png)
   
    您也可以移至 [模組]，然後按一下 [設定計算]。

    ![設定計算角色](./media/data-box-edge-deploy-configure-compute/setup-compute-2.png)
 
2. 從下拉式清單中，選取您在上一個步驟中建立的 [IoT 中樞資源]。 目前只有 Linux 平台適用於 IoT Edge 裝置。 按一下頁面底部的 [新增] 。

    ![設定計算角色](./media/data-box-edge-deploy-configure-compute/setup-compute-3.png)
 
3. 建立計算角色需要幾分鐘的時間。 由於此版本中的 Bug，即使在計算角色建立後，畫面仍不會重新整理。 移至 [模組]，您就會看到 Edge 計算已設定完成。  

    ![設定計算角色](./media/data-box-edge-deploy-configure-compute/setup-compute-4.png)

4. 再次移至 [概觀]，此時畫面已更新而指出計算角色已設定完成。

    ![設定計算角色](./media/data-box-edge-deploy-configure-compute/setup-compute-5.png)
 
5. 移至您建立 Edge 計算角色時所使用的 IoT 中樞。 移至 [IoT 裝置]。 您會看到 IoT 裝置現在已啟用。 

    ![設定計算角色](./media/data-box-edge-deploy-configure-compute/setup-compute-6.png)

6. 移至 [IoT Edge]，您會看到 IoT Edge 裝置也已啟用。

    ![設定計算角色](./media/data-box-edge-deploy-configure-compute/setup-compute-7.png)
 
7. 選取並按一下 IoT Edge 裝置。 Edge 代理程式會在此 IoT Edge 裝置上執行。 

    ![設定計算角色](./media/data-box-edge-deploy-configure-compute/setup-compute-8.png) 

不過，此 Edge 裝置上並沒有自訂模組。 您現在可以將自訂模組新增至此裝置。


## <a name="add-a-custom-module"></a>新增自訂模組

在本節中，您會將自訂模組新增至 IoT Edge 裝置。 

在此程序使用的範例中，使用的自訂模組會從 Edge 裝置上的本機共用中取用檔案，並將其移至裝置上的雲端共用。 接著，雲端共用會將檔案推送至與雲端共用相關聯的 Azure 儲存體帳戶。 

1. 第一個步驟是在 Edge 裝置上新增本機共用。 在您的 Data Box Edge資源中，移至 [共用]。 按一下 [+ 新增共用]。 提供共用名稱，然後選取共用類型。 若要建立本機共用，請核取 [設定為 Edge 本機共用] 選項。 選取**現有的使用者**或**建立新的**使用者。 按一下頁面底部的 [新增] 。

    ![新增自訂模組](./media/data-box-edge-deploy-configure-compute/add-a-custom-module-1.png) 

    如果您已建立本機 NFS 共用，請使用下列 rsync 命令選項將檔案複製到共用上：

    `rsync --inplace <source file path> < destination file path>`

     如需 rsync 命令的詳細資訊，請移至 [Rsync 文件](https://www.computerhope.com/unix/rsync.htm)。 

 
2. 當本機共用已建立，且您已收到成功建立通知後 (共用清單可能會先更新，但您必須等候共用建立完成)，請移至共用清單。 

    ![新增自訂模組](./media/data-box-edge-deploy-configure-compute/add-a-custom-module-2.png) 
 
3. 選取並按一下新建立的本機共用，檢視共用的屬性。 複製並儲存對應於此共用的 **Edge 模組的本機掛接點**。

    ![新增自訂模組](./media/data-box-edge-deploy-configure-compute/add-a-custom-module-3.png) 
 
    移至您 Data Box Edge 裝置上建立的現有雲端共用。 再次為此雲端共用複製並儲存 Edge 計算模組的本機掛接點。 在部署模組時將會使用這些本機掛接點。

    ![新增自訂模組](./media/data-box-edge-deploy-configure-compute/add-a-custom-module-4.png)  

4. 若要將自訂模組新增至 IoT Edge 裝置，請移至您的 IoT 中樞資源，然後移至 [IoT Edge 裝置]。 選取並按一下裝置。 在 [裝置詳細資料] 中，從頂端的命令列中按一下 [設定模組]。 

    ![新增自訂模組](./media/data-box-edge-deploy-configure-compute/add-a-custom-module-5.png) 

5. 在 [新增模組] 下，執行下列步驟：

    1. 為自訂模組的 [容器登錄設定] 提供 [名稱]、[位址]、[使用者名稱] 和 [密碼]。 名稱、位址和列出的認證會用來擷取具有相符 URL 的模組。 若要部署此模組，請在 [部署模組] 下方選取 [IoT Edge 模組]。 此 IoT Edge 模組是一個 Docker 容器，且可供您部署至與 Data Box Edge 裝置相關聯的 IoT Edge 裝置。

        ![新增自訂模組](./media/data-box-edge-deploy-configure-compute/add-a-custom-module-6.png) 
 
    2. 指定 IoT Edge 自訂模組的設定。 提供模組的**名稱**和**映像 URI**。 
    
        ![新增自訂模組](./media/data-box-edge-deploy-configure-compute/add-a-custom-module-7.png) 

    3. 在 [容器建立選項] 中，為您在先前的步驟中針對雲端和本機共用而複製的 Edge 模組提供本機掛接點 (請務必使用這些路徑，而不是建立新的)。 這些共用會對應至相對應的容器掛接點。 此外也請在此處提供模組的任何環境變數。

        ![新增自訂模組](./media/data-box-edge-deploy-configure-compute/add-a-custom-module-8.png) 
 
    4. 如有需要，請**設定進階 Edge 執行階段設定**，然後按 [下一步]。

        ![新增自訂模組](./media/data-box-edge-deploy-configure-compute/add-a-custom-module-9.png) 
 
6.  在 [指定路由] 下方，設定模組之間的路由。 在此案例中，請提供會將資料推送至雲端共用的本機共用名稱。 按 [下一步] 。

    ![新增自訂模組](./media/data-box-edge-deploy-configure-compute/add-a-custom-module-10.png) 
 
7.  在 [檢閱部署] 下方檢閱所有設定，若沒問題，則**提交**模組以進行部署。

    ![新增自訂模組](./media/data-box-edge-deploy-configure-compute/add-a-custom-module-11.png) 
 
這會啟動模組部署，如 [模組] 下方的 [IoT Edge 自訂模組] 所示。

![新增自訂模組](./media/data-box-edge-deploy-configure-compute/add-a-custom-module-12.png) 

### <a name="verify-data-transform-and-transfer"></a>確認資料轉換和傳輸

最後一個步驟是確定模組已如預期連線並執行。 請執行下列步驟以確認模組正在執行中。

1. 在 IoT 中樞資源中，模組應處於為您的 IoT Edge 裝置執行中的執行階段狀態。

    ![確認資料轉換](./media/data-box-edge-deploy-configure-compute/verify-data-transform-1.png) 
 
2. 選取並按一下模組，然後查看 [模組身分識別對應項]。 Edge 裝置和模組的用戶端狀態應顯示為 [已連線]。

    ![確認資料轉換](./media/data-box-edge-deploy-configure-compute/verify-data-transform-2.png) 
 
3.  模組開始執行後，也會顯示在您 Data Box Edge 資源中的 Edge 模組清單下。 您新增的模組處於**執行中**的**執行階段狀態**。

    ![確認資料轉換](./media/data-box-edge-deploy-configure-compute/verify-data-transform-3.png) 
 
4.  透過 [檔案總管] 連線到您建立的本機和雲端共用。

    ![確認資料轉換](./media/data-box-edge-deploy-configure-compute/verify-data-transform-4.png) 
 
5.  將資料新增至本機共用。

    ![確認資料轉換](./media/data-box-edge-deploy-configure-compute/verify-data-transform-5.png) 
 
6.  資料會移至雲端共用。

    ![確認資料轉換](./media/data-box-edge-deploy-configure-compute/verify-data-transform-6.png)  

7.  然後，資料會從雲端共用推送至儲存體帳戶。 請移至 [儲存體總管] 以檢視資料。

    ![確認資料轉換](./media/data-box-edge-deploy-configure-compute/verify-data-transform-7.png) 
 
驗證程序到此結束。


## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解 Data Box Edge 的相關主題，像是：

> [!div class="checklist"]
> * 建立 IoT 中樞資源
> * 設定計算角色
> * 新增計算模組
> * 確認資料轉換和傳輸

請繼續進行下一個教學課程，以了解如何管理 Data Box Edge。

> [!div class="nextstepaction"]
> [使用本機 Web UI 來管理 Data Box Edge](http://aka.ms/dbg-docs)


