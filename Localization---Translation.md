If you [set up mail](https://github.com/dani-garcia/vaultwarden/wiki/SMTP-Configuration) Vaultwarden will send its mails in English. Since the server does not know about a user's preferred language settings (which is done completely client-side) it is currently not possible to provide multiple languages for different users on the same server. 

If your users don't understand English you could translate the provided handlebar templates into your preferred language.

## How to translate / customize the templates?

You can use customized templates by

1. copying the respective files from the repository (from `src/static/templates/email`) to the corresponding `TEMPLATES_FOLDER` with the same folder structure (e.g. to `data/templates/email`)
2. changing them (e.g. translate them - but be sure to keep the `{{variables}}` in the link intact so everything still works) and
3. restarting vaultwarden to load the new (overwritten) templates.

**Note:** For ensuring compatibility you should initially download the templates for your version and you will also have to keep them uptodate yourself if they ever change (or new templates get added).