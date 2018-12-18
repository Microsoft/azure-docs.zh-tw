<!--author=alkohli last changed:02/10/2017-->

---
title: 包含檔案
description: 包含檔案
services: storage
author: alkohli
ms.service: storage
ms.topic: include
ms.date: 08/20/2018
ms.author: alkohli
ms.custom: include file
---

#### <a name="to-create-a-new-service"></a>建立新服務

1. 使用您的 Microsoft 帳戶認證以登入 [Azure 入口網站](https://portal.azure.com/)。

2. 在 Azure 入口網站中，按一下 [建立資源]，然後在 Marketplace 中按一下 [查看全部]。

    ![建立 StorSimple 裝置管理員](./media/storsimple-8000-create-new-service/createssdevman1.png)

    搜尋 [StorSimple 實體]。 選取並按一下 [StorSimple 實體裝置系列]，然後按一下 [建立]。 或者，在 Azure 入口網站中按一下 [+]，然後在 [儲存體]按一下 [StorSimple 實體裝置系列]。

    ![建立 StorSimple 裝置管理員](./media/storsimple-8000-create-new-service/createssdevman11.png)

3. 在 [StorSimple 裝置管理員] 刀鋒視窗中，執行下列步驟：
   
   1. 為服務提供唯一的**資源名稱** 。 這個名聲是可用來識別服務的易記名稱。 名稱長度可介於 2 到 50 個字元之間，且可以是字母、數字和連字號。 名稱必須以字母或數字為開頭或結尾。

   2. 從下拉式清單選擇 [訂用帳戶]  。 訂用帳戶會連結到您的帳單帳戶。 如果您只擁有一個訂用帳戶，則此欄位不存在。

   3. 針對**資源群組**，**使用現有群組**或**建立新的群組**。 如需詳細資訊，請參閱 [Azure 資源群組](https://azure.microsoft.com/documentation/articles/virtual-machines-windows-infrastructure-resource-groups-guidelines/)。
   
   4. 提供服務的 [位置]  。 一般而言，請選擇最接近您想要部署裝置之地理區域的位置。 您可能也需要計入下列考量因素： 
      
      * 如果您在 Azure 中有工作負載，且想要與您的 StorSimple 裝置部署，請使用該資料中心。
      * 您的 StorSimple 裝置管理員服務和 Azure 儲存體可以位於兩個不同的位置。 如此一來，您需要分別建立 StorSimple 裝置管理員和 Azure 儲存體帳戶。 若要建立 Azure 儲存體帳戶，請前往 Azure 入口網站中的 [Azure 儲存體服務]，依[建立 Azure 儲存體帳戶](../articles/storage/common/storage-quickstart-create-account.md)中的步驟進行。 建立此帳戶之後，請依[設定服務的新儲存體帳戶](../articles/storsimple/storsimple-8000-deployment-walkthrough-u2.md#configure-a-new-storage-account-for-the-service)中的步驟，將此帳戶新增到 StorSimple 裝置管理員服務。

   5. 選取 [建立新的儲存體帳戶]  ，自動建立具有此服務的儲存體帳戶。 指定此儲存體帳戶的名稱。 如果您需要將資料放在不同的位置，取消核取此方塊。

   6. 如果您想在儀表板上快速連結到此服務，請勾選 [釘選到儀表板]。
      
   7. 按一下 [建立] 以建立 StorSimple 裝置管理員。

       ![建立 StorSimple 裝置管理員](./media/storsimple-8000-create-new-service/createssdevman2.png)
   
服務建立需要幾分鐘的時間。 成功建立服務之後，您會看到通知，並隨即開啟新的服務刀鋒視窗。
   
![建立 StorSimple 裝置管理員](./media/storsimple-8000-create-new-service/createssdevman5.png)


