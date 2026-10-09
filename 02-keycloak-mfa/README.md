# Lab 2: MFA and brute-force protection in Keycloak

**Goal:** enforce TOTP MFA, then test how the realm behaves under wrong, replayed and brute-forced codes.

## What I did
- Duplicated the default browser flow, made the OTP step **Required**, and bound it as the realm's browser flow.
- Turned on user event logging.
- Logged in as a test user: registered an authenticator app, then logged in with password + OTP.
- Entered wrong OTPs with brute-force detection **off** (no lockout at all), then **on** (temporary lockout after 3 failures, 5-minute wait).
- Unlocked the user manually from the admin console.

## What I observed
| Step | Screenshot |
|---|---|
| OTP step set to Required | [01](screenshots/01-otp-required.png) |
| Custom MFA flow bound as browser flow | [02](screenshots/02-mfa-flow-bound.png) |
| Wrong OTP rejected | [03](screenshots/03-invalid-otp.png) |
| Brute-force detection settings | [04](screenshots/04-brute-force-settings.png) |
| Test user shown as temporarily locked | [05](screenshots/05-user-temporarily-locked.png) |
| `LOGIN_ERROR` events in the event log | [06](screenshots/06-login-error-events.png) |

## Takeaway
MFA stops a stolen password, but without brute-force detection a 6-digit code can simply be guessed. Lockout events are also a detection signal: a burst of `LOGIN_ERROR` for one user is worth alerting on.

## Attacker view / Defender view
| | |
|---|---|
| **Attack** | Guessing or spraying passwords and OTP codes. **Brute Force, T1110.** A 6-digit code is only 1,000,000 combinations, so without a limit it can be guessed. |
| **What this lab showed** | With brute-force detection off, repeated wrong OTPs never locked the account. With it on, 3 failures caused a temporary lockout and Keycloak logged `LOGIN_ERROR` events. |
| **What I would monitor** | Bursts of `LOGIN_ERROR` for one user (guessing) or across many users from one IP (spraying). A lockout followed by a success from a different IP. |
| **Hardening I would add** | Keep lockout on with increasing wait times. Ship Keycloak events to a SIEM: the lab kept them for only 15 minutes, which is far too short for an investigation. |
