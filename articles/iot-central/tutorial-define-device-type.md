---
title: 在 Azure IoT Central 中定義新的裝置類型 | Microsoft Docs
description: 本教學課程將為建置者說明如何在 Azure IoT Central 應用程式中定義新的裝置類型。 您會定義類型的遙測、狀態、屬性和設定。
author: tbhagwat3
ms.author: tanmayb
ms.date: 04/16/2018
ms.topic: tutorial
ms.service: iot-central
services: iot-central
ms.custom: mvc
manager: peterpr
ms.openlocfilehash: 246d7f837fdf72ddb2d24139d2d3d83fcd43b640
ms.sourcegitcommit: 3a02e0e8759ab3835d7c58479a05d7907a719d9c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/13/2018
ms.locfileid: "49310643"
---
# <a name="tutorial-define-a-new-device-type-in-your-azure-iot-central-application"></a>教學課程：在 Azure IoT 中心應用程式中定義新的裝置類型

本教學課程將為建置者說明如何使用裝置範本在 Microsoft Azure IoT Central 應用程式中定義新的裝置類型。 裝置範本會定義裝置類型的遙測、狀態、屬性和設定。

為了讓您能夠先測試應用程式再連接實際裝置，Azure IoT Central 會在您建立裝置範本時，從範本產生模擬裝置。

在本教學課程中，您會建立**連線的空調**裝置範本。 連線的空調裝置：

* 傳送遙測資料，例如溫度和濕度。
* 回報狀態，例如裝置處於開啟還是關閉狀態。
* 具有裝置屬性，例如裝置的韌體版本和序號。
* 具有設定，例如目標溫度。

在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 建立新的裝置範本
> * 將遙測資料新增至裝置
> * 檢視模擬的遙測資料
> * 定義事件測量
> * 檢視模擬的事件
> * 定義狀態測量
> * 檢視模擬的狀態
> * 使用設定和屬性
> * 使用命令
> * 在儀表板中檢視模擬裝置

## <a name="prerequisites"></a>必要條件

若要完成本教學課程，您必須要有 Azure IoT 中心應用程式。 如果您已完成[建立 Azure IoT Central 應用程式](quick-deploy-iot-central.md)快速入門，則可以重複使用您在該快速入門中建立的應用程式。 否則，請完成下列步驟以建立空的 Azure IoT Central 應用程式：

1. 瀏覽至 Azure IoT Central 的[應用程式管理員](https://aka.ms/iotcentral)頁面。

2. 輸入您用來存取 Azure 訂用帳戶的電子郵件地址和密碼：

   ![輸入您的組織帳戶](./media/tutorial-define-device-type/sign-in.png)

3. 若要開始建立新的 Azure IoT Central 應用程式，請選擇 [新增應用程式]：

    ![Azure IoT Central 應用程式管理員頁面](./media/tutorial-define-device-type/iotcentralhome.png)

4. 若要建立新的 Azure IoT Central 應用程式：
    
    * 選擇 [免費]。 7 天免費試用版不需要訂用帳戶。
    
       如需目錄和訂用帳戶的詳細資訊，請參閱[建立 Azure IoT Central 應用程式](howto-create-application.md)。
    
    * 選擇 [自訂應用程式]。
    
    * (選擇性) 您可以選擇易記的應用程式名稱，例如 **Contoso Air Conditioners**。 Azure IoT Central 會為您產生唯一的 URL 前置詞。 您可以將其變更為更好記的 URL 前置詞。
    
    * 選擇 [建立] 。

    ![Azure IoT Central 的建立應用程式頁面](./media/tutorial-define-device-type/iotcentralcreatenew.png)

    如需詳細資訊，請參閱[如何建立 Azure IoT Central 應用程式](howto-create-application.md)。

## <a name="create-a-new-custom-device-template"></a>建立新的自訂裝置範本

身為建置者，您可以在應用程式中建立和編輯裝置範本。 當您建立裝置範本時，Azure IoT Central 會從範本產生模擬裝置。 模擬裝置會產生遙測資料，讓您可先測試應用程式的行為，再連接實體裝置。

若要將新的裝置範本新增至應用程式，您必須移至 [應用程式建置者] 頁面。 若要這麼做，請選擇左側導覽功能表上的 [應用程式建立器]。

![應用程式建置者頁面](./media/tutorial-define-device-type/builderhome.png)

## <a name="add-a-device-and-define-telemetry"></a>新增裝置並定義遙測

下列步驟說明如何為要將溫度遙測資料傳送至應用程式的裝置建立新的**連線的空調**裝置範本：

1. 在 [應用程式建置者] 頁面上，選擇 [建立裝置範本]：

    ![應用程式建置者頁面，建立裝置範本](./media/tutorial-define-device-type/builderhomedevices.png)

2. 在 [裝置範本] 頁面上，選擇 [自訂]。 [自訂] 裝置範本可讓您為連線的空調您定義所有特性和行為：

    ![裝置](./media/tutorial-define-device-type/builderhomedevicescustom.png)

3. 在 [新增裝置範本] 頁面上，輸入**連線的空調**作為裝置名稱，然後選擇 [建立]。 您也可以上傳操作員在裝置總管中可見的裝置影像：

    ![自訂裝置](./media/tutorial-define-device-type/createcustomdevice.png)

4. 在 [連線的空調] 裝置範本中，確定您是在定義遙測資料的 [測量] 頁面上。 您所定義的每個裝置範本都有個別的頁面，以供您：

    * 指定裝置所傳送的測量，例如遙測、事件和狀態。
    
    * 定義用來控制裝置的設定。
    
    * 定義裝置中繼資料的屬性。

    * 定義要直接在裝置上執行的命令。
    
    * 定義與裝置相關聯的規則。
    
    * 自訂操作員的裝置儀表板。

    在您每次定義裝置範本時，均請先選擇 [編輯範本] 以編輯範本。 完成之後，選擇 [完成]。 

    ![空調測量](./media/tutorial-define-device-type/airconmeasurements.png)

    > [!NOTE]
    > 若要變更裝置或裝置範本的名稱，請按一下頁面頂端的文字。

5. 若要新增溫度遙測測量，請選擇 [新增測量]。 然後，選擇 [遙測] 作為測量類型：

    ![連線的空調測量](./media/tutorial-define-device-type/airconmeasurementsnew.png)

6. 您為裝置範本定義的每種遙測類型都會包含[組態選項](howto-set-up-template.md)，例如：

    * 顯示選項。

    * 遙測的詳細資料。

    * 模擬參數。

    若要設定**溫度**遙測，請使用下表中的資訊：

    | 設定              | 值         |
    | -------------------- | -----------   |
    | 顯示名稱         | 溫度   |
    | 欄位名稱           | 溫度   |
    | Units                | F             |
    | Min                  | 60            |
    | max                  | 110           |
    | 小數位數       | 0             |

    您也可以選擇遙測顯示的色彩。 若要儲存遙測定義，請選擇 [儲存]：

    ![設定溫度模擬](./media/tutorial-define-device-type/temperaturesimulation.png)

7. 不久之後，[測量] 頁面即會顯示從模擬的連線空調裝置產生的溫度遙測資料的圖表。 您可以使用控制項管理可見性和彙總，或編輯遙測定義：

    ![檢視溫度模擬](./media/tutorial-define-device-type/viewsimulation.png)

8. 您也可以使用 [列]、[堆疊] 和 [編輯時間範圍] 等控制項來自訂圖表：

    ![自訂圖表](./media/tutorial-define-device-type/customizechart.png)

## <a name="define-event-measurement"></a>定義事件測量

您可以使用事件來定義裝置所傳送的時間點資料，以指出具有重要性的事項，例如錯誤或元件失敗。 如同遙測測量，Azure IoT Central 也可模擬裝置事件，而讓您能夠先測試應用程式的行為，再連接實體裝置。 您可以在 [測量] 檢視中為您的裝置類型定義事件測量。

1. 若要新增 [風扇馬達錯誤] 事件測量，請選擇 [新增測量]。 然後，選擇 [事件] 作為測量類型：

    ![連線的空調測量](./media/tutorial-define-device-type/eventnew.png)

2. 您為裝置範本定義的每種事件類型都會包含[組態選項](howto-set-up-template.md)，例如：

   * 顯示名稱。

   * 欄位名稱。

   * 嚴重性。

    若要設定 [風扇馬達錯誤] 事件，請使用下表中的資訊：

    | 設定              | 值             |
    | -------------------- | -----------       |
    | 顯示名稱         | 風扇馬達錯誤   |
    | 欄位名稱           | fanmotorerr       |
    | 嚴重性             | Error             |

    若要儲存事件定義，請選擇 [儲存]：

    ![設定事件測量](./media/tutorial-define-device-type/eventconfiguration.png)

3. 不久之後，[測量] 頁面即會顯示從模擬的連線空調裝置隨機產生之事件的圖表。 您可以使用控制項管理可見性，或編輯事件定義：

    ![檢視事件模擬](./media/tutorial-define-device-type/eventview.png)

1. 若要檢視關於事件的其他詳細資料，請按一下圖表上的事件：

    ![檢視事件詳細資料](./media/tutorial-define-device-type/eventviewdetail.png)

## <a name="define-state-measurement"></a>定義狀態測量

您可以使用 [狀態] 來定義和視覺化裝置或其元件在一段時間內的狀態。 如同遙測測量，Azure IoT Central 也可模擬裝置狀態，而讓您能夠先測試應用程式的行為，再連接實體裝置。 您可以在 [測量] 檢視中為您的裝置類型定義狀態測量。

1. 若要新增 [風扇模式]，請選擇 [新增測量]。 然後，選擇 [狀態] 作為測量類型：

    ![連線的空調狀態測量](./media/tutorial-define-device-type/statenew.png)

2. 您為裝置範本定義的每種狀態類型都會包含[組態選項](howto-set-up-template.md)，例如：

   * 顯示名稱。

   * 欄位名稱。

   * 選用顯示標籤的值。

   * 每個值的色彩。

    若要設定 [風扇模式] 狀態，請使用下表中的資訊：

    | 設定              | 值             |
    | -------------------- | -----------       |
    | 顯示名稱         | 風扇模式          |
    | 欄位名稱           | fanmode           |
    | 值                | 1                 |
    | 顯示標籤        | 運作中         |
    | 值                | 0                 |
    | 顯示標籤        | 已停止           |

    若要儲存狀態測量定義，請選擇 [儲存]：

    ![設定狀態測量](./media/tutorial-define-device-type/stateconfiguration.png)

3. 不久之後，[測量] 頁面即會顯示從模擬的連線空調裝置隨機產生之狀態的圖表。 您可以使用控制項管理可見性，或編輯狀態定義：

    ![檢視狀態模擬](./media/tutorial-define-device-type/stateview.png)

4. 如果裝置在一小段時間內傳送太多資料點，狀態測量在顯示時就會使用不同的視覺效果，如下所示。 如果您按一下圖表，則該時段內的所有資料點將會依時間先後順序顯示。 您也可以縮小時間範圍，以查看詳細的測量。

    ![檢視狀態詳細資料](./media/tutorial-define-device-type/stateviewdetail.png)

## <a name="settings-properties-and-commands"></a>設定、屬性和命令

設定、屬性和命令是定義於裝置範本中，且與個別裝置相關聯的不同值：

* 您可以使用_設定_將組態資料從您的應用程式傳送至裝置。 例如，操作員可以使用設定將裝置的遙測間隔從兩秒變更為五秒。 當操作員變更設定時，在裝置確認它已處理設定變更之前，設定在 UI 中會標示為擱置中。

* 您可以使用_屬性_來定義與裝置與相關聯的中繼資料。 屬性分為兩種類別：
    
    * 您可以使用_應用程式屬性_來記錄與應用程式中的裝置有關的資訊。 例如，您可以使用應用程式屬性來記錄裝置的位置及其上次維修日期。 這些屬性會儲存在應用程式中，且不會與裝置同步處理。 操作員可以將值指派給屬性。

    * 您可以使用_裝置屬性_，讓裝置能夠將屬性值傳送至您的應用程式。 只有裝置可以變更這些屬性。 對操作員而言，裝置屬性是唯讀的。 在這個連線空調的案例中，裝置所報告的裝置屬性是韌體版本和裝置序號。 
    
    如需詳細資訊，請參閱裝置範本設定操作說明指南中的[屬性][lnk-define-template]。

* 您可使用 _commands_ 從應用程式遠端管理您的裝置。 您可以從雲端直接在裝置上執行命令來控制裝置。 例如，操作員可以執行命令 (例如 reboot)，以立即重新啟動裝置。

## <a name="use-settings"></a>使用設定

您可以使用*設定*，讓操作員能夠將組態資料傳送至裝置。 在本節中，您會將設定新增**連線的空調**裝置範本，讓操作員為連線的空調設定目標溫度。

1. 瀏覽至 [連線的空調] 裝置範本的 [設定] 頁面：

    ![準備新增設定](./media/tutorial-define-device-type/deviceaddsetting.png)

    您可以建立不同類型的設定，例如數值或文字。

2. 選擇 [數值]，將數值設定新增至裝置。

3. 若要設定 [設定溫度] 設定，請使用下表中的資訊：

    | 欄位                | 值           |
    | -------------------- | -----------     |
    | 顯示名稱         | 設定溫度 |
    | 欄位名稱           | setTemperature  |
    | 測量單位      | F               |
    | 小數位數       | 1               |
    | 最小值        | 20              |
    | 最大值        | 200             |
    | 初始值        | 80              |
    | 說明          | 設定空調的目標溫度 |

    然後，選擇 [儲存]：

    ![設定「設定溫度」設定](./media/tutorial-define-device-type/configuresetting.png)

    > [!NOTE]
    > 當裝置確認設定變更之後，該設定的狀態將會變更為 [已同步]。

4. 您可以藉由移動設定圖格和調整其大小，來自訂 [設定] 頁面的版面配置：

    ![自訂設定版面配置](./media/tutorial-define-device-type/settingslayout.png)

## <a name="use-properties"></a>使用屬性 

您可以使用*應用程式屬性*來儲存與應用程式中的裝置有關的資訊。 在本節中，您會將應用程式屬性新增至**連線的空調**裝置範本，以儲存裝置位置和上次維修日期。 請注意，這兩者都是可編輯的裝置屬性。 裝置也會報告不可變更的唯讀裝置屬性，例如裝置序號和韌體版本。
 
1. 瀏覽至 [連線的空調] 裝置範本的 [屬性] 頁面：

    ![準備新增屬性](./media/tutorial-define-device-type/deviceaddproperty.png)

    您可以建立不同類型裝置的屬性，例如數值或文字。 若要將位置屬性新增至裝置範本，請選擇 [位置]。

1. 若要設定位置屬性，請使用下表中的資訊：

    | 欄位                | 值                |
    | -------------------- | -------------------- |
    | 顯示名稱         | 位置             |
    | 欄位名稱           | location             |
    | 初始值        | 美國華盛頓州西雅圖          |
    | 說明          | 裝置位置      |

    讓其他欄位保留其預設值。

    ![設定裝置屬性](./media/tutorial-define-device-type/configureproperties.png)

    選擇 [儲存]。

1. 若要將上次維修日期屬性新增至裝置範本，請選擇 [日期]。

1. 若要設定上次維修日期屬性，請使用下表中的資訊：

    | 欄位                | 值                   |
    | -------------------- | ----------------------- |
    | 顯示名稱         | 上次維修日期       |
    | 欄位名稱           | serviceDate             |
    | 初始值        | 1/1/2018                |
    | 說明          | 上次維修日期           |

    ![設定裝置屬性](./media/tutorial-define-device-type/configureproperties2.png)

    選擇 [儲存]。

5. 您可以藉由移動屬性圖格和調整其大小，來自訂 [屬性] 頁面的版面配置：

    ![自訂屬性版面配置](./media/tutorial-define-device-type/propertieslayout.png)

1. 若要將裝置屬性 (例如韌體版本) 新增至您的裝置範本，請選擇 [裝置屬性]。

1.  若要設定韌體版本，請使用下表中的資訊：

    | 欄位                | 值                   |
    | -------------------- | ----------------------- |
    | 顯示名稱         | 韌體版本        |
    | 欄位名稱           | firmwareVersion         |
    | 資料類型            | text                    |
    | 說明          | 空調的韌體版本 |

    ![設定韌體版本](./media/tutorial-define-device-type/configureproperties3.png)
    
    選擇 [儲存]。

1. 若要將裝置屬性 (例如序號) 新增至您的裝置範本，請選擇 [裝置屬性]。

1. 若要設定序號，請使用下表中的資訊：

    | 欄位                | 值                   |
    | -------------------- | ----------------------- |
    | 顯示名稱         | 序號           |
    | 欄位名稱           | serialNumber            |
    | 資料類型            | text                    |
    | 說明          | 空調的序號  |

    ![設定序號](./media/tutorial-define-device-type/configureproperties4.png)
    
    選擇 [儲存]。
    
    > [!NOTE]
    > 裝置屬性會從裝置傳送至應用程式。 當實際的裝置連線到 IoT Central 時，將會更新韌體版本和序號的值。

## <a name="use-commands"></a>使用命令

您可使用 _commands_ 讓操作員在裝置上直接執行命令。 在本節中，您會將命令新增至**連線的空調**裝置範本，讓操作員能夠回應連線空調的特定訊息。

1. 瀏覽至 [連線的空調] 裝置範本的 [命令] 頁面，以編輯範本。 

1. 按一下 [新增命令] 以將命令新增至您的裝置，並開始設定新命令。

   您可以根據需求建立不同類型的命令。 

1. 若要設定新的命令，請使用下表中的資訊：

    | 欄位                | 值           |
    | -------------------- | -----------     |
    | 顯示名稱         | 回應命令    |
    | 欄位名稱           | 回應            |
    | 預設逾時      | 30              |
    | 顯示類型         | text            |
    | 說明          | 裝置命令  |  

    您可以按一下**輸入欄位**的 **+**，將其他輸入新增至命令。

    ![準備新增設定](media/tutorial-define-device-type/commandsecho1.png)

     選擇 [儲存]。

1. 您可以藉由移動命令圖格和調整其大小，來自訂 [命令] 頁面的版面配置：

    ![自訂設定版面配置](media/tutorial-define-device-type/commandstileresize1.png)

## <a name="view-your-simulated-device"></a>檢視模擬裝置

現在，您已定義**連線的空調**裝置範本，接下來可以自訂其**儀表板**，以包含您所定義的測量、設定和屬性。 然後，您可以用操作員的身分預覽儀表板：

1. 選擇 [連線的空調] 裝置範本的 [儀表板] 頁面：

    ![連線的空調儀表板](./media/tutorial-define-device-type/aircondashboards.png)

1. 選擇 [折線圖]，以在 [儀表板] 上新增元件：

    ![儀表板元件](./media/tutorial-define-device-type/dashboardcomponents1.png)

1. 使用下表中的資訊設定 [折線圖] 元件：

    | 設定      | 值       |
    | ------------ | ----------- |
    | 標題        | 溫度 |
    | 時間範圍   | 過去 30 分鐘 |
    | 量值     | 溫度 (選擇 [溫度] 旁的 [可見性]) |

    ![折線圖設定](./media/tutorial-define-device-type/linechartsettings.png)

    然後，選擇 [儲存]。

1. 使用下表中的資訊設定 [事件歷程記錄] 元件：

    | 設定      | 值       |
    | ------------ | ----------- |
    | 標題        | 活動 |
    | 時間範圍   | 過去 30 分鐘 |
    | 量值     | 風扇馬達錯誤 (選擇 [風扇馬達錯誤] 旁的 [可見性]) |

    ![折線圖設定](./media/tutorial-define-device-type/dashboardeventchartsetting.png)

    然後，選擇 [儲存]。

1. 使用下表中的資訊設定 [狀態歷程記錄] 元件：

    | 設定      | 值       |
    | ------------ | ----------- |
    | 標題        | 風扇模式 |
    | 時間範圍   | 過去 30 分鐘 |
    | 量值 | 風扇馬達 (選擇 [風扇模式] 旁的 [可見性]) |

    ![折線圖設定](./media/tutorial-define-device-type/dashboardstatechartsetting.png)

    然後，選擇 [儲存]。

1. 若要將「設定溫度」設定新增至儀表板，請選擇 [設定和屬性]。 按一下 [新增/移除]，以新增您要在儀表板中檢視的設定或屬性。 

    ![儀表板元件](./media/tutorial-define-device-type/dashboardcomponents4.png)

1. 使用下表中的資訊設定 [設定和屬性] 元件：

    | 設定                 | 值         |
    | ----------------------- | ------------- |
    | 標題                   | 設定目標溫度 |
    | 設定和屬性 | 設定溫度 |

    您先前在 [設定] 和 [屬性] 頁面上定義的設定和屬性，會顯示 [可用的資料行] 中。 

    ![設定溫度屬性設定](./media/tutorial-define-device-type/propertysettings4.png)

    然後選擇 [確定]。

1. 若要將裝置序號新增至儀表板，請選擇 [設定和屬性]：

    ![儀表板元件](./media/tutorial-define-device-type/dashboardcomponents3.png)

1. 使用下表中的資訊設定 [設定和屬性] 元件：

    | 設定                 | 值         |
    | ----------------------- | ------------- |
    | 標題                   | 序號 |
    | 設定和屬性 | 序號 |

    ![序號屬性設定](./media/tutorial-define-device-type/propertysettings5.png)

    然後選擇 [確定]。

1. 若要將裝置韌體版本新增至儀表板，請選擇 [設定和屬性]：

    ![儀表板元件](./media/tutorial-define-device-type/dashboardcomponents4.png)

1. 使用下表中的資訊設定 [設定和屬性] 元件：

    | 設定                 | 值            |
    | ----------------------- | ---------------- |
    | 標題                   | 韌體版本 |
    | 設定和屬性 | 韌體版本 |

    ![序號屬性設定](./media/tutorial-define-device-type/propertysettings6.png)

    然後選擇 [確定]。

1. 若要以操作員的身分檢視儀表板，請將頁面右上方的 [編輯範本] 切換為關閉。

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何：

<!-- Repeat task list from intro -->
> [!div class="nextstepaction"]
> * 建立新的裝置範本
> * 將遙測資料新增至裝置
> * 檢視模擬的遙測資料
> * 定義裝置事件
> * 檢視模擬的事件
> * 定義狀態
> * 檢視模擬的狀態
> * 使用設定和屬性
> * 使用命令
> * 在儀表板中檢視模擬裝置

現在，您已在 Azure IoT Central 應用程式中定義裝置範本，以下是建議的後續步驟：

* [為您的裝置設定規則和動作](tutorial-configure-rules.md)
* [自訂操作員的檢視](tutorial-customize-operator.md)

[lnk-define-template]: /azure/iot-central/howto-set-up-template#properties