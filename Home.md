# Vaultwarden

Vaultwarden is an unofficial Bitwarden server implementation written in Rust. It is compatible with the [official Bitwarden clients](https://bitwarden.com/download/), and is ideal for self-hosted deployments where running the official resource-heavy service is undesirable.

Vaultwarden is targeted towards individuals, families, and smaller organizations. Development of features that are mainly useful to larger organizations (e.g., single sign-on, directory syncing, etc.) is not a priority, though high-quality PRs that implement such features would be welcome.

## Supported features

Vaultwarden implements the Bitwarden APIs required for most functionality, including:

* Web interface (equivalent to https://vault.bitwarden.com/)
* Personal vault support
* [Organization](https://bitwarden.com/help/getting-started-organizations/) vault support
* [Password sharing](https://bitwarden.com/help/sharing/) and [access control](https://bitwarden.com/help/user-types-access-control/)
* [Collections](https://bitwarden.com/help/about-collections/)
* [File attachments](https://bitwarden.com/help/attachments/)
* [Folders](https://bitwarden.com/help/folders/)
* [Favorites](https://bitwarden.com/help/favorites/)
* [Website icons](https://bitwarden.com/help/website-icons/)
* [Bitwarden Authenticator (TOTP)](https://bitwarden.com/help/authenticator-keys/)
* [Bitwarden Send](https://bitwarden.com/help/about-send/)
* [Emergency Access](https://bitwarden.com/help/emergency-access/)
* [Live sync](https://bitwarden.com/blog/post/live-sync/) (WebSocket only) for desktop/browser clients/extensions
* [Trash](https://bitwarden.com/help/managing-items/#items-in-the-trash) (soft delete)
* [Master password re-prompt](https://bitwarden.com/help/managing-items/#protect-individual-items)
* [Personal API key](https://bitwarden.com/help/personal-api-key/)
* Two-step login via [email](https://bitwarden.com/help/setup-two-step-login-email/), [Duo](https://bitwarden.com/help/setup-two-step-login-duo/), [YubiKey](https://bitwarden.com/help/setup-two-step-login-yubikey/), and [FIDO2 WebAuthn](https://bitwarden.com/help/setup-two-step-login-fido/) (including Nitrokeys and Solokeys)
* [Directory Connector](https://bitwarden.com/help/directory-sync/) support (basic implementation, no group support)
  <br>Only version [v2.9.2](https://github.com/bitwarden/directory-connector/releases/tag/v2.9.2) and lower is supported, v2.9.3 and up use a different login method not supported yet.
* Certain enterprise policies:
  * [Two-Step Login](https://bitwarden.com/help/policies/#two-step-login)
  * [Master Password](https://bitwarden.com/help/policies/#master-password)
  * [Password Generator](https://bitwarden.com/help/policies/#password-generator)
  * [Personal Ownership](https://bitwarden.com/help/policies/#personal-ownership)
  * [Disable Send](https://bitwarden.com/help/policies/#disable-send)
  * [Send Options](https://bitwarden.com/help/policies/#send-options)
  * [Single Organization](https://bitwarden.com/help/policies/#single-organization)

## Missing features

Issue [#246](https://github.com/dani-garcia/vaultwarden/issues/246) contains the comprehensive list of feature requests, both features of the official server that are missing in Vaultwarden, as well as enhancements specific to Vaultwarden.

To simplify comparison with the official server, this section summarizes the features implemented in the official server that are not currently available in Vaultwarden.

Features that may be added as time permits (contributions are always welcome):

* [Bitwarden Public API](https://bitwarden.com/help/public-api/) / [Organization API key](https://bitwarden.com/help/public-api/#authentication)
* [Event Logs](https://bitwarden.com/help/event-logs/)
* [Live sync](https://bitwarden.com/blog/post/live-sync/) (push notifications) for mobile clients (Android/iOS)
* [Admin Password Reset](https://bitwarden.com/help/admin-reset/)
* Certain enterprise policies:
  * [Master Password Reset](https://bitwarden.com/help/policies/#master-password-reset)

Features that probably won't be added unless contributed:

* [Single Sign-On (SSO)](https://bitwarden.com/help/about-sso/)
* [Groups](https://bitwarden.com/help/about-groups/)
* [Custom roles](https://bitwarden.com/help/user-types-access-control/#custom-role)
* Certain enterprise policies ([UI not open source](https://github.com/bitwarden/web/tree/master/bitwarden_license/src/app/policies), would probably need to be configured via admin page):
  * [Vault Timeout](https://bitwarden.com/help/policies/#vault-timeout)
  * [Disable Personal Vault Export](https://bitwarden.com/help/policies/#disable-personal-vault-export)

## Get in touch

To ask a question, offer suggestions, request new features, or get help configuring or installing the software, please [use the forum](https://vaultwarden.discourse.group/).

If you spot any bugs or crashes with Vaultwarden itself, please [create an issue](https://github.com/dani-garcia/vaultwarden/issues/). Make sure there aren't any similar issues open, though!

If you prefer to chat, we're usually hanging around at [#vaultwarden:matrix.org](https://matrix.to/#/#vaultwarden:matrix.org) room on Matrix. Feel free to join us!