# bitwarden_rs

bitwarden_rs is an unofficial Bitwarden server implementation written in Rust. It is compatible with the [official Bitwarden clients](https://bitwarden.com/download/), and is ideal for self-hosted deployments where running the official resource-heavy service is undesirable.

bitwarden_rs is targeted towards individuals, families, and smaller organizations. Development of features that are mainly useful to larger organizations (e.g., single sign-on, directory syncing, etc.) is not a priority, though high-quality PRs that implement such features would be welcome.

## Supported features

bitwarden_rs implements the Bitwarden APIs required for most functionality, including:

* Web interface (equivalent to https://vault.bitwarden.com/)
* Personal vault support
* [Organization](https://bitwarden.com/help/article/getting-started-organizations/) vault support
* [Password sharing](https://bitwarden.com/help/article/share-to-a-collection/) and [access control](https://bitwarden.com/help/article/user-types-access-control/)
* [Collections](https://bitwarden.com/help/article/about-collections/)
* [File attachments](https://bitwarden.com/help/article/attachments/)
* [Folders](https://bitwarden.com/help/article/folders/)
* [Favorites](https://bitwarden.com/help/article/favorites/)
* [Website icons](https://bitwarden.com/help/article/website-icons/)
* [Bitwarden Authenticator (TOTP)](https://bitwarden.com/help/article/authenticator-keys/)
* [Live sync](https://bitwarden.com/blog/post/live-sync/) (WebSocket only) for desktop/browser clients
* [Trash](https://bitwarden.com/help/article/managing-items/#items-in-the-trash) (soft delete)
* Two-step login via [email](https://bitwarden.com/help/article/setup-two-step-login-email/), [Duo](https://bitwarden.com/help/article/setup-two-step-login-duo/), [YubiKey](https://bitwarden.com/help/article/setup-two-step-login-yubikey/), and [FIDO U2F](https://bitwarden.com/help/article/setup-two-step-login-u2f/)
* Certain enterprise policies:
  * [Master Password](https://bitwarden.com/help/article/policies/#master-password)
  * [Password Generator](https://bitwarden.com/help/article/policies/#password-generator)
  * [Personal Ownership](https://bitwarden.com/help/article/policies/#personal-ownership)

## Missing features

Issue [#246](https://github.com/dani-garcia/bitwarden_rs/issues/246) contains the comprehensive list of feature requests, both features of the official server that are missing in bitwarden_rs, as well as enhancements specific to bitwarden_rs.

To simplify comparison with the official server, this section summarizes the features implemented in the official server that are not currently available in bitwarden_rs.

Features that may be added as time permits (contributions are always welcome):

* [Emergency Access](https://bitwarden.com/help/article/emergency-access/)
* [Bitwarden Public API](https://bitwarden.com/help/article/public-api/)
* [Event Logs](https://bitwarden.com/help/article/event-logs/)
* [Live sync](https://bitwarden.com/blog/post/live-sync/) (push notifications) for mobile clients (Android/iOS)
* Certain enterprise policies:
  * [Two-Step Login](https://bitwarden.com/help/article/policies/#two-step-login) ([#981](https://github.com/dani-garcia/bitwarden_rs/issues/981))
  * [Single Organization](https://bitwarden.com/help/article/policies/#single-organization)

Features that probably won't be added unless contributed:

* [Single Sign-On (SSO)](https://bitwarden.com/help/article/about-sso/)
* [Directory Connector](https://bitwarden.com/help/article/directory-sync/) support
* [Groups](https://bitwarden.com/help/article/about-groups/)
* [Custom roles](https://bitwarden.com/help/article/user-types-access-control/#custom-role)

## Get in touch

To ask a question, offer suggestions, request new features, or get help configuring or installing the software, please [use the forum](https://bitwardenrs.discourse.group/).

If you spot any bugs or crashes with bitwarden_rs itself, please [create an issue](https://github.com/dani-garcia/bitwarden_rs/issues/). Make sure there aren't any similar issues open, though!

If you prefer to chat, we're usually hanging around at [#bitwarden_rs:matrix.org](https://matrix.to/#/#bitwarden_rs:matrix.org) room on Matrix. Feel free to join us!
