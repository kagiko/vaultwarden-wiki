If you [set up mail](https://github.com/dani-garcia/vaultwarden/wiki/SMTP-Configuration) Vaultwarden will send its mails in English. Since the server does not know about a user's preferred language settings (which is done completely client-side) it is currently not possible to provide multiple languages for different users on the same server. 

If your users don't understand English you could translate the provided handlebar templates into your preferred language.

## How to translate / customize the templates?

You can use customized templates by

1. copying the respective files from the repository (from `src/static/templates/email`) to the corresponding `TEMPLATES_FOLDER` with the same folder structure (e.g. to `data/templates/email`)
2. changing them (e.g. translate them - but be sure to keep the `{{variables}}` in the link intact so everything still works) and
3. restarting vaultwarden to load the new (overwritten) templates.

**Note:** For ensuring compatibility you should initially download the templates for your version and you will also have to keep them up to date yourself if they ever change (or should new templates get added).

## Translations

* German by @kennymc-c: https://github.com/kennymc-c/vaultwarden-lang-de
* French by @YoanSimco: https://github.com/YoanSimco/vaultwarden-lang-fr
* Polish by @olokelo: https://github.com/olokelo/vaultwarden-lang-pl
* Simplified Chinese by @wcjxixi: https://github.com/wcjxixi/vaultwarden-lang-zhcn
* Simplified Chinese by @zituoguan: https://github.com/zituoguan/vaultwarden-lang-zh_CN
* Italian by @rizlas: https://github.com/rizlas/vaultwarden-lang-it
* Spanish by @javier-varez: https://github.com/javier-varez/vaultwarden-lang-es
* Russian by @marat2509: https://github.com/marat2509/vaultwarden-lang-ru
* Brazilian Portuguese by @marivaldojr: https://github.com/marivaldojr/vaultwarden-lang-pt_br

> [!WARNING]
> Translations are provided as is by community members and we have not tested them. So use them at your own risk. If there has been a breaking change (e.g. caused by a new release of Vaultwarden) inform the maintainer and/or make a note here.