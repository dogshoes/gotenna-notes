## The goTenna

The goTenna is a neat little device which permits peer to peer communication over distances without the need for an established wireless backbone (cell phone network) or radio license.  Kind of neat.  Here are some of my notes on the device.

## Connecting

**Connecting to your goTenna outside the official app will probably brick your unit.  It might cause it to launch off your desk and off in to space.  Who knows.  Read this document at your own risk.**

Connect the goTenna over USB to any desktop computer and it shows up as a "goTenna" (swell!), and exposes a serial port over USB.  Connecting to this serial port at 9600 baud gives us an undocumented CLI.  Neat-o.  I can't determine whether this mode is purely diagnostic or whether it might be possible to fully control the goTenna.

Periodically send a message over the serial port or else you'll get this message and the serial port will drop!

```
[1085000-18984] USB Asleep
```

Send a CR to get a `goTenna>` prompt.

### The Periodic Event

Every minute (?) goTenna outputs diagnostic information about the current state.

```
[984000-40984] Periodic Event
[984000-000] Batt Voltage : 3981, selected pap: 6
[984015-015] scan rssi[0=81  1=76  2=0  3=91  4=75]
[984016-001] MAC bg_rssi=73
```

Battery voltage is what it says on the tin.  PAP appears to be "PA Power", refer to the `pap` command documented below.  [RSSI](https://en.wikipedia.org/wiki/Received_signal_strength_indication) is a pretty familiar RF concept.

### Version Command

Show the goTenna firmware version.

```
[095843] goTenna> version
[121435] goTenna> [121445-39429] cli_parse_command >> version
[121445-000] FW Version : 00.12.02
[121445-000] Build Date : Oct 22 2015 13:05:48

[121445-000] Boot Version: 01.07
[121445-000] printer: cli-cmd = version

```

### RSSI Command

Get information about the RSSI.

```
[165182] goTenna> rssi
[166444] goTenna>
[166454-2438] cli_parse_command >> rssi
[166454-000] CLI cmd=RSSI
[166454-000] printer: cli-cmd = rssi
[166475-021] log received.
[166490-015] RSSI :
Channel 0 RSSI 85
Channel 1 RSSI 70
Channel 2 RSSI 83
Channel 3 RSSI 80
Channel 4 RSSI 82
```

### PAP Command

Set the PA Power (PAP).  Not sure what this is, or whether it's a great idea to change it.  However, changing it doesn't seem to change the "selected pap" value that appears in the periodic event. 

```
[364773] goTenna> pap 2
[366776] goTenna> [366786-2745] cli_parse_command >> pap 2
[366786-000] PA Power value changed to 2
[366786-000] printer: cli-cmd = pap 2
```

### Batt Command

Get information about the battery.

```
[541125] goTenna> batt
[544393] goTenna>
[544404-4274] cli_parse_command >> batt
[544405-001] Raw Value :40627 Battery= 4.091
[544405-000] printer: cli-cmd = batt
```

### Flash Command

Perform a flash check?  I'm a bit afraid of this command.

```
[650554] goTenna> flash
[651526] goTenna> 
[651536-21415] cli_parse_command >> flash
[651536-000] Flash check passed.
[651536-000] printer: cli-cmd = flash
```

### Power Command

Seems to immediately reboot the device.  Does not return any output.  Seems to disregard arguments.

```
[946820] goTenna> power
[948519] goTenna>
```

### Random Observed Messages

This is a pile of things to look at later.

```
[000022-022] PA ------------ off
[000032-010] TRX Current State: STATE_POWER_UP *
[000093-061] Msg Add: 19XXX0, Wrt Msg Add: 19XXX8, Del Msg Num: 0, Total Msg Nu1
[000095-00
```

How can we read this diag block?  Is there wear leveling in place or does this block just get constantly overwritten?

```

Write Diag Block, 433 CRC: 56364

Write Diag Block, 434 CRC: 56364
[330141-2125] Storing Diag Info in Flash
```