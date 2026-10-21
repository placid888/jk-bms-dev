# Recovered phone backup changes

The uploaded phone backup reports `version.txt = 1.2.2`, matching the current public upstream `TSDZ2-ESP32/TSDZ2-ESP32-Main` version marker. Treat upstream as the clean baseline and this folder as recovery evidence/overlay, not as a new independent implementation.

A scan of the phone backup for JK-specific identifiers found JK-related changes in exactly these source paths:

- `main/jk_bms_ble.c` (new recovery module)
- `main/jk_bms_ble.h` (new recovery module)
- `main/main.c`
- `main/tsdz_data.h`
- `main/tsdz_data.c`

## Exact recovered additions in main.c

The phone copy includes:

```c
#include "jk_bms_ble.h"
```

and after `tsdz_tmp112_init();`:

```c
ESP_LOGI(TAG, "JK BMS init ...");
jk_bms_init();
```

## Exact recovered JK data structure in tsdz_data.h

```c
#pragma pack(1)
typedef struct _jk_bms_data {
    volatile uint16_t ui16_voltage_x100; // BMS 總電壓 (V * 100)
    volatile int16_t  i16_current_x100;  // BMS 總電流 (A * 100，正為充電，負為放電)
    volatile uint8_t  ui8_soc;           // 剩餘電量 (0-100%)
    volatile uint8_t  ui8_temp_fet;      // 功率管溫度
    volatile uint8_t  ui8_temp_bat;      // 電池組溫度
    volatile uint16_t ui16_cell_max_mv;  // 最高單體電壓 (mV)
    volatile uint16_t ui16_cell_min_mv;  // 最低單體電壓 (mV)
    volatile uint16_t ui16_alarms;       // 警報代碼
    volatile uint8_t  ui8_connected;     // 藍牙連線狀態 (0: 未連線, 1: 已連線)
} struct_jk_bms_data;
```

and later:

```c
// JK BMS data message
extern struct_jk_bms_data jk_bms_data;
```

## Exact recovered initialization in tsdz_data.c

```c
struct_jk_bms_data jk_bms_data = {
    .ui16_voltage_x100 = 0,
    .i16_current_x100 = 0,
    .ui8_soc = 0,
    .ui8_temp_fet = 0,
    .ui8_temp_bat = 0,
    .ui16_cell_max_mv = 0,
    .ui16_cell_min_mv = 0,
    .ui16_alarms = 0,
    .ui8_connected = 0
};
```

The phone copy also replaces the TSDZ battery voltage with JK voltage when JK is marked connected:

```c
// 若藍牙連線成功且電壓大於 0，將系統總電壓替換為 JK BMS 讀數 (BMS 解析度為 10mV，需乘 10 轉換為 1mV)
if (jk_bms_data.ui8_connected && jk_bms_data.ui16_voltage_x100 > 0) {
    tsdz_data.ui16_battery_voltage_x1000 = jk_bms_data.ui16_voltage_x100 * 10;

    // 若需同時替換電流，亦可解除下方註解
    // tsdz_data.ui8_battery_current_x10 = (uint8_t)(abs(jk_bms_data.i16_current_x100) / 10);
}
```

## Important inconsistency to audit

The recovered `jk_bms_ble.c` currently updates the separate globals `jk_bms_voltage`, `jk_bms_current`, and `jk_bms_soc`. It does not visibly populate `jk_bms_data` in the recovered copy. Therefore the `jk_bms_data.ui8_connected` voltage override path may never become active in this early snapshot. Codex should resolve this by comparing with upstream and then wiring parsed values into the canonical structure rather than creating duplicate state.

## JK BLE facts from recovered phone module

- Device name prefix: `JK-`
- Service UUID: `0xFFE0`
- RX/TX characteristic UUID: `0xFFE1`
- receive buffer: 512 bytes
- request starts `4E 57 00 13 ...`
- parser recognizes the adjacent tags `0x83`, `0x84`, `0x85` for voltage/current/SOC.
- SPIFFS recovery log path: `/spiffs/jk_log.txt`

Do not confuse these JK-side UUIDs with the TSDZ2 ESP32 server UUIDs used by the Android app.
