# macOS autostart (LaunchAgent)

Keeps `cli-proxy-api` running at all times: starts at login, restarts if it
crashes or is killed.

A LaunchAgent (not a LaunchDaemon) is correct here — the proxy reads
`~/.cli-proxy-api` and the user's config, so it must run as the logged-in user,
not root.

## Install

```sh
sed "s#__HOME__#$HOME#g" deploy/macos/com.yagyesh.cliproxyapi.plist \
  > ~/Library/LaunchAgents/com.yagyesh.cliproxyapi.plist
plutil -lint ~/Library/LaunchAgents/com.yagyesh.cliproxyapi.plist
launchctl bootstrap gui/$(id -u) ~/Library/LaunchAgents/com.yagyesh.cliproxyapi.plist
launchctl enable gui/$(id -u)/com.yagyesh.cliproxyapi
```

Assumes the binary at `<repo>/bin/cli-proxy-api` and config at
`<repo>/config.yaml`. Edit the plist's `ProgramArguments` if either moves.

## Operate

```sh
launchctl print gui/$(id -u)/com.yagyesh.cliproxyapi | grep -E 'state|pid'
launchctl kickstart -k gui/$(id -u)/com.yagyesh.cliproxyapi   # restart (after config edits)
launchctl bootout gui/$(id -u)/com.yagyesh.cliproxyapi        # stop + unload
tail -f ~/Library/Logs/cliproxyapi.log
```

`KeepAlive` is on, so `kill` alone will not stop it — launchd respawns within
~10s (`ThrottleInterval`). Use `bootout` to actually stop it.

## Health check

```sh
curl -s -o /dev/null -w '%{http_code}\n' \
  -H "Authorization: Bearer $API_KEY" http://127.0.0.1:8317/v1/models
```

## Notes

- `~/Library/Logs/cliproxyapi.log` is not rotated by launchd; truncate it
  occasionally, or add a `newsyslog.d` entry if it grows.
- `KeepAlive: true` covers crashes, not a wedged-but-alive process. Add a
  periodic health probe if that ever happens in practice.
