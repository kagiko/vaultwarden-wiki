## Inviting users into organization

### With SMTP enabled

Invited users will receive an email containing a link that is valid for 5 days. Upon clicking the link, users can choose to create an account or log in. New users will need to create a new account; existing users who are being invited to a new organization will simply need to log in. After either step, they will show up as "Accepted" in the admin interface, and will be added to the organization when an orgnization admin confirms them.

### Without SMTP enabled

The invited users won't get an invitation email; instead all already registered users will appear in the interface as if they already accepted the invitation. Organization admin then just needs to confirm them to be proper Organization members and to give them access to the shared secrets.

Invited users that aren't registered yet will show up in the Organization admin interface as "Invited". At the same time an invitation record is created that allows the users to register even if [[user registration is disabled|disable-registration-of-new-users]]. (unless you [[disable this functionality|disable-invitations]]) They will automatically become "Accepted" once they register. From there Organization admin can confirm them to give them access to Organization.

## Running on unencrypted connection

It is strongly recommended to run bitwarden_rs service over HTTPS. However the server itself while [[supporting it|enabling-https]] does not strictly require such setup. This makes it a bit easier to spin up the service in cases where you can generally trust the connection (internal and secure network, access over VPN,..) or when you want to put the service behind HTTP proxy, that will do the encryption on the proxy end.

Running over HTTP is still reasonably secure provided you use really strong master password and that you avoid using web Vault over connection that is vulnerable to MITM attacks where attacker could inject javascript into your interface. However some forms of 2FA might not work in this setup and [Vault doesn't work in this configuration in Chrome](https://github.com/bitwarden/web/issues/254).