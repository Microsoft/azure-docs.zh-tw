---
title: 測試 Azure 受控應用程式的 UI 定義 | Microsoft Docs
description: 說明如何測試透過入口網站建立 Azure 受控應用程式的使用者體驗。
services: managed-applications
documentationcenter: na
author: tfitzmac
ms.service: managed-applications
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 08/22/2018
ms.author: tomfitz
ms.openlocfilehash: c88bdce64e88f8639da2c4ebb01f4594fccff8a0
ms.sourcegitcommit: b5ac31eeb7c4f9be584bb0f7d55c5654b74404ff
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/23/2018
ms.locfileid: "42747083"
---
# <a name="test-azure-portal-interface-for-your-managed-application"></a>測試您受控應用程式的 Azure 入口網站介面
為您的 Azure 受控應用程式[建立 createUiDefinition.json 檔案](create-uidefinition-overview.md)後，您必須測試使用者體驗。 若要簡化測試，請使用可在入口網站中載入檔案的指令碼。 您不需要實際部署受控應用程式。

## <a name="prerequisites"></a>必要條件

* **createUiDefinition.json** 檔案。 如果您沒有此檔案，請複製[範例檔案](https://github.com/Azure/azure-quickstart-templates/blob/master/100-marketplace-sample/createUiDefinition.json)，並將其儲存於本機。

* Azure 訂用帳戶。 如果您沒有 Azure 訂用帳戶，請在開始之前先[建立免費帳戶](https://azure.microsoft.com/free/)。

## <a name="download-test-script"></a>下載測試指令碼

若要在入口網站中測試您的介面，請將下列其中一個指令碼複製到本機電腦：

* [PowerShell 側載指令碼](https://github.com/Azure/azure-quickstart-templates/blob/master/SideLoad-CreateUIDefinition.ps1)
* [Azure CLI 側載指令碼](https://github.com/Azure/azure-quickstart-templates/blob/master/sideload-createuidef.sh)

## <a name="run-script"></a>執行指令碼

若要在入口網站中查看您的介面檔案，請執行您已下載的指令碼。 此指令碼會在您的 Azure 訂用帳戶中建立儲存體帳戶，並將 createUiDefinition.json 檔案上傳至儲存體帳戶。 第一次執行指令碼時或儲存體帳戶已刪除後，將會建立儲存體帳戶。 如果 Azure 訂用帳戶中已有儲存體帳戶，則指令碼會重複使用該帳戶。 指令碼會開啟入口網站，並從儲存體帳戶載入您的檔案。

請提供儲存體帳戶的位置，並指定含有 createUiDefinition.json 檔案的資料夾。

對於 PowerShell，請使用：

```powershell
.\SideLoad-CreateUIDefinition.ps1 `
  -StorageResourceGroupLocation southcentralus `
  -ArtifactsStagingDirectory .\100-Marketplace-Sample
```

對於 Azure CLI，請使用：

```azurecli
./sideload-createuidef.sh \
  -l southcentralus \
  -a .\100-Marketplace-Sample
```

如果 createUiDefinition.json 檔案位於和指令碼相同的資料夾內，而且您已建立儲存體帳戶，則不需要提供這些參數。

對於 PowerShell，請使用：

```powershell
.\SideLoad-CreateUIDefinition.ps1
```

對於 Azure CLI，請使用：

```azurecli
./sideload-createuidef.sh
```

## <a name="test-your-interface"></a>測試您的介面

指令碼會在瀏覽器中開啟新的索引標籤。 它會顯示入口網站，其中包含您用來建立受控應用程式的介面。

![檢視入口網站](./media/test-createuidefinition/view-portal.png)

在填寫欄位之前，請在瀏覽器中開啟 Web Developer Tool。 [主控台] 會顯示關於介面的重要訊息。

![選取主控台](./media/test-createuidefinition/select-console.png)

如果您的介面定義有錯誤，您會在主控台中看到描述。

![顯示錯誤](./media/test-createuidefinition/show-error.png)

提供欄位的值。 作業完成後，您會看到傳至範本的值。

![顯示值](./media/test-createuidefinition/show-json.png)

您可以使用這些值作為參數檔案來測試部署範本。

## <a name="troubleshooting-the-interface"></a>針對介面進行疑難排解

您可能會看到的一些常見錯誤包括：

* 入口網站未載入介面。 相反地，它顯示了有淚滴的雲端圖示。 通常，您會在檔案有語法錯誤時看到這個圖示。 在 VS Code (或其他具有結構描述驗證的 JSON 編輯器) 中開啟檔案並尋找語法錯誤。

* 入口網站在 [摘要] 畫面停止回應。 通常，當 [輸出] 區段有錯誤時，就會發生此中斷。 例如，您可能參考了不存在的控制項。

* 輸出中的參數是空的。 該參數可能參考了不存在的屬性。 例如，控制項的參考有效，但屬性參考無效。

## <a name="test-your-solution-files"></a>測試您的解決方案檔案

現在您已確認入口網站介面正常運作，接下來可以驗證您的 createUiDefinition 檔案是否與 mainTemplate.json 檔案正確整合。 您可以執行驗證指令碼測試來測試解決方案檔案的內容，包括 createUiDefinition 檔案。 指令碼會驗證 JSON 語法、檢查文字欄位上的 regex 運算式，並確定入口網站介面的輸出值符合您的範本參數。 如需執行此指令碼的相關資訊，請參閱[執行範本的靜態驗證檢查](https://github.com/Azure/azure-quickstart-templates/tree/master/test/template-validation-tests)。

## <a name="next-steps"></a>後續步驟

驗證入口網站介面後，請了解如何建立[在 Marketplace 中提供的 Azure 受控應用程式](publish-marketplace-app.md)。
