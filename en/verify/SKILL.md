---
name: verify
description: Truly verifies that a change works before calling it done. Use it when creating or modifying apps (Flutter/Android), server configurations, Docker, Frigate, Home Assistant, n8n, or any home security and automation project. It forces real checks and evidence instead of claiming "it should work".
---

# Skill: verify

## Principle
You are not the verifier by intuition: you are the verifier by EVIDENCE.
Never say "it's fixed", "it should work" or "I found the problem" without having run a real check and seen its result.
If you cannot run something, say so and mark that point as NOT VERIFIED.

## Fixed protocol (always the same 6 steps, in this order)

1. **Identify the project and its profile.** Decide which applies (see "Profiles" below). If unclear, ask once.
2. **Read before claiming.** Open the real files and configuration affected. Do not assume their contents. If you are going to say "the fault is in X", first show that you read X.
3. **Define the success criterion.** Write in one or two lines what "it works" means for this change (e.g.: "the app builds and the settings screen opens without errors").
4. **Run the profile's checks.** Use real commands (not invented ones). If a command fails, include it as-is in the report.
5. **Test the case that used to fail.** Reproduce the original problem and confirm it no longer happens. If you cannot reproduce it, say so.
6. **Report using the mandatory format** (see below).

## Mandatory report format

```
VERIFICATION
Project: <name/type>
Success criterion: <one line>

Checks run:
- [OK] <command or test> → <relevant result>
- [FAIL] <command or test> → <error>
- [NOT VERIFIED] <what is missing and why>

What I read: <files/configs reviewed>
Conclusion: VERIFIED / PARTIAL / NOT VERIFIED
Pending risks: <short list or "none known">
Recommended next step: <one line>
```

Format rules:
- Only mark VERIFIED if ALL critical checks are OK.
- Quote errors literally and briefly; do not summarize them from memory.
- Separate what was checked from what was assumed. Assumptions go as "I assume", never as fact.

## Profile A: apps (Flutter / Android)

Checks, from cheapest to most expensive (stop at the first one that fails):
1. `flutter analyze` → no errors (list any warnings).
2. `flutter test` → all tests pass. If there are no tests for what changed, say so and propose one.
3. `flutter build apk --debug` (or `./gradlew assembleDebug` for native Android) → it builds.
4. If an emulator/device is available: `flutter run` or install the APK and test the affected flow.
5. Review `adb logcat` (filtering by the app's package) looking for exceptions when opening the touched screen.

Extras depending on the case:
- Dependency changes: `flutter pub get` without conflicts; check the minimum SDK version.
- Permission or notification changes: review `AndroidManifest.xml` and test on a device with a recent Android version.
- Before uploading to Google Play: signed release build and check that `versionCode` was incremented.
- If the change touches data storage: test with existing data (migration), not just a clean install.

## Profile B: home security and automation

Golden rule: **read-only first**. Do not change production configuration without (a) a backup, (b) explicit confirmation from the user and (c) a rollback plan.

Typical checks (adapt to the real installation; do not assume a command exists, check it):
- **Docker / docker-compose:** `docker compose config` (validates syntax), `docker compose ps` (status), `docker logs --tail 100 <container>` (recent errors). Review volume paths and ports before bringing anything up.
- **Frigate:** validate the configuration file with Frigate's own tool if there is one; check in the logs that each camera connects; confirm detection is active and recordings are being written to disk. Check CPU usage if detection runs on CPU.
- **Cameras:** confirm the RTSP stream responds (for example with `ffprobe` on the URL, without showing credentials in the report). Review the switch's PoE budget (sum of camera consumption versus total power).
- **Home Assistant:** run the configuration check before restarting; afterwards review the log for integration errors.
- **n8n or other automations:** run the flow in test mode with sample data and check each node's output; confirm time zone and credentials.
- **NAS, network and UPS:** check connectivity (`ping`, open ports), free disk space, and UPS status (for example with `upsc` if it uses NUT). After a real or simulated power cut, confirm that the orderly shutdown works.
- **Security:** do not expose cameras, panels or the NAS to the internet without strong authentication and, preferably, a VPN. Review open ports and the system's own alerts. Never include passwords, tokens or URLs with credentials in the report.

For any network or firewall change: check beforehand that you will not lock yourself out (keep a way back in).

## What to do when it cannot be verified
- Say exactly what you could not check and why (no access to the device, no network, no permissions).
- Propose the specific command or test the user can run and what result to expect.
- Mark the conclusion as PARTIAL or NOT VERIFIED. Do not fill gaps with assumptions.

## Anti-patterns (forbidden)
- Saying "it works now" without running anything.
- Giving the cause of a failure without having read the relevant code or log.
- Inventing names of commands, options, files or ports. If you are not sure, check or ask.
- Changing several things at once: one change, one verification.
- Hiding failed checks from the report.

## Maintaining this skill
Every time a failure slips through (something "verified" that later fails), add a new check here that would have caught it. This list improves with real errors.
