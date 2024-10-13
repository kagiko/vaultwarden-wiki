# Customize Vaultwarden CSS

> [!IMPORTANT]
> **This functionality is not yet merged or released!**
> It will probably land as stable in v1.33.0

Since version v1.33.0 you can modify the CSS which Vaultwarden previously embedded in the web-vault.<br>
This way it makes it more easier for users to tweak the style and layout or even hide items.

To modify the CSS you need to add a `templates` directory in your `data` directory, or provide the correct path via the `TEMPLATES_FOLDER` environment variable.<br>
Within this directory you need to create another directory called `scss` which will hold the file(s) for modifying the stylesheet Vaultwarden will serve.


There are two files you can place here:
- **`user.vaultwarden.scss.hbs`**<br>
  This file is the file you want to edit and add your custom styles to.

- **`vaultwarden.scss.hbs`**<br>
  This file should not exist, since it will overwrite the built-in defaults.<br>
  _**Only override this if you really know what you are doing!**_

```tree
.
├── templates
│   └── scss
│       ├── user.vaultwarden.scss.hbs
│       └── vaultwarden.scss.hbs
```



**Some examples which you can place inside `user.vaultwarden.scss.hbs`:**
```css
/* Hide `Authenticator app` 2FA (First item of the list) */
app-two-factor-setup ul.list-group.list-group-2fa li.list-group-item:nth-child(1) {
  @extend %vw-hide;
}

/* Hide `YubiKey OTP security key` 2FA (Second item of the list) */
/* Note: This will also be hidden automatically if the Yubikey config is net set */
app-two-factor-setup ul.list-group.list-group-2fa li.list-group-item:nth-child(2) {
  @extend %vw-hide;
}

/* Hide `Duo` 2FA (Third item of the list) */
app-two-factor-setup ul.list-group.list-group-2fa li.list-group-item:nth-child(3) {
  @extend %vw-hide;
}

/* Hide `FIDO2 WebAuthn` 2FA (Fourth item of the list) */
app-two-factor-setup ul.list-group.list-group-2fa li.list-group-item:nth-child(4) {
  @extend %vw-hide;
}

/* Hide `Email` 2FA (Fifth item of the list) */
/* Note: This will also be hidden automatically if email is not enabled */
app-two-factor-setup ul.list-group.list-group-2fa li.list-group-item:nth-child(5) {
  @extend %vw-hide;
}

/* Use a custom login logo */
app-login img.logo {
	content: url(../images/my-custom-login.logo.png) !important;
}

/* Use a custom top left logo global */
bit-icon > svg {
  display: none !important;
}
bit-icon::before {
  display: block;
  content: "" !important;
  width: 175px !important;
  height: 30px !important;
  background-image: url(../images/my-custom-global-logo.png) !important;
  background-repeat: no-repeat !important;
  background-size: contain;
}

/* Use a custom top left logo different per vault/admin */
bit-icon > svg {
  display: none !important;
}
bit-icon::before {
  display: block;
  content: "" !important;
  width: 175px !important;
  height: 30px !important;
  background-repeat: no-repeat !important;
  background-size: contain;
}
app-user-layout bit-icon::before {
  background-image: url(../images/my-custom-password-manager-logo.png) !important;
}
app-organization-layout bit-icon::before {
  background-image: url(../images/my-custom-admin-console-logo.png) !important;
}
```
