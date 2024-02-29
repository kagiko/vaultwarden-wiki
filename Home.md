# Vaultwarden

Vaultwarden is an unofficial Bitwarden server implementation written in Rust. It is compatible with the [official Bitwarden clients](https://bitwarden.com/download/), and is ideal for self-hosted deployments where running the official resource-heavy service is undesirable.

Vaultwarden is targeted towards individuals, families, and smaller organizations. Development of features that are mainly useful to larger organizations (e.g., single sign-on, directory syncing, etc.) is not a priority, though high-quality PRs that implement such features would be welcome.

## Supported features

Vaultwarden implements the Bitwarden APIs required for most functionality, including:

* Web interface (equivalent to https://vault.bitwarden.com/)
* Personal vault support
* [Organization](https://bitwarden.com/help/getting-started-organizations/) vault support
* [Groups](https://bitwarden.com/help/about-groups/) (unstable ([#3624](https://github.com/dani-garcia/vaultwarden/issues/3624), [#3413](https://github.com/dani-garcia/vaultwarden/issues/3413)), setting an [environment variable](https://github.com/dani-garcia/vaultwarden/blob/bb2412d0339e1da5dee99fc566a2b2aab5d2808c/.env.template#L409-L414) is required in order to enable it)
* [Event Logs](https://bitwarden.com/help/event-logs/)
* [Password sharing](https://bitwarden.com/help/sharing/) and [access control](https://bitwarden.com/help/user-types-access-control/)
* [Collections](https://bitwarden.com/help/about-collections/)
* [File attachments](https://bitwarden.com/help/attachments/)
* [Folders](https://bitwarden.com/help/folders/)
* [Favorites](https://bitwarden.com/help/favorites/)
* [Website icons](https://bitwarden.com/help/website-icons/)
* [Bitwarden Authenticator (TOTP)](https://bitwarden.com/help/authenticator-keys/)
* [Bitwarden Send](https://bitwarden.com/help/about-send/)
* [Emergency Access](https://bitwarden.com/help/emergency-access/)
* [Live sync](https://bitwarden.com/blog/live-sync/) (WebSocket only) for desktop/browser clients/extensions
* [Trash](https://bitwarden.com/help/managing-items/#vault-trash) (soft delete)
* [Master password re-prompt](https://bitwarden.com/help/managing-items/#protect-individual-items)
* [Personal API key](https://bitwarden.com/help/personal-api-key/)
* Two-step login via [email](https://bitwarden.com/help/setup-two-step-login-email/), [Duo](https://bitwarden.com/help/setup-two-step-login-duo/), [YubiKey](https://bitwarden.com/help/setup-two-step-login-yubikey/), and [FIDO2 WebAuthn](https://bitwarden.com/help/setup-two-step-login-fido/) (including Nitrokeys and Solokeys)
* Username generator integration with SimpleLogin, AnonAddy, or Firefox Relay
* [Directory Connector](https://bitwarden.com/help/directory-sync/) support
* [Admin Password Reset](https://bitwarden.com/help/admin-reset/)
* [Live sync](https://bitwarden.com/blog/live-sync/) (push notifications) for mobile clients (Android/iOS)

* Certain enterprise policies:
  * [Require two-step login](https://bitwarden.com/help/policies/#require-two-step-login)
  * [Master password requirements](https://bitwarden.com/help/policies/#master-password-requirements)
  * [Master password reset](https://bitwarden.com/help/policies/#master-password-reset)
  * [Password generator](https://bitwarden.com/help/policies/#password-generator)
  * [Single organization](https://bitwarden.com/help/policies/#single-organization)
  * [Remove individual vault](https://bitwarden.com/help/policies/#remove-individual-vault)
  * [Remove Send](https://bitwarden.com/help/policies/#remove-send)
  * [Send options](https://bitwarden.com/help/policies/#send-options)

## Missing features

Issue [#246](https://github.com/dani-garcia/vaultwarden/issues/246) contains the comprehensive list of feature requests, both features of the official server that are missing in Vaultwarden, as well as enhancements specific to Vaultwarden.

To simplify comparison with the official server, this section summarizes the features implemented in the official server that are not currently available in Vaultwarden.

Features that may be added as time permits (contributions are always welcome):

* [Bitwarden Public API](https://bitwarden.com/help/public-api/) / [Organization API key](https://bitwarden.com/help/public-api/#authentication)
  This feature is partially added, but only to support the Bitwarden Directory Connector.

Features that probably won't be added unless contributed:

* [Single Sign-On (SSO)](https://bitwarden.com/help/about-sso/)
* [Custom roles](https://bitwarden.com/help/user-types-access-control/#custom-role)
* Certain enterprise policies ([UI not open source](https://github.com/bitwarden/clients/tree/main/bitwarden_license/bit-web/src/app/admin-console/policies), would probably need to be configured via admin page):
  * [Require single sign-on authentication](https://bitwarden.com/help/policies/#require-single-sign-on-authentication)
  * [Vault timeout](https://bitwarden.com/help/policies/#vault-timeout)
  * [Remove individual vault export](https://bitwarden.com/help/policies/#remove-individual-vault-export)

## Get in touch

To ask a question, offer suggestions, request new features, or get help configuring or installing the software, please [use the forum](https://vaultwarden.discourse.group/).

If you spot any bugs or crashes with Vaultwarden itself, please [create an issue](https://github.com/dani-garcia/vaultwarden/issues/). Make sure there aren't any similar issues open, though!

If you prefer to chat, we're usually hanging around at [#vaultwarden:matrix.org](https://matrix.to/#/#vaultwarden:matrix.org) room on Matrix. Feel free to join us!