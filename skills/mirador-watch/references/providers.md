# Provider-specific config blocks

Drop into `~/.config/mirador/config.toml`. Only the `backend.host` / `backend.port` / `backend.login` and credential setup differ — everything else is identical.

## Aliyun enterprise mail (`qiye.aliyun.com`)

```toml
backend.host = "imap.qiye.aliyun.com"
backend.port = 993
backend.encryption.type = "tls"
backend.login = "<full-email-address>"
backend.auth.type = "password"
backend.auth.cmd = "security find-generic-password -a <full-email-address> -s mirador-aliyun -w"
```

- Account password works directly — no app password needed.
- IDLE cuts at ~5 minutes (mirador reconnects automatically, logs are noisy).
- CAPABILITY does not advertise AUTH=PLAIN/LOGIN. Mirador falls back to bare LOGIN which works. Three "trying auth mechanism…" failures in debug logs before login succeeds is expected.

## Gmail (`gmail.com`)

```toml
backend.host = "imap.gmail.com"
backend.port = 993
backend.encryption.type = "tls"
backend.login = "<full-email-address>"
backend.auth.type = "password"
backend.auth.cmd = "security find-generic-password -a <full-email-address> -s mirador-gmail -w"
```

- **App password required.** 2FA must be enabled, generate at https://myaccount.google.com/apppasswords. Store the 16-char app password (no spaces) via `security add-generic-password`.
- Gmail uses labels exposed as IMAP folders. `INBOX` works; for "All Mail" use `[Gmail]/All Mail`.
- IDLE timeout ~29 minutes.

## NetEase 163 (`163.com`)

```toml
backend.host = "imap.163.com"
backend.port = 993
backend.encryption.type = "tls"
backend.login = "<full-email-address>"
backend.auth.type = "password"
backend.auth.cmd = "security find-generic-password -a <full-email-address> -s mirador-163 -w"
```

- **Client authorization code required** (not the web password). Generate in 163 mail web UI: 设置 → POP3/SMTP/IMAP → 开启服务 → 授权码管理.
- 163 enforces IMAP ID (RFC 2971). Mirador 1.0 sends an ID by default — login should succeed. If you see `Unsafe Login`, file an upstream issue at https://github.com/pimalaya/mirador.

## Tencent exmail (`exmail.qq.com`)

```toml
backend.host = "imap.exmail.qq.com"
backend.port = 993
backend.encryption.type = "tls"
backend.login = "<full-email-address>"
backend.auth.type = "password"
backend.auth.cmd = "security find-generic-password -a <full-email-address> -s mirador-exmail -w"
```

- **Client authorization code required.**
- Known auth-mechanism quirks (see [himalaya issue #622](https://github.com/pimalaya/himalaya/issues/622)). Test with `mirador doctor` immediately after config.

## Custom IMAP server

Same template, just replace `backend.host` / `backend.port`. Common ports:
- 993 with `encryption.type = "tls"` (implicit TLS, recommended)
- 143 with `encryption.type = "start-tls"` (STARTTLS upgrade)
- 143 with `encryption.type = "none"` (cleartext — never use over the internet)
