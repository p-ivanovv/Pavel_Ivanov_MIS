# 08 — Identichnost i dostap (Keycloak)

## Summary

Vsichko ot test matricata (T1-T8) e izpalneno i raboti.

- T1: staff login — OK.
- T2: TOTP za admin — OK.
- T3: greshen TOTP kod — OK.
- T4: slaba parola — OK.
- T5: brute force (5 greshni opita) — OK.
- T6: nov admin s TOTP — OK.
- T7: leaver (disable) — OK.
- T8: password grant — OK.

## Konfiguratsii

- Password policy: `length(15) and notUsername(true)`.
- Brute force: `bruteForceProtected: true`, `failureFactor: 5`.
- MFA: `browserFlow: browser-mfa-admins`.
- Session timeout: `accessTokenLifespan` 300, `ssoSessionIdleTimeout` 1800, `ssoSessionMaxLifespan` 36000.
- Audit log: `kc-sabitiya` pokazva sabitiyata.
- Backup: `/tmp/gamestudio-neo-backup.json`.

## Dokazatelstva

- `kc-sabitiya gamestudio-neo 30` — pokazva LOGIN, LOGIN_ERROR, UPDATE_TOTP, REVOKE_GRANT i dr.
- `get realms/gamestudio-neo` — pokazva passwordPolicy, bruteForceProtected, browserFlow, session timeouts.
- `curl` za password grant — vrashta access_token i refresh_token.
- `curl` za refresh — vrashta novi tokeni.
- `curl` za revoke — uspeshno otmenyava token.
