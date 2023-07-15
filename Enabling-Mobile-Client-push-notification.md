Since the version 1.29.0 of Vaultwarden, you can activate the Mobile Client push notification to seamlessly sync your vault between the mobile app, the web extension and the web vault without the need to sync manually.

### Enable Mobile Client push notification

Edit your  vaultwarden docker compose file and add this lines in the environnement part:
```yaml
      - PUSH_ENABLED=true
      - PUSH_INSTALLATION_ID=
      - PUSH_INSTALLATION_KEY=
```

To get the PUSH_INSTALLATION_ID and PUSH_INSTALLATION_KEY go to [https://bitwarden.com/host/](https://bitwarden.com/host/), put an email address and you'll get your ID and KEY.

Once it's done, restart your docker container with

```bash 
docker compose up -d vaultwarden
```

💡 At first the sync will not work with the mobile app. You have to reinstall the app and relog to you Vaultwarden to make the push notification work.