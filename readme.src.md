# Titan Control System

## Cryochillers

To change the operating mode of the cryochillers, set `PC2500.DesiredOperatingMode` or `PC4000.DesiredOperatingMode`. It may take several seconds to switch to the new mode. There is not even a guarantee that the new mode will be accepted by the cryochiller. The current, actual operating mode of each cryochiller can be determined from an integer code that is stored in `PC2500.OperatingMode` and `PC4000.OperatingMode`:

Operating Mode | Code
-|-
Power off or no communication | -2
Inactive, compressor off | -1
Standby | 0
Cool | 1
Defrost | 2

### Cryochiller Safety

The control system will prevent the operator from making invalid transitions from one mode to the other. These are the valid transitions:

Current Mode | Allowed Transitions
-|-
Inactive | Standby
Standby | Inactive, Cool
Cool | Inactive, Standby, Defrost
Defrost | Inactive, Standby

TODO: Also prevent transitions from standby to cool if compressure suction pressure or temperature are not low enough, so that you have to run in standby for a while before switching to cool.


## Cooling Loop

When cooling down isopentane, condensing vapors can form a vacuum within the cooling loop. To prevent this, the control system injects nitrogen automatically. If `OutputDB.P-102 1` is on and `OutputDB.T-101 Vent` is closed, then `OutputDB.T-101 N2` will open as necessary to keep `InputDB.PT-104` greater than 1 psig. It will close again when the pressure is at least 1.5 psig.

`OutputDB.E-105` can optionally be manipulated with a PID loop to control the output temperature, `InputDB.TT-105`, to follow `PID E-105.Setpoint`. Set `PID E-105.Auto` to enable the controller. Use `PID E-105.Gain` and `PID E-105.TI` to tune the controller.

### Cooling Loop Safety

There are several valves downstream of P-102. If either of the discharge pressures of P-102, `InputDB.PT-102` or `InputDB.PT-104`, exceed 60 psig, then `OutputDB.P-102 1` will be immediately turned off.

To protect against the operator accidentally leaving `OutputDB.T-101 N2` open, it will automatically close if either `InputDB.PT-102` or `InputDB.PT-104` exceed 50 psig.

The control system protects against running the heater E-105 without flow. `OutputDB.P-102 1` must be on at a speed of 5% or greater, otherwise `OutputDB.E-105` will be set to 0%. If `PID E-105.Auto` is already on when `OutputDB.P-102 1` is started, the heater is allowed to begin ramping up automatically.

The maximum value of `PID E-105.Setpoint` is 0 °C. Additionally, if either `InputDB.TT-104` or `InputDB.TT-105` exceed 0 °C, then `OutputDB.E-105` will be set to 0%, even if `PID E-105.Auto` is on. The heater may ramp back up from 0% if the temperature drops again.

## Emergency Stop

When `SafetyDB.Stopped` is set, the following variables are set to these safe states:

Variable | Safe state
-|-
`PC2500.DesiredOperatingMode` | -1 (Inactive)
`PC2500.DesiredOperatingMode` | -1 (Inactive)
`OutputDB.P-102 1` | 0 (Off)
`OutputDB.E-105` | 0 % 
`PID E-105.Auto` | 0 (Off)
`OutputDB.T-101 N2` | 0 (Closed)
`OutputDB.T-101 Vent` | 0 (Closed)
