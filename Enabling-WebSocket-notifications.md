WebSocket notifications are used to inform the browser, desktop and Browser Extension Bitwarden clients that some event of interest has occurred, such as when an entry in the password database has been modified or deleted. Upon receiving the notification, the client can take an appropriate action, such as refresh the modified entry, or removing the deleted entry from its local cache. In this notification scheme, the Bitwarden client establishes a persistent WebSocket connection with the Bitwarden server (Vaultwarden in this case). Whenever the server has an event to report, it sends it to the client via this persistent connection.

Note that WebSocket notifications are not applicable to the mobile (Android/iOS) Bitwarden clients. These clients use the native push notification service instead ([FCM](https://firebase.google.com/docs/cloud-messaging) for Android, [APNs](https://developer.apple.com/go/?id=push-notifications) for iOS). These have to be configured separately using push credentials from Bitwarden's cloud service, also available since v1.29.0.

<br>

WebSocket's are enabled by default since v1.29.0 of Vaultwarden. Previous versions needed a reverse proxy because WebSockets were running on a different port than then default HTTPS port.<br>
The old implementation is still available in v1.29.0 to not break during updates for now. But this will be removed in the future.<br>

<br>

If you do use a reverse proxy like nginx or Apache HTTPd, then you need to make sure you configure it correctly to pass through the WebSocket `Upgrade` and `Connection` headers. Some reverse proxies do this by default like Traefik for example.

<br>

The old `WEBSOCKET_ENABLED` and `WEBSOCKET_PORT` are not needed anymore since v1.29.0 of Vaultwarden and can be ignored.<br>
In fact, if you use the native implementation setting `WEBSOCKET_ENABLED` back to the default `false` value will reduce resources used by Vaultwarden (though not that much).

<br>

Example configurations are included in [[Proxy examples|proxy-examples]].
<br>
**Note that some examples are not yet updated for the v1.29.0 version.**

<br>

## Test the WebSockets connection

Testing if a connection is working correctly can be done in two ways:

1. Open the developer tools of your browser, go to the network tab and filter for `WS`/`WebSockets`. Logout or refresh the page and login again and you you should see a 101 response for the upgraded WebSocket connection. If you click on that line you should be able to see the messages. If you do not get the status code 101 on `/notifications/hub` then something is configured incorrectly.
Messages will be shown in the console window of your browser:

`[2023-12-01T00:00:00.000Z] Information: WebSocket connected to wss://HOST_NAME/notifications/hub?access_token=eyJ0eX......`

2. Open two different browsers or an incognito/private window. Login into your account on both. Either create a new entry, or rename the cipher in one, and that should instantly change also in the other.
