Since version `1.29.0` of Vaultwarden, you can activate Mobile Client push notifications to [automatically sync](https://bitwarden.com/help/vault-sync/#automatic-sync) your personal vault between the mobile app, the web extension and the web vault without the need to sync manually.

### Enable Mobile Client push notification

1. Go to [https://bitwarden.com/host/](https://bitwarden.com/host/) insert your email address and you'll get an INSTALLATION ID and KEY.  

2. Add the following settings to your `docker-compose.yml` (and make sure you insert the correct ID and the KEY from the previous step):
```yaml
    environment:
      - PUSH_ENABLED=true
      - PUSH_INSTALLATION_ID=
      - PUSH_INSTALLATION_KEY= 
```
> :information_source: If you have requested an INSTALLATION ID and KEY for `bitwarden.eu (European Union)` in the previous step, you also have to set
```yaml
      - PUSH_RELAY_URI=https://push.bitwarden.eu
      - PUSH_IDENTITY_URI=https://identity.bitwarden.eu
```

3. Recreate your container, e.g. with

```bash 
docker compose up -d vaultwarden
```

4. Connect your app to your Vaultwarden instance.  
> :warning: If you have connected your Bitwarden app before [v1.30.2](https://github.com/dani-garcia/vaultwarden/releases/tag/1.30.2) push notifications **will not work** for your device. You have to **clear the app data** of your mobile app (or **reinstall the app**) and connect your Vaultwarden account again to [register the push token with the Bitwarden's Azure Notification Hub](https://contributing.bitwarden.com/architecture/deep-dives/push-notifications/mobile/#self-hosted-implementation).

> :bulb: Push notifications will also only work on Bitwarden apps installed from the official mobile stores (App Store, Google Play Store) or when using alternative clients for the Google Play Store (such as Aurora Store). Push notifications **will not work** using Bitwarden clients installed from [F-Droid](https://mobileapp.bitwarden.com/fdroid/), NeoStore or other alternative stores. Those apps have been built without support for Firebase Messaging.

5. Test if mobile push notifications work, for example by renaming a folder in the web vault and see if it changes after a few seconds in your mobile app.