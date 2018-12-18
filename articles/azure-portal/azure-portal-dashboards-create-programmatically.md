---
title: 以程式設計方式建立 Azure 儀表板 | Microsoft Docs
description: 本文說明如何以程式設計方式建立 Azure 儀表板。
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
ms.openlocfilehash: 8ac3bb2c95420eb4a608f003f3d937e3a47c272b
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39448225"
---
# <a name="programmatically-create-azure-dashboards"></a>以程式設計方式建立 Azure Dashboards

本文會逐步解說以程式設計方式建立和發佈 Azure 儀表板的流程。 文件全程參照以下所示的儀表板。

![範例儀表板](./media/azure-portal-dashboards-create-programmatically/sample-dashboard.png)

## <a name="overview"></a>概觀

Azure 中的共用儀表板屬於[資源](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-overview)，如同虛擬機器和儲存體帳戶。  因此，透過 [Azure Resource Manager REST API](/rest/api/)、[Azure CLI](https://docs.microsoft.com/cli/azure)、[Azure PowerShell 命令](https://docs.microsoft.com/powershell/azure/get-started-azureps?view=azurermps-4.2.0)，以及以這些 API 為基礎的許多 [Azure 入口網站](https://portal.azure.com)，都能夠以程式設計方式管理這些儀表板，因此更容易管理資源。  

這些 API 和工具都能夠建立、列出、擷取、修改和刪除資源。  由於儀表板屬於資源，因此可以挑選自己喜愛的 API/工具來使用。

無論使用哪種工具，在您呼叫任何資源建立 API 前，都需要建構儀表板物件的 JSON 表示法。 此物件包含儀表板組件 (也稱為 圖格) 的資訊。 包含大小、位置、繫結的目的地資源，以及任何使用者自訂的資訊。

若要建置此 JSON 文件，最實際的方式是使用[入口網站](https://portal.azure.com/)以互動方式新增和放置圖格。 接著即可匯出 JSON。 最後可以從結果建立範本，以便稍後在指令碼、程式和部署工具中使用。

## <a name="create-a-dashboard"></a>建立儀表板

若要建立新的儀表板，請使用入口網站主畫面上的新儀表板命令。

![新儀表板命令](./media/azure-portal-dashboards-create-programmatically/new-dashboard-command.png)

接下來可以使用圖格資源庫尋找和新增圖格。 透過拖曳和置放即可新增圖格。 有些圖格支援透過拖曳控點調整大小，而其他則支援可在快顯功能表中呈現的固定大小。

### <a name="drag-handle"></a>拖曳控點
![拖曳控點](./media/azure-portal-dashboards-create-programmatically/drag-handle.png)

### <a name="fixed-sizes-via-context-menu"></a>透過操作功能表進行的固定大小
![大小內容功能表](./media/azure-portal-dashboards-create-programmatically/sizes-context-menu.png)

## <a name="share-the-dashboard"></a>共用儀表板

按照所需的方式設定儀表板之後，下一個步驟是發佈儀表板 (使用「共用」命令)，然後使用資源總管擷取 JSON。

![共用命令](./media/azure-portal-dashboards-create-programmatically/share-command.png)

按一下「共用」命令即會顯示對話方塊，要求您選擇要發佈的目的地訂用帳戶和資源群組。 請注意，對於您選擇的訂用帳戶和資源群組，[您必須擁有寫入權限](https://docs.microsoft.com/azure/role-based-access-control/role-assignments-portal)。

![共用和存取](./media/azure-portal-dashboards-create-programmatically/sharing-and-access.png)

## <a name="fetch-the-json-representation-of-the-dashboard"></a>擷取儀表板的 JSON 表示法

發佈只需要幾秒鐘。  完成後，下一個步驟是移至[資源總管](https://portal.azure.com/#blade/HubsExtension/ArmExplorerBlade)擷取 JSON。

![瀏覽資源總管](./media/azure-portal-dashboards-create-programmatically/browse-resource-explorer.png)

請從資源總管中，瀏覽至您選擇的訂用帳戶和資源群組。 接著，按一下新發行的儀表板資源來顯示 JSON。

![資源總管 json](./media/azure-portal-dashboards-create-programmatically/resource-explorer-json.png)

## <a name="create-a-template-from-the-json"></a>從 JSON 建立範本

下一個步驟是從這個 JSON 建立範本，如此便能使用合適的資源管理 API、命令列工具或在入口網站中，以程式設計方式重複使用。

建立範本不需要完全了解儀表板 JSON 結構。 在大部分情況下，您會想要保留每個圖格的架構和組態，然後將圖格指向的 Azure 資源集合參數化。 請查看您匯出的 JSON 儀表板，並尋找所有出現的 Azure 資源識別碼。 我們的範例儀表板有多個圖格全部指向單一 Azure 虛擬機器。 這是因為該儀表板僅查看此單一資源。 如果您搜尋 “/subscriptions” 的範例 json (附於文件結尾)，您會發現此識別碼多次出現。

`/subscriptions/6531c8c8-df32-4254-d717-b6e983273e5d/resourceGroups/contoso/providers/Microsoft.Compute/virtualMachines/myVM1`

未來若要對於任何虛擬機器發佈此儀表板，您需要將 JSON 中每次出現的此字串參數化。 

有兩種 API 可在 Azure 中建立資源。 [命令式 API](https://docs.microsoft.com/rest/api/resources/resources) 每次會建立一個資源，[範本型部署](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-template-deploy)系統可透過單一 API 呼叫來協調建立多個相依的資源。 後者本身支援參數化和範本化，因此在此範例中使用後者。

## <a name="programmatically-create-a-dashboard-from-your-template-using-a-template-deployment"></a>以程式設計方式使用範本部署從範本建立儀表板

Azure 可讓您協調多個資源的部署。 您可建立部署範本表示要部署的資源集合，以及這些資源之間的關聯性。  每個資源的 JSON 格式都相同，就如同您逐一建立這些資源一般。 差異在於，[範本語言](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-authoring-templates)會加入一些概念，例如變數、參數、基本功能等等。 此擴充語法只在範本部署的內容中受支援，如果搭配稍早討論的命令式 API 則無法運作。

如果您採用此方式，則應該使用範本的參數語法執行參數化。  您可取代我們先前找到全部出現的資源識別碼，如下所示。

### <a name="example-json-property-with-hard-coded-resource-id"></a>使用硬式編碼資源識別碼的 JSON 屬性範例
`id: "/subscriptions/6531c8c8-df32-4254-d717-b6e983273e5d/resourceGroups/contoso/providers/Microsoft.Compute/virtualMachines/myVM1"`

### <a name="example-json-property-converted-to-a-parameterized-version-based-on-template-parameters"></a>按照範本參數轉換為參數化版本的 JSON 屬性範例

`id: "[resourceId(parameters('virtualMachineResourceGroup'), 'Microsoft.Compute/virtualMachines', parameters('virtualMachineName'))]"`

在 json 範本的頂端，您也需要宣告一些所需的範本中繼資料和參數，如下所示：

```json

{
    "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "virtualMachineName": {
            "type": "string"
        },
        "virtualMachineResourceGroup": {
            "type": "string"
        },
        "dashboardName": {
            "type": "string"
        }
    },
    "variables": {},

    ... rest of template omitted ...
```

__這份文件的結尾有提供完整的可行範本。__

您製作範本後，可以使用 [REST API](https://docs.microsoft.com/rest/api/resources/deployments)、[PowerShell](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-template-deploy)、[Azure CLI](https://docs.microsoft.com/cli/azure/group/deployment#az-group-deployment-create) 或[入口網站的範本部署頁面](https://portal.azure.com/#create/Microsoft.Template)部署範本。

以下是範例儀表板 JSON 的兩個版本。 第一個是我們從已經繫結至資源的入口網站匯出的版本。 第二個是能夠以程式設計方式繫結至任何 VM 並使用 Azure Resource Manager 部署的範本版本。

## <a name="json-representation-of-our-example-dashboard-before-templating"></a>範例儀表板的 JSON 表示法 (範本化之前)

如果您依照先前的指示擷取已部署的儀表板本身的 JSON 表示法，應該會看到這種情況。 請注意，顯示此儀表板的硬式編碼資源識別碼指向特定的 Azure 虛擬機器。

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
        "metadata": { }
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

### <a name="template-representation-of-our-example-dashboard"></a>範例儀表板的範本表示法

儀表板的範本版本定義稱為 __virtualMachineName__、__virtualMachineResourceGroup__ 和 __dashboardName__ 的三個參數。  這些參數可讓您在每次部署時，將此儀表板指向不同的 Azure 虛擬機器。 參數化識別碼會醒目呈現，顯示能夠以程式設計方式設和部署此儀表板定，以便指向任何 Azure 虛擬機器。 若要測試此功能，最簡單的方式是複製下列範本，並在 [Azure 入口網站的範本部署頁面](https://portal.azure.com/#create/Microsoft.Template)貼上。 

此範例是自行部署儀表板，但是範本語言可讓您部署多個資源，並組合一個或多個儀表板。 

```json
{
    "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "virtualMachineName": {
            "type": "string"
        },
        "virtualMachineResourceGroup": {
            "type": "string"
        },
        "dashboardName": {
            "type": "string"
        }
    },
    "variables": {},
    "resources": [
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
                                                "id": "[resourceId(parameters('virtualMachineResourceGroup'), 'Microsoft.Compute/virtualMachines', parameters('virtualMachineName'))]",
                                                "chartType": 0,
                                                "metrics": [
                                                    {
                                                        "name": "Percentage CPU",
                                                        "resourceId": "[resourceId(parameters('virtualMachineResourceGroup'), 'Microsoft.Compute/virtualMachines', parameters('virtualMachineName'))]"
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
                                                "id": "[resourceId(parameters('virtualMachineResourceGroup'), 'Microsoft.Compute/virtualMachines', parameters('virtualMachineName'))]",
                                                "chartType": 0,
                                                "metrics": [
                                                    {
                                                        "name": "Disk Read Operations/Sec",
                                                        "resourceId": "[resourceId(parameters('virtualMachineResourceGroup'), 'Microsoft.Compute/virtualMachines', parameters('virtualMachineName'))]"
                                                    },
                                                    {
                                                        "name": "Disk Write Operations/Sec",
                                                        "resourceId": "[resourceId(parameters('virtualMachineResourceGroup'), 'Microsoft.Compute/virtualMachines', parameters('virtualMachineName'))]"
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
                                                "id": "[resourceId(parameters('virtualMachineResourceGroup'), 'Microsoft.Compute/virtualMachines', parameters('virtualMachineName'))]",
                                                "chartType": 0,
                                                "metrics": [
                                                    {
                                                        "name": "Disk Read Bytes",
                                                        "resourceId": "[resourceId(parameters('virtualMachineResourceGroup'), 'Microsoft.Compute/virtualMachines', parameters('virtualMachineName'))]"
                                                    },
                                                    {
                                                        "name": "Disk Write Bytes",
                                                        "resourceId": "[resourceId(parameters('virtualMachineResourceGroup'), 'Microsoft.Compute/virtualMachines', parameters('virtualMachineName'))]"
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
                                                "id": "[resourceId(parameters('virtualMachineResourceGroup'), 'Microsoft.Compute/virtualMachines', parameters('virtualMachineName'))]",
                                                "chartType": 0,
                                                "metrics": [
                                                    {
                                                        "name": "Network In",
                                                        "resourceId": "[resourceId(parameters('virtualMachineResourceGroup'), 'Microsoft.Compute/virtualMachines', parameters('virtualMachineName'))]"
                                                    },
                                                    {
                                                        "name": "Network Out",
                                                        "resourceId": "[resourceId(parameters('virtualMachineResourceGroup'), 'Microsoft.Compute/virtualMachines', parameters('virtualMachineName'))]"
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
                                            "value": "[resourceId(parameters('virtualMachineResourceGroup'), 'Microsoft.Compute/virtualMachines', parameters('virtualMachineName'))]"
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
                }
            },
            "metadata": { },
            "apiVersion": "2015-08-01-preview",
            "type": "Microsoft.Portal/dashboards",
            "name": "[parameters('dashboardName')]",
            "location": "westus",
            "tags": {
                "hidden-title": "[parameters('dashboardName')]"
            }
        }
    ]
}


```
