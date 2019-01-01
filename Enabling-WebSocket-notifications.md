*Important: This does not apply to the mobile clients, which use push notifications.*

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
  mprasil/bitwarden:latest
```

Note: The reason for this workaround is the lack of support for WebSockets from Rocket (though [it's a planned feature](https://github.com/SergioBenitez/Rocket/issues/90)), which forces us to launch a secondary server on a separate port.