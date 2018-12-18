---
title: Azure Blockchain Workbench 組態參考
description: Azure Blockchain Workbench 應用程式組態概觀。
services: azure-blockchain
keywords: ''
author: PatAltimore
ms.author: patricka
ms.date: 10/1/2018
ms.topic: article
ms.service: azure-blockchain
ms.reviewer: zeyadr
manager: femila
ms.openlocfilehash: fd3ff0087ee51c392d9cebb32c8bcc969f9a4601
ms.sourcegitcommit: 1981c65544e642958917a5ffa2b09d6b7345475d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/03/2018
ms.locfileid: "48241236"
---
# <a name="azure-blockchain-workbench-configuration-reference"></a>Azure Blockchain Workbench 組態參考

 Azure Blockchain Workbench 應用程式是以組態中繼資料和智慧合約程式碼所定義的多方工作流程。 組態中繼資料會定義區塊鏈應用程式的高階工作流程和互動模型。 智慧合約則會定義區塊鏈應用程式的商務邏輯。 Workbench 會使用組態和智慧合約程式碼來產生區塊鏈應用程式使用者體驗。

組態中繼資料會指定每個區塊鏈應用程式的下列資訊： 

* 區塊鏈應用程式的名稱和說明
* 可在區塊鏈應用程式中採取行動或參與的使用者唯一角色
* 一或多個工作流程。 每個工作流程都可作為狀態機器以控制商務邏輯的流程。 工作流程既可彼此獨立，也可彼此互動。

所定義的每個工作流程都會指定下列項目：

* 工作流程的名稱和說明
* 工作流程的狀態。  每個狀態都是商務邏輯控制流程中的一個階段。 
* 轉換至下一個狀態的動作
* 允許將每個動作初始化的使用者角色
* 代表程式碼檔案中商務邏輯的智慧合約

## <a name="application"></a>Application

區塊鏈應用程式包含組態中繼資料、工作流程，以及可在應用程式中採取行動或參與的使用者角色。

| 欄位 | 說明 | 必要 |
|-------|-------------|:--------:|
| ApplicationName | 唯一的應用程式名稱。 對應的智慧合約必須針對適用的合約類別使用相同的 **ApplicationName**。  | 是 |
| DisplayName | 應用程式的易記顯示名稱。 | 是 |
| 說明 | 應用程式的說明。 | 否 |
| ApplicationRoles | [ApplicationRoles](#application-roles) 的集合。 可在應用程式中採取行動或參與的使用者角色。  | 是 |
| 工作流程 | [工作流程](#workflows)的集合。 每個工作流程都可作為狀態機器以控制商務邏輯的流程。 | 是 |

如需範例，請參閱[組態檔範例](#configuration-file-example)。

## <a name="workflows"></a>工作流程

您可以將應用程式的商務邏輯塑造為狀態機器，在此機器中採取動作將會造成商務邏輯的流程從某個狀態進入到另一個狀態。 工作流程是這類狀態和動作的集合。 每個工作流程都是由一或多個智慧合約所組成，這些合約可代表程式碼檔案中的商務邏輯。 可執行檔合約是工作流程的執行個體。

| 欄位 | 說明 | 必要 |
|-------|-------------|:--------:|
| 名稱 | 唯一的工作流程名稱。 對應的智慧合約必須針對適用的合約類別使用相同的**名稱**。 | 是 |
| DisplayName | 工作流程的易記顯示名稱。 | 是 |
| 說明 | 工作流程的說明。 | 否 |
| Initiators | [ApplicationRoles](#application-roles) 的集合。 對已獲授權可在工作流程中建立合約的使用者所指派的角色。 | 是 |
| StartState | 工作流程初始狀態的名稱。 | 是 |
| properties | [識別碼](#identifiers)的集合。 代表可在使用者體驗工具中進行鏈結關閉讀取或視覺化的資料。 | 是 |
| 建構函式 | 定義用於建立工作流程執行個體的輸入參數。 | 是 |
| Functions | 可在工作流程中執行的[函式](#functions)集合。 | 是 |
| 狀態 | 工作流程[狀態](#states)的集合。 | 是 |

如需範例，請參閱[組態檔範例](#configuration-file-example)。

## <a name="type"></a>類型

支援的資料類型。

| 類型 | 說明 |
|-------|-------------|
| 位址  | 區塊鏈位址類型，例如「合約」或「使用者」 |
| 布林     | 布林值資料類型 |
| 合約 | 合約類型的位址 |
| 列舉     | 列舉的具名值組。 使用列舉類型時，您也可以指定一份 EnumValues 清單。 每個值的上限為 255 個字元。 有效值的字元包括大小寫字母 (A-Z、a-z) 和數字 (0-9)。 |
| int      | 整數資料類型 |
| money    | 金錢資料類型 |
| state    | 工作流程狀態 |
| 字串   | 字串資料類型 |
| user     | 使用者類型的位址 |
| 分析     | 時間資料類型 |
|`[ Application Role Name ]`| 應用程式角色中所指定的任何名稱。 限制使用者只能成為該角色類型。 |

### <a name="example-configuration-of-type-string"></a>字串類型的組態範例

``` json
{
  "Name": "description",
  "Description": "Descriptive text",
  "DisplayName": "Description",
  "Type": {
    "Name": "string"
  }
}
```

### <a name="example-configuration-of-type-enum"></a>列舉類型的組態範例

``` json
{
  "Name": "PropertyType",
  "DisplayName": "Property Type",
  "Description": "The type of the property",
  "Type": {
    "Name": "enum",
    "EnumValues": ["House", "Townhouse", "Condo", "Land"]
  }
}
```

#### <a name="using-enumeration-type-in-solidity"></a>在 Solidity 中使用列舉類型

一旦列舉定義於組態中，您就可以在 Solidity 中使用列舉類型。 例如，您可以定義稱為 PropertyTypeEnum 的列舉。

```
enum PropertyTypeEnum {House, Townhouse, Condo, Land} PropertyTypeEnum public PropertyType; 
```

字串清單必須在組態和智慧合約之間進行比對，以成為 Blockchain Workbench 中有效且一致的宣告。

指派範例：

```
PropertyType = PropertyTypeEnum.Townhouse;
```

函式參數範例： 

``` 
function AssetTransfer(string description, uint256 price, PropertyTypeEnum propertyType) public
{
    InstanceOwner = msg.sender;
    AskingPrice = price;
    Description = description;
    PropertyType = propertyType;
    State = StateType.Active;
    ContractCreated();
}

```

## <a name="constructor"></a>建構函式

定義工作流程執行個體的輸入參數。

| 欄位 | 說明 | 必要 |
|-------|-------------|:--------:|
| 參數 | 需要起始智慧合約的[識別碼](#identifiers)集合。 | 是 |

### <a name="constructor-example"></a>建構函式範例

``` json
{
  "Parameters": [
    {
      "Name": "description",
      "Description": "The description of this asset",
      "DisplayName": "Description",
      "Type": {
        "Name": "string"
      }
    },
    {
      "Name": "price",
      "Description": "The price of this asset",
      "DisplayName": "Price",
      "Type": {
        "Name": "money"
      }
    }
  ]
}
```

## <a name="functions"></a>Functions

定義可在工作流程中執行的函式。

| 欄位 | 說明 | 必要 |
|-------|-------------|:--------:|
| 名稱 | 函式的唯一名稱。 對應的智慧合約必須針對適用的函式使用相同的**名稱**。 | 是 |
| DisplayName | 函式的易記顯示名稱。 | 是 |
| 說明 | 函式的說明 | 否 |
| 參數 | 對應至函式參數的[識別碼](#identifiers)集合。 | 是 |

### <a name="functions-example"></a>函式範例

``` json
"Functions": [
  {
    "Name": "Modify",
    "DisplayName": "Modify",
    "Description": "Modify the description/price attributes of this asset transfer instance",
    "Parameters": [
      {
        "Name": "description",
        "Description": "The new description of the asset",
        "DisplayName": "Description",
        "Type": {
          "Name": "string"
        }
      },
      {
        "Name": "price",
        "Description": "The new price of the asset",
        "DisplayName": "Price",
        "Type": {
          "Name": "money"
        }
      }
    ]
  },
  {
    "Name": "Terminate",
    "DisplayName": "Terminate",
    "Description": "Used to cancel this particular instance of asset transfer",
    "Parameters": []
  }
]

```

## <a name="states"></a>狀態

工作流程內唯一狀態的集合。 每個狀態都會擷取商務邏輯控制流程中的一個步驟。 

| 欄位 | 說明 | 必要 |
|-------|-------------|:--------:|
| 名稱 | 狀態的唯一名稱。 對應的智慧合約必須針對適用的狀態使用相同的**名稱**。 | 是 |
| DisplayName | 狀態的易記顯示名稱。 | 是 |
| 說明 | 狀態的說明。 | 否 |
| PercentComplete | Blockchain Workbench 使用者介面中所顯示的整數值，用來顯示商務邏輯控制流程內的進度。 | 是 |
| Style | 指出狀態是表示成功還是失敗狀態的視覺提示。 有效值有兩個：`Success` 或 `Failure`。 | 是 |
| 轉換 | 可從目前狀態進入到下一組狀態的[轉換](#transitions)集合。 | 否 |

### <a name="states-example"></a>狀態範例

``` json
"States": [
    {
      "Name": "Active",
      "DisplayName": "Active",
      "Description": "The initial state of the asset transfer workflow",
      "PercentComplete": 20,
      "Style": "Success",
      "Transitions": [
        {
          "AllowedRoles": [],
          "AllowedInstanceRoles": [ "InstanceOwner" ],
          "Description": "Cancels this instance of asset transfer",
          "Function": "Terminate",
          "NextStates": [ "Terminated" ],
          "DisplayName": "Terminate Offer"
        },
        {
          "AllowedRoles": [ "Buyer" ],
          "AllowedInstanceRoles": [],
          "Description": "Make an offer for this asset",
          "Function": "MakeOffer",
          "NextStates": [ "OfferPlaced" ],
          "DisplayName": "Make Offer"
        },
        {
          "AllowedRoles": [],
          "AllowedInstanceRoles": [ "InstanceOwner" ],
          "Description": "Modify attributes of this asset transfer instance",
          "Function": "Modify",
          "NextStates": [ "Active" ],
          "DisplayName": "Modify"
        }
      ]
    },
    {
      "Name": "Accepted",
      "DisplayName": "Accepted",
      "Description": "Asset transfer process is complete",
      "PercentComplete": 100,
      "Style": "Success",
      "Transitions": []
    },
    {
      "Name": "Terminated",
      "DisplayName": "Terminated",
      "Description": "Asset transfer has been cancelled",
      "PercentComplete": 100,
      "Style": "Failure",
      "Transitions": []
    }
  ]
```

## <a name="transitions"></a>轉換

下一個狀態的可用動作。 一或多個使用者角色可在每個狀態執行動作，動作可能會讓某個狀態轉換為工作流程中的另一個狀態。 

| 欄位 | 說明 | 必要 |
|-------|-------------|:--------:|
| AllowedRoles | 允許起始轉換的應用程式角色清單。 屬於指定角色的所有使用者都可執行該動作。 | 否 |
| AllowedInstanceRoles | 參與或指定於智慧合約中、允許起始轉換的使用者角色清單。 執行個體角色會定義在工作流程的**屬性**中。 AllowedInstanceRoles 代表參與智慧合約執行個體的使用者。 AllowedInstanceRoles 可讓您對合約執行個體中的使用者角色限制採取動作。  例如，如果您只想讓建立合約的使用者 (InstanceOwner) 可以執行終止作業，而不是讓角色類型 (Owner) 中的所有使用者可這麼做，您可以在 AllowedRoles 中指定該角色。 | 否 |
| DisplayName | 轉換的易記顯示名稱。 | 是 |
| 說明 | 轉換的說明。 | 否 |
| 函式 | 要起始轉換的函式名稱。 | 是 |
| NextStates | 成功轉換之後下一個可能狀態的集合。 | 是 |

### <a name="transitions-example"></a>轉換範例

``` json
"Transitions": [
  {
    "AllowedRoles": [],
    "AllowedInstanceRoles": [ "InstanceOwner" ],
    "Description": "Cancels this instance of asset transfer",
    "Function": "Terminate",
    "NextStates": [ "Terminated" ],
    "DisplayName": "Terminate Offer"
  },
  {
    "AllowedRoles": [ "Buyer" ],
    "AllowedInstanceRoles": [],
    "Description": "Make an offer for this asset",
    "Function": "MakeOffer",
    "NextStates": [ "OfferPlaced" ],
    "DisplayName": "Make Offer"
  },
  {
    "AllowedRoles": [],
    "AllowedInstanceRoles": [ "InstanceOwner" ],
    "Description": "Modify attributes of this asset transfer instance",
    "Function": "Modify",
    "NextStates": [ "Active" ],
    "DisplayName": "Modify"
  }
]

```

## <a name="application-roles"></a>應用程式角色

應用程式角色會針對要在應用程式中採取行動或參與的使用者，定義一組可對其指派的角色。 應用程式角色可用來限制區塊鏈應用程式和對應的工作流程內所能進行的行動和參與。 

| 欄位 | 說明 | 必要 |
|-------|-------------|:--------:|
| 名稱 | 應用程式角色的唯一名稱。 對應的智慧合約必須針對適用的角色使用相同的**名稱**。 系統會保留基底類型名稱。 您不能使用和[類型](#type)相同的名稱來為應用程式角色命名| 是 |
| 說明 | 應用程式角色的說明。 | 否 |

### <a name="application-roles-example"></a>應用程式角色範例

``` json
"ApplicationRoles": [
  {
    "Name": "Appraiser",
    "Description": "User that signs off on the asset price"
  },
  {
    "Name": "Buyer",
    "Description": "User that places an offer on an asset"
  }
]
```
## <a name="identifiers"></a>識別項

識別碼代表用來說明工作流程屬性、建構函式和函式參數的資訊集合。 

| 欄位 | 說明 | 必要 |
|-------|-------------|:--------:|
| 名稱 | 屬性或參數的唯一名稱。 對應的智慧合約必須針對適用的屬性或參數使用相同的**名稱**。 | 是 |
| DisplayName | 屬性或參數的易記顯示名稱。 | 是 |
| 說明 | 屬性或參數的說明。 | 否 |

### <a name="identifiers-example"></a>識別碼範例

``` json
"Properties": [
  {
    "Name": "State",
    "DisplayName": "State",
    "Description": "Holds the state of the contract",
    "Type": {
      "Name": "state"
    }
  },
  {
    "Name": "Description",
    "DisplayName": "Description",
    "Description": "Describes the asset being sold",
    "Type": {
      "Name": "string"
    }
  }
]
```

## <a name="configuration-file-example"></a>組態檔範例

資產轉送是購買和銷售高價值資產的智慧合約案例，而這需要檢查員和評鑑員。 賣方可以藉由具現化資產轉送智慧合約來列出其資產。 買方可以藉由在智慧合約上採取動作來取得供應項目，而其他合作對象可以採取動作來檢查或評鑑資產。 一旦資產標記為已檢查和已評鑑，買方與賣方會再次確認此銷售，然後將合約設定為完成。 在處理程序中的每一個位置上，所有參與者都可看見合約更新時的狀態。 

如需詳細資訊 (包括在程式碼檔案)，請參閱 [Azure Blockchain Workbench 的資產轉送範例](https://github.com/Azure-Samples/blockchain/tree/master/blockchain-workbench/application-and-smart-contract-samples/asset-transfer)

下列是資產轉送範例的組態檔：

``` json
{
  "ApplicationName": "AssetTransfer",
  "DisplayName": "Asset Transfer",
  "Description": "Allows transfer of assets between a buyer and a seller, with appraisal/inspection functionality",
  "ApplicationRoles": [
    {
      "Name": "Appraiser",
      "Description": "User that signs off on the asset price"
    },
    {
      "Name": "Buyer",
      "Description": "User that places an offer on an asset"
    },
    {
      "Name": "Inspector",
      "Description": "User that inspects the asset and signs off on inspection"
    },
    {
      "Name": "Owner",
      "Description": "User that signs off on the asset price"
    }
  ],
  "Workflows": [
    {
      "Name": "AssetTransfer",
      "DisplayName": "Asset Transfer",
      "Description": "Handles the business logic for the asset transfer scenario",
      "Initiators": [ "Owner" ],
      "StartState":  "Active",
      "Properties": [
        {
          "Name": "State",
          "DisplayName": "State",
          "Description": "Holds the state of the contract",
          "Type": {
            "Name": "state"
          }
        },
        {
          "Name": "Description",
          "DisplayName": "Description",
          "Description": "Describes the asset being sold",
          "Type": {
            "Name": "string"
          }
        },
        {
          "Name": "AskingPrice",
          "DisplayName": "Asking Price",
          "Description": "The asking price for the asset",
          "Type": {
            "Name": "money"
          }
        },
        {
          "Name": "OfferPrice",
          "DisplayName": "Offer Price",
          "Description": "The price being offered for the asset",
          "Type": {
            "Name": "money"
          }
        },
        {
          "Name": "InstanceAppraiser",
          "DisplayName": "Instance Appraiser",
          "Description": "The user that appraises the asset",
          "Type": {
            "Name": "Appraiser"
          }
        },
        {
          "Name": "InstanceBuyer",
          "DisplayName": "Instance Buyer",
          "Description": "The user that places an offer for this asset",
          "Type": {
            "Name": "Buyer"
          }
        },
        {
          "Name": "InstanceInspector",
          "DisplayName": "Instance Inspector",
          "Description": "The user that inspects this asset",
          "Type": {
            "Name": "Inspector"
          }
        },
        {
          "Name": "InstanceOwner",
          "DisplayName": "Instance Owner",
          "Description": "The seller of this particular asset",
          "Type": {
            "Name": "Owner"
          }
        }
      ],
      "Constructor": {
        "Parameters": [
          {
            "Name": "description",
            "Description": "The description of this asset",
            "DisplayName": "Description",
            "Type": {
              "Name": "string"
            }
          },
          {
            "Name": "price",
            "Description": "The price of this asset",
            "DisplayName": "Price",
            "Type": {
              "Name": "money"
            }
          }
        ]
      },
      "Functions": [
        {
          "Name": "Modify",
          "DisplayName": "Modify",
          "Description": "Modify the description/price attributes of this asset transfer instance",
          "Parameters": [
            {
              "Name": "description",
              "Description": "The new description of the asset",
              "DisplayName": "Description",
              "Type": {
                "Name": "string"
              }
            },
            {
              "Name": "price",
              "Description": "The new price of the asset",
              "DisplayName": "Price",
              "Type": {
                "Name": "money"
              }
            }
          ]
        },
        {
          "Name": "Terminate",
          "DisplayName": "Terminate",
          "Description": "Used to cancel this particular instance of asset transfer",
          "Parameters": []
        },
        {
          "Name": "MakeOffer",
          "DisplayName": "Make Offer",
          "Description": "Place an offer for this asset",
          "Parameters": [
            {
              "Name": "inspector",
              "Description": "Specify a user to inspect this asset",
              "DisplayName": "Inspector",
              "Type": {
                "Name": "Inspector"
              }
            },
            {
              "Name": "appraiser",
              "Description": "Specify a user to appraise this asset",
              "DisplayName": "Appraiser",
              "Type": {
                "Name": "Appraiser"
              }
            },
            {
              "Name": "offerPrice",
              "Description": "Specify your offer price for this asset",
              "DisplayName": "Offer Price",
              "Type": {
                "Name": "money"
              }
            }
          ]
        },
        {
          "Name": "Reject",
          "DisplayName": "Reject",
          "Description": "Reject the user's offer",
          "Parameters": []
        },
        {
          "Name": "AcceptOffer",
          "DisplayName": "Accept Offer",
          "Description": "Accept the user's offer",
          "Parameters": []
        },
        {
          "Name": "RescindOffer",
          "DisplayName": "Rescind Offer",
          "Description": "Rescind your placed offer",
          "Parameters": []
        },
        {
          "Name": "ModifyOffer",
          "DisplayName": "Modify Offer",
          "Description": "Modify the price of your placed offer",
          "Parameters": [
            {
              "Name": "offerPrice",
              "DisplayName": "Price",
              "Type": {
                "Name": "money"
              }
            }
          ]
        },
        {
          "Name": "Accept",
          "DisplayName": "Accept",
          "Description": "Accept the inspection/appraisal results",
          "Parameters": []
        },
        {
          "Name": "MarkInspected",
          "DisplayName": "Mark Inspected",
          "Description": "Mark the asset as inspected",
          "Parameters": []
        },
        {
          "Name": "MarkAppraised",
          "DisplayName": "Mark Appraised",
          "Description": "Mark the asset as appraised",
          "Parameters": []
        }
      ],
      "States": [
        {
          "Name": "Active",
          "DisplayName": "Active",
          "Description": "The initial state of the asset transfer workflow",
          "PercentComplete": 20,
          "Style": "Success",
          "Transitions": [
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceOwner" ],
              "Description": "Cancels this instance of asset transfer",
              "Function": "Terminate",
              "NextStates": [ "Terminated" ],
              "DisplayName": "Terminate Offer"
            },
            {
              "AllowedRoles": [ "Buyer" ],
              "AllowedInstanceRoles": [],
              "Description": "Make an offer for this asset",
              "Function": "MakeOffer",
              "NextStates": [ "OfferPlaced" ],
              "DisplayName": "Make Offer"
            },
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceOwner" ],
              "Description": "Modify attributes of this asset transfer instance",
              "Function": "Modify",
              "NextStates": [ "Active" ],
              "DisplayName": "Modify"
            }
          ]
        },
        {
          "Name": "OfferPlaced",
          "DisplayName": "Offer Placed",
          "Description": "Offer has been placed for the asset",
          "PercentComplete": 30,
          "Style": "Success",
          "Transitions": [
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceOwner" ],
              "Description": "Accept the proposed offer for the asset",
              "Function": "AcceptOffer",
              "NextStates": [ "PendingInspection" ],
              "DisplayName": "Accept Offer"
            },
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceOwner" ],
              "Description": "Reject the proposed offer for the asset",
              "Function": "Reject",
              "NextStates": [ "Active" ],
              "DisplayName": "Reject"
            },
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceOwner" ],
              "Description": "Cancel this instance of asset transfer",
              "Function": "Terminate",
              "NextStates": [ "Terminated" ],
              "DisplayName": "Terminate"
            },
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceBuyer" ],
              "Description": "Rescind the offer you previously placed for this asset",
              "Function": "RescindOffer",
              "NextStates": [ "Active" ],
              "DisplayName": "Rescind Offer"
            },
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceBuyer" ],
              "Description": "Modify the price that you specified for your offer",
              "Function": "ModifyOffer",
              "NextStates": [ "OfferPlaced" ],
              "DisplayName": "Modify Offer"
            }
          ]
        },
        {
          "Name": "PendingInspection",
          "DisplayName": "Pending Inspection",
          "Description": "Asset is pending inspection",
          "PercentComplete": 40,
          "Style": "Success",
          "Transitions": [
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceOwner" ],
              "Description": "Reject the offer",
              "Function": "Reject",
              "NextStates": [ "Active" ],
              "DisplayName": "Reject"
            },
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceOwner" ],
              "Description": "Cancel the offer",
              "Function": "Terminate",
              "NextStates": [ "Terminated" ],
              "DisplayName": "Terminate"
            },
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceBuyer" ],
              "Description": "Rescind the offer you placed for this asset",
              "Function": "RescindOffer",
              "NextStates": [ "Active" ],
              "DisplayName": "Rescind Offer"
            },
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceInspector" ],
              "Description": "Mark this asset as inspected",
              "Function": "MarkInspected",
              "NextStates": [ "Inspected" ],
              "DisplayName": "Mark Inspected"
            },
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceAppraiser" ],
              "Description": "Mark this asset as appraised",
              "Function": "MarkAppraised",
              "NextStates": [ "Appraised" ],
              "DisplayName": "Mark Appraised"
            }
          ]
        },
        {
          "Name": "Inspected",
          "DisplayName": "Inspected",
          "PercentComplete": 45,
          "Style": "Success",
          "Transitions": [
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceOwner" ],
              "Description": "Reject the offer",
              "Function": "Reject",
              "NextStates": [ "Active" ],
              "DisplayName": "Reject"
            },
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceOwner" ],
              "Description": "Cancel the offer",
              "Function": "Terminate",
              "NextStates": [ "Terminated" ],
              "DisplayName": "Terminate"
            },
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceBuyer" ],
              "Description": "Rescind the offer you placed for this asset",
              "Function": "RescindOffer",
              "NextStates": [ "Active" ],
              "DisplayName": "Rescind Offer"
            },
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceAppraiser" ],
              "Description": "Mark this asset as appraised",
              "Function": "MarkAppraised",
              "NextStates": [ "NotionalAcceptance" ],
              "DisplayName": "Mark Appraised"
            }
          ]
        },
        {
          "Name": "Appraised",
          "DisplayName": "Appraised",
          "Description": "Asset has been appraised, now awaiting inspection",
          "PercentComplete": 45,
          "Style": "Success",
          "Transitions": [
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceOwner" ],
              "Description": "Reject the offer",
              "Function": "Reject",
              "NextStates": [ "Active" ],
              "DisplayName": "Reject"
            },
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceOwner" ],
              "Description": "Cancel the offer",
              "Function": "Terminate",
              "NextStates": [ "Terminated" ],
              "DisplayName": "Terminate"
            },
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceBuyer" ],
              "Description": "Rescind the offer you placed for this asset",
              "Function": "RescindOffer",
              "NextStates": [ "Active" ],
              "DisplayName": "Rescind Offer"
            },
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceInspector" ],
              "Description": "Mark the asset as inspected",
              "Function": "MarkInspected",
              "NextStates": [ "NotionalAcceptance" ],
              "DisplayName": "Mark Inspected"
            }
          ]
        },
        {
          "Name": "NotionalAcceptance",
          "DisplayName": "Notional Acceptance",
          "Description": "Asset has been inspected and appraised, awaiting final sign-off from buyer and seller",
          "PercentComplete": 50,
          "Style": "Success",
          "Transitions": [
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceOwner" ],
              "Description": "Sign-off on inspection and appraisal",
              "Function": "Accept",
              "NextStates": [ "SellerAccepted" ],
              "DisplayName": "SellerAccept"
            },
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceOwner" ],
              "Description": "Reject the proposed offer for the asset",
              "Function": "Reject",
              "NextStates": [ "Active" ],
              "DisplayName": "Reject"
            },
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceOwner" ],
              "Description": "Cancel this instance of asset transfer",
              "Function": "Terminate",
              "NextStates": [ "Terminated" ],
              "DisplayName": "Terminate"
            },
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceBuyer" ],
              "Description": "Sign-off on inspection and appraisal",
              "Function": "Accept",
              "NextStates": [ "BuyerAccepted" ],
              "DisplayName": "BuyerAccept"
            },
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceBuyer" ],
              "Description": "Rescind the offer you placed for this asset",
              "Function": "RescindOffer",
              "NextStates": [ "Active" ],
              "DisplayName": "Rescind Offer"
            }
          ]
        },
        {
          "Name": "BuyerAccepted",
          "DisplayName": "Buyer Accepted",
          "Description": "Buyer has signed-off on inspection and appraisal",
          "PercentComplete": 75,
          "Style": "Success",
          "Transitions": [
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceOwner" ],
              "Description": "Sign-off on inspection and appraisal",
              "Function": "Accept",
              "NextStates": [ "SellerAccepted" ],
              "DisplayName": "Accept"
            },
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceOwner" ],
              "Description": "Reject the proposed offer for the asset",
              "Function": "Reject",
              "NextStates": [ "Active" ],
              "DisplayName": "Reject"
            },
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceOwner" ],
              "Description": "Cancel this instance of asset transfer",
              "Function": "Terminate",
              "NextStates": [ "Terminated" ],
              "DisplayName": "Terminate"
            }
          ]
        },
        {
          "Name": "SellerAccepted",
          "DisplayName": "Seller Accepted",
          "Description": "Seller has signed-off on inspection and appraisal",
          "PercentComplete": 75,
          "Style": "Success",
          "Transitions": [
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceBuyer" ],
              "Description": "Sign-off on inspection and appraisal",
              "Function": "Accept",
              "NextStates": [ "Accepted" ],
              "DisplayName": "Accept"
            },
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceBuyer" ],
              "Description": "Rescind the offer you placed for this asset",
              "Function": "RescindOffer",
              "NextStates": [ "Active" ],
              "DisplayName": "Rescind Offer"
            }
          ]
        },
        {
          "Name": "Accepted",
          "DisplayName": "Accepted",
          "Description": "Asset transfer process is complete",
          "PercentComplete": 100,
          "Style": "Success",
          "Transitions": []
        },
        {
          "Name": "Terminated",
          "DisplayName": "Terminated",
          "Description": "Asset transfer has been cancelled",
          "PercentComplete": 100,
          "Style": "Failure",
          "Transitions": []
        }
      ]
    }
  ]
}
```
## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [Azure Blockchain Workbench REST API 參考](https://docs.microsoft.com/rest/api/azure-blockchain-workbench) (英文)

