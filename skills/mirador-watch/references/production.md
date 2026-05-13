# Production deployment

Bare `mirador watch &` dies on terminal close. For unattended operation use the OS service manager.

## macOS — launchd user agent

Write `~/Library/LaunchAgents/dev.boshenzh.mirador.<account>.plist`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN"
  "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>Label</key>
  <string>dev.boshenzh.mirador.<account></string>
  <key>ProgramArguments</key>
  <array>
    <string>/Users/<you>/.cargo/bin/mirador</string>
    <string>watch</string>
    <string><account></string>
    <string>INBOX</string>
  </array>
  <key>RunAtLoad</key>
  <true/>
  <key>KeepAlive</key>
  <true/>
  <key>StandardOutPath</key>
  <string>/Users/<you>/Library/Logs/mirador/<account>.log</string>
  <key>StandardErrorPath</key>
  <string>/Users/<you>/Library/Logs/mirador/<account>.log</string>
  <key>EnvironmentVariables</key>
  <dict>
    <key>OPENCLAW_HOOK_TOKEN</key>
    <string><your-token></string>
  </dict>
</dict>
</plist>
```

Load:
```bash
launchctl load ~/Library/LaunchAgents/dev.boshenzh.mirador.<account>.plist
launchctl start dev.boshenzh.mirador.<account>
```

`KeepAlive=true` means launchd restarts mirador if it exits for any reason. `RunAtLoad=true` starts it at login.

Status:
```bash
launchctl list | grep mirador
```

Unload (stop and disable):
```bash
launchctl unload ~/Library/LaunchAgents/dev.boshenzh.mirador.<account>.plist
```

## Linux — systemd user unit

Write `~/.config/systemd/user/mirador@.service` (templated):

```ini
[Unit]
Description=mirador IMAP IDLE watcher for %i
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
ExecStart=%h/.cargo/bin/mirador watch %i INBOX
Restart=always
RestartSec=10
Environment=OPENCLAW_HOOK_TOKEN=<your-token>
StandardOutput=append:%h/.local/state/mirador/%i.log
StandardError=append:%h/.local/state/mirador/%i.log

[Install]
WantedBy=default.target
```

Enable per-account:
```bash
systemctl --user daemon-reload
systemctl --user enable --now mirador@<account>.service
```

Status:
```bash
systemctl --user status mirador@<account>.service
journalctl --user -u mirador@<account>.service -f
```

## Health check

`mirador watch` is silent during IDLE — no log lines means it is working. To confirm liveness independently of mail traffic:

```bash
# expect mirador process listed
pgrep -fl "mirador watch <account>"
```

Or wrap the watcher in a process that exports a `/healthz` HTTP endpoint. Out of scope for this skill — write a separate sidecar.
