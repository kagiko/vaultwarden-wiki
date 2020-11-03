WebSocket notifications are used to inform the browser and desktop Bitwarden clients that some event of interest has occurred, such as when an entry in the password database has been modified or deleted. Upon receiving the notification, the client can take an appropriate action, such as refetching the modified entry, or removing the deleted entry from its local copy of the database. In this notification scheme, the Bitwarden client establishes a persistent WebSocket connection with the Bitwarden server (bitwarden_rs in this case). Whenever the server has an event to report, it sends it to the client via this persistent connection.

Note that WebSocket notifications are not applicable to the mobile (Android/iOS) Bitwarden clients. These clients use the native push notification service instead ([FCM](https://firebase.google.com/docs/cloud-messaging) for Android, [APNs](https://developer.apple.com/go/?id=push-notifications) for iOS). bitwarden_rs does not currently support push notifications to mobile clients.

To enable WebSockets notifications, an external reverse proxy is necessary, and it must be configured to do the following:
- Route the `/notifications/hub` endpoint to the WebSocket server, by default at port `3012`, making sure to pass the `Connection` and `Upgrade` headers. (Note the port can be changed with `WEBSOCKET_PORT` variable)
- Route everything else, including `/notifications/hub/negotiate`, to the standard Rocket server, by default at port `80`.
- If using Docker, you may need to map both ports with the `-p` flag

Example configurations are included in [[Proxy examples|proxy-examples]].

Then you need to enable WebSockets negotiation on the bitwarden_rs side by setting the `WEBSOCKET_ENABLED` variable to `true`:

```sh
docker run -d --name bitwarden \
  -e WEBSOCKET_ENABLED=true \
  -v /bw-data/:/data/ \
  -p 80:80 \
  -p 3012:3012 \
  bitwardenrs/server:latest
```

Note: The reason for this workaround is the lack of support for WebSockets from Rocket (though [it's a planned feature](https://github.com/SergioBenitez/Rocket/issues/90)), which forces us to launch a secondary server on a separate port.