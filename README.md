## The goTenna

The goTenna is a neat little device which permits peer to peer communication over distances without the need for an established wireless backbone (cell phone network) or radio license.  Kind of neat.  Here are some of my notes on the device.

## Connecting

**Connecting to your goTenna outside the official app will probably brick your unit.  It might cause it to launch off your desk and off in to space.  Who knows.  Read this document at your own risk.**

Connect the goTenna over USB to any desktop computer and it shows up as a "goTenna" (swell!), and exposes a serial port over USB.  The driver seems to load in Windows 10, but not Windows 7.  Linux is happy to load a generic ACM driver (like TTY but for modems.)  Connecting to this serial port at 9600 baud gives us an undocumented CLI.  I can't yet determine whether this mode is purely diagnostic or whether it might be possible to fully control the goTenna.

Periodically send a message over the serial port or else you'll get this message and the serial port will drop!

```
[543600-20940]  USB   USB Asleep
```

### The Periodic Event

Every minute (?) goTenna outputs diagnostic information about the current state.

```
[522600-40141]  TRX   Periodic Event
[522600-000]  TRX   Batt Voltage : 4141, selected pap: 5
[522603-003]  TRX   RSSI: ch=0, rssi=78
[522606-003]  TRX   RSSI: ch=1, rssi=84
[522609-003]  TRX   RSSI: ch=2, rssi=81
[522612-003]  TRX   RSSI: ch=3, rssi=74
[522615-003]  TRX   Min rssi 74
[522615-000]  TRX   Bg index val : 3 bin 0 rssi=75  bin 1 rssi=68  bin 2 rssi=74  bin 3 rssi=83  bin 4 rssi=82  bin 5 rssi=78  bin 6 rssi=67  bin 7 rssi=76
[522616-001]  TRX   MAC bg_rssi=75
[522616-000]  TRX   MAC Current State: MAC_PERIODIC_AWAKE_IDLE*
[522617-001]  TRX   TRX channel id : 4
[522619-002]  TRX   TRX Current State: PDU_IDLE_LONG *
[522619-000]  TRX   PA --- OFF
[522640-021]  TRX   MAC Expired State: MAC_PERIODIC_AWAKE_IDLE*
[522640-000]  TRX   Batt Voltage : 4140, selected pap: 5
[522643-003]  TRX   RSSI: ch=0, rssi=74
[522646-003]  TRX   RSSI: ch=1, rssi=76
[522649-003]  TRX   RSSI: ch=2, rssi=78
[522652-003]  TRX   RSSI: ch=3, rssi=65
[522655-003]  TRX   Min rssi 69
[522655-000]  TRX   Bg index val : 4 bin 0 rssi=75  bin 1 rssi=68  bin 2 rssi=74  bin 3 rssi=69  bin 4 rssi=82  bin 5 rssi=78  bin 6 rssi=67  bin 7 rssi=76
[522656-001]  TRX   MAC bg_rssi=73
[522656-000]  TRX   HDNA Mode 0, advertised 0, observed 0
[522657-001]  TRX   Sleep TRX Chip for LDC mode
[522657-000]  TRX   MAC Current State: Idle*
[522658-001]  TRX   TRX channel id : 4
[522660-002]  TRX   TRX Current State: PDU_IDLE_LONG *
[522660-000]  TRX   PA --- OFF
```

Battery voltage is what it says on the tin.  PAP appears to be "PA Power", refer to the `pap` command documented below.  [RSSI](https://en.wikipedia.org/wiki/Received_signal_strength_indication) is a pretty familiar RF concept.

### CLI Behavior

The CLI seems to accept some valid commands with any extraneous trailing characters.  For example, the `version` command can be invoked by `versionxy`, or `version;`, or `version;temp;`.  Some commands will default given no arguments (`pap` for example), but some commands will report as unrecongnized unless an argument is passed (`chn` and `ctx` to name a few).

As of the latest firmware release, 0.23.2, the `goTenna>` prompt is no longer displayed.

### CLI Commands

### version

Show the goTenna firmware version.  The version reported by the CLI is hexidecimally encoded (0x00, 0x12, 0x02), while the verion reported in the application is convered to its numerical equivalent (0, 21, 4).

| Observed version | Notes |
|------------------|-------| 
| 00.12.02         | First generation firmware. |
| 00.15.04         | Included in 1.12 of the goTenna app. |
| 00.17.02         | Included in 1.23 of the goTenna app. |

```
version
```

```
[224521-23461]  PRNT  FW Version : 00.17.02
[224521-000]  PRNT  Build Date : Feb 18 2016 13:40:31

[224522-001]  PRNT  Boot Version: 01.07
[224522-000]  PRNT  Boot flag: aa. Firmware CRC: d06f2af1
```

#### rssi

Get the [RSSI](https://en.wikipedia.org/wiki/Received_signal_strength_indication) of the five [MURS](https://en.wikipedia.org/wiki/Multi-Use_Radio_Service) channels used for communication.

Firmware versions prior to 00.23.2 reported only the first five channels.

```
rssi
```

```
[123303-2645]  PRNT  CLI cmd=RSSI
[123314-011]  PRNT  log received.
[123327-013]  TRX   RSSI:
Channel 0 RSSI 79
Channel 1 RSSI 77
Channel 2 RSSI 89
Channel 3 RSSI 77
Channel 4 RSSI 77
Channel 5 RSSI 1937006955
Channel 6 RSSI 1937006955
Channel 7 RSSI 1937006955
Channel 8 RSSI 1937006955
Channel 9 RSSI 1937006955
Channel 10 RSSI 1937006955
Channel 11 RSSI 1937006955
Channel 12 RSSI 1937006955
Channel 13 RSSI 1937006955
Channel 14 RSSI 1937006955
Channel 15 RSSI 1937006955
```

#### pap [val]

Set the PA Power (PAP).  Not sure what this is, or whether it's a great idea to change it.  However, changing it doesn't seem to change the "selected pap" value that appears in the periodic event.   The largest PAP that can be entered is 255.  Larger values will be truncated to 255.

```
[364773] goTenna> pap 2
[366776] goTenna> 
[366786-2745] cli_parse_command >> pap 2
[366786-000] PA Power value changed to 2
[366786-000] printer: cli-cmd = pap 2
```

#### batt

Get information about the battery.  Issuing the command as `battery` works just as well, but this could be due to the cli ignoring extraneous trailing characters.  The values this command gives are sometimes wonky, so several readings should be taken any outliers should be discarded.

```
batt
```

```
[073972-33712]  PRNT  Raw Value :41269 Battery= 4.156
```

#### flash

Perform a flash check?  I'm a bit afraid of this command.

```
flash
```

```
[091966-11507]  PRNT  Flash check passed.
```

#### power

Seems to immediately reboot the device.  Does not return any output.  Your serial connection will drop as the USB device disconnects and reconnects.

```
power
```

#### log

Not at all sure what this does.

```
[804819] goTenna> log
[806217] goTenna> 
[806227-2421] cli_parse_command >> log
[806227-000] printer: cli-cmd = log
[806237-010] log received.
```

#### temp

Read the internal temperature.  Could be the temperature of the Freescale ARM processor or the Silicon Labs RF chip, not entirely sure yet.  Given the preoccupation with the TRX (tranceiver?) temperature during `send` I would imagine this is temperature of the Silicon Labs RF chip.

```
temp
```

```
[014708-13898]  PRNT  Temp raw: e
[014708-000]  PRNT  Temp: 14
```

#### res

Reset RF hardware?  The [Si4460](https://www.silabs.com/Support%20Documents/TechnicalDocs/Si4464-63-61-60.pdf), mentioned in the resulting output, is a High-Performance Low-Current Tranceiver by Silicon Labs.

Interesting to note, this command is sesensitive to trailing extraneous characters.

```
res
```

```
[035634-20926]  TRX   Si4460 initialized.
[035642-008]  TRX   Batt Voltage : 4148, selected pap: 5
[035643-001]  TRX   Batt Voltage : 4154, selected pap: 5
[035646-003]  TRX   RSSI: ch=0, rssi=96
[035649-003]  TRX   RSSI: ch=1, rssi=97
[035652-003]  TRX   RSSI: ch=2, rssi=76
[035655-003]  TRX   RSSI: ch=3, rssi=87
[035658-003]  TRX   Min rssi 81
[035658-000]  TRX   Bg index val : 3 bin 0 rssi=40  bin 1 rssi=40  bin 2 rssi
[035659-001]  TRX   MAC bg_rssi=45
[035659-000]  TRX   MAC : STATE_POWER_UP*
[035659-000]  TRX   Reset Cause : 0, 4
[035660-001]  TRX   Batt Voltage : 4155, selected pap: 5
[035663-003]  TRX   RSSI: ch=0, rssi=86
[035666-003]  TRX   RSSI: ch=1, rssi=80
[035669-003]  TRX   RSSI: ch=2, rssi=88
[035672-003]  TRX   RSSI: ch=3, rssi=92
[035675-003]  TRX   Min rssi 83
[035675-000]  TRX   Bg index val : 4 bin 0 rssi=40  bin 1 rssi=40  bin 2 rssi
[035676-001]  TRX   MAC bg_rssi=50
[035676-000]  TRX   HDNA Mode 0, advertised 0, observed 0
[035676-000]  TRX   Sleep TRX Chip for LDC mode
[035677-001]  TRX   MAC Current State: Idle*
[035678-001]  TRX   TRX channel id : 4
[035679-001]  TRX   TRX Current State: PDU_IDLE_LONG *
[035680-001]  TRX   PA --- OFF
```

#### ? (help)

Issuing a `?` to the CLI returns some help.

```
?
```

```
[057424-17165]  PRNT  res->     Reset network SM IDLE.
[057424-000]  PRNT  ctx {0/1}->  Continuous transmit(en=1).
[057425-001]  PRNT  cfp {0/1}->  Corrupt First Packet(en=1).
[057425-000]  PRNT  cap {0/1}->  Corrupt All Packets(en=1).
```

#### ctx [0|1]

Enable continuous transmission mode.

```
[1477193] goTenna> ctx 1
[1480391] goTenna>
[1480402-4386] cli_parse_command >> ctx 1
[1480402-000] printer: cli-cmd = ctx 1
[1480413-011] Continuous Transmission Enabled.
[1480421-008] PA ------------on
[1480421-000] vapc value  3400
```

Disable continuous transmission mode.

```
[024889] goTenna> ctx 0
[027132] goTenna>
[027143-4549] cli_parse_command >> ctx 0
[027143-000] printer: cli-cmd = ctx 0
[027154-011] PA ------------ off
[027720-566] Si4460 initialized.
[027727-007] Continuous Transmission Disabled.
```

#### cfp [0|1]

Enable first packet corruption.  Used for testing?

```
[056441] goTenna> cfp 1
[058044] goTenna>
[058054-27934] cli_parse_command >> cfp 1
[058054-000] printer: cli-cmd = cfp 1
[058065-011] First Packet Corruption Enabled.
```

Disable first packet corruption.

```
[059257] goTenna> cfp 0
[060669] goTenna>
[060679-2614] cli_parse_command >> cfp 0
[060679-000] printer: cli-cmd = cfp 0
[060690-011] First Packet Corruption Disabled.
```

#### cap [0|1]

Enable all packet corruption.

```
[133133] goTenna> cap 1
[134233] goTenna>
[134243-73553] cli_parse_command >> cap 1
[134243-000] printer: cli-cmd = cap 1
[134254-011] All Packet Corruption Enabled.
```

Disable all packet corruption.

```
[134750] goTenna> cap 0
[135746] goTenna> 
[135756-1502] cli_parse_command >> cap 0
[135756-000] printer: cli-cmd = cap 0
[135767-011] All Packet Corruption Disabled.
```

#### chn [num]

Select a channel.  The maximum channel that can be selected is 2147483647 (32 bits).  However, during `send` it appears as though the channel is handled as only 8 bits and wraps around, implying maximum of 256 channels.

```
[188229] goTenna> chn 1
[189336] goTenna>
[189347-1726] cli_parse_command >> chn 1
[189347-000] printer: cli-cmd = chn 1
[189358-011] Channel 1 selected.
```

#### send

Send... something?  Uses previously set values from `vapc`, `chn`, `pap` commands.  Unsure how to set the payload, destination, or anything else about what's being sent.

```
[342917] goTenna> send
[343867] goTenna>
[343877-13038] cli_parse_command >> send
[343877-000] printer: cli-cmd = send
[343888-011] Sending PN9 packets
[344454-566] Si4460 initialized.
[344462-008] Before Temp
[344472-010] TRX *** Temp AD Value= 0x5ea0 ***
[344472-000] TRX Temp 17
[344473-001] TRX valid temp
[344473-000] Sending PN9 on chan 0, at vapc adc 3400 and pap 1
[344473-000] PA ------------on
[344474-001] vapc value  3400[344731] INT PACKET_SENT
[344775-301] PA ------------ off
[344775-000] After Temp
[344785-010] TRX *** Temp AD Value= 0x5dbd ***
[344785-000] TRX Temp 29
[344785-000] TRX valid temp
[344786-001] TRX Current State: STATE_POWER_UP *
[345301] INT Sync Word
[345352-566] Si4460 initialized.
[345359-007] Batt Voltage : 4131, selected pap: 6
[345359-000] Batt Voltage : 4131, selected pap: 6
[345375-016] scan rssi[0=98  1=82  2=91  3=96  4=87]
[345375-000] MAC bg_rssi=42
[345375-000] MAC : STATE_POWER_UP*
[345375-000] Reset Cause : 82, 0
[345376-001] MAC Current State: Idle*
[345376-000] Batt Voltage : 4134, selected pap: 6
[345391-015] scan rssi[0=93  1=81  2=91  3=94  4=77]
[345392-001] MAC bg_rssi=45
[345392-000] TRX Current State: STATE_POWER_UP *
[345958-566] Si4460 initialized.
[345965-007] Batt Voltage : 4132, selected pap: 6
```

#### vapc [val]

Set the VAPC?  Probably best to leave this be.  Leave it at 3400?

```
[484758] goTenna> vapc 3400
[486969] goTenna>
[486980-35964] cli_parse_command >> vapc 3400
[486980-000] vapc value changed
[486980-000] printer: cli-cmd = vapc 3400
```

#### read_eflash

Output the contents of the internal flash chip over the serial port.  I've created some tools to aid with extracting the flash this way: https://github.com/dogshoes/gotenna-flash-dumper.

```
read_eflash
```

```
000000: 3A1008000000600020C10800004D2A01 
000010: 00512A0100AB0A3A10081000552A0100 
000020: 592A01005D2A0100612A0100C00A3A10 
000030: 082000652A0100692A01006D2A010071 
...
```

#### disp-gid

Display the registered GID(s).

```
disp-gid
```

```
RAM: Number of GID = 1
[ k] T    APPID  GID              M    HASH
---- --- ------ ---------------- ---- -----
[ 0] 00 : 3fff : 53xx xxd4 ecxx : 00 : f1xx
```

#### getgid

Displays detailed information about the registered GIDs.

```
getgid
```

```
[437860-4707]  FLSH  ERROR: MSG buf pointer invalid
[437861-001]  FLSH  Public Key len : 49 Public key : c56010439fdf9f727d3e6083e71f6a40d1381c02c74aa4b4f95d2682f5785349a1423f691ae59a492bad9ddefb4531bfbd
[437869-008]  FLSH  Private GIDlen :12: packet: 00 3f ff 53 xx xx d4 ec xx 00 73 f1

[437872-003]  FLSH   GID Count: 1
[437873-001]  FLSH  Stored GIDlen :12: packet: 00 3f ff 53 xx xx d4 ec xx 00 73 f1
```

#### dev_info

Return some useful information about the device.

```
dev_info
```

```
[020470-19661]  PRNT  PAP Offset 0
[020470-000]  PRNT  SERIAL no: MX154XXXXX
[020470-000]  PRNT  Bluetooth flag 255
[020470-000]  PRNT  Device Type :01, Hardware Version 01
```

#### sweep [0|1]

Perform some sort of RF test sweep?  Is any RF emitted during this?

```
sweep 1
```

```
[220372-001]  TRX   Current Frequency 142179750
[220392-020]  TRX   Frequency band 142 to 175 selected
[220392-000]  TRX   Frequency band 142 - 175 configured
[220394-002]  TRX   Frac   1078634656 -2147483648
[220395-001]  TRX   PA --- ON. Vapc value:  0
[220396-001]  TRX   Current Frequency 142180000
[220416-020]  TRX   Frequency band 142 to 175 selected
[220416-000]  TRX   Frequency band 142 - 175 configured
[220418-002]  TRX   Frac   1078634659 -2147483648
[220419-001]  TRX   PA --- ON. Vapc value:  0
[220420-001]  TRX   Current Frequency 142180250
[220440-020]  TRX   Frequency band 142 to 175 selected
[220440-000]  TRX   Frequency band 142 - 175 configured
[220442-002]  TRX   Frac   1078634663 -1610612736
[220443-001]  TRX   PA --- ON. Vapc value:  0
[220444-001]  TRX   Current Frequency 142180500
[220464-020]  TRX   Frequency band 142 to 175 selected
[220464-000]  TRX   Frequency band 142 - 175 configured
[220466-002]  TRX   Frac   1078634666 -1610612736
[220467-001]  TRX   PA --- ON. Vapc value:  0
[220468-001]  TRX   Current Frequency 142180750
[220488-020]  TRX   Frequency band 142 to 175 selected
[220488-000]  TRX   Frequency band 142 - 175 configured
[220490-002]  TRX   Frac   1078634669 -1073741824
[220491-001]  TRX   PA --- ON. Vapc value:  0
[220492-001]  TRX   Current Frequency 142181000
[220512-020]  TRX   Frequency band 142 to 175 selected
[220512-000]  TRX   Frequency band 142 - 175 configured
[220514-002]  TRX   Frac   1078634672 -1610612736
[220515-001]  TRX   PA --- ON. Vapc value:  0
[220516-001]  TRX   Current Frequency 142181250
[220536-020]  TRX   Frequency band 142 to 175 selected
[220536-000]  TRX   Frequency band 142 - 175 configured
[220538-002]  TRX   Frac   1078634675 -1073741824
[220539-001]  TRX   PA --- ON. Vapc value:  0
[220540-001]  TRX   Current Frequency 142181500
```

```
sweep 0
```

```
[220541-001]  PRNT  Sweep Disabled.
```

#### get_serial

Returns the serial number.

```
get_serial
```

```
[011816-4014]  PRNT  SERIAL no:MX154XXXXX
[011816-000]  PRNT
```

#### getdiag

Get diagnostic information.

```
getdiag
```

```
RST CAUSE:8200
PWR RST: 65
SOFT RST: 156
WDOG RST: 1

Total Mins of Operation: 214280
Total BT Connections Mins: 37010
Total Transmissions: 14

Max Recorded Temperature: 33

Total Private TX: 0
Total Group TX: 0
Total Shout TX: 23
Failed Private TX: 0

Total Private RX: 0
Total Group RX: 0
Total Shout RX: 164

Total MSG Dropped RSSI: 3
Total MSG Dropped Thermal Backoff 0

RSSI Stats:
RSSI 000 - 020 Pass:00000001 Fail:00000020 CRC Recov:00000001
RSSI 020 - 040 Pass:00000001 Fail:00000007 CRC Recov:00000000
RSSI 040 - 060 Pass:00000019 Fail:00000054 CRC Recov:00000000
RSSI 060 - 080 Pass:00000077 Fail:00000021 CRC Recov:00000000
RSSI 080 - 100 Pass:00000225 Fail:00000020 CRC Recov:00000000
RSSI 100 - 120 Pass:00000154 Fail:00000005 CRC Recov:00000000
RSSI 120 - 140 Pass:00000008 Fail:00000001 CRC Recov:00000000
RSSI 140 - 160 Pass:00000002 Fail:00000000 CRC Recov:00000000
RSSI 160 - 180 Pass:00000006 Fail:00000000 CRC Recov:00000000
RSSI 180 - 200 Pass:00000000 Fail:00000000 CRC Recov:00000000
RSSI 200 - 220 Pass:00000000 Fail:00000000 CRC Recov:00000000
RSSI 220 - 240 Pass:00000000 Fail:00000001 CRC Recov:00000000
RSSI 240 - 260 Pass:00000000 Fail:00000000 CRC Recov:00000000
```

#### clrdiag

Clear the diagnostic blocks from flash.  All mertrics reported by `getdiag` are reset.

```
clrdiag
```

```
[214941-13881]  FLSH  ERROR: MSG buf pointer out of range
[215076-135]  FLSH  Block: 433, Invalid CRC, Expected:29739, IS:-1
[215083-007]  FLSH  Block: 434, Invalid CRC, Expected:29739, IS:-1
[215084-001]  FLSH  Diag Block corruption. Erasing Diag Block.[215159-075]  FLSH  Write Diag Block, 433 CRC: 0
[215225-066]  FLSH  Write Diag Block, 434 CRC: 0
[215232-007]  FLSH  Block: 433, Invalid CRC, Expected:10804, IS:0
```

### The Button

There's a tiny button near the USB port on the goTenna.  A single press results in the following message.

```
[108550-000]  INT   BUTTON PRESSED
[108833-000]  INT   BUTTON RELEASED
```

It's smart enough to see a double press as well.

```
[105684-000]  INT   BUTTON PRESSED
[105788-000]  INT   BUTTON RELEASED
[105925-000]  INT   BUTTON PRESSED
[106000-000]  INT   BUTTON DOUBLE PRESSED
[106011-000]  INT   BUTTON RELEASED
```

And, things go a little crazy when you triple press it...

```
[106289-000]  INT   BUTTON PRESSED
[106389-000]  INT   BUTTON RELEASED
[108317-000]  INT   BUTTON PRESSED
[108400-000]  INT   BUTTON TRIPLE PRESSED
[108422-000]  INT   BUTTON RELEASED
[108550-000]  INT   BUTTON PRESSED
[108833-000]  INT   BUTTON RELEASED
[109000-000]  INT   BUTTON TRIPLE RELEASED. Send Emergency
[109001-28543]  FLSH  Sending Out Emergency Beacon
[109002-001]  FLSH  Buffer Allocated 1FFFE5B8. Num Free: 6
...
```

Oops.

Holding the button down for over 10 seconds does this!  Does this reset the data stored in flash?

```
[051174-10159] Block: 433, Invalid CRC, Expected:29739, IS:-1
[051181-007] Block: 434, Invalid CRC, Expected:29739, IS:-1
[051181-000] Diag Block corruption. Erasing Diag Block.
Write Diag Block, 433 CRC: 0

Write Diag Block, 434 CRC: 0
[051309-128] Block: 433, Invalid CRC, Expected:10804, IS:0
[052028] BUTTON RELEASED
[056970-5661] Binary Log deleted, 1
[056970-000] Current Write Index, 0222
[056971-001] Current Write Offset, 0032
[056971-000] Current Read Index, 0222
[056971-000] Current Read Offset, 0032
[056971-000] Device Data Reset successful
```

Additionally, for extra excitement, there's a red LED which illuminates while the device is charging.

### Additional Notes and Things to Look at

This is a pile of things to look at later.

This appears at boot.

```
[000022-022] PA ------------ off
[000032-010] TRX Current State: STATE_POWER_UP *
[000093-061] Msg Add: 19XXX0, Wrt Msg Add: 19XXX8, Del Msg Num: 0, Total Msg Num: 251. Max Msg: 261
[000095-00
```

This also appears at boot, sometimes?

```
[000022-022] PA ------------ off
[000032-010] TRX Current State: STATE_POWER_UP *
[000093-061] Msg Add: 19a000, Wrt Msg Add: 19a000, Del Msg Num: 0, Total Msg Nu1
[000095-002] PAP Offset 0
[000095-000] SERIAL no: MX1542XXXX[000096-001]
[000096-000]
Bluetooth flag 255
[000107-011] Flash Initialized.
[000598-491] Si4460 initialized.
[000605-007] Batt Voltage : 4157, selected pap: 6
[000605-000] Batt Voltage : 4152, selected pap: 6
[000621-016] scan rssi[0=44  1=44  2=35  3=44  4=44]
[000621-000] MAC bg_rssi=39
[000621-000] MAC : STATE_POWER_UP*
[000621-000] Reset Cause : 82, 0
[000622-001] MAC Current State: Idle*
[000622-000] Batt Voltage : 4153, selected pap: 6
[000637-015] scan rssi[0=47  1=40  2=35  3=47  4=40]
[000637-000] MAC bg_rssi=39
[000638-001] TRX Current State: STATE_POWER_UP *
[000639-001] Flash Message invalid Metadata
[001204-565] Si4460 initialized.
[001211-007] Batt Voltage : 4155, selected pap: 6
[001212-001] TRX channel id waiting idle: 4
```

How can we read this diag block?  Is there wear leveling in place or does this block just get constantly overwritten?  Diag block is always 433 and 434 on my goTenna.

```

Write Diag Block, 433 CRC: 56364

Write Diag Block, 434 CRC: 56364
[330141-2125] Storing Diag Info in Flash
```

Had this appear.

```
[523195] INT PREAMBLE_DETECT
[523195-10875] Preamble detect1
[523196-001] TRX Current State: RX_SYNC_WORD_WAIT *
[523196-000] MAC Current State: MAC_RX_PRE_RTS*
[523376-180] TRX State Expired: RX_SYNC_WORD_WAIT *
[523377-001] TRX channel id : 4
[523378-001] TRX Current State: PDU_IDLE_LONG *
[523378-000] turning off the HW
[523378-000] PA ------------ off
[523399-021] MAC Receive PreRTS Timeout expired
[523399-000] MAC Current State: Idle*
```

Happened again?

```
[539606] INT PREAMBLE_DETECT
[539607-6591] Preamble detect1
[539608-001] TRX Current State: RX_SYNC_WORD_WAIT *
[539608-000] MAC Current State: MAC_RX_PRE_RTS*
[539611] INT Sync Word
[539611-003] TRX Sync Word detected. *
[539612-001] MODEM STATUS : 00 01 56 58 00 00 00
[539612-000] MAC RX_Pre-RTS stage=0, sync detect received
[539613-001] TRX Current State: RX_PDU_PKT_HEADER_WAIT *
[539671] INT RX_FIFO_ALMOST_FULL
[539672-059] RX fifo len 18/n
[539672-000] crc is being checked
[539673-001] crc is corrupted
[539681-008] crc packet not recovered
[539682-001] TRX channel id : 4
[539684-002] TRX Current State: PDU_IDLE_LONG *
[539684-000] turning off the HW
[539684-000] PA ------------ off
[539694-010] MAC Receive PreRTS Timeout expired
[539694-000] MAC Current State: Idle*
[539695-001] Batt Voltage : 4154, selected pap: 6
[539710-015] scan rssi[0=89  1=84  2=90  3=90  4=86]
[539710-000] MAC bg_rssi=55
[539711-001] TRX channel id : 4
[539712-001] TRX Current State: PDU_IDLE_LONG *
[539713-001] turning off the HW
[539713-000] PA ------------ off
```

This occurs when pairing the goTenna.

```
BLE set to connected
[026584-8392] Bluetooth Response Sent.
[026703-119] NRF Message sent
[026734-031] system info asked.
[026744-010] TRX *** Temp AD Value= 0x5efd ***
[026744-000] TRX Temp 12
[026745-001] TRX valid temp
[026789-044] Bluetooth Response Sent.
[026943-154] NRF Message sent
[027014-071] Bluetooth Response Sent.
[027094-080] NRF Message sent
[027123-029] Bluetooth Response Sent.
[027274-151] NRF Message sent
[027303-029] Received get stored message.
[027304-001] Get message NACK sent.
[027304-000] Bluetooth Response Sent.
[027424-120] NRF Message sent
[027574-150]  Received date and time
[027575-001] Bluetooth Response Sent.
[027693-118] NRF Message sent
```

On setting the GID.

```
[156336-33320] stored hash17920
[156338-002] Bluetooth Response Sent.
[156455-117] NRF Message sent
```

This happened after.

```
[175686-11670]  Received date and time
[175687-001] Bluetooth Response Sent.
[175805-118] NRF Message sent
```

Beginning firmware upgrade.

```
[083825-151]  Received date and time
[083825-000] Bluetooth Response Sent.
[083944-119] NRF Message sent
[083974-030] FW Exchange initiated.
[088346-4372] Bluetooth Response Sent.
[088384-038] NRF Message sent
[089225-841] FW data packet received
[089226-001] seqid 36[089226-000] Bluetooth Response Sent.
[089344-118] NRF Message sent
[090155-811] FW data packet received
[090156-001] seqid 37[090156-000] stored pkt 1 of 655 stored length 250 of total length 163652
[090167-011] Bluetooth Response Sent.
[090274-107] NRF Message sent
[091085-811] FW data packet received
[091086-001] seqid 38[091086-000] stored pkt 2 of 655 stored length 500 of total length 163652
[091098-012] Bluetooth Response Sent.
[091205-107] NRF Message sent
```


## USB

### Output From `lsusb`

```
Bus 002 Device 004: ID 1fc9:2047 NXP Semiconductors 
Device Descriptor:
  bLength                18
  bDescriptorType         1
  bcdUSB               1.10
  bDeviceClass            2 Communications
  bDeviceSubClass         0 
  bDeviceProtocol         0 
  bMaxPacketSize0        64
  idVendor           0x1fc9 NXP Semiconductors
  idProduct          0x2047 
  bcdDevice            3.10
  iManufacturer           1 goTenna, Inc
  iProduct                2 goTenna
  iSerial                 3 123456789ABCDEF
  bNumConfigurations      1
  Configuration Descriptor:
    bLength                 9
    bDescriptorType         2
    wTotalLength           67
    bNumInterfaces          2
    bConfigurationValue     1
    iConfiguration          4 123456789ABCDEF
    bmAttributes         0xc0
      Self Powered
    MaxPower               64mA
    Interface Descriptor:
      bLength                 9
      bDescriptorType         4
      bInterfaceNumber        0
      bAlternateSetting       0
      bNumEndpoints           1
      bInterfaceClass         2 Communications
      bInterfaceSubClass      2 Abstract (modem)
      bInterfaceProtocol      1 AT-commands (v.25ter)
      iInterface              0 
      CDC Header:
        bcdCDC               1.10
      CDC Union:
        bMasterInterface        0
        bSlaveInterface         1 
      CDC ACM:
        bmCapabilities       0x00
      CDC Call Management:
        bmCapabilities       0x01
          call management
        bDataInterface          1
      Endpoint Descriptor:
        bLength                 7
        bDescriptorType         5
        bEndpointAddress     0x82  EP 2 IN
        bmAttributes            3
          Transfer Type            Interrupt
          Synch Type               None
          Usage Type               Data
        wMaxPacketSize     0x0008  1x 8 bytes
        bInterval             128
    Interface Descriptor:
      bLength                 9
      bDescriptorType         4
      bInterfaceNumber        1
      bAlternateSetting       0
      bNumEndpoints           2
      bInterfaceClass        10 CDC Data
      bInterfaceSubClass      0 Unused
      bInterfaceProtocol      0 
      iInterface              0 
      Endpoint Descriptor:
        bLength                 7
        bDescriptorType         5
        bEndpointAddress     0x81  EP 1 IN
        bmAttributes            2
          Transfer Type            Bulk
          Synch Type               None
          Usage Type               Data
        wMaxPacketSize     0x0040  1x 64 bytes
        bInterval               0
      Endpoint Descriptor:
        bLength                 7
        bDescriptorType         5
        bEndpointAddress     0x01  EP 1 OUT
        bmAttributes            2
          Transfer Type            Bulk
          Synch Type               None
          Usage Type               Data
        wMaxPacketSize     0x0040  1x 64 bytes
        bInterval               0
Device Status:     0x0000
  (Bus Powered)
```

### Output From `dmesg`

The serial number here has not been obfuscated.  Every goTenna I've tried reports this serial number.

```
[125726.694882] usb 2-2.1: new full-speed USB device number 6 using uhci_hcd
[125726.826823] usb 2-2.1: New USB device found, idVendor=1fc9, idProduct=2047
[125726.826826] usb 2-2.1: New USB device strings: Mfr=1, Product=2, SerialNumber=3
[125726.826828] usb 2-2.1: Product: goTenna
[125726.826830] usb 2-2.1: Manufacturer: goTenna, Inc
[125726.826831] usb 2-2.1: SerialNumber: 123456789ABCDEF
```

## Possible Leads

* http://tieba.baidu.com/p/3858941725
* https://github.com/akimtke/akim-gotenna/blob/04223025e1a93c9e518d54000696509abdedcc85/src/tests/wireless.py