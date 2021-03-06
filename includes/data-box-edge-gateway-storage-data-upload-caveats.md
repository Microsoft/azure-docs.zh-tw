---
author: alkohli
ms.service: databox
ms.topic: include
ms.date: 09/25/2020
ms.author: alkohli
ms.openlocfilehash: 6dc201af2271909de15af9bac1a2e2bb68faed1a
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/09/2020
ms.locfileid: "91400968"
---
下列注意事項適用於正移至 Azure 的資料。

- 我們建議不要將多個裝置寫入至相同的容器。
- 如果雲端中現有的 Azure 物件 (例如 Blob 或檔案) 與要複製的物件同名，裝置將會覆寫雲端中的檔案。
- 在共用資料夾下建立的空目錄階層 (不含任何檔案) 則不會上傳至 Blob 容器。
- 您可以使用拖放檔案總管或透過命令列來複製資料。 如果要複製的檔案匯總大小超過 10 GB，建議您使用大量複製程式，例如 Robocopy 或 rsync。 大量複製工具會重試複製作業的間歇性錯誤，並提供額外的復原能力。
- 如果與 Azure 儲存體容器相關的共用上傳不符合於建立時針對共用所定義之 Blob 類型的 Blob，則這類 Blob 將不會被更新。 例如，您在裝置上建立區塊 Blob 共用。 將該共用與具有分頁 Blob 的現有雲端容器相關聯。 重新整理該共用以下載檔案。 修改部分已重新整理，且已經在雲端中儲存為分頁 Blob 的檔案。 您將會看見上傳失敗。
- 在共用中建立檔案之後，不支援重新命名檔案。
- 從共用中刪除檔案並不會刪除儲存體帳戶中的項目。
- 如果使用 rsync 複製資料，則 `rsync -a` 不支援此選項。

