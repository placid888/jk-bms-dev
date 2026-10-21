# Start here in Codex

Work only on branch `codex/reconstruct-jk-bms`.

1. Read `CODEX_TASK.md` and `recovery_phone_overlay/RECOVERED_PHONE_CHANGES.md` first.
2. Use public upstream `TSDZ2-ESP32/TSDZ2-ESP32-Main` version 1.2.2 as the clean ESP32 baseline. The recovered phone backup also reports version `1.2.2`.
3. Compare the upstream source with the recovered JK overlay/evidence. Do not assume the recovered snippets are final or internally consistent.
4. First deliverable is `docs/RECOVERY_AUDIT.md`. Do not implement speculative protocol fields before the audit is committed.
5. After the audit, reconstruct the verified JK-BMS integration in small commits. Preserve original TSDZ2 Android BLE protocol, ESP32 OTA, STM8 OTA, configuration and screens.
6. The `recovery_phone_overlay` directory is evidence only; production code should live in the reconstructed upstream-compatible tree.

Important: `recovery_phone_overlay/main/jk_bms_ble.c` is a recovery copy for analysis. Validate it against the notes and source history before treating it as production code.