## LHandProLib + IgH EtherCAT Integration
### Background
The **LHandProLib** library provides a callback-based architecture that can work with any *EtherCAT* master:

* `lhandprolib_set_send_rpdo_callback()` - register callback to send PDO data
* `lhandprolib_set_tpdo_data_decode()` - pass received PDO data for parsing

This allows integrating with **IgH** *EtherCAT* master instead of the default **SOEM**.

### Benefits

Aspect |	Current `ls_hand.c`  |  New `ls_hand_pro.c`
----|----|----
Protocol parsing |	Manual |	LHandProLib handles it
Sensor data |	Basic |	Rich API (normal, tangential, proximity)
Motor control |	Raw PDO |	High-level API (angles, velocity, homing)
Finger identification |	Generic |	Named (thumb, index, etc.)

### Proposed Changes

#### [NEW] `ls_hand_pro.c`

New source file that combines:
1. **IgH EtherCAT master** - for real-time PDO exchange
2. **LHandProLib** - for protocol parsing and high-level control

Key integration points:
 - Cyclic task calls lhandprolib_get_pre_send_rpdo_data() and writes to EtherCAT 
 - Cyclic task reads EtherCAT and calls `lhandprolib_set_tpdo_data_decode()`
 - RPDO callback bridges library to IgH master

#### [MODIFY] `Makefile`
Add build target for ls_hand_pro with LHandProLib linking.

#### Verification Plan
1. Build with make ls_hand_pro
2. Run and verify device enters OP state
3. Test enable/disable through LHandProLib API
4. Read sensor values using library functions
