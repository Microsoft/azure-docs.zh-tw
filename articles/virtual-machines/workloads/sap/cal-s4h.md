---
title: 在 Azure VM 上部署 SAP S/4HANA 或 BW/4HANA | Microsoft Docs
description: 在 Azure VM 上部署 SAP S/4HANA 或 BW/4HANA
services: virtual-machines-linux
documentationcenter: ''
author: hermanndms
manager: jeconnoc
editor: ''
tags: azure-resource-manager
keywords: ''
ms.assetid: 44bbd2b6-a376-4b5c-b824-e76917117fa9
ms.service: virtual-machines-linux
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 09/15/2016
ms.author: hermannd
ms.openlocfilehash: a99fb959ae1ac1434bedffd782a7c4e0a302d361
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39431400"
---
# <a name="deploy-sap-s4hana-or-bw4hana-on-azure"></a>在 Azure 上部署 SAP S/4HANA 或 BW/4HANA
本文說明如何使用 SAP Cloud Appliance Library (SAP CAL) 3.0 在 Azure 上部署 S/4HANA。 若要部署其他 SAP HANA 型解決方案 (例如 BW/4HANA)，請遵循相同的步驟。

> [!NOTE]
如需 SAP CAL 的詳細資訊，請前往 [SAP Cloud Appliance Library](https://cal.sap.com/) 網站。 SAP 也有關於 [SAP Cloud Appliance Library 3.0](http://scn.sap.com/community/cloud-appliance-library/blog/2016/05/27/sap-cloud-appliance-library-30-came-with-a-new-user-experience) 的部落格。

> [!NOTE]
自 2017 年 5 月 29 日起，除了較不建議使用的傳統部署模型外，您還可以使用 Azure Resource Manager 部署模型來部署 SAP CAL。 我們建議您使用新的 Resource Manager 部署模型，而不要使用傳統部署模型。

## <a name="step-by-step-process-to-deploy-the-solution"></a>部署解決方案的逐步程序

以下螢幕擷取畫面的順序會示範如何使用 SAP CAL 在 Azure 上部署 S/4HANA。 對於 BW/4 HANA 等其他解決方案，此程序的運作方式相同。

[解決方案] 頁面會顯示 Azure 上可用的某些 SAP CAL HANA 型解決方案。 **SAP S/4HANA 1610 FPS01, Fully-Activated Appliance** 位於中間的資料列：

![SAP CAL 解決方案](./media/cal-s4h/s4h-pic-1c.png)

### <a name="create-an-account-in-the-sap-cal"></a>在 SAP CAL 中建立帳戶
1. 若要第一次進行 SAP CAL 登入，請使用您的 SAP S-User 或其他已向 SAP 註冊的使用者。 然後定義 SAP CAL 帳戶，以供 SAP CAL 用來在 Azure 上部署應用裝置。 在帳戶定義中，您需要：

    a. 選取 Azure 上的部署模型 (Resource Manager 或傳統)。

    b.這是另一個 C# 主控台應用程式。 輸入您的 Azure 訂用帳戶。 一個 SAP CAL 帳戶只能指派給一個訂用帳戶。 如果您需要多個訂用帳戶，則必須另外建立 SAP CAL 帳戶。

    c. 賦予 SAP CAL 部署到 Azure 訂用帳戶的權限。

    > [!NOTE]
    接下來的步驟會示範如何針對 Resource Manager 部署建立 SAP CAL 帳戶。 如果您已有連結至傳統部署模型的 SAP CAL 帳戶，則您「必須」遵循下列步驟來建立新的 SAP CAL 帳戶。 新的 SAP CAL 帳戶必須以 Resource Manager 模式部署。

1. 建立新的 SAP CAL 帳戶。 [帳戶] 頁面會顯示三個適用於 Azure 的選項： 

    a. **Microsoft Azure (傳統)** 是傳統部署模型，目前不再建議使用。

    b. **Microsoft Azure** 是新的 Resource Manager 部署模型。

    c. **Windows Azure operated by 21Vianet** 是為中國提供的選項，此選項使用傳統部署模型。

    若要在 Resource Manager 模型中部署，請選取 [Microsoft Azure]。

    ![SAP CAL 帳戶詳細資料](./media/cal-s4h/s4h-pic-2a.png)

1. 輸入可在 Azure 入口網站上找到的 Azure **訂用帳戶識別碼**。

   ![SAP CAL 帳戶](./media/cal-s4h/s4h-pic3c.png)

1. 若要授權 SAP CAL 部署到您所定義的 Azure 訂用帳戶，請按一下 [授權]。 瀏覽器索引標籤中會出現下列頁面：

   ![Internet Explorer 雲端服務登入](./media/cal-s4h/s4h-pic4c.png)

1. 如果頁面中列出多個使用者，請選擇連結至成為您所選 Azure 訂用帳戶共同管理員的 Microsoft 帳戶。 瀏覽器索引標籤中會出現下列頁面：

   ![Internet Explorer 雲端服務確認](./media/cal-s4h/s4h-pic5a.png)

1. 按一下 [接受]。 如果授權成功，SAP CAL 帳戶定義便會再次顯示。 片刻之後便會出現訊息來確認授權程序已成功。

1. 若要將新建立的 SAP CAL 帳戶指派給您的使用者，請在右邊的文字方塊中輸入您的 [使用者識別碼]，然後按一下 [新增]。

   ![帳戶與使用者的關聯](./media/cal-s4h/s4h-pic8a.png)

1. 若要讓您的帳戶與您用來登入 SAP CAL 的使用者產生關聯，請按一下 [檢閱]。 
 
1. 若要在使用者與新建立的 SAP CAL 帳戶之間建立關聯，請按一下 [建立]。

   ![使用者與 SAP CAL 帳戶的關聯](./media/cal-s4h/s4h-pic9b.png)

您已成功建立可執行下列作業的 SAP CAL 帳戶：

- 使用 Resource Manager 部署模型。
- 將 SAP 系統部署到您的 Azure 訂用帳戶。

現在您可以開始將 S/4HANA 部署到您在 Azure 中的使用者訂用帳戶。

> [!NOTE]
在繼續之前，請先確定您是否有適用於 Azure H 系列 VM 的 Azure vCPU 配額。 目前，SAP CAL 會使用 Azure 的 H 系列 VM 來部署某些 SAP HANA 型解決方案。 您的 Azure 訂用帳戶可能沒有 H 系列的任何 H 系列 vCPU 配額。 如果是這樣，您可能需要連絡 Azure 支援人員以取得至少 16 個 H 系列 vCPU 的配額。

> [!NOTE]
當您在 SAP CAL 中於 Azure 部署解決方案時，您可能會發現您只能選擇一個 Azure 區域。 若要部署到非 SAP CAL 所建議的 Azure 區域，您需要從 SAP 購買 CAL 訂用帳戶。 您也可能需要與 SAP 連絡，請他們讓您的 CAL 帳戶能夠傳遞到非一開始所建議的 Azure 區域。

### <a name="deploy-a-solution"></a>部署解決方案

讓我們從 SAP CAL 的 [解決方案] 頁面部署解決方案。 SAP CAL 有兩個部署順序：

- 基本順序會使用一個頁面來定義要部署的系統
- 進階順序則會有某些 VM 大小可供您選擇 

在此我們會示範基本部署路徑。

1. 在 [帳戶詳細資料] 頁面上，您需要：

    a. 選取 SAP CAL 帳戶  (使用與以 Resource Manager 部署模型來進行部署相關聯的帳戶)。

    b. 輸入執行個體**名稱**。

    c. 選取 Azure **區域**。 SAP CAL 會建議一個區域。 如果您需要其他 Azure 區域，而且您沒有 SAP CAL 訂用帳戶，您必須向 SAP 訂購 CAL 訂用帳戶。

    d. 為解決方案輸入主要**密碼**，此密碼要有 8 或 9 個字元。 此密碼會用於不同元件的系統管理員。

   ![SAP CAL 基本模式：建立執行個體](./media/cal-s4h/s4h-pic10a.png)

1. 按一下 [建立]，並在出現的訊息方塊中，按一下 [確定]。

   ![SAP CAL 支援的 VM 大小](./media/cal-s4h/s4h-pic10b.png)

1. 在 [私密金鑰] 對話方塊中，按一下 [儲存] 以將私密金鑰儲存在 SAP CAL 中。 若要對私密金鑰使用密碼保護，請按一下 [下載]。 

   ![SAP CAL 私密金鑰](./media/cal-s4h/s4h-pic10c.png)

1. 閱讀 SAP CAL 的**警告**訊息，然後按一下 [確定]。

   ![SAP CAL 警告](./media/cal-s4h/s4h-pic10d.png)

    現在會開始部署。 在一段時間之後，視解決方案的大小和複雜性 (SAP CAL 會提供預估) 而定，狀態將會顯示為作用中並可供使用。

1. 若要尋找隨著某個資源群組中的其他相關聯資源一起收集的虛擬機器，請移至 Azure 入口網站： 

   ![部署在新入口網站中的 SAP CAL 物件](./media/cal-s4h/sapcaldeplyment_portalview.png)

1. 在 SAP CAL 入口網站上，狀態會顯示為 [作用中]。 若要連線到此解決方案，請按一下 [連線]。 此解決方案中會部署連線到不同元件的不同選項。

   ![SAP CAL 執行個體](./media/cal-s4h/active_solution.png)

1. 請先按一下 [開始使用指南]，之後您才能使用其中一個選項來連線至已部署的系統。 

   ![連線到執行個體](./media/cal-s4h/connect_to_solution.png)

    文件會指出每個連線方法的使用者。 這些使用者的密碼會設定為您在部署程序開始時所定義的主要密碼。 文件中會另外列出多名功能使用者及其密碼，以供您用來登入已部署的系統。 

    例如，如果您使用 Windows 遠端桌面電腦上預先安裝的 SAP GUI，S/4 系統看起來可能像這樣：

   ![預先安裝之 SAP GUI 中的 SM50](./media/cal-s4h/gui_sm50.png)

    或者，如果您使用 DBACockpit，執行個體看起來可能像這樣：

   ![DBACockpit SAP GUI 中的 SM50](./media/cal-s4h/dbacockpit.png)

在幾個小時內，狀況良好的 SAP S/4 應用裝置便會部署在 Azure 中。

如果您已購買 SAP CAL 訂用帳戶，SAP 便可完全支援透過 SAP CAL 在 Azure 上進行部署。 支援佇列是 BC-VCM-CAL。







