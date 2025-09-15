# MututallyHuman ESP IDF v5.5.1 Changes

The following is an analysis of the changes to [ESP IDF](https://github.com/espressif/esp-idf) `v5.5.1` in this [MutuallyHuman fork](https://github.com/mhs/esp-idf) on branch `mhs_changes_v5.5.1`. The purpose is to categorize the changes into features and describe their purpose and usage in the [Callbox Embedded Starter Kit](https://github.com/msx/callbox-embedded/). The analysis was originally performed for the `spindance_changes_v5.0` branch against the Espressif `v5.0` tag and has been updated after porting the MutuallyHuman branch to v5.5.1.

Because the esp-idf uprevs from 4.x to 5.0 and from 5.0 to 5.5.1 used squashed patch generation,
the `spindance_changes_v5.0`  and `mhs_changes_v5.5.1` branches do not include specific commits
for each MutuallyHuman fork feature. To help provide context, the feature sections include pointers to the original commits where possible (usually on a 4.x branch).

A handy way to view the diff is via this [Github pull request link](TBD).  // FIXME

The original ESP repo and this MutuallyHuman fork can be cloned and viewed separately via:
```
git clone git@github.com:espressif/esp-idf.git esp-idf
cd esp-idf
git checkout v5.5.1

cd ..
git clone git@github.com:mhs/esp-idf.git callbox-embedded
cd callbox-embedded
git checkout mhs_changes_v5.5.1
```

## Changed Files

```(text)
.github/workflows/docker.yml            // DELETED
MutuallyHuman_Fork_Changes.md           // Renamed from SpinDance_Fork_Changes.md
components/esp_netif/include/lwip/esp_netif_net_stack.h  (was components/lwip/port/esp32/include/netif/wlanif.h before v5.0)
components/esp_netif/lwip/netif/wlanif.c
components/freertos/CMakeLists.txt
components/json/json.mk
components/protocomm/include/security/protocomm_security.h
components/protocomm/include/transports/protocomm_ble.h
components/protocomm/src/common/protocomm.c
components/protocomm/src/security/security1.c
components/protocomm/src/transports/protocomm_nimble.c
components/wifi_provisioning/include/wifi_provisioning/manager.h
components/wifi_provisioning/include/wifi_provisioning/wifi_config.h
components/wifi_provisioning/include/wifi_provisioning/wifi_scan.h
components/wifi_provisioning/proto-c/wifi_config.pb-c.c
components/wifi_provisioning/proto-c/wifi_config.pb-c.h
components/wifi_provisioning/proto-c/wifi_scan.pb-c.c
components/wifi_provisioning/proto-c/wifi_scan.pb-c.h
components/wifi_provisioning/proto/wifi_config.proto
components/wifi_provisioning/proto/wifi_scan.proto
components/wifi_provisioning/python/wifi_config_pb2.py
components/wifi_provisioning/python/wifi_constants_pb2.py
components/wifi_provisioning/python/wifi_scan_pb2.py
components/wifi_provisioning/src/manager.c
components/wifi_provisioning/src/wifi_config.c
components/wifi_provisioning/src/wifi_provisioning_priv.h
components/wifi_provisioning/src/wifi_scan.c
tools/idf_tools.py
```

## Feature List
The following is a list of the MutuallyHuman Embedded Starter Kit features that the changes in this fork support:
- Network (NW) Metrics Reporting
- Protocomm BLE Connectivity Reporting
- Provisioning State in BLE Advertisement Data (AD)
- JWT Authorization for Protocomm WiFi Provisioning
- WPA2 Enterprise NW Support
- WiFi Provisioning Sequence Changes
- Python Version Change
- WICED build system support

## Feature Details
The features, their associated changes to ESP IDF and how the change is related to the Embedded Starter Kit are described in more detail below:
- **Network (NW) Metrics Reporting**
  - Added support for simple bytes in/bytes out tracking and reporting in ESP's LWIP implementation
  - Embedded Starter Kit accesses these values via its PAL and includes them in metrics reported to MQTT
  - Impacted ESP IDF files:
    - esp_netif_net_stack.h
    - wlanif.c
  - Original Commits:
    - [lwip/port/esp32: add metrics for wlan bytes in/out](https://github.com/mhs/esp-idf/commit/ef9b870d0dad29b4993815c323c57202f5c1700b)
- **Protocomm BLE Connectivity Reporting**
  - Added reporting of BLE peer connectivity state (connected, connected securely, not connected)
  - Embedded Starter Kit reports these states to analagous pubsub topic IDs in the pubsub namespace `WIFI_TOPIC_NS`
  - Note: We'd likely want to implement this for `Security2` if/when we migrate to ESP IDF v5. Used to display LEDs of different colors depending upon the state.
  - Note 2:  The esp-idf v5.0 added a new `protocomm_transport_ble_event_t` type that had two out of the 3 events that we added in our fork change to add `ble_event`. Since they did NOT include the secure connection event, we continue to use our fork changes. As a result, you'll see a little duplication of events, e.g. in `protocomm_nimble.c`, between our ble_event and their protocomm_transport_ble_event_t. We should keep an eye on the evolution of protocomm_transport_ble_event_t and see if they ever add the secure connection, at which point we would be able to undo our fork changes and be closer to the unmodified esp-idf.
  - Impacted ESP IDF files:
    - protocomm_security.h
    - protocomm_ble.h
    - protocomm.c
    - security1.c
    - protocomm_nimble.c
  - Original Commits:
    - [mutually human changes for wifi_provisioning](https://github.com/mhs/esp-idf/commit/56c743a69cf9dce0bf4ce4eab4048a2c1088fdee)
- **Provisioning State in BLE Advertisement Data (AD)**
  - Added ability to directly modify the manufacturer specific data in the BLE advertisement.
  - Embedded Starter Kit uses this to include the WiFi provisioning state in the advertisement
  - Note: It maybe possible that the ESP IDF provides a different way to modify the advertisement.
  - Impacted ESP IDF files:
    - protocomm_ble.h
    - protocomm_nimble.c
  - Original Commits:
    - [mutually human changes for wifi_provisioning](https://github.com/mhs/esp-idf/commit/56c743a69cf9dce0bf4ce4eab4048a2c1088fdee)
- **JWT Authorization for Protocomm WiFi Provisioning**
  - Added an auth token property to the Protocomm protobuf messages, which is supplied to an also added optional authorization callback for validation prior to scanning for or configuring a WiFi access point.
  - Embedded Starter Kit registers an authorization handler that validates the token as a JWT. This feature is disabled in the WiFi configuration in `devkit`.
  - Note: This was added to validate device claiming.
  - Impacted ESP IDF files:
    - Protobuf definition files:
      - wifi_config.proto
      - wifi_scan.proto
      - wifi_config.pb-c.h
      - wifi_config.pb-c.c
      - wifi_scan.pb-c.c
      - wifi_scan.pb-c.h
      - wifi_config_pb2.py
      - wifi_constants_pb2.py
      - wifi_scan_pb2.py
    - manager.h
    - manager.c
    - wifi_config.h
    - wifi_config.c
    - wifi_scan.h
    - wifi_scan.c
  - Original Commits:
    - [mutually human changes for wifi_provisioning](https://github.com/mhs/esp-idf/commit/56c743a69cf9dce0bf4ce4eab4048a2c1088fdee)
- **WPA2 Enterprise NW Support**
  - Existing internal IDF helper functions that provide information about scanned WiFi access points were made public by moving their declarations to a public IDF header file.
  - Embedded Starter Kit uses these functions to determine if the access point attempting to be configured is a WPA2 enterprise NW, and then enables or disables support for WPA2 enterprise.
  - Note: It's possible that a newer version of ESP IDF may sufficiently support WPA2 enterprise. _Also, it's possible that WPA2 enterprise NW support inside the Starter Kit's `wifi` component is not tested or fully functional._
  - Impacted ESP IDF files:
    - manager.h
    - wifi_provisioning_priv.h
  - Original Commits:
    - [mutually human changes for wifi_provisioning](https://github.com/mhs/esp-idf/commit/56c743a69cf9dce0bf4ce4eab4048a2c1088fdee)
- **WiFi Provisioning Sequence Changes**
  - `WIFI_PROV_SCAN_STARTED` was added to the `wifi_prov_cb_event_t` enum and is reported via the `app_event_handler` in manager.c once a WiFi access point scan is started.
    - Embedded Starter kit wifi.c listens for `WIFI_PROV_SCAN_STARTED`, at which point it sets a boolean flag `_provisioning_in_progress` indicating WiFi is being used to scan for and configure an access points. It also disconnects wifi via `esp_wifi_disconnect()`, which happens to be one of the calls that is removed in manager.c `wifi_prov_mgr_start_provisioning()`, as discussed below.
  - Code was removed in manager.c `wifi_prov_mgr_start_provisioning()` to remove steps from the setup when WiFi provisioning is started.
    - Removed "Start Wi-Fi in Station Mode":
      - Removed calls: `esp_wifi_set_mode(WIFI_MODE_STA); esp_wifi_start();`
      - Embedded Starter Kit calls these two functions once, at startup, in `wifi_init()`. It's not clear if calling these again in `wifi_prov_mgr_start_provisioning()` is therefore necessary (apparently it is not), _nor is it clear if calling them again by re-including the removed calls would be problematic_.
    - Removed "Change Wi-Fi storage to RAM temporarily and erase any old credentials in RAM"
      - Removed calls: `esp_wifi_set_storage(WIFI_STORAGE_RAM); esp_wifi_set_config(WIFI_IF_STA, &wifi_cfg_empty);`
      - Based upon the comments, this appears to be a sort of back door means of preventing the **application code** from attempting to auto-reconnect when it receives `WIFI_EVENT_STA_DISCONNECTED` due to the call to `esp_wifi_disconnect()` discussed below. Presumably the auto-reconnect cannot occur or won't succeed due to there being no stored access point credentials.
      - Embedded Starter Kit wifi.c appears to manage not attempting to reconnect during provisioning. It controls when it calls `esp_wifi_connect()` when it receives `WIFI_EVENT_STA_DISCONNECTED`, only calling it if `_provisioning_in_progress` is false and WiFi is not actively being disconnected from.
    - Removed "Disconnect to make sure device doesn't remain connected to the AP whose credentials were present earlier"
      - Removed call: `esp_wifi_disconnect();`
      - Embedded Starter Kit calls this function when it receives the `WIFI_PROV_SCAN_STARTED` event, as discussed above.
  - Impacted ESP IDF files:
    - manager.h
    - manager.c
  - Original Commits:
    - [mutually human changes for wifi_provisioning](https://github.com/mhs/esp-idf/commit/56c743a69cf9dce0bf4ce4eab4048a2c1088fdee)
- **Python Version Change**
  - idf_tools.py was modified to use `python3` instead of the Mac default (`python2`)
  - Impacted ESP IDF files:
    - idf_tools.py
  - Original Commits:
    - [mutually human changes for wifi_provisioning](https://github.com/mhs/esp-idf/commit/56c743a69cf9dce0bf4ce4eab4048a2c1088fdee)

- **WICED Build System Support Changes**
  - components/freertos/CMakeLists.txt was modified to include a path needed by the WICED build
  - components/json/json.mk was added to provide a pure makefile build of the cJSON component used in the example WICED build.

## Feature / File Association

| File                                                                 | SK Feature(s) |
|:-------------------------------------------------------------------- | -------------- |
|components/lwip/port/esp32/include/esp_netif_net_stack.h              | NW Metrics Reporting |
|components/lwip/port/esp32/netif/wlanif.c                             | NW Metrics Reporting |
| components/freertos/CMakeLists.txt                                   | WICED Build System Support Changes |
| components/json/json.mk                                              | WICED Build System Support Changes |
|components/protocomm/include/security/protocomm_security.h            | Protocomm BLE Connectivity Reporting |
|components/protocomm/include/transports/protocomm_ble.h               | Protocomm BLE Connectivity Reporting, Provisioning State in BLE AD |
|components/protocomm/src/common/protocomm.c                           | Protocomm BLE Connectivity Reporting |
|components/protocomm/src/security/security1.c                         | Protocomm BLE Connectivity Reporting |
|components/protocomm/src/transports/protocomm_nimble.c                | Protocomm BLE Connectivity Reporting, Provisioning State in BLE AD |
|components/wifi_provisioning/include/wifi_provisioning/manager.h      | WiFi Provisioning Sequence Changes, JWT Authorization for Protocomm WiFi provisioning, WPA2 Enterprise NW Support |
|components/wifi_provisioning/include/wifi_provisioning/wifi_config.h  | JWT Authorization for Protocomm WiFi Provisioning |
|components/wifi_provisioning/include/wifi_provisioning/wifi_scan.h    | JWT Authorization for Protocomm WiFi Provisioning |
|components/wifi_provisioning/proto/wifi_config.proto                  | JWT Authorization for Protocomm WiFi Provisioning |
|components/wifi_provisioning/proto/wifi_scan.proto                    | JWT Authorization for Protocomm WiFi Provisioning |
|components/wifi_provisioning/proto-c/wifi_config.pb-c.c               | JWT Authorization for Protocomm WiFi Provisioning |
|components/wifi_provisioning/proto-c/wifi_config.pb-c.h               | JWT Authorization for Protocomm WiFi Provisioning |
|components/wifi_provisioning/proto-c/wifi_scan.pb-c.c                 | JWT Authorization for Protocomm WiFi Provisioning |
|components/wifi_provisioning/proto-c/wifi_scan.pb-c.h                 | JWT Authorization for Protocomm WiFi Provisioning |
|components/wifi_provisioning/python/wifi_config_pb2.py                | JWT Authorization for Protocomm WiFi Provisioning |
|components/wifi_provisioning/python/wifi_constants_pb2.py             | JWT Authorization for Protocomm WiFi Provisioning |
|components/wifi_provisioning/python/wifi_scan_pb2.py                  | JWT Authorization for Protocomm WiFi Provisioning |
|components/wifi_provisioning/src/manager.c                            | JWT Authorization for Protocomm WiFi Provisioning, WiFi Provisioning Sequence Changes |
|components/wifi_provisioning/src/wifi_config.c                        | JWT Authorization for Protocomm WiFi Provisioning |
|components/wifi_provisioning/src/wifi_provisioning_priv.h             | WPA2 Enterprise NW Support |
|components/wifi_provisioning/src/wifi_scan.c                          | JWT Authorization for Protocomm WiFi Provisioning |
|tools/idf_tools.py                                                    | Python Version Change |

# Comparison of the v5.0 and v5.5.1 fork changes

After generating the new branch `mhs_changes_v5.5.1`, we did a comparison of the diffs
1. Diff `mhs_changes_v5.5.1` branch  vs `v5.5.1 ` tag
1. Diff `spindance_changes_v5.0` branch vs `v5.0` tag
to ensure the update was complete.

## Summary

For most files, the changes were applied exactly the same in both diffs. But for a small number of files there were minor differences. The only possibly functional difference is the addition of authorization checks inside `wifi_config.c` and `wifi_scan.c`. Since JWT authentication is empirically working during testing, the addition of those extra checks seems safe.

## Details for the diff of diffs

Here are the files that show non-trivial differences between the two diffs along with some notes.

| File                                                                 | Notes |
|:-------------------------------------------------------------------- | -------------- |
| protocomm_nimble.c |  The changes to report PROTOCOMM_BLE_PEER_DISCONNECTED and PROTOCOMM_BLE_PEER_CONNECTED events applied exactly. </br>Note that v5.0 added `protocomm_transport_ble_event_t` type that had two out of the 3 events that we added in our fork change to add `ble_event`. But since they did NOT include the secure connection event, we continue to use our fork changes. As a result, you'll see a little duplication of events in this file between our `ble_event` and their `protocomm_transport_ble_event_t`. </br>We should keep an eye on the evolution of `protocomm_transport_ble_event_t` and see if Espressif ever adds the secure connection, at which point we would be able to undo our fork changes and be closer to the unmodified esp-idf. |
| wifi_constants_pb2.py | Mostly changes applied exactly, including the "WifiConnectedState = _reflection.GeneratedProtocolMessageType" paragraph. But the v5.5.1 port added a "WifiAttemptFailed = _reflection.GeneratedProtocolMessageType  paragraph as needed in the v5.5.1 code. |
|  manager.c | The code structure around our line `ESP_LOGI(TAG, "Delaying %lu ms", cleanup_delay);` changed but the logging was added into the new structure successfully. Otherwise, all changes applied exactly. |
|  wifi_config.c | Our v5.0 fork added to the `wifi_prov_config_data_handler` function the code paragraph "Authorize before dispatching command". And that was applied exactly in the new branch. But in creating our v5.5.1 fork, claude code added to the `cmd_set_config_handler` function the code paragraph "Check authorization if callback is set" |
|  wifi_scan.c | Similar to `wifi_config.c`, our v5.0 fork added to the `wifi_prov_scan_handler`  function the code paragraph "Authorize before dispatching command". And that was applied exactly in the new branch. But in creating our v5.5.1 fork, claude code added to `cmd_scan_start_handler`  function the code paragraph "Check authorization if callback is set". |
