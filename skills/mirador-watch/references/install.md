# Installing mirador

## CLI binary

Mirador is not on Homebrew, crates.io, or as a prebuilt release. Install from git:

```bash
cargo install --git https://github.com/pimalaya/mirador.git \
  --features imap,keyring --locked
```

Verify:
```bash
mirador --version
# expect: mirador v1.x.x +imap +keyring (+maildir +wizard)
```

If `cargo` is missing: `brew install rustup-init && rustup-init -y --default-toolchain stable --profile minimal --no-modify-path` then `source $HOME/.cargo/env`.

### Why `--locked` is required

Mirador's `Cargo.lock` pins `pimalaya-tui 0.2.2` against `toml 0.8`. Without `--locked`, cargo resolves `toml 0.9` which breaks the build with `expected toml::value::Value, found toml::Value` type-mismatch errors.

The yanked-keccak warning that `--locked` prints is non-fatal — yank does not delete the crate from the registry, it only hides it from new resolutions.

### When upstream pins update

If a future release removes the toml conflict, drop `--locked` and let cargo resolve fresh. Always re-verify with `mirador --version` after install.

## Storing the mailbox credential

Mirador's `backend.auth.cmd` is a shell command whose stdout is treated as the password. **Default: macOS Keychain.**

```bash
security add-generic-password \
  -a <email-address> \
  -s mirador-<account-name> \
  -w '<password-or-app-password>'
```

Then in `config.toml`:
```toml
backend.auth.cmd = "security find-generic-password -a <email-address> -s mirador-<account-name> -w"
```

### Alternatives (only when macOS Keychain is unavailable)

- **Linux GNOME Keyring / KWallet** via `secret-tool`:
  ```toml
  backend.auth.cmd = "secret-tool lookup mirador <account-name>"
  ```
- **Unix `pass`** (gpg-backed):
  ```toml
  backend.auth.cmd = "pass show mail/<account-name>"
  ```
- **1Password CLI**:
  ```toml
  backend.auth.cmd = "op read 'op://Private/<account-name>/password'"
  ```

Do not place plaintext credentials in `config.toml`. Do not use `backend.auth.raw`.

## Gmail / Outlook caveat

Account password (the one used to log into the web UI) **does not work** for IMAP on Gmail or Outlook. Generate an **app password** instead:
- Gmail: enable 2FA, then go to https://myaccount.google.com/apppasswords
- Outlook: similar at https://account.live.com/proofs/AppPassword
Store the app password in the secret store the same way as a regular password.
