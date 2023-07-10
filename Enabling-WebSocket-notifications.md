WebSocket notifications are used to inform the browser and desktop Bitwarden clients that some event of interest has occurred, such as when an entry in the password database has been modified or deleted. Upon receiving the notification, the client can take an appropriate action, such as refetching the modified entry, or removing the deleted entry from its local copy of the database. In this notification scheme, the Bitwarden client establishes a persistent WebSocket connection with the Bitwarden server (vaultwarden in this case). Whenever the server has an event to report, it sends it to the client via this persistent connection.

Note that WebSocket notifications are not applicable to the mobile (Android/iOS) Bitwarden clients. These clients use the native push notification service instead ([FCM](https://firebase.google.com/docs/cloud-messaging) for Android, [APNs](https://developer.apple.com/go/?id=push-notifications) for iOS). These have to be configured separately using push credentials from Bitwarden's cloud service.

To enable WebSockets notifications, an external reverse proxy is necessary, and it must be configured to do the following:
- Route the `/notifications/hub` endpoint to the WebSocket server, by default at port `3012`, making sure to pass the `Connection` and `Upgrade` headers. (Note the port can be changed with `WEBSOCKET_PORT` variable)
- Route everything else, including `/notifications/hub/negotiate`, to the standard Rocket server, by default at port `80`.
- If using Docker, you may need to map both ports with the `-p` flag

Example configurations are included in [[Proxy examples|proxy-examples]].

Then you need to enable WebSockets negotiation on the vaultwarden side by setting the `WEBSOCKET_ENABLED` variable to `true`:

```sh
docker run -d --name vaultwarden \
  -e WEBSOCKET_ENABLED=true \
  -v /vw-data/:/data/ \
  -p 80:80 \
  -p 3012:3012 \
  vaultwarden/server:latest
```

Note: Port 3012 is only required when using an old reverse proxy configuration. From version 1.29.0, vaultwarden support WebSocket notifications through port 80.

## Test the WebSockets connection

Testing if a connection is working correctly can be done in two ways:

1. Open the developer tools of your browser, go to the network tab and filter for `WS`/`WebSockets`. Logout or refresh the page and login again and you you should see a 101 response for the upgraded WebSocket connection. If you click on that line you should be able to see the messages. If you do not get the status code 101 on `/notifications/hub` then something is configured incorrectly.

2. Open two different browsers or an incognito/private window. Login into your account on both. Either create a new entry, or rename the cipher in one, and that should instantly change also in the other.
