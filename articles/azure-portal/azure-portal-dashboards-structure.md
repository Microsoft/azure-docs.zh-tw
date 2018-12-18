---
title: Azure 儀表板結構 | Microsoft Docs
description: 本文說明 Azure 儀表板的 JSON 結構
services: azure-portal
documentationcenter: ''
author: adamabmsft
manager: dougeby
editor: tysonn
ms.service: azure-portal
ms.devlang: NA
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: na
ms.date: 09/01/2017
ms.author: adamab
ms.openlocfilehash: 2eb9289957968db04b78087413fb9df8ed1b085b
ms.sourcegitcommit: ea5193f0729e85e2ddb11bb6d4516958510fd14c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/21/2018
ms.locfileid: "36301667"
---
# <a name="the-structure-of-azure-dashboards"></a>Azure 儀表板結構
此文件將使用下列儀表板做為範例，逐步說明 Azure 儀表板的結構：

![範例儀表板](./media/azure-portal-dashboards-structure/sample-dashboard.png)

因為共用 [Azure 儀表板屬於資源](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-overview)，所以此儀表板能夠以 JSON 的形式呈現。  以下 JSON 所呈現的便是上方視覺化的儀表板。

```json

{
    "properties": {
        "lenses": {
            "0": {
                "order": 0,
                "parts": {
                    "0": {
                        "position": {
                            "x": 0,
                            "y": 0,
                            "rowSpan": 2,
                            "colSpan": 3
                        },
                        "metadata": {
                            "inputs": [],
                            "type": "Extension[azure]/HubsExtension/PartType/MarkdownPart",
                            "settings": {
                                "content": {
                                    "settings": {
                                        "content": "## Azure Virtual Machines Overview\r\nNew team members should watch this video to get familiar with Azure Virtual Machines.",
                                        "title": "",
                                        "subtitle": ""
                                    }
                                }
                            }
                        }
                    },
                    "1": {
                        "position": {
                            "x": 3,
                            "y": 0,
                            "rowSpan": 4,
                            "colSpan": 8
                        },
                        "metadata": {
                            "inputs": [],
                            "type": "Extension[azure]/HubsExtension/PartType/MarkdownPart",
                            "settings": {
                                "content": {
                                    "settings": {
                                        "content": "This is the team dashboard for the test VM we use on our team. Here are some useful links:\r\n\r\n1. [Getting started](https://www.contoso.com/tsgs)\r\n1. [Troubleshooting guide](https://www.contoso.com/tsgs)\r\n1. [Architecture docs](https://www.contoso.com/tsgs)",
                                        "title": "Test VM Dashboard",
                                        "subtitle": "Contoso"
                                    }
                                }
                            }
                        }
                    },
                    "2": {
                        "position": {
                            "x": 0,
                            "y": 2,
                            "rowSpan": 2,
                            "colSpan": 3
                        },
                        "metadata": {
                            "inputs": [],
                            "type": "Extension[azure]/HubsExtension/PartType/VideoPart",
                            "settings": {
                                "content": {
                                    "settings": {
                                        "title": "",
                                        "subtitle": "",
                                        "src": "https://www.youtube.com/watch?v=YcylDIiKaSU&list=PLLasX02E8BPCsnETz0XAMfpLR1LIBqpgs&index=4",
                                        "autoplay": false
                                    }
                                }
                            }
                        }
                    },
                    "3": {
                        "position": {
                            "x": 0,
                            "y": 4,
                            "rowSpan": 3,
                            "colSpan": 11
                        },
                        "metadata": {
                            "inputs": [
                                {
                                    "name": "queryInputs",
                                    "value": {
                                        "timespan": {
                                            "duration": "PT1H",
                                            "start": null,
                                            "end": null
                                        },
                                        "id": "/subscriptions/6531c8c8-df32-4254-d717-b6e983273e5d/resourceGroups/contoso/providers/Microsoft.Compute/virtualMachines/myVM1",
                                        "chartType": 0,
                                        "metrics": [
                                            {
                                                "name": "Percentage CPU",
                                                "resourceId": "/subscriptions/6531c8c8-df32-4254-d717-b6e983273e5d/resourceGroups/contoso/providers/Microsoft.Compute/virtualMachines/myVM1"
                                            }
                                        ]
                                    }
                                }
                            ],
                            "type": "Extension/Microsoft_Azure_Monitoring/PartType/MetricsChartPart",
                            "settings": {}
                        }
                    },
                    "4": {
                        "position": {
                            "x": 0,
                            "y": 7,
                            "rowSpan": 2,
                            "colSpan": 3
                        },
                        "metadata": {
                            "inputs": [
                                {
                                    "name": "queryInputs",
                                    "value": {
                                        "timespan": {
                                            "duration": "PT1H",
                                            "start": null,
                                            "end": null
                                        },
                                        "id": "/subscriptions/6531c8c8-df32-4254-d717-b6e983273e5d/resourceGroups/contoso/providers/Microsoft.Compute/virtualMachines/myVM1",
                                        "chartType": 0,
                                        "metrics": [
                                            {
                                                "name": "Disk Read Operations/Sec",
                                                "resourceId": "/subscriptions/6531c8c8-df32-4254-d717-b6e983273e5d/resourceGroups/contoso/providers/Microsoft.Compute/virtualMachines/myVM1"
                                            },
                                            {
                                                "name": "Disk Write Operations/Sec",
                                                "resourceId": "/subscriptions/6531c8c8-df32-4254-d717-b6e983273e5d/resourceGroups/contoso/providers/Microsoft.Compute/virtualMachines/myVM1"
                                            }
                                        ]
                                    }
                                }
                            ],
                            "type": "Extension/Microsoft_Azure_Monitoring/PartType/MetricsChartPart",
                            "settings": {}
                        }
                    },
                    "5": {
                        "position": {
                            "x": 3,
                            "y": 7,
                            "rowSpan": 2,
                            "colSpan": 3
                        },
                        "metadata": {
                            "inputs": [
                                {
                                    "name": "queryInputs",
                                    "value": {
                                        "timespan": {
                                            "duration": "PT1H",
                                            "start": null,
                                            "end": null
                                        },
                                        "id": "/subscriptions/6531c8c8-df32-4254-d717-b6e983273e5d/resourceGroups/contoso/providers/Microsoft.Compute/virtualMachines/myVM1",
                                        "chartType": 0,
                                        "metrics": [
                                            {
                                                "name": "Disk Read Bytes",
                                                "resourceId": "/subscriptions/6531c8c8-df32-4254-d717-b6e983273e5d/resourceGroups/contoso/providers/Microsoft.Compute/virtualMachines/myVM1"
                                            },
                                            {
                                                "name": "Disk Write Bytes",
                                                "resourceId": "/subscriptions/6531c8c8-df32-4254-d717-b6e983273e5d/resourceGroups/contoso/providers/Microsoft.Compute/virtualMachines/myVM1"
                                            }
                                        ]
                                    }
                                }
                            ],
                            "type": "Extension/Microsoft_Azure_Monitoring/PartType/MetricsChartPart",
                            "settings": {}
                        }
                    },
                    "6": {
                        "position": {
                            "x": 6,
                            "y": 7,
                            "rowSpan": 2,
                            "colSpan": 3
                        },
                        "metadata": {
                            "inputs": [
                                {
                                    "name": "queryInputs",
                                    "value": {
                                        "timespan": {
                                            "duration": "PT1H",
                                            "start": null,
                                            "end": null
                                        },
                                        "id": "/subscriptions/6531c8c8-df32-4254-d717-b6e983273e5d/resourceGroups/contoso/providers/Microsoft.Compute/virtualMachines/myVM1",
                                        "chartType": 0,
                                        "metrics": [
                                            {
                                                "name": "Network In",
                                                "resourceId": "/subscriptions/6531c8c8-df32-4254-d717-b6e983273e5d/resourceGroups/contoso/providers/Microsoft.Compute/virtualMachines/myVM1"
                                            },
                                            {
                                                "name": "Network Out",
                                                "resourceId": "/subscriptions/6531c8c8-df32-4254-d717-b6e983273e5d/resourceGroups/contoso/providers/Microsoft.Compute/virtualMachines/myVM1"
                                            }
                                        ]
                                    }
                                }
                            ],
                            "type": "Extension/Microsoft_Azure_Monitoring/PartType/MetricsChartPart",
                            "settings": {}
                        }
                    },
                    "7": {
                        "position": {
                            "x": 9,
                            "y": 7,
                            "rowSpan": 2,
                            "colSpan": 2
                        },
                        "metadata": {
                            "inputs": [
                                {
                                    "name": "id",
                                    "value": "/subscriptions/6531c8c8-df32-4254-d717-b6e983273e5d/resourceGroups/contoso/providers/Microsoft.Compute/virtualMachines/myVM1"
                                }
                            ],
                            "type": "Extension/Microsoft_Azure_Compute/PartType/VirtualMachinePart",
                            "asset": {
                                "idInputName": "id",
                                "type": "VirtualMachine"
                            },
                            "defaultMenuItemId": "overview"
                        }
                    }
                }
            }
        },
        "metadata": {
            "model": {
                "timeRange": {
                    "value": {
                        "relative": {
                            "duration": 24,
                            "timeUnit": 1
                        }
                    },
                    "type": "MsPortalFx.Composition.Configuration.ValueTypes.TimeRange"
                }
            }
        }
    },
    "id": "/subscriptions/6531c8c8-df32-4254-d717-b6e983273e5d/resourceGroups/dashboards/providers/Microsoft.Portal/dashboards/aa9786ae-e159-483f-b05f-1f7f767741a9",
    "name": "aa9786ae-e159-483f-b05f-1f7f767741a9",
    "type": "Microsoft.Portal/dashboards",
    "location": "eastasia",
    "tags": {
        "hidden-title": "Created via API"
    }
}

```

## <a name="common-resource-properties"></a>通用資源屬性

接下來要將 JSON 的相關區段加以細分。  最上層屬性、__識別碼__、__名稱__、__類型__、__位置__和__標籤__是所有 Azure 資源類型共同的屬性。 也就是說，這些與儀表板的內容無關。

### <a name="the-id-property"></a>識別碼屬性

Azure 資源識別碼，受限於 [Azure 資源的命名慣例](https://docs.microsoft.com/azure/architecture/best-practices/naming-conventions)。 入口網站建立儀表板時，通常會選擇 guid 形式的識別碼，但是以程式設計方式建立儀表板時，可以隨意使用任何有效的名稱。 

### <a name="the-name-property"></a>名稱屬性
名稱是資源識別碼的區段，不包含訂用帳戶、資源類型或資源群組資訊。 基本上，它是資源識別碼的最後一個區段。

### <a name="the-type-property"></a>類型屬性
所有儀表板均屬於類型 __Microsoft.Portal/dashboards__。

### <a name="the-location-property"></a>位置屬性
不同於其他資源，儀表板沒有執行階段元件。  對於儀表板，位置代表的是儲存儀表板的 JSON 表示法所在的主要地理位置。 該值應為可使用[訂用帳戶資源上的位置 API](https://docs.microsoft.com/rest/api/resources/subscriptions) 擷取到的其中一個位置代碼。

### <a name="the-tags-property"></a>標記屬性
標記是 Azure 資源的共通功能，可讓您按照任意名稱值組排列您的資源。 至於儀表板，有一個稱為__隱藏標題__的特殊標籤。 如果您的儀表板已填入這個屬性，則會使用該屬性做為儀表板在入口網站的顯示名稱。 Azure 資源識別碼無法重新命名，但是標記可以。 此標籤可讓您的儀表板有可重新命名的顯示名稱。

`"tags": { "hidden-title": "Created via API" }`

### <a name="the-properties-object"></a>屬性物件
屬性物件包含兩個屬性：__功能濾鏡__和__中繼資料__。 __功能濾鏡__屬性包含有關儀表板圖格 (也稱為 組件) 的資訊。  __中繼資料__屬性將供未來可能的功能使用。

### <a name="the-lenses-property"></a>功能濾鏡屬性
__功能濾鏡__屬性包含儀表板。 請注意，此範例中的功能濾鏡物件包含名為 "0" 的單一屬性。 功能濾鏡是儀表板中目前未實作的群組概念。 目前，所有儀表板都有在功能濾鏡物件上稱為 "0" 的這個單一屬性。

### <a name="the-lens-object"></a>功能濾鏡物件
"0" 下方的物件包含兩個屬性：__順序__和__組件__。  在目前版本的儀表板中，__順序__永遠為 0。 __組件__屬性包含定義儀表板上個別組件 (也稱為 圖格) 的資訊。

__組件__物件包含每個組件的屬性，其中的屬性名稱是數字。 這個數字並不重要。 

### <a name="the-part-object"></a>組件物件
每個個別組件物件都有__位置__和__中繼資料__。

### <a name="the-position-object"></a>位置物件
__位置__屬性包含組件的大小和位置資訊，分別以 __x__、__y__、__rowSpan__ 和 __colSpan__ 表示。 這些值會以格線單位呈現。 儀表板處於如下所示的自訂模式時，這些格線單位便會顯示。 如果您想要圖格的寬度為兩個格線單位、高度為一個格線單位、位在儀表板左上角，則位置物件如下所示：

`location: { x: 0, y: 0, rowSpan: 2, colSpan: 1 }`

![格線單位](./media/azure-portal-dashboards-structure/grid-units.png)

### <a name="the-metadata-object"></a>中繼資料物件
每個組件都有中繼資料屬性，物件只有一個稱為__類型__的必要屬性。 這個字串告知入口網站要顯示的圖格。 我們的範例儀表板使用下列類型的圖格：


1. `Extension/Microsoft_Azure_Monitoring/PartType/MetricsChartPart` – 用來顯示監視計量
1. `Extension[azure]/HubsExtension/PartType/MarkdownPart` – 用來顯示清單、連結等等的基本格式所用的文字或影像
1. `Extension[azure]/HubsExtension/PartType/VideoPart`– 用來顯示 YouTube、Channel9 和 html 視訊標記中其他任何類型的視訊提供的視訊。
1. `Extension/Microsoft_Azure_Compute/PartType/VirtualMachinePart` – 用來顯示 Azure 虛擬機器的名稱和狀態。

每個類型的組件都有本身的設定。 可能的組態屬性稱為__輸入__、__設定__和__資產__。 

### <a name="the-inputs-object"></a>輸入物件
輸入物件通常包含將圖格繫結至資源執行個體的資訊。  我們範例儀表板中的虛擬機器組件包含使用 Azure 資源識別碼表示繫結的單一輸入。  此資源識別碼格式在所有的 Azure 資源完全一致。

```json
"inputs":
[
    {
        "name": "id",
        "value": "/subscriptions/6531c8c8-df32-4254-d717-b6e983273e5d/resourceGroups/contoso/providers/Microsoft.Compute/virtualMachines/myVM1"
    }
]

```
計量圖表組件有表示要繫結目的地資源的單一輸入，以及所示計量的資訊。 以下是顯示網路輸入和網路輸出計量的圖格輸入。

```json
“inputs”:
[
    {
        "name": "queryInputs",
        "value": 
        {
            "timespan": 
            {
                "duration": "PT1H",
                "start": null,
                "end": null
           },
            "id": "/subscriptions/6531c8c8-df32-4254-d717-b6e983273e5d/resourceGroups/contoso/providers/Microsoft.Compute/virtualMachines/myVM1",
            "chartType": 0,
            "metrics": 
            [
                {
                    "name": "Network In",
                    "resourceId": "/subscriptions/6531c8c8-df32-4254-d717-b6e983273e5d/resourceGroups/contoso/providers/Microsoft.Compute/virtualMachines/myVM1"
                },
                {
                    "name": "Network Out",
                    "resourceId": "/subscriptions/6531c8c8-df32-4254-d717-b6e983273e5d/resourceGroups/contoso/providers/Microsoft.Compute/virtualMachines/myVM1"
                }
            ]
        }
    }
]

```

### <a name="the-settings-object"></a>設定物件
設定物件包含可設定的組件元素。  在我們的範例儀表板中，Markdown 組件使用儲存自訂 Markdown 內容的設定，以及可設定的標題和副標題。

```json
"settings": 
{
    "content": 
    {
        "settings": 
        {
            "content": "This is the team dashboard for the test VM we use on our team. Here are some useful links:\r\n\r\n1. [Getting started](https://www.contoso.com/tsgs)\r\n1. [Troubleshooting guide](https://www.contoso.com/tsgs)\r\n1. [Architecture docs](https://www.contoso.com/tsgs)",
            "title": "Test VM Dashboard",
            "subtitle": "Contoso"
        }
    }
}

```

同樣地，影片圖格有自己的設定，其中包含播放視訊的指標、自動播放設定，以及選用的標題資訊。

```json
"settings": 
{
   "content": 
    {
        "settings": 
        {
            "title": "",
            "subtitle": "",
            "src": "https://www.youtube.com/watch?v=YcylDIiKaSU&list=PLLasX02E8BPCsnETz0XAMfpLR1LIBqpgs&index=4",
            "autoplay": false
        }
    }
}

```

### <a name="the-asset-object"></a>資產物件
繫結至第一個可管理入口網站物件 (稱為資產) 的圖格透過資產物件表示此關聯性。  在我們的範例儀表板中，虛擬機器圖格包含此資產描述。  __IdInputName__ 屬性告知入口網站識別碼輸入包含資產的唯一識別碼，在此案例中是資源識別碼。大部分的 Azure 資源類型都有在入口網站中定義的資產。

`"asset": {    "idInputName": "id",    "type": "VirtualMachine"    }`