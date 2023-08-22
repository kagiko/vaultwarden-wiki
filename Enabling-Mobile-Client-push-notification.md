Since version `1.29.0` of Vaultwarden, you can activate Mobile Client push notifications to [automatically sync](https://bitwarden.com/help/vault-sync/#automatic-sync) your personal vault between the mobile app, the web extension and the web vault without the need to sync manually.

### Enable Mobile Client push notification

1. Go to [https://bitwarden.com/host/](https://bitwarden.com/host/) insert your email address and you'll get an INSTALLATION ID and KEY.  
:warning: Until [#3752](https://github.com/dani-garcia/vaultwarden/pull/3752) is implemented, make sure to select `bitwarden.com (United States)` as Data Region.  
:bulb: The EU Data Region is currently **not supported** by Vaultwarden. If you have requested an INSTALLATION ID and KEY for `bitwarden.eu (European Union)`, you need to either wait until the PR is merged and released, [build Vaultwarden](https://github.com/dani-garcia/vaultwarden/wiki/Building-your-own-docker-image) with necessary changes yourself, or you can simply request a new id/key pair for the US Data Region.

2. Add the following settings to your `docker-compose.yml` (and make sure you insert the correct ID and the KEY from the previous step):
```yaml
    environment:
      - PUSH_ENABLED=true
      - PUSH_INSTALLATION_ID=
      - PUSH_INSTALLATION_KEY= 
```

3. Recreate your container, e.g. with

```bash 
docker compose up -d vaultwarden
```

4. Connect your app to your Vaultwarden instance.  
:warning: Unless you're using a freshly installed Bitwarden app, push notifications **will not work** with already connected clients. You have to **clear the app data** of your mobile app (or **reinstall the app**) and connect your Vaultwarden account again to [register the push token with the Bitwarden's Azure Notification Hub](https://contributing.bitwarden.com/architecture/deep-dives/push-notifications/mobile/#self-hosted-implementation).  
:bulb: Push notifications will also only work on Bitwarden apps installed from the official mobile stores (App Store, Google Play Store) or when using alternative clients for the Google Play Store (such as Aurora Store). Push notifications **will not work** using Bitwarden clients installed from [F-Droid](https://mobileapp.bitwarden.com/fdroid/), NeoStore or other alternative stores. Those apps have been built without support for Firebase Messaging.

5. Test if mobile push notifications work, for example by renaming a folder in the web vault and see if it changes after a few seconds in your mobile app.