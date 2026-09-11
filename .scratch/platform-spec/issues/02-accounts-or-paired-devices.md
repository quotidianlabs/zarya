# Accounts or paired devices?

Type: grilling
Status: open
Blocked by: -

## Question

`nooka` and `habbits` deliberately have no account. Sync needs *some* way to know that two
devices belong to the same person, but an account is not the only shape that can take.

- **Accounts**: email + password, magic link, or OAuth. Familiar, recoverable, and it means
  running identity infrastructure and holding a credential for a personal tool.
- **Device pairing**: one device shows a code or QR, another scans it, and they share a keypair
  or a sync secret. No email, no password, no recovery - lose every device and the data is gone.
- **Bring-your-own-storage**: point every device at a store the user already owns (their own
  server, a cloud drive, a self-hosted instance). Continuous with `nooka`'s Google Drive backup.

Constraints worth weighing: Google sign-in is not usable in the RuStore market; Sign in with
Apple is iOS-shaped; and at the personal-tool bar, "recovery" may be worth less than "no
credential to hold".

This is a product decision that should *drive* the sync engine choice, not follow it - several
candidate engines bundle an auth model, and picking the engine first would silently decide this.
