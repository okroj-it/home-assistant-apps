# doorkey

A guest taps the NFC tag at your door, a keypad opens on their phone, and the
right code opens the lock. Tap-gated actions go further: tap a tag, confirm
with a fingerprint (a passkey), and an allowlisted Home Assistant script
runs. Full documentation: <https://github.com/okroj-it/doorkey>.

## How the app is laid out

| | where | who |
|---|---|---|
| **Keypad, tap pages, passkey prompts** | port `8080`, published by you over HTTPS | anyone with a tag |
| **Admin page** | the **doorkey** entry in the sidebar (Ingress) | Home Assistant administrators |

The admin page is never on port 8080. doorkey checks with Home Assistant
that whoever opens the panel is an administrator; hiding the panel from
other users is not relied on.

## Setup

1. **Give doorkey a public HTTPS address.** Passkeys need a secure origin,
   and guests are on their own phones. Either:
   - the **Cloudflared** app: a tunnel hostname (e.g. `door.example.com`)
     pointing to `http://<your HA IP>:8080`; or
   - your reverse proxy, forwarding the hostname to port 8080.

   Do **not** put SSO or forward-auth in front: a guest has no account, the
   code or passkey is the credential.
2. **Set the options** (Configuration tab):
   - **Public address**: that address, scheme and host only.
   - **Lock**: the `lock.*` entity to open.
   - **Tap mode**: `sun` for NTAG 424 DNA tags (needs both tag keys), or
     `static` for a secret link on any NFC tag (needs the static path).
   - Optionally a notify service, your home networks, and admin users.
3. **Start** the app and open **doorkey** in the sidebar.

MQTT is picked up automatically when the Mosquitto broker app is installed
and its MQTT integration is set up (Settings → Devices & services offers it
once the broker runs). doorkey then shows up as a "Door Keypad" device:
locked out, failed attempts today, and the last entry.

## Tags

`sun` tags are provisioned on a computer with a PN532 reader, using
`tagtui` or `provision.py` from the doorkey repository. Both print a
one-line **tag code**: paste it into the admin page (Door → DNA tags). doorkey
refuses a code its own tag key-encryption key can't open, so a tag made with
the wrong key is caught at enrolment.

## Backups

The database, the generated secrets and the options live in the app's
`/data`, which Home Assistant backups include. doorkey folds its write-ahead
log into the database before each backup, so a hot backup is consistent.

The pepper in `/data/secrets.json` hashes every guest code: restoring the
app without it invalidates all codes. Keep your backups.

## Commands

`doorkey` is the command line inside the app's container, for example from
the **Advanced SSH & Web Terminal** app with protection mode off:

```sh
docker exec app_<repository>_doorkey doorkey help
```

(`docker ps` shows the exact container name.)
