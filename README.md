# IPATool

On a headless Ubuntu VPS, login fails because ipatool can't reach a
keychain/secret-service backend.

Fix — force the file-based keychain backend and pass everything inline:

DBUS_SESSION_BUS_ADDRESS='unix:path=/nonexistent' \
ipatool --keychain-passphrase 'any_passphrase_you_choose' \
auth login -e 'email_here' -p 'pass_here' --auth-code 'your_2fa_code'

Notes:
- DBUS_SESSION_BUS_ADDRESS=/nonexistent makes go-keyring's D-Bus backend
  fail and fall back to the encrypted file store.
