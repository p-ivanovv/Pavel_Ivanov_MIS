# 08 - Идентичност и достъп (Keycloak)

## Обобщение

Всичко от тест матрицата (T1-T8) е изпълнено и работи.

- T1: staff login - ОК.
- T2: TOTP за admin - ОК.
- T3: грешен TOTP код - ОК.
- T4: слаба парола - ОК.
- T5: brute force (5 грешни опита) - ОК.
- T6: нов admin с TOTP - ОК.
- T7: leaver (disable) - ОК.
- T8: password grant - ОК.

## Конфигурации

- Password policy: `length(15) and notUsername(true)`.
- Brute force: `bruteForceProtected: true`, `failureFactor: 5`.
- MFA: `browserFlow: browser-mfa-admins`.
- Session timeout: `accessTokenLifespan` 300, `ssoSessionIdleTimeout` 1800, `ssoSessionMaxLifespan` 36000.
- Audit log: `kc-sabitiya` показва събитията.
- Backup: `/tmp/gamestudio-neo-backup.json`.

## Доказателства

- `kc-sabitiya gamestudio-neo 30` - показва LOGIN, LOGIN_ERROR, UPDATE_TOTP, REVOKE_GRANT и др.
- `get realms/gamestudio-neo` - показва passwordPolicy, bruteForceProtected, browserFlow, session timeouts.
- `curl` за password grant - връща access_token и refresh_token.
- `curl` за refresh - връща нови tokeni.
- `curl` за revoke - успешно отменя token.
