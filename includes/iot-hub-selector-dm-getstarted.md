> [!div class="op_single_selector"]
> * [裝置︰Node.js 服務︰Node.js](../articles/iot-hub/iot-hub-node-node-device-management-get-started.md)
> * [裝置：C# 服務：C#](../articles/iot-hub/iot-hub-csharp-csharp-device-management-get-started.md)
> * [裝置︰Java 服務︰Java](../articles/iot-hub/iot-hub-java-java-device-management-getstarted.md)
> * [Python：Python 服務：Python](../articles/iot-hub/iot-hub-python-python-device-management-get-started.md)

後端應用程式可以使用 Azure IoT 中樞基元 (例如[裝置對應項][lnk-devtwin]和[直接方法][lnk-c2dmethod])，從遠端啟動並監視裝置上的裝置管理動作。 本教學課程會示範後端應用程式和裝置應用程式如何共同運作，以使用 IoT 中樞初始化和監視遠端裝置重新啟動。

[!INCLUDE [iot-hub-basic](iot-hub-basic-whole.md)]

使用直接方法從雲端中的後端應用程式起始裝置管理動作 (例如重新啟動、恢復出廠預設值，以及韌體更新)。 該裝置將負責：

* 處理從 IoT 中樞傳送的方法要求。
* 在裝置上初始化相對應的裝置特定動作。
* 透過針對 IoT 中樞的*報告屬性*提供狀態更新。

您可以在雲端中使用後端 App 來執行裝置對應項查詢，以報告裝置管理動作的進度。

[lnk-devtwin]: ../articles/iot-hub/iot-hub-devguide-device-twins.md
[lnk-c2dmethod]: ../articles/iot-hub/iot-hub-devguide-direct-methods.md
