---
title: Azure FileUpload UI 元素 | Microsoft Docs
description: 描述 Azure 入口網站的 Microsoft.Common.FileUpload UI 元素。
services: managed-applications
documentationcenter: na
author: tfitzmac
manager: timlt
editor: tysonn
ms.service: managed-applications
ms.devlang: na
ms.topic: reference
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 09/05/2018
ms.author: tomfitz
ms.openlocfilehash: 2886dbafe6bf20718f4e3cd2976764fc432dbb04
ms.sourcegitcommit: d211f1d24c669b459a3910761b5cacb4b4f46ac9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/06/2018
ms.locfileid: "44021747"
---
# <a name="microsoftcommonfileupload-ui-element"></a>Microsoft.Common.FileUpload UI 元素
控制項可讓使用者指定要上傳的一個或多個檔案。

## <a name="ui-sample"></a>UI 範例
![Microsoft.Common.FileUpload](./media/managed-application-elements/microsoft.common.fileupload.png)

## <a name="schema"></a>結構描述
```json
{
  "name": "element1",
  "type": "Microsoft.Common.FileUpload",
  "label": "Some file upload",
  "toolTip": "",
  "constraints": {
    "required": true,
    "accept": ".doc,.docx,.xml,application/msword"
  },
  "options": {
    "multiple": false,
    "uploadMode": "file",
    "openMode": "text",
    "encoding": "UTF-8"
  },
  "visible": true
}
```

## <a name="remarks"></a>備註
- `constraints.accept` 會指定在瀏覽器的 [檔案] 對話方塊中顯示的檔案類型。 請參閱 [HTML5 規格](http://www.w3.org/TR/html5/forms.html#attr-input-accept) 以取得允許的值。 預設值為 **null**。
- 如果將 `options.multiple` 設為 **true**，使用者就允許在瀏覽器的 [檔案] 對話方塊中選取一個以上的檔案。 預設值為 **false**。
- 這個元素會根據 `options.uploadMode` 的值，支援兩種檔案上傳模式。 如果是指定 **file**，輸出就會包含檔案內容作為 Blob。 如果是指定 **URL**，檔案就會上傳至暫存位置，而輸出會包含 Blob 的 URL。 24 小時之後，就會清除暫存 blob。 預設值為 **file**。
- 上傳的檔案已受保護。 輸出 URL 包含 [SAS 權杖](../storage/common/storage-dotnet-shared-access-signature-part-1.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)，可供在部署期間存取檔案。
- `options.openMode` 的值會決定讀取檔案的方式。 如果預期是純文字檔案，則指定為 **text**；否則，指定為 **binary**。 預設值為 **text**。
- 如果將 `options.uploadMode` 設為 **file**，且將 `options.openMode` 設為 **binary**，輸出就會是 base64 編碼。
- `options.encoding` 會指定讀取檔案時要使用的編碼方式。 預設值為 **UTF-8**，且僅在 `options.openMode` 設為 **text** 時才會使用。

## <a name="sample-output"></a>範例輸出
如果 options.multiple 為 false 且 options.uploadMode 為 file，輸出就會包含檔案內容作為 JSON 字串：

```json
"Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua."
```

如果 options.multiple 為 true 且 options.uploadMode 為 file，輸出就會包含檔案內容作為 JSON 陣列：

```json
[
  "Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua.",
  "Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat.",
  "Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur.",
  "Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum."
]
```

如果 options.multiple 為 false 且 options.uploadMode 為 URL，輸出就會包含 URL 作為 JSON 字串：

```json
"https://myaccount.blob.core.windows.net/pictures/profile.jpg?sv=2013-08-15&st=2013-08-16&se=2013-08-17&sr=c&sp=r&rscd=file;%20attachment&rsct=binary &sig=YWJjZGVmZw%3d%3d&sig=a39%2BYozJhGp6miujGymjRpN8tsrQfLo9Z3i8IRyIpnQ%3d"
```

如果 options.multiple 為 true 且 options.uploadMode 為 URL，輸出就會包含 URL 清單作為 JSON 陣列：
```json
[
  "https://myaccount.blob.core.windows.net/pictures/profile1.jpg?sv=2013-08-15&st=2013-08-16&se=2013-08-17&sr=c&sp=r&rscd=file;%20attachment&rsct=binary &sig=YWJjZGVmZw%3d%3d&sig=a39%2BYozJhGp6miujGymjRpN8tsrQfLo9Z3i8IRyIpnQ%3d",
  "https://myaccount.blob.core.windows.net/pictures/profile2.jpg?sv=2013-08-15&st=2013-08-16&se=2013-08-17&sr=c&sp=r&rscd=file;%20attachment&rsct=binary &sig=YWJjZGVmZw%3d%3d&sig=a39%2BYozJhGp6miujGymjRpN8tsrQfLo9Z3i8IRyIpnQ%3d",
  "https://myaccount.blob.core.windows.net/pictures/profile3.jpg?sv=2013-08-15&st=2013-08-16&se=2013-08-17&sr=c&sp=r&rscd=file;%20attachment&rsct=binary &sig=YWJjZGVmZw%3d%3d&sig=a39%2BYozJhGp6miujGymjRpN8tsrQfLo9Z3i8IRyIpnQ%3d"
]
```

測試 CreateUiDefinition 時，某些瀏覽器 (例如 Google Chrome) 會將瀏覽器主控台的 Microsoft.Common.FileUpload 元素所產生的 URL 截斷。 您可能需要以滑鼠右鍵按一下個別連結來複製完整的 URL。


## <a name="next-steps"></a>後續步驟
* 如需建立 UI 定義的簡介，請參閱[開始使用 CreateUiDefinition](create-uidefinition-overview.md)。
* 如需 UI 元素中通用屬性的說明，請參閱 [CreateUiDefinition 元素](create-uidefinition-elements.md)。
