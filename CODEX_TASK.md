# JK-BMS / TSDZ2-ESP32 reconstruction task

## Goal
Reconstruct the lost later-stage TSDZ2-ESP32 + JK-BMS project safely, using the early phone backup as the baseline and recovered code fragments only as evidence. Do not blindly replace upstream/original TSDZ2 functionality with simplified recovery snippets.

## Known architecture
- Original TSDZ2 status payload is 39 bytes (indices 0..38).
- A recovered later Android fragment sets `STATUS_ADV_SIZE = 53` and parses JK summary data starting at byte 39.
- Recovered mapping:
  - 39..40: JK pack voltage, little-endian, /100 V
  - 41..42: JK current, signed 16-bit little-endian, /100 A
  - 43: SOC %
  - 44: FET temperature
  - 45: battery temperature 1
  - 46..47: max cell voltage mV
  - 48..49: min cell voltage mV
  - 50..52: unknown; do not invent meanings without evidence.
- The Android data model also contains fields for second battery temperature, balancing current/state, cycle count/capacity, total/remain capacity, active cell count, 17 cell voltages, and 17 balance-wire resistances. Those cannot all fit in the 53-byte status packet, so treat them as requiring a separate detailed-data transport unless source evidence proves otherwise.
- ESP32 <-> JK-BMS uses JK BLE service/characteristic FFE0/FFE1 in the early backup. Do NOT reuse FFE0/FFE1 as the Android <-> ESP32 service merely because a recovered Android stub did so.
- The original ESP32 <-> Android BLE side uses its own TSDZ2 service/characteristics. Preserve the original protocol/UUID definitions from the baseline source.

## Recovered Android fragments
The later fragments recovered from broken records include:
- `TSDZ_Status.java`
- `FragmentJkBattery.java`
- `MainPagerAdapter.java`
- `TSDZConst.java`
- `MainActivity.java`
- `MyFragmentListener.java`
- `FragmentStatus.java`
- `res/layout/activity_main.xml`
- `res/layout/fragment_status.xml`
- `AndroidManifest.xml`
- `app/build.gradle`
- `res/values/strings.xml`
- `BluetoothLeService.java`
- `res/values/colors.xml`
- `res/values/themes.xml` (or styles.xml)
- `settings.gradle`
- `gradle.properties`
- `proguard-rules.pro`

These are fragments from different points in development. There are known inconsistencies: theme naming differs between Manifest and theme resource; the recovered BluetoothLeService contains a TODO and likely incorrect FFE0/FFE1 UUIDs for ESP32-to-Android; FragmentStatus is a simplified replacement that could remove original TSDZ2 features; comments about 17-cell data fitting in 53 bytes are impossible and must not be trusted as protocol truth.

## Implementation plan
1. Import/restore the complete early phone backup as the baseline source tree before attempting feature reconstruction.
2. Make a clean baseline commit preserving that source unchanged as much as possible.
3. Audit the existing ESP32 BLE protocol and Android app protocol before modifying packet sizes or UUIDs.
4. Integrate JK-BMS parsing on ESP32 into the existing JK data structure. Preserve existing TSDZ2 behavior.
5. Implement the verified 53-byte summary status extension first, with explicit constants/packing and bounds checks on both ESP32 and Android.
6. Add tests or deterministic packet fixtures for the 39-byte legacy packet and 53-byte extended packet. Preserve backward compatibility if practical.
7. Implement detailed JK data (up to 17 cell voltages/resistances plus capacities/balancing data) using a separate, versioned transport/message. Do not overload the 53-byte summary packet.
8. Merge JK UI into the original Android app instead of replacing the original MainActivity/FragmentStatus wholesale. Preserve original BLE services, configuration, OTA, and existing screens.
9. Keep ESP32 OTA and STM8 OTA functionality intact.
10. Build/test each side and document any protocol assumptions that cannot be proven from source.

## Safety / source-of-truth rules
- Baseline source wins over comments in recovered fragments when they conflict.
- Do not guess UUIDs, byte meanings, STM8 protocol details, or hardware pins.
- Do not delete original functionality just to make the recovered fragments compile.
- Commit in small stages so recovery work cannot be lost again.
- Prefer a feature branch and PR; do not develop directly on `main`.

## First Codex task
Once the early phone backup source tree is present on this branch, inspect it and produce `docs/RECOVERY_AUDIT.md` containing:
1. project tree and build systems,
2. ESP32 <-> Android BLE UUIDs and packet definitions with source paths,
3. ESP32 <-> JK-BMS BLE implementation and parsed tags,
4. exact location/definition of the 39-byte status structure,
5. OTA paths for ESP32 and STM8,
6. proposed minimal file-by-file changes for the verified 53-byte JK summary extension,
7. unresolved questions for the detailed 17-cell transport.

Do not implement speculative protocol fields in this first task.