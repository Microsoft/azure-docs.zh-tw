---
title: Azure IoT 中樞裝置布建服務中的重新布建裝置
description: 瞭解如何使用裝置布建服務重新布建裝置 (DPS) 實例，以及您可能需要這麼做的原因。
author: wesmc7777
ms.author: wesmc
ms.date: 01/25/2021
ms.topic: conceptual
ms.service: iot-dps
services: iot-dps
ms.openlocfilehash: d704e8f9687f3987d80018d84b41c0fd519da172
ms.sourcegitcommit: a055089dd6195fde2555b27a84ae052b668a18c7
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/26/2021
ms.locfileid: "98791891"
---
# <a name="how-to-reprovision-devices"></a>如何重新佈建裝置

在 IoT 解決方案的生命週期中，在 IoT 中樞之間移動裝置是很常見的。 本主題的撰寫是為了協助解決方案操作員設定重新布建原則。

如需更詳細的重新布建案例總覽，請參閱 [IoT 中樞裝置重新布建概念](concepts-device-reprovision.md)。


## <a name="configure-the-enrollment-allocation-policy"></a>設定註冊配置原則

配置原則會決定如何在重新佈建裝置之後，將與註冊關聯的裝置配置或指派給 IoT 中樞。

下列步驟會設定裝置註冊的配置原則：

1. 登入 [Azure 入口網站](https://portal.azure.com)，並瀏覽至您的裝置佈建服務執行個體。

2. 按一下 [管理註冊]，然後按一下想要針對重新佈建設定的註冊群組或個別註冊。 

3. 在 [選取要如何將裝置指派到中樞] 下，選取下列其中一個配置原則：

    * **最低延遲**：此原則會將裝置指派到將在裝置與 IoT 中樞之間產生最低延遲通訊的已連結 IoT 中樞。 此選項可讓裝置與位置最接近的 IoT 中樞通訊。 
    
    * **權重相等的分佈**：此原則會根據指派給每個連結的 IoT 中樞的配置權重，在所有連結的 IoT 中樞分佈裝置。 此原則可根據在一群連結的中樞中設定的配置權重，跨該連結的中樞群組進行裝置負載平衡。 如果您只要將裝置重新佈建到一個 IoT 中樞，建議使用此設定。 這是預設設定。 
    
    * **靜態** 設定：此原則需要在要布建之裝置的註冊專案中，列出所需的 IoT 中樞。 此原則可讓您指定要指派裝置的單一特定 IoT 中樞。

4. 在 [選取可指派此群組的 IoT 中樞] 下，選取要在配置原則中包含之連結的 IoT 中樞。 (選擇性) 使用 [連結新的 IoT 中樞] 按鈕新增新的連結的 IoT 中樞。

    您可以使用 [最低延遲] 配置原則，將選取的中樞包含在延遲評估中，判斷最接近的中樞以指派裝置。

    您可以使用 [權重相等的分佈] 配置原則，根據中樞中設定的配置權重和其目前的裝置負載，跨所有選取的中樞進行裝置負載平衡。

    您可以使用 [靜態設定] 配置原則，選取要指派裝置到其中的 IoT 中樞。

4. 按一下 [儲存]，或繼續下一節以設定重新佈建原則。

    ![選取註冊配置原則](./media/how-to-reprovision/enrollment-allocation-policy.png)



## <a name="set-the-reprovisioning-policy"></a>設定重新佈建原則

1. 登入 [Azure 入口網站](https://portal.azure.com)，並瀏覽至您的裝置佈建服務執行個體。

2. 按一下 [管理註冊]，然後按一下想要針對重新佈建設定的註冊群組或個別註冊。

3. 在 [選取在重新佈建至其他 IoT 中樞時如何處理裝置資料] 下，選擇下列其中一個重新佈建原則：

    * **重新佈建並移轉資料**：此原則會在與註冊項目關聯的裝置提交新的佈建要求時採取動作。 視註冊項目設定而定，裝置可能會重新指派給其他 IoT 中樞。 如果裝置所屬的 IoT 中樞有所變更，將會移除初始 IoT 中樞中的裝置註冊。 該初始 IoT 中的所有裝置狀態資訊將會移轉到新的 IoT 中樞。 移轉期間，裝置的狀態將回報為 **指派中**

    * **重新佈建並重設為初始設定**：此原則會在與註冊項目關聯的裝置提交新的佈建要求時採取動作。 視註冊項目設定而定，裝置可能會重新指派給其他 IoT 中樞。 如果裝置所屬的 IoT 中樞有所變更，將會移除初始 IoT 中樞中的裝置註冊。 系統會將佈建服務執行個體在佈建裝置時收到的初始設定資料提供給新的 IoT 中樞。 在遷移期間，裝置的狀態將會回報為 [ **指派**]。

4. 按一下 [儲存] 以根據您所做的變更重新部署裝置。

    ![醒目顯示您所做的變更和 [儲存] 按鈕的螢幕擷取畫面。](./media/how-to-reprovision/reprovisioning-policy.png)



## <a name="send-a-provisioning-request-from-the-device"></a>從裝置傳送佈建要求

若要根據前面幾節中所做的設定變更重新部署裝置，這些裝置必須要求重新部署。 

裝置提交佈建要求的頻率需視情況而定。 但是，建議將裝置設定為在重新開機時傳送佈建要求至佈建服務執行個體，並支援一種視需要手動觸發佈建的[方法](../iot-hub/iot-hub-devguide-direct-methods.md)。 也可以設定[所需屬性](../iot-hub/iot-hub-devguide-device-twins.md#desired-property-example)來觸發佈建。 

註冊項目上的重新佈建原則會決定裝置佈建服務執行個體如何處理這些佈建要求，以及是否應該在重新佈建期間移轉裝置狀態資料。 相同原則可供個別註冊與註冊群組使用：

如需在開機序列期間從裝置傳送佈建要求的範例程式碼，請參閱[自動佈建模擬裝置](quick-create-simulated-device.md)。


## <a name="next-steps"></a>後續步驟

- 若要深入瞭解重新布建，請參閱 [IoT 中樞裝置重新布建概念](concepts-device-reprovision.md) 
- 若要深入瞭解解除布建，請參閱如何取消布 [建先前自動布建的裝置](how-to-unprovision-devices.md) 











