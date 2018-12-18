## <a name="webapi-project"></a>WebAPI 專案
1. 在 Visual Studio 中，開啟您在 [通知使用者] 教學課程中所建立的 **AppBackend** 專案。
2. 在 Notifications.cs 中，使用下列程式碼來取代整個 **Notifications** 類別。 請確定使用通知中樞的連接字串 (包含完整存取權) 和中樞名稱來取代預留位置。 您可以從 [Azure 入口網站](http://portal.azure.com)取得這些值。 此模組現會顯示即將傳送的不同安全通知。 在完整的實作中，通知會儲存在資料庫中。為了本案例的方便起見，我們會將通知儲存在記憶體中。
   
        public class Notification
        {
            public int Id { get; set; }
            public string Payload { get; set; }
            public bool Read { get; set; }
        }

        public class Notifications
        {
            public static Notifications Instance = new Notifications();

            private List<Notification> notifications = new List<Notification>();

            public NotificationHubClient Hub { get; set; }

            private Notifications() {
                Hub = NotificationHubClient.CreateClientFromConnectionString("{conn string with full access}",     "{hub name}");
            }

            public Notification CreateNotification(string payload)
            {
                var notification = new Notification() {
                Id = notifications.Count,
                Payload = payload,
                Read = false
                };

                notifications.Add(notification);

                return notification;
            }

            public Notification ReadNotification(int id)
            {
                return notifications.ElementAt(id);
            }
        }

1. 在 NotificationsController.cs 中，使用下列程式碼來取代 **NotificationsController** 類別定義內的程式碼。 此元件會實作一個可供裝置以安全的方式擷取通知的方法，它還提供一個可觸發安全推送至裝置的方法 (依照本教學課程的目的)。 請注意，將通知傳送至通知中樞時，我們只會傳送包含通知 ID 的原始通知 (而非實際訊息)。
   
       public NotificationsController()
       {
           Notifications.Instance.CreateNotification("This is a secure notification!");
       }
   
       // GET api/notifications/id
       public Notification Get(int id)
       {
           return Notifications.Instance.ReadNotification(id);
       }
   
       public async Task<HttpResponseMessage> Post()
       {
           var secureNotificationInTheBackend = Notifications.Instance.CreateNotification("Secure confirmation.");
           var usernameTag = "username:" + HttpContext.Current.User.Identity.Name;
   
           // windows
           var rawNotificationToBeSent = new Microsoft.Azure.NotificationHubs.WindowsNotification(secureNotificationInTheBackend.Id.ToString(),
                           new Dictionary<string, string> {
                               {"X-WNS-Type", "wns/raw"}
                           });
           await Notifications.Instance.Hub.SendNotificationAsync(rawNotificationToBeSent, usernameTag);
   
           // apns
           await Notifications.Instance.Hub.SendAppleNativeNotificationAsync("{\"aps\": {\"content-available\": 1}, \"secureId\": \"" + secureNotificationInTheBackend.Id.ToString() + "\"}", usernameTag);
   
           // gcm
           await Notifications.Instance.Hub.SendGcmNativeNotificationAsync("{\"data\": {\"secureId\": \"" + secureNotificationInTheBackend.Id.ToString() + "\"}}", usernameTag);

            return Request.CreateResponse(HttpStatusCode.OK);
        }


請注意 `Post` 方法目前不會傳送快顯通知。 它會傳送只包含通知 ID 且非任何敏感內容的原始通知。 另外，針對您未在通知中樞上設定其認證的平台，請務必為傳送作業加上註解，因為他們將會導致錯誤。

1. 為了可以從所有裝置存取此應用程式，我們現在可以將它重新部署到 Azure 網站。 以滑鼠右鍵按一下 **AppBackend** 專案，然後選取 [發佈]。
2. 選取 Azure 網站作為您的發行目標。 使用您的 Azure 帳戶登入，並選取現有或新的網站，然後記下 [連線] 索引標籤中的 [目的地 URL] 屬性。我們後續將在本教學課程中參考此 URL 作為您的 *後端端點* 。 按一下 [發佈] 。

