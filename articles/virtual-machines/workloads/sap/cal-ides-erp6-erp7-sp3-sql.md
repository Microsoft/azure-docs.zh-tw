---
title: 在 Azure 上部署適用於 SAP ERP 6.0 的 SAP IDES EHP7 SP3 | Microsoft Docs
description: 在 Azure 上部署適用於 SAP ERP 6.0 的 SAP IDES EHP7 SP3
services: virtual-machines-windows
documentationcenter: ''
author: hermanndms
manager: jeconnoc
editor: ''
tags: azure-resource-manager
keywords: ''
ms.assetid: 626c1523-1026-478f-bd8a-22c83b869231
ms.service: virtual-machines-windows
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 09/16/2016
ms.author: hermannd
ms.openlocfilehash: 93824c8f0e7667fcb58fd6b8292cddfa2b4a482a
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39441454"
---
# <a name="deploy-sap-ides-ehp7-sp3-for-sap-erp-60-on-azure"></a>在 Azure 上部署適用於 SAP ERP 6.0 的 SAP IDES EHP7 SP3
本文說明如何透過 SAP Cloud Appliance Library (SAP CAL) 3.0 在 Azure 上部署與 SQL Server 和 Windows 作業系統搭配執行的 SAP IDES 系統。 螢幕擷取畫面會顯示逐步程序。 若要部署不同的解決方案，請遵循相同的步驟。

若要開始使用 SAP CAL，請前往 [SAP Cloud Appliance Library](https://cal.sap.com/) 網站。 SAP 也有關於全新 [SAP Cloud Appliance Library 3.0](http://scn.sap.com/community/cloud-appliance-library/blog/2016/05/27/sap-cloud-appliance-library-30-came-with-a-new-user-experience) 的部落格。 

> [!NOTE]
自 2017 年 5 月 29 日起，除了較不建議使用的傳統部署模型外，您還可以使用 Azure Resource Manager 部署模型來部署 SAP CAL。 我們建議您使用新的 Resource Manager 部署模型，而不要使用傳統部署模型。

如果您已建立使用傳統模型的 SAP CAL 帳戶，則必須建立其他 SAP CAL 帳戶。 您必須使用 Resource Manager 模型，以獨佔方式將此帳戶部署至 Azure。

在登入 SAP CAL 之後，第一個頁面通常會帶您進入 [解決方案] 頁面。 SAP CAL 上所提供的解決方案會不斷增加，所以您可能必須捲動很久才能找到您要的解決方案。 反白顯示的 Azure 專用 Windows 型 SAP IDE 解決方案會示範部署程序：

![SAP CAL 解決方案](./media/cal-ides-erp6-ehp7-sp3-sql/ides-pic1.jpg)

### <a name="create-an-account-in-the-sap-cal"></a>在 SAP CAL 中建立帳戶
1. 若要第一次進行 SAP CAL 登入，請使用您的 SAP S-User 或其他已向 SAP 註冊的使用者。 然後定義 SAP CAL 帳戶，以供 SAP CAL 用來在 Azure 上部署應用裝置。 在帳戶定義中，您需要：

    a. 選取 Azure 上的部署模型 (Resource Manager 或傳統)。

    b.這是另一個 C# 主控台應用程式。 輸入您的 Azure 訂用帳戶。 一個 SAP CAL 帳戶只能指派給一個訂用帳戶。 如果您需要多個訂用帳戶，則必須另外建立 SAP CAL 帳戶。
    
    c. 賦予 SAP CAL 部署到 Azure 訂用帳戶的權限。

    > [!NOTE]
    接下來的步驟會示範如何針對 Resource Manager 部署建立 SAP CAL 帳戶。 如果您已有連結至傳統部署模型的 SAP CAL 帳戶，則您「必須」遵循下列步驟來建立新的 SAP CAL 帳戶。 新的 SAP CAL 帳戶必須以 Resource Manager 模式部署。

1. 若要建立新的 SAP CAL 帳戶，[帳戶] 頁面會顯示兩個適用於 Azure 的選項： 

    a. **Microsoft Azure (傳統)** 是傳統部署模型，目前不再建議使用。

    b. **Microsoft Azure** 是新的 Resource Manager 部署模型。

    ![SAP CAL 帳戶](./media/cal-ides-erp6-ehp7-sp3-sql/s4h-pic-2a.PNG)

    若要在 Resource Manager 模型中部署，請選取 [Microsoft Azure]。

    ![SAP CAL 帳戶](./media/cal-ides-erp6-ehp7-sp3-sql/s4h-pic3c.PNG)

1. 輸入可在 Azure 入口網站上找到的 Azure **訂用帳戶識別碼**。 

    ![SAP CAL 訂用帳戶識別碼](./media/cal-ides-erp6-ehp7-sp3-sql/s4h-pic3c.PNG)

1. 若要授權 SAP CAL 部署到您所定義的 Azure 訂用帳戶，請按一下 [授權]。 瀏覽器索引標籤中會出現下列頁面：

    ![Internet Explorer 雲端服務登入](./media/cal-ides-erp6-ehp7-sp3-sql/s4h-pic4c.PNG)

1. 如果頁面中列出多個使用者，請選擇連結至成為您所選 Azure 訂用帳戶共同管理員的 Microsoft 帳戶。 瀏覽器索引標籤中會出現下列頁面：

    ![Internet Explorer 雲端服務確認](./media/cal-ides-erp6-ehp7-sp3-sql/s4h-pic5a.PNG)

1. 按一下 [接受]。 如果授權成功，SAP CAL 帳戶定義便會再次顯示。 片刻之後便會出現訊息來確認授權程序已成功。

1. 若要將新建立的 SAP CAL 帳戶指派給您的使用者，請在右邊的文字方塊中輸入您的 [使用者識別碼]，然後按一下 [新增]。 

    ![帳戶與使用者的關聯](./media/cal-ides-erp6-ehp7-sp3-sql/s4h-pic8a.PNG)

1. 若要讓您的帳戶與您用來登入 SAP CAL 的使用者產生關聯，請按一下 [檢閱]。 

1. 若要在使用者與新建立的 SAP CAL 帳戶之間建立關聯，請按一下 [建立]。

    ![使用者與帳戶的關聯](./media/cal-ides-erp6-ehp7-sp3-sql/s4h-pic9b.PNG)

您已成功建立可執行下列作業的 SAP CAL 帳戶：

- 使用 Resource Manager 部署模型。
- 將 SAP 系統部署到您的 Azure 訂用帳戶。

> [!NOTE]
您可能需要先註冊 SAP CAL 訂用帳戶，才可以部署以 Windows 和 SQL Server 為基礎的 SAP IDES 解決方案。 否則在概觀頁面上，解決方案可能會顯示為 [已鎖定]。

### <a name="deploy-a-solution"></a>部署解決方案
1. 在設定 SAP CAL 帳戶之後，請選取 [Windows 和 SQL Server 上的 SAP IDES 解決方案] 解決方案。 按一下 [建立執行個體]，並確認使用條款和條件。 

1. 在 [基本模式: 建立執行個體] 頁面上，您必須：

    a. 輸入執行個體**名稱**。

    b. 選取 Azure **區域**。 您可能需要 SAP CAL 訂用帳戶才能取得所提供的多個 Azure 區域。

    c.  輸入解決方案的主要**密碼**，如下所示：

    ![SAP CAL 基本模式：建立執行個體](./media/cal-ides-erp6-ehp7-sp3-sql/ides-pic10a.png)

1. 按一下頁面底部的 [新增] 。 在一段時間之後，視解決方案的大小和複雜性 (SAP CAL 會提供預估) 而定，狀態將會顯示為作用中並可供使用： 

    ![SAP CAL 執行個體](./media/cal-ides-erp6-ehp7-sp3-sql/ides-pic12a.png)

1. 若要尋找 SAP CAL 所建立的資源群組及其所有物件，請移至 Azure 入口網站。 您會發現以 SAP CAL 中所提供的相同執行個體名稱開頭的虛擬機器。

    ![資源群組物件](./media/cal-ides-erp6-ehp7-sp3-sql/ides_resource_group.PNG)

1. 在 SAP CAL 入口網站中，移至已部署的執行個體，然後按一下 [連線]。 下列快顯視窗隨即出現： 

    ![連線到執行個體](./media/cal-ides-erp6-ehp7-sp3-sql/ides-pic14a.PNG)

1. 請先按一下 [開始使用指南]，之後您才能使用其中一個選項來連線至已部署的系統。 文件會指出每個連線方法的使用者。 這些使用者的密碼會設定為您在部署程序開始時所定義的主要密碼。 文件中會另外列出多名功能使用者及其密碼，以供您用來登入已部署的系統。

    ![SAP 歡迎使用文件](./media/cal-ides-erp6-ehp7-sp3-sql/ides-pic15.jpg)

在幾個小時內，狀況良好的 SAP IDES 系統便會部署在 Azure 中。

如果您已購買 SAP CAL 訂用帳戶，SAP 便可完全支援透過 SAP CAL 在 Azure 上進行部署。 支援佇列是 BC-VCM-CAL。

