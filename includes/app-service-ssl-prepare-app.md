---
title: 包含檔案
description: 包含檔案
services: app-service
author: cephalin
ms.service: app-service
ms.topic: include
ms.date: 10/15/2018
ms.author: cephalin
ms.custom: include file
ms.openlocfilehash: 858d098f18604d1360af8509eb8a0cec6fa7d0c7
ms.sourcegitcommit: 8e06d67ea248340a83341f920881092fd2a4163c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49354527"
---
## <a name="prepare-your-web-app"></a>準備您的 Web 應用程式

若要將自訂 SSL 憑證 (第三方憑證或 App Service 憑證) 繫結至 Web 應用程式，[App Service 方案](https://azure.microsoft.com/pricing/details/app-service/)必須為**基本**、**標準**、**進階**或**隔離**層。 在此步驟中，您要確定 Web 應用程式在支援的定價層。

### <a name="log-in-to-azure"></a>登入 Azure

開啟 [Azure 入口網站](https://portal.azure.com)。

### <a name="navigate-to-your-web-app"></a>瀏覽至您的 Web 應用程式

按一下左側功能表中的 [應用程式服務]，然後按一下 Web 應用程式的名稱。

![選取 Web 應用程式](./media/app-service-ssl-prepare-app/select-app.png)

您已經位於 Web 應用程式的管理頁面。  

### <a name="check-the-pricing-tier"></a>檢查定價層

在 Web 應用程式頁面的左側導覽中，捲動到 [設定] 區段，然後選取 [相應增加 (App Service 方案)]。

![相應增加功能表](./media/app-service-ssl-prepare-app/scale-up-menu.png)

請檢查以確定您的 Web 應用程式不在 **F1** 或 **D1** 層中。 系統會以深藍色方塊醒目顯示 Web 應用程式目前的層。

![檢查定價層](./media/app-service-ssl-prepare-app/check-pricing-tier.png)

**F1** 或 **D1** 層中不支援自訂 SSL。 如果您需要相應增加，請遵循下一節中的步驟來進行。 否則，請關閉 [相應增加] 頁面，然後跳至[上傳並繫結 SSL 憑證](#upload)。

### <a name="scale-up-your-app-service-plan"></a>相應增加您的 App Service 方案

選取任何非免費層 (**B1**、**B2****B3**或「生產」類別中的任何一層)。 如需其他選項，請按一下 [查看其他選項]。

按一下 [套用]。

![選擇定價層](./media/app-service-ssl-prepare-app/choose-pricing-tier.png)

當您看見下列通知時，表示擴充作業已完成。

![相應增加通知](./media/app-service-ssl-prepare-app/scale-notification.png)

