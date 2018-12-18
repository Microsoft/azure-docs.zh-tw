---
title: 包含檔案
description: 包含檔案
services: iot-edge
author: kgremban
ms.service: iot-edge
ms.topic: include
ms.date: 08/20/2018
ms.author: kgremban
ms.custom: include file
ms.openlocfilehash: 8aa4695ea1175fe9d558e02bae661c9610123299
ms.sourcegitcommit: f6e2a03076679d53b550a24828141c4fb978dcf9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43086387"
---
## <a name="associate-an-azure-storage-account-to-iot-hub"></a>讓 Azure 儲存體帳戶與 IoT 中樞產生關聯

由於模擬裝置應用程式會將檔案上傳至 Blob，因此您必須擁有和 IoT 中樞關聯的 [Azure 儲存體](../articles/storage/common/storage-quickstart-create-account.md)帳戶。 當您將 Azure 儲存體帳戶關聯 IoT 中樞時，IoT 中樞會產生 SAS URI。 裝置可以使用此 SAS URI，安全地將檔案上傳至 blob 容器。 IoT 中樞服務和裝置 SDK 會協調產生 SAS URI 的程序，並使其可供裝置用來上傳檔案。

依照[使用 Azure 入口網站設定檔案上傳](../articles/iot-hub/iot-hub-configure-file-upload.md)中的指示，讓 Azure 儲存體帳戶與 IoT 中樞產生關聯。 請確定 blob 容器與 IoT 中樞相關聯，並已啟用檔案通知。

![在入口網站中啟用檔案通知](./media/iot-hub-associate-storage/enable-file-notifications.png)