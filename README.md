# IPATool (fork)

A fork of [majd/ipatool](https://github.com/majd/ipatool) with fixes for Apple's
June 2026 authentication changes and for headless-server usage.

## Fixes in this fork

### 1. Apple App Store login broken — `something went wrong`

In June 2026 (Apple's `26HOTFIX24`) Apple changed the native auth flow, breaking
login for everyone with a generic `ERR error="something went wrong"` (sometimes
surfacing as `unexpected hex digit 'h'`). Two things changed:

- The `authenticateAccount` endpoint key moved to the **Bag root** (it used to
  live under `urlBag`), so the client resolved an empty endpoint.
- The native auth endpoint now requires a **trailing slash** —
  `https://auth.itunes.apple.com/auth/v1/native/fast/`. Without it Apple returns
  `301` + an HTML redirect page, which the plist parser rejects.

This fork reads `authenticateAccount` from the Bag root (with `urlBag` as a
fallback) and normalizes the resolved endpoint to always end in `/fast/`. It also
surfaces Apple's `AMD-Action::SP` response as a clear "account requires browser
sign-in (2FA or Apple ID review required)" message instead of the generic error.

Ports [majd/ipatool#486](https://github.com/majd/ipatool/pull/486); fixes
[#488](https://github.com/majd/ipatool/issues/488),
[#485](https://github.com/majd/ipatool/issues/485), and
[#499](https://github.com/majd/ipatool/issues/499).

### 2. Stale cookie lock file hangs the tool

A zero-byte `cookies.lock` file (left behind if a process is killed mid-write)
would block every subsequent run forever, because the locking library only does
PID-based stale detection when the lock file has content. The tool now removes a
zero-byte lock file on startup.

## Headless / VPS usage

On a headless Ubuntu VPS, login fails because ipatool can't reach a
keychain / secret-service backend. Force the file-based keychain backend and pass
everything inline:

```bash
DBUS_SESSION_BUS_ADDRESS='unix:path=/nonexistent' \
ipatool --keychain-passphrase 'any_passphrase_you_choose' \
auth login -e 'email_here' -p 'pass_here' --auth-code 'your_2fa_code'
```

<img width="1394" height="364" alt="image" src="https://github.com/user-attachments/assets/b198f01f-607f-4c4c-a293-ee649a210818" />

Notes:

- Setting `DBUS_SESSION_BUS_ADDRESS` to a nonexistent path makes go-keyring's
  D-Bus backend fail and fall back to the encrypted file store.

## Building

```bash
go generate ./...
go build -o ipatool .
```
