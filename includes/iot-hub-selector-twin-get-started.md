> [!div class="op_single_selector"]
> * [Node.js](../articles/iot-hub/iot-hub-node-node-twin-getstarted.md)
> * [C#](../articles/iot-hub/iot-hub-csharp-csharp-twin-getstarted.md)
> * [Java](../articles/iot-hub/iot-hub-java-java-twin-getstarted.md)
> * [Python](../articles/iot-hub/iot-hub-python-twin-getstarted.md)

「裝置對應項」是存放裝置狀態資訊 (中繼資料、組態和條件) 的 JSON 文件。 IoT 中樞會為其連線的每個裝置保存裝置對應項。

[!INCLUDE [iot-hub-basic](iot-hub-basic-whole.md)]

使用裝置對應項可以：

* 從您的解決方案後端儲存裝置中繼資料。
* 從您的裝置應用程式報告目前狀態資訊，例如可用功能和狀況 (例如，使用的連接方法)。
* 同步處理裝置應用程式與後端應用程式之間長時間執行之工作流程的狀態 (例如韌體和組態更新)。
* 查詢裝置中繼資料、組態或狀態。

裝置對應項是設計來進行同步處理和查詢裝置組態和條件。 如需何時使用裝置對應項的詳細資訊，請參閱[了解裝置對應項][lnk-twins]。

裝置對應項會儲存在 IoT 中樞，並且包含︰

* 標籤，只有解決方案後端可存取裝置中繼資料；
* 所需屬性，JSON 物件可由解決方案後端修改並且可由裝置應用程式觀察，以及
* 報告屬性，JSON 物件可由裝置應用程式修改並且可由解決方案後端讀取。 標籤和屬性不能包含陣列，但物件可以是巢狀的。

![][img-twin]

此外，解決方案後端可以根據上述的所有資料查詢裝置對應項。
請參閱[了解裝置對應項][lnk-twins]以取得裝置對應項的詳細資訊，以及參閱 [IoT 中樞查詢語言][lnk-query]參考以進行查詢。


本教學課程說明如何：

* 建立後端應用程式，將「標籤」新增至裝置對應項，以及建立模擬裝置應用程式，以裝置對應項上的「報告屬性」來報告其連線通道。
* 使用先前建立的標籤和屬性上的篩選器，從您的後端應用程式查詢裝置。

<!-- images -->
[img-twin]: media/iot-hub-selector-twin-get-started/twin.png

<!-- links -->
[lnk-query]: ../articles/iot-hub/iot-hub-devguide-query-language.md
[lnk-twins]: ../articles/iot-hub/iot-hub-devguide-device-twins.md
[lnk-d2c]: ../articles/iot-hub/iot-hub-devguide-messaging.md#device-to-cloud-messages
[lnk-methods]: ../articles/iot-hub/iot-hub-devguide-direct-methods.md
[lnk-devguide-mqtt]: ../articles/iot-hub/iot-hub-mqtt-support.md