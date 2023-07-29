Since the version 1.29.0 of Vaultwarden, you can activate the Mobile Client push notification to seamlessly sync your vault between the mobile app, the web extension and the web vault without the need to sync manually.

### Enable Mobile Client push notification

Edit your  vaultwarden docker compose file and add this lines in the environnement part:
```yaml
      - PUSH_ENABLED=true
      - PUSH_INSTALLATION_ID=
      - PUSH_INSTALLATION_KEY=
```

To get the PUSH_INSTALLATION_ID and PUSH_INSTALLATION_KEY go to [https://bitwarden.com/host/](https://bitwarden.com/host/), put an email address and you'll get your ID and KEY. Make sure to only select the US region, the EU region does not seem to work yet.

Once it's done, restart your docker container with

```bash 
docker compose up -d vaultwarden
```

💡 Unless you're using a freshly installed Bitwarden app, push notifications will not work with the mobile app straight away. You have to reinstall the app and login again to your Vaultwarden account to make the push notifications work.

💡 Push notifications will only work on Bitwarden app installed from the official mobile stores (App Store, Google Play Store) or when using alternative clients for the Google Play Store (such as Aurora Store). Push notifications **will not work** on Bitwarden installed from F-Droid, NeoStore or alternative stores.

💡Data Region EU is currently not supported by Vaultwarden push. If you requested an INSTALLATION_ID and -KEY for Data Region EU, you need to request a new one for the US Data Region and use those in your Vaultwarden config to successfully enable push. 