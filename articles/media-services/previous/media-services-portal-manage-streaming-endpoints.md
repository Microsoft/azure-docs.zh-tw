---
title: 透過 Azure 入口網站管理串流端點 | Microsoft Docs
description: 本主題說明如何透過 Azure 入口網站管理串流端點。
services: media-services
documentationcenter: ''
author: Juliako
writer: juliako
manager: cfowler
editor: ''
ms.assetid: bb1aca25-d23a-4520-8c45-44ef3ecd5371
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 09/10/2017
ms.author: juliako
ms.openlocfilehash: 542780766cfa90026d5ff492fcf7b579cb2d7029
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33785827"
---
# <a name="manage-streaming-endpoints-with-the-azure-portal"></a>透過 Azure 入口網站管理串流端點

本文說明如何使用 Azure 入口網站來管理串流端點。 

>[!NOTE]
>請務必檢閱[概觀](media-services-streaming-endpoints-overview.md)一文。 

如需調整串流端點的相關資訊，請參閱[這篇](media-services-portal-scale-streaming-endpoints.md)文章。

## <a name="start-managing-streaming-endpoints"></a>開始管理串流端點 

若要開始管理您帳戶的串流端點，請執行下列作業。

1. 在 [Azure 入口網站](https://portal.azure.com/)中，選取您的 Azure 媒體服務帳戶。
2. 在 [設定] 刀鋒視窗中，選取 [串流端點]。
   
    ![串流端點](./media/media-services-portal-manage-streaming-endpoints/media-services-manage-streaming-endpoints1.png)

> [!NOTE]
> 只有當串流端點處於執行中狀態時，才會向您收取費用。

## <a name="adddelete-a-streaming-endpoint"></a>新增/移除串流端點

>[!NOTE]
>預設串流端點不可刪除。

若要使用 Azure 入口網站來新增或移除串流端點，以執行下列作業：

1. 若要新增串流端點，請按一下頁面頂端的 [+端點]  。 

    如果您打算有不同的 CDN 或一個 CDN 和直接存取，您可能需要有多個串流端點。

2. 若要刪除串流端點，請按下 [刪除]  按鈕。      
3. 按一下 [啟動]  按鈕以啟動串流端點。
   
    ![串流端點](./media/media-services-portal-manage-streaming-endpoints/media-services-manage-streaming-endpoints2.png)


## <a id="configure_streaming_endpoints"></a>設定串流端點
串流端點可讓您設定下列屬性：

* 存取控制
* 快取控制
* 跨站台存取原則

如需這些屬性的詳細資訊，請參閱 [StreamingEndpoint](https://docs.microsoft.com/rest/api/media/operations/streamingendpoint)。

>[!NOTE]
>啟用 CDN 時，您無法使用 IP 存取。 當您沒有 CDN 時，才可使用 IP 存取。

若要設定串流端點，請執行以下作業：

1. 選取您想要設定的串流端點。
2. 按一下 [設定] 。

隨時顯示簡要的欄位說明。

![串流端點](./media/media-services-portal-manage-streaming-endpoints/media-services-manage-streaming-endpoints4.png)

1. 最大快取原則：用來設定透過此串流端點所提供的資源快取存留期。 如果沒有設定任何值，則會使用預設值。 預設值也可以直接在 Azure 儲存體中定義。 如果 Azure CDN 已針對串流端點啟用，您便不應該將快取原則值設定成少於 600 秒。  
2. 允許的 IP 位址：用來指定能夠連接到已發佈之串流端點的 IP 位址。 若未指定 IP 位址，則任何 IP 位址都可連接。 IP 位址可以指定為單一 IP 位址 (例如 ‘10.0.0.1’)、使用 IP 位址和 CIDR 子網路遮罩的 IP 範圍 (例如 ‘10.0.0.1/22’)，或是使用 IP 位址和小數點十進位子網路遮罩的 IP 範圍 (例如 ‘10.0.0.1(255.255.255.0)’)。
3. 針對 Akamai 簽章標頭驗證的設定：用來指定來自 Akamai 伺服器的簽章標頭驗證的設定方式。 到期時間的格式為 UTC。

## <a name="scale-your-premium-streaming-endpoint"></a>調整您的進階串流端點

如需詳細資訊，請參閱[本篇文章](media-services-portal-scale-streaming-endpoints.md)。

## <a id="enable_cdn"></a>啟用 Azure CDN 整合

當您建立新的帳戶時，依預設會啟用預設串流端點 Azure CDN 整合。

如果您稍後想要停用/啟用 CDN，串流端點必須處於**已停止**狀態。 可能需要將近 2 小時，Azure CDN 整合才會啟用，變更也才會遍及所有 CDN POP。 不過，您可以啟動串流端點，並從串流端點不停地串流，等到整合完成後，則會從 CDN 傳送資料流。 在佈建期間，串流端點會處於**啟動中**狀態，您可能會發現效能下降。

中國和聯邦政府區域除外，其他所有 Azure 資料中心都啟用 CDN 整合。

啟用後，[存取控制]、 ** [自訂主機名稱]和 [Akamai 簽章驗證] 設定就會停用。
 
> [!IMPORTANT]
> 如果是標準串流端點，Azure 媒體服務與 Azure CDN 的整合是在**來自 Verizon 的 Azure CDN** 上實作。 您可以使用所有 **Azure CDN 定價層和提供者**來設定進階串流端點。 如需 Azure CDN 功能的詳細資訊，請參閱 [CDN 概觀](../../cdn/cdn-overview.md)。
 
### <a name="additional-considerations"></a>其他考量

* 對串流端點啟用 CDN 時，用戶端無法直接從來源要求內容。 如果您需要在具有 CDN 或不具有 CDN 的情況下測試您內容的能力，您可以建立另一個未啟用 CDN 的串流端點。
* 您的串流端點主機名稱在啟用 CDN 之後維持不變。 您在啟用 CDN 之後不需要對您的媒體服務工作流程進行任何變更。 例如，如果您的串流端點主機名稱是 strasbourg.streaming.mediaservices.windows.net，則在啟用 CDN 之後，會使用完全相同的主機名稱。
* 對於新的串流端點，只要建立新的端點就會啟用 CDN；對於現有的串流端點，您必須先停止端點，然後啟用/停用 CDN。
* 只有透過 Azure 傳統入口網站使用 **Verizon 標準 CDN 提供者**，才能設定標準串流端點。 不過，您可以使用 REST API 來啟用其他 Azure CDN 提供者。

## <a name="configure-cdn-profile"></a>設定 CDN 設定檔

您可以選取頂端的 [管理 CDN] 按鈕來設定 CDN 設定檔。

![串流端點](./media/media-services-portal-manage-streaming-endpoints/media-services-manage-streaming-endpoints6.png)

## <a name="next-steps"></a>後續步驟
檢閱媒體服務學習路徑。

[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>提供意見反應
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

