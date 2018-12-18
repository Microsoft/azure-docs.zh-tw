---
title: 為 DevTest Labs 虛擬機器建立自訂構件 | Microsoft Docs
description: 了解如何撰寫自己的構件以用於 Azure DevTest Labs。
services: devtest-lab,virtual-machines
documentationcenter: na
author: spelluru
manager: ''
editor: ''
ms.assetid: 32dcdc61-ec23-4a01-b731-78c029ea5316
ms.service: devtest-lab
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/11/2018
ms.author: spelluru
ms.openlocfilehash: ad9e9e893dc831530b69a30cc3dd930e879e9d7b
ms.sourcegitcommit: bf522c6af890984e8b7bd7d633208cb88f62a841
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/20/2018
ms.locfileid: "39185113"
---
# <a name="create-custom-artifacts-for-your-devtest-labs-virtual-machine"></a>為 DevTest Labs 虛擬機器建立自訂構件

觀看下列影片以概略了解本文所說明的步驟：

> [!VIDEO https://channel9.msdn.com/Blogs/Azure/how-to-author-custom-artifacts/player]
> 
> 

## <a name="overview"></a>概觀
在佈建 VM 之後，您可以使用「構件」來部署和設定應用程式。 構件包含構件定義檔和其他儲存於 Git 存放庫之資料夾中的指令碼檔案。 構件定義檔是由 JSON 和可用來指定您想要在 VM 上安裝的運算式所組成。 例如，您可以定義構件名稱、要執行的命令，以及執行命令時可用的參數。 您可以依照名稱來參考構件定義檔中的其他指令碼檔案。

## <a name="artifact-definition-file-format"></a>構件定義檔格式
下列範例顯示組成定義檔基本結構的區段：

    {
      "$schema": "https://raw.githubusercontent.com/Azure/azure-devtestlab/master/schemas/2016-11-28/dtlArtifacts.json",
      "title": "",
      "description": "",
      "iconUri": "",
      "targetOsType": "",
      "parameters": {
        "<parameterName>": {
          "type": "",
          "displayName": "",
          "description": ""
        }
      },
      "runCommand": {
        "commandToExecute": ""
      }
    }

| 元素名稱 | 必要？ | 說明 |
| --- | --- | --- |
| $schema |否 |JSON 結構描述檔案的位置。 JSON 結構描述檔案可協助您測試定義檔是否有效。 |
| title |是 |實驗室中顯示的構件名稱。 |
| 說明 |是 |實驗室中顯示的構件說明。 |
| iconUri |否 |實驗室中顯示的圖示 URI。 |
| targetOsType |是 |構件安裝所在之 VM 的作業系統。 支援的選項為 Windows 和 Linux。 |
| parameters |否 |在電腦上執行構件安裝命令時所提供的值。 這可協助您自訂構件。 |
| runCommand |是 |在 VM 上執行的構件安裝命令。 |

### <a name="artifact-parameters"></a>構件參數
在定義檔的參數區段中，指定可供使用者在安裝構件時輸入的值。 您可以在構件安裝命令中參考這些值。

若要定義參數，請使用下列結構：

    "parameters": {
        "<parameterName>": {
          "type": "<type-of-parameter-value>",
          "displayName": "<display-name-of-parameter>",
          "description": "<description-of-parameter>"
        }
      }

| 元素名稱 | 必要？ | 說明 |
| --- | --- | --- |
| type |是 |參數值類型。 請參閱下列清單以了解允許的類型。 |
| displayName |是 |為實驗室中的使用者顯示的參數名稱。 | |
| 說明 |是 |在實驗室中顯示的參數說明。 |

允許的類型為：

* string (任何有效的 JSON 字串)
* int (任何有效的 JSON 整數)
* bool (任何有效的 JSON 布林值)
* array (任何有效的 JSON 陣列)

## <a name="artifact-expressions-and-functions"></a>構件運算式和函式
您可以使用運算式和函式來建構構件安裝命令。
運算式是以方括號 ([ 與 ]) 括住，並會在安裝構件後加以評估。 運算式可以出現在 JSON 字串值的任何位置。 運算式一律會傳回另一個 JSON 值。 如果您必須使用開頭為括號 ([) 的常數字串，您必須使用兩個括號 ([[)。
通常您會使用運算式搭配函式來建構值。 正如同在 JavaScript 中，函式呼叫的格式為 **functionName(arg1,arg2,arg3)**。

下列清單顯示常見的函式：

* **parameters(parameterName)**：傳回在執行構件命令時所提供的參數值。
* **concat(arg1, arg2, arg3,….. )**：結合多個字串值。 此函式可以接受各種引數。

下列範例示範如何使用運算式和函式來建構值：

    runCommand": {
         "commandToExecute": "[concat('powershell.exe -ExecutionPolicy bypass \"& ./startChocolatey.ps1'
    , ' -RawPackagesList ', parameters('packages')
    , ' -Username ', parameters('installUsername')
    , ' -Password ', parameters('installPassword'))]"
    }

## <a name="create-a-custom-artifact"></a>建立自訂構件

1. 安裝 JSON 編輯器。 您需要 JSON 編輯器才能使用構件定義檔。 建議您使用 [Visual Studio 程式碼](https://code.visualstudio.com/)，這適用於 Windows、Linux 和 OS X。
2. 取得 artifactfile.json 定義檔範例。 請參閱 [GitHub 存放庫](https://github.com/Azure/azure-devtestlab)中由 DevTest Labs 小組所建立的構件。 我們已建立了豐富的構件庫，以協助您建立您自己的構件。 下載構件定義檔，並對它進行變更以建立您自己的構件。
3. 利用 IntelliSense。 使用 IntelliSense 來查看可用於建構構件定義檔的有效元素。 您也可以看到適用於元素值的不同選項。 例如，當您編輯 **targetOsType** 元素時，IntelliSense 會顯示 Windows 或 Linux 這兩個選項。
4. 將成品儲存在[適用於 DevTest Labs 的公開 Git 存放庫](https://github.com/Azure/azure-devtestlab/tree/master/Artifacts)或[您自己的 Git 存放庫](devtest-lab-add-artifact-repo.md)中。 在公用存放庫中，您可以檢視其他人分享的成品，並且可以直接使用這些成品，或根據您的需求來自訂這些成品。 
   
   1. 為每個構件建立個別的目錄。 目錄名稱應該和構件名稱相同。
   2. 將構件定義檔 (artifactfile.json) 儲存於您建立的目錄中。
   3. 儲存從構件安裝命令參考的指令碼。
      
      構件資料夾的可能外觀範例如下：
      
      ![構件資料夾範例](./media/devtest-lab-artifact-author/git-repo.png)
5. 如果您使用自己的存放庫來儲存成品，請依照[新增成品和範本的 Git 存放庫](devtest-lab-add-artifact-repo.md)一文中的指示，將存放庫新增至實驗室。


## <a name="related-articles"></a>相關文章
* [如何診斷 DevTest Labs 中的構件失敗](devtest-lab-troubleshoot-artifact-failure.md)
* [Join a VM to an existing Active Directory domain by using a Resource Manager template in DevTest Labs](http://www.visualstudiogeeks.com/blog/DevOps/Join-a-VM-to-existing-AD-domain-using-ARM-template-AzureDevTestLabs) (在 DevTest Labs 中使用 Resource Manager 範本將 VM 加入至現有 Active Directory 網域)

## <a name="next-steps"></a>後續步驟
* 了解如何 [將 Git 構件儲存機制加入實驗室](devtest-lab-add-artifact-repo.md)。

