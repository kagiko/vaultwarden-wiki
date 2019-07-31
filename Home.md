# bitwarden_rs
This is a Bitwarden server API implementation written in Rust compatible with upstream Bitwarden clients*, perfect for self-hosted deployment where running the official resource-heavy service might not be ideal.

## Features
Basically full implementation of Bitwarden API is provided including:

* Basic single user functionality
* Organizations support
* Attachments
* Vault API support
* Serving the static files for Vault interface
* Website icons API
* Authenticator and U2F support
* YubiKey OTP

## Missing Features
* Email confirmation
* Other two-factor systems:
  * Email codes

## Get in touch
To ask an question, [raising an issue](https://github.com/dani-garcia/bitwarden_rs/issues/new) is fine, also please report any bugs spotted here.

If you prefer to chat, we're usually hanging around at [#bitwarden_rs:matrix.org](https://matrix.to/#/#bitwarden_rs:matrix.org) room on Matrix. Feel free to join us!